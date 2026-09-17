# Node 1.3.3 アーカイブカード修正パッチ

Luna-Frontier（Node テーマ 1.3.2）向けの表示修正です。

## ダウンロード

- **このリポジトリの ZIP**: https://github.com/wingzone94/node-1.3.3-archive-cards/archive/refs/heads/main.zip
- Codex 用手順: [CODEX.md](./CODEX.md)

## 直すこと

| 対象 | 現行 1.3.2 | このパッチ |
|---|---|---|
| カテゴリ／タグ／日付のカード角 | 角丸 | 四角（LATEST と同じ） |
| パソコン・タブレットのアイキャッチ | 上下に空き | 枠が画像実寸に追従。全体表示 |
| スマートフォン（≤700px）の行 | cover の横並び | **変更しない** |

## 適用

1. `CODEX.md` を Codex に貼る（推奨）。または
2. `src/styles/_archive.css` と `src/styles/_cards.css` をテーマの同パスへ上書きし、`CHANGELOG.snippet.md` を `CHANGELOG.md` の `[1.3.2]` の前に挿入する。
3. `bun x vite build`

PHP / JS / バージョン番号は変更しません。
