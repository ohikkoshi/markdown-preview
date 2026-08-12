# GitHub Flavored Markdown Preview

単一ファイルのMarkdownプレビューツールです。
ブラウザで開くだけで利用可能でビルドや依存関係のインストールは不要です。

## 機能

- リアルタイムの分割画面エディタとプレビュー
- GitHub Flavored Markdown (GFM) のサポート
- シンタックスハイライティング付きコードブロック（highlight.js）
- GitHub スタイルのアラート（`> [!NOTE]`, `> [!TIP]`, `> [!IMPORTANT]`, `> [!WARNING]`, `> [!CAUTION]`）
- フットノート（`[^1]` 記法 → 脚注 + バックリンク）
- 折りたたみセクション（`<details>`/`<summary>` に GitHub 風トグルアイコンスタイル）
- エモジショートコード（33種の `:name:` 記法を Unicode に変換）
- Mermaid ダイアグラム（` ```mermaid ` フェンスコードブロックを図として描画）
- プレビューのスタンドアロン HTML ファイルとしてエクスポート

## 使い方

`markdown-preview.html` をブラウザで開きます。
左ペインにMarkdownを入力すると、右ペインにプレビューがリアルタイムで表示されます。
**Export (HTML)** ボタンをクリックすることで、結果をスタンドアロンのHTMLファイルとしてダウンロードできます。

## 依存関係

すべての依存関係はCDN経由で読み込まれます：

- [marked](https://github.com/markedjs/marked) — Markdownパーサー
- [highlight.js](https://highlightjs.org/) — シンタックスハイライティング（JS + github テーマCSS）
- [marked-footnote](https://www.npmjs.com/package/marked-footnote) — フットノート拡張
- [DOMPurify](https://github.com/cure53/DOMPurify) — HTMLサニタイズ（XSS対策）
- [mermaid](https://mermaid.js.org/) — ダイアグラム描画（ESM）

## ライセンス

MIT
