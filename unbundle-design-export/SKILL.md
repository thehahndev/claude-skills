---
name: unbundle-design-export
description: "Converts a Claude Design export bundle (typically named index.html, usually 1-5 MB) into plain static files - HTML, CSS, and font files with no JavaScript required. Use this skill whenever the user mentions a Claude Design export, a bundled HTML file from Claude Design, or asks to unbundle, extract, or convert a Claude Design project. Also trigger this skill if the user has a large single-file HTML export and wants to separate it into individual CSS and font files."
---

## What this skill does

Claude Design exports are single self-extracting HTML files. They pack all assets — fonts, CSS, and JavaScript — as base64+gzip blobs inside custom `<script>` tags. When opened in a browser, a runtime unpacker reconstructs these assets on the fly.

This skill decodes those blobs and produces a clean, static file structure:

```
project/
├── index.html    <- plain HTML (no script tags)
├── css/
│   └── app.css   <- all extracted styles
└── fonts/
    ├── inter-1.woff2
    └── ... (all font files)
```

The JavaScript in the bundle is Claude Design's own edit-mode overlay (React + a TweaksPanel component). It is not user content and can be safely discarded for static deployment.

---

## How to perform the extraction

Write the script below to `_extract_bundle.mjs` in the project directory, run it, then delete it. No npm install needed — it uses only Node.js built-ins.

```javascript
import fs from 'fs';
import path from 'path';
import zlib from 'zlib';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

// Find the bundle file — accept index.html, index.php, or any .html/.php in cwd
const candidates = ['index.html', 'index.php'].concat(
  fs.readdirSync(__dirname).filter(f => /\.(php|html)$/i.test(f))
);
const bundleFile = candidates.find(f => {
  const p = path.join(__dirname, f);
  if (!fs.existsSync(p)) return false;
  const content = fs.readFileSync(p, 'utf8');
  return content.includes('__bundler/manifest');
});
if (!bundleFile) throw new Error('No Claude Design bundle found in ' + __dirname);

const src = fs.readFileSync(path.join(__dirname, bundleFile), 'utf8');
console.log('Bundle file:', bundleFile, '(' + (src.length / 1024).toFixed(0) + ' KB)');

// Parse manifest
const manifestMatch = src.match(/<script type="__bundler\/manifest">([\s\S]*?)<\/script>/);
if (!manifestMatch) throw new Error('No manifest found');
const manifest = JSON.parse(manifestMatch[1]);

function decode(entry) {
  const buf = Buffer.from(entry.data, 'base64');
  return entry.compressed ? zlib.gunzipSync(buf) : buf;
}

// Extract fonts
fs.mkdirSync(path.join(__dirname, 'fonts'), { recursive: true });
const uuidToFont = {};
let fontIndex = 1;
for (const [uuid, entry] of Object.entries(manifest)) {
  if (entry.type === 'font/woff2') {
    const name = `inter-${fontIndex++}.woff2`;
    fs.writeFileSync(path.join(__dirname, 'fonts', name), decode(entry));
    uuidToFont[uuid] = `fonts/${name}`;
    console.log('Font:', name);
  }
}

// Parse template (it is JSON-encoded)
const templateMatch = src.match(/<script type="__bundler\/template">([\s\S]*?)<\/script>/);
if (!templateMatch) throw new Error('No template found');
let html = JSON.parse(templateMatch[1].trim());

// Replace UUID font references with relative paths
for (const [uuid, fontPath] of Object.entries(uuidToFont)) {
  html = html.split(uuid).join(fontPath);
}

// Extract <style> blocks into css/app.css
fs.mkdirSync(path.join(__dirname, 'css'), { recursive: true });
const styleBlocks = [];
html = html.replace(/<style[^>]*>([\s\S]*?)<\/style>/gi, (_, css) => {
  styleBlocks.push(css);
  return '';
});
const css = styleBlocks.join('\n\n');
fs.writeFileSync(path.join(__dirname, 'css', 'app.css'), css);
console.log('CSS:', (css.length / 1024).toFixed(1) + ' KB');

// Strip all <script> tags (Claude Design runtime — not user content)
html = html.replace(/<script[\s\S]*?<\/script>/gi, '');

// Remove Google Fonts preconnect hints (no longer needed)
html = html.replace(/<link[^>]*fonts\.googleapis[^>]*>/gi, '');
html = html.replace(/<link[^>]*fonts\.gstatic[^>]*>/gi, '');

// Inject CSS link in <head>
html = html.replace(/<\/head>/i, '  <link rel="stylesheet" href="css/app.css">\n</head>');

// Write output — overwrite the bundle file with clean HTML
fs.writeFileSync(path.join(__dirname, bundleFile), html);
console.log('Output HTML:', (html.length / 1024).toFixed(1) + ' KB');
console.log('Done. Script tags remaining:', (html.match(/<script/gi) || []).length);
```

Run it:
```bash
node _extract_bundle.mjs
```

**If the script exits with an error:** stop immediately. Do not delete `_extract_bundle.mjs`. Report the full error output to the user and wait for instructions.

Only if the script succeeds: **immediately** delete the temporary script.
```bash
# Windows
del _extract_bundle.mjs
# macOS/Linux
rm _extract_bundle.mjs
```

Verify the deletion succeeded (e.g. confirm the file no longer exists) before proceeding.

---

## Done when

**The task is not complete until every item on this list is verified.** Each item must be confirmed by actively reading or inspecting the relevant file — do not assume correctness based on the extraction script having run without errors. Complete any unmet item before reporting success.

- [ ] **Temp script deleted**: confirm `_extract_bundle.mjs` does not exist in the project directory by checking for it explicitly
- [ ] **CSS link present**: read the output HTML file and confirm it contains `<link rel="stylesheet" href="css/app.css">` inside `<head>`
- [ ] **CSS non-empty**: read `css/app.css` and confirm it contains actual CSS rules (not an empty or whitespace-only file)
- [ ] **Fonts extracted**: list the `fonts/` directory and confirm at least one `.woff2` file is present
- [ ] **No script tags**: read the output HTML file and confirm it contains zero `<script` occurrences — if any remain, resolve them per the Troubleshooting section before proceeding
- [ ] **No other temp files**: confirm no other files written during this run (e.g. intermediate files, renamed copies) remain in the project directory

If any item is not yet true, complete it before finishing.

---

## Troubleshooting

**"No Claude Design bundle found"** — The file may be named differently. Check that `__bundler/manifest` appears in the file. Pass the filename explicitly by editing the `bundleFile` constant.

**Script tags remain after extraction** — Read the HTML file and list every remaining `<script>` tag. Show the list to the user and ask explicitly whether each should be kept or removed. Do not decide independently. Do not mark the task complete until the user has answered and the appropriate action has been taken.

**`node` not found** — Node.js must be installed. Download from nodejs.org. Version 18+ is recommended (ESM support).

**If the user wants JavaScript preserved** — Some projects include interactive components. In that case, keep the `text/jsx` asset, compile it with Babel (`npx babel --presets @babel/preset-react`), write it to `js/app.js`, and add a `<script src="js/app.js">` tag. React and ReactDOM from the manifest can also be extracted and served locally.
