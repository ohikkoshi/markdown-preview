# AGENTS.md

## What this is
Single-file GitHub Flavored Markdown preview tool. **All code lives in `markdown-preview.html`** (~1250 lines: inline `<style>`, sample content, and `<script>`). There is no build, no `package.json`, no dependency install — open the HTML in a browser to run it.

## Commands
- Lint/format check: `biome check markdown-preview.html` (uses the global `biome`; there is no npm wrapper).
- Autofix: `biome check --write markdown-preview.html`.
- Indentation is **tabs** (Biome default). Note the exported-HTML template literal in `generateFullHTML` is intentionally space-indented; leave it as-is.
- `biome.json` disables `noUnusedVariables`, `noControlCharactersInRegex`, and `noUselessEscapeInString` for `*.html` — do not re-enable via inline changes.
- No test runner, no CI, no `package.json` — verification is `biome check` plus opening the file in a browser. Do not hunt for a test command.

## Dependencies (CDN, `@latest`)
All loaded from jsDelivr `@latest` in `<head>`; there is no local vendoring and no version pinning. The same dependency set (minus DOMPurify/marked, which export omits on purpose) is re-declared inside `generateFullHTML` (see below). If you change a URL, update both the `<head>` script tags and the copy inside `generateFullHTML`. Current: marked (jsDelivr `@latest`), highlight.js via `@highlightjs/cdn-assets@latest` (classic script that publishes the `hljs` global), marked-footnote `@latest` (UMD, `window.markedFootnote`, `peerDeps marked >=7.0.0`), DOMPurify `@latest` (UMD, `window.DOMPurify` + `dist/purify.min.js`), mermaid `@latest` (ESM `mermaid.esm.min.mjs`). marked's custom `renderer.code` uses the **object** signature `code({ text, lang })` (marked >= v13); the positional `code(code, infostring)` (marked <= v9) was removed when `@latest` was adopted, so a further marked major bump can still break the renderer — keep an eye on `code({ text, lang })` and the `false` fallback.

## Render pipeline (order matters)
`renderMarkdown()` is the single source of truth: normalize CRLF→LF → `marked.parse` → `DOMPurify.sanitize`. After injecting HTML, `updatePreview()` then applies, in this exact order:
1. `marked.parse` (runs the custom extensions below)
2. `hljs.highlightElement` on `pre code` — applied to the built DOM, **not** via marked's highlight callback (alert bodies get re-parsed, so post-DOM highlighting avoids double work). Do not switch to a marked highlight callback.
3. `renderMermaid()` — mermaid is loaded as async ESM; it awaits `window.mermaidReady` before `mermaid.run(... suppressErrors:true)`. Mermaid blocks render as `<pre class="mermaid">` (no `code` child) so they deliberately skip the highlight selector.

The custom `renderer.code({ text, lang })` handles only ```` ```mermaid ````; for any other language it returns `false` on purpose to fall back to marked's default code renderer (the `marked.use` fallback mechanism). Returning a string for non-mermaid would bypass the `pre code` highlight path — keep the `false` return.

## Custom features (where to edit)
- Emoji shortcodes: `EMOJI_MAP` + `EMOJI_START`/`EMOJI_TOKEN`; implemented as an inline `emoji` tokenizer extension (`level:'inline'`), not `walkTokens`. Because it is inline-level, `code`/`codespan` are untouched, so `:name:` inside code stays literal.
- GitHub alerts (`> [!NOTE]` etc.): `ALERT_TYPES` (SVG icon/class/label) + `ALERT_PATTERN`, converted from `blockquote` tokens via a `walkTokens` + `alert` extension.
- `<details>`/`<summary>` use the native browser toggle (only `cursor: pointer` + margin CSS at `.preview-content details`/`summary`); no JS extension is attached.
- Sanitize allowlist: `SANITIZE_CONFIG` (`USE_PROFILES` html+svg, plus `ADD_ATTR` for footnote `data-*`). New HTML output that DOMPurify would strip must be allowlisted here or it silently disappears from the preview.

## The dual-template gotcha (read before large edits)
`markdown-preview.html` contains two embedded documents inside string literals, so grep/line reasoning is misleading:
- The `<textarea id="markdown-input">` holds the **default sample markdown** (headings, `hello()`, mermaid diagrams, etc.). That inner `<body>`/`function`/code is content, not app code.
- `generateFullHTML(markdown)` contains a **complete standalone HTML document** as a template literal used by the Export button. It **re-declares its own CSS and its own highlight.js/mermaid loader** (`</script>` escaped as `<\/script>`). Styling or dependency changes meant to affect exported files must be made here too; the live preview CSS/JS and the export template are separate copies that can drift.

## Intentional behavior — do not "fix"
- Export omits DOMPurify on purpose: content is already sanitized at generation time (`renderMarkdown`), so the exported file does not ship DOMPurify.
- Footnote back-link/jump interactions are pre-rendered in exported HTML and are not re-processed there — non-functional links in the export are expected, not a bug.
