# AGENTS.md

## What this is
Single-file GitHub Flavored Markdown preview tool. **All code lives in `markdown-preview.html`** (~1200 lines: inline `<style>`, sample content, and `<script>`). There is no build, no `package.json`, no dependency install — open the HTML in a browser to run it.

## Commands
- Lint/format check: `biome check markdown-preview.html` (uses the global `biome`; there is no npm wrapper).
- Autofix: `biome check --write markdown-preview.html`.
- Indentation is **tabs** (Biome default). Note the exported-HTML template literal in `generateFullHTML` is intentionally space-indented; leave it as-is.
- `biome.json` disables `noUnusedVariables`, `noControlCharactersInRegex`, and `noUselessEscapeInString` for `*.html` — do not re-enable via inline changes.

## Dependencies (CDN, version-pinned)
Loaded from cdnjs/jsdelivr in `<head>`; there is no local vendoring. Versions are pinned and duplicated in `README.md` — if you change a version, update both the `<head>` script tags and README, and also the copy inside `generateFullHTML` (see below). Current: marked 9.1.6, highlight.js 11.9.0, marked-footnote 1.4.0, DOMPurify 3.4.8, mermaid 11.16.0 (ESM). marked 9's `renderer.code(code, infostring)` signature is assumed; a marked major bump will break the custom code renderer and alert extension.

## Render pipeline (order matters)
`renderMarkdown()` is the single source of truth: normalize CRLF→LF → `marked.parse` → `DOMPurify.sanitize`. After injecting HTML, `updatePreview()` then applies, in this exact order:
1. `marked.parse` (runs the custom extensions below)
2. `hljs.highlightElement` on `pre code` — applied to the built DOM, **not** via marked's highlight callback (alert bodies get re-parsed, so post-DOM highlighting avoids double work). Do not switch to a marked highlight callback.
3. `renderMermaid()` — mermaid is loaded as async ESM; it awaits `window.mermaidReady` before `mermaid.run(... suppressErrors:true)`. Mermaid blocks render as `<pre class="mermaid">` (no `code` child) so they deliberately skip the highlight selector.

The custom `renderer.code` handles only ```` ```mermaid ````; for any other language it returns `false` on purpose to fall back to marked's default code renderer (the `marked.use` fallback mechanism). Returning a string for non-mermaid would bypass the `pre code` highlight path — keep the `false` return.

## Custom features (where to edit)
- Emoji shortcodes: `EMOJI_MAP` + `EMOJI_START`/`EMOJI_TOKEN`; implemented as an inline `emoji` tokenizer extension (`level:'inline'`), not `walkTokens`. Because it is inline-level, `code`/`codespan` are untouched, so `:name:` inside code stays literal.
- GitHub alerts (`> [!NOTE]` etc.): `ALERT_TYPES` (SVG icon/class/label) + `ALERT_PATTERN`, converted from `blockquote` tokens via a `walkTokens` + `alert` extension.
- Sanitize allowlist: `SANITIZE_CONFIG` (`USE_PROFILES` html+svg, plus `ADD_ATTR` for footnote `data-*`). New HTML output that DOMPurify would strip must be allowlisted here or it silently disappears from the preview.

## The dual-template gotcha (read before large edits)
`markdown-preview.html` contains two embedded documents inside string literals, so grep/line reasoning is misleading:
- The `<textarea id="markdown-input">` holds the **default sample markdown** (headings, `hello()`, mermaid diagrams, etc.). That inner `<body>`/`function`/code is content, not app code.
- `generateFullHTML(markdown)` contains a **complete standalone HTML document** as a template literal used by the Export button. It **re-declares its own CSS and its own highlight.js/mermaid loader** (`</script>` escaped as `<\/script>`). Styling or dependency changes meant to affect exported files must be made here too; the live preview CSS/JS and the export template are separate copies that can drift.

## Intentional behavior — do not "fix"
- Export omits DOMPurify on purpose: content is already sanitized at generation time (`renderMarkdown`), so the exported file does not ship DOMPurify.
- Footnote back-link/jump interactions are pre-rendered in exported HTML and are not re-processed there — non-functional links in the export are expected, not a bug.
