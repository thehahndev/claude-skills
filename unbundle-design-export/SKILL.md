---
name: unbundle-design-export
description: "Converts a Claude Design export bundle (typically named index.php or index.html, usually 1-5 MB) into plain static files - HTML, CSS, and font files with no JavaScript required. Use this skill whenever the user mentions a Claude Design export, a bundled HTML file from Claude Design, or asks to unbundle, extract, or convert a Claude Design project. Also trigger this skill if the user has a large single-file HTML export and wants to separate it into individual CSS and font files."
---

## What this skill does

Claude Design exports are single self-extracting HTML files (often saved as `.php` despite containing no PHP). They pack all assets — fonts, CSS, and JavaScript — as base64+gzip blobs inside custom `<script>` tags. When opened in a browser, a runtime unpacker reconstructs these assets on the fly.

This skill decodes those blobs and produces a clean, static file structure:

```
project/
├── index.php     <- plain HTML (no script tags)
├── css/
│   └── app.css   <- all extracted styles
└── fonts/
    ├── inter-1.woff2
    └── ... (all font files)
```

The JavaScript in the bundle is Claude Design's own edit-mode overlay (React + a TweaksPanel component). It is not user content and can be safely discarded for static deployment.

---

## Bundle format reference

A Claude Design bundle contains three custom script tags:

```html
<script type="__bundler/manifest">{ "UUID": { "type": "font/woff2"|"text/javascript"|"text/jsx", "data": "<base64>", "compressed": true|false }, ... }</script>
<script type="__bundler/template">JSON-encoded string of the full page HTML</script>
<script type="__bundler/ext_resources">[]</script>
```

**Important:** The template content is a JSON-encoded string — you must `JSON.parse()` it before use, not just extract it with a regex.

Asset types:
- `font/woff2` — Inter font subsets; extract these
- `text/javascript` — React, ReactDOM, Babel — Claude Design runtime only; discard
- `text/jsx` — TweaksPanel edit overlay — Claude Design utility; discard

The actual page HTML (the user's content) is already static inside the template. There is nothing to pre-render.

---

## How to perform the extraction

Write the script below to `_extract_bundle.mjs` in the project directory, run it, then delete it. No npm install needed — it uses only Node.js built-ins.

```javascript
import fs from 'fs';
import path from 'path';
import zlib from 'zlib';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

// Find the bundle file — accept index.php, index.html, or any .html/.php in cwd
const candidates = ['index.php', 'index.html'].concat(
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

Then clean up:
```bash
# Windows
del _extract_bundle.mjs
# macOS/Linux
rm _extract_bundle.mjs
```

---

## After extraction

Verify the output:
1. Open the HTML file in a browser — it should render identically to the original
2. DevTools -> Network: only `app.css` and font files should load (no `.js` files)
3. DevTools -> Console: should be clean (no errors)
4. Check `<script` tags remaining — the extraction script prints this count; it should be 0

---

## Troubleshooting

**"No Claude Design bundle found"** — The file may be named differently. Check that `__bundler/manifest` appears in the file. Pass the filename explicitly by editing the `bundleFile` constant.

**Fonts not loading** — Check `@font-face` declarations in `app.css` use relative paths like `fonts/inter-1.woff2`. If they still contain UUIDs, check the UUID-to-font mapping loop.

**Script tags remain after extraction** — Some inline `<script>` tags (e.g., analytics, third-party widgets) may be legitimate user content. Review them before removing manually.

**`node` not found** — Node.js must be installed. Download from nodejs.org. Version 18+ is recommended (ESM support).

**Page looks wrong after extraction** — The CSS may depend on class names injected by the React overlay. This is rare but possible. In this case, open the original bundle in a browser, use DevTools to copy the fully-rendered HTML, and use that as the base instead.

**If the user wants JavaScript preserved** — Some projects include interactive components. In that case, keep the `text/jsx` asset, compile it with Babel (`npx babel --presets @babel/preset-react`), write it to `js/app.js`, and add a `<script src="js/app.js">` tag. React and ReactDOM from the manifest can also be extracted and served locally.
