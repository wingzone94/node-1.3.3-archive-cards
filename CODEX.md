# Node 1.3.3 — アーカイブ記事カード修正（Codex 用）

リポジトリ: Luna-Frontier（Node テーマ）。現行は **1.3.2**。この作業は **unreleased の 1.3.3** として入れる。

## 制約

- **git commit / push はしない。**
- 関係ないファイルを触らない。
- `sslverify => false` を新規追加しない。
- PHP / JS は変更しない。CSS と CHANGELOG のみ。
- バージョン番号（`style.css` の Version、`package.json`）は **まだ上げない**（unreleased のまま）。
- CSS を変えたら **`bun x vite build`** を実行して `assets/css/` を更新する。
- 既存の 700px 以下コンパクト行（`object-fit: cover`）を壊さない。`bun run verify:mobile` が通ること。

必読: `AGENTS.md` の CSS / カード規約。対象はカテゴリ・タグ・日付アーカイブ（`.m3-archive-layout`）だけ。トップの LATEST・検索結果には波及させない。

---

## 背景（何が壊れているか）

本番のカテゴリアーカイブ（例: `/category/ゲーム/`）で、

1. **カードが角丸**のまま。トップの LATEST は 1.3.0 で四角にしたのに、アーカイブだけ `_archive.css` が `border-radius: var(--m3-radius-large)` を直接指定しているため `--node-card-radius` が効かない。
2. **アイキャッチの上下にグレーの空き（レターボックス）**が出る。原因はリスト表示の
   `:is(.c-card__visual) { display: flex; align-items: center; }` と
   カードの `min-height: 180px` / `height: 100%`。画像は `height: auto` なので、本文が画像より高いと枠だけ伸びて上下が空く。

スマートフォン幅（≤700px）の横並び行は 1.3.1 で `cover` にして密度を上げた。ここは **触らない**。前回案でモバイルにも `contain` / `height: auto` を当てると、詳細度の負けで visual が stretch したまま画像だけ縮み、逆にレターボックスが増える。

---

## タスク1: `src/styles/_archive.css`

`.m3-archive-layout` に `--node-card-radius: 0` を追加する。

```css
.m3-archive-layout {
    --m3-archive-hero-radius: 48px;
    --m3-archive-panel-radius: 28px;
    --m3-archive-accent: var(--category-color, var(--md-sys-color-primary));
    /* 1.3.3: アーカイブの記事カードをトップの LATEST と同じ四角に揃える */
    --node-card-radius: 0;
```

`.m3-archive-post-grid__cards .m3-card` の `border-radius: var(--m3-radius-large)` を **`0` に変更**する（変数経由では勝てないため、直接上書きする）。

```css
.m3-archive-post-grid__cards .m3-card {
    border-radius: 0;
    transition:
        transform 0.35s var(--m3-motion-easing, cubic-bezier(0.2, 0, 0, 1)),
        box-shadow 0.35s var(--m3-motion-easing, cubic-bezier(0.2, 0, 0, 1));
}
```

カテゴリバッジ・シリーズピルの radius は触らない。

---

## タスク2: `src/styles/_cards.css`

ファイル末尾（`@media (max-width: 600px)` のカテゴリ幅制限ブロックの **後**、EOF の前）に次を追加する。既存のモバイル 700px ブロックは **1文字も変えない**。

```css
/* ===========================================================================
   Node 1.3.3: カテゴリ／タグ／日付アーカイブの記事カード
   - カードをトップの LATEST と同じ四角にする
   - パソコン・タブレット幅ではアイキャッチ枠を画像実寸に追従させ、
     レターボックス（上下の空き）を出さない
   - 700px 以下のコンパクト行（object-fit: cover）は触らない
     （トップ・検索結果のモバイル行に波及させない／アーカイブの密度も維持）
   =========================================================================== */

.m3-archive-layout {
    --node-card-radius: 0;
}

.m3-archive-layout .m3-archive-post-grid__cards :is(.c-card, .m3-card) {
    border-radius: 0;
}

@media (min-width: 701px) {
    .m3-archive-layout .m3-archive-post-grid__cards :is(.c-card, .m3-card) {
        align-items: flex-start;
        min-height: 0;
        height: auto;
    }

    .m3-archive-layout .m3-archive-post-grid__cards :is(.c-card__visual, .m3-card__visual) {
        display: block;
        align-self: flex-start;
        align-items: flex-start;
        height: auto;
        padding: 0;
        margin: 0;
        line-height: 0;
    }

    .m3-archive-layout .m3-archive-post-grid__cards :is(.c-card__image-link, .m3-card__image-link) {
        display: block;
        height: auto;
        line-height: 0;
        padding: 0;
    }

    .m3-archive-layout .m3-archive-post-grid__cards :is(.c-card__visual, .m3-card__visual) img {
        width: 100%;
        height: auto;
        object-fit: contain;
        object-position: center top;
        display: block;
        vertical-align: top;
    }
}
```

セレクタを `.m3-archive-layout .m3-archive-post-grid__cards` に限定すること。トップ（`.m3-surface--latest` / `--articles`）と検索結果はクラスが違うので巻き込まれない。

`min-width: 701px` は必須。モバイル行の `object-fit: cover`（詳細度 0,3,1〜0,4,1）と戦わせない。

---

## タスク3: `CHANGELOG.md`

冒頭の説明段落の直後、現行 `## [1.3.2]` の **前** に挿入する。

```md
## [1.3.3] - unreleased

1.3.2 公開後の表示まわりの修正です。

### カテゴリ／タグ／日付アーカイブ
- **記事カードを四角に**: トップの LATEST と同じく、カテゴリ／タグ／日付アーカイブの記事カードから角丸をなくしました。カテゴリバッジはこれまでどおりピルのままです。
- **アイキャッチの上下余白を解消（パソコン・タブレット幅）**: 画像表示枠が画像の実寸に追従するようにし、レターボックス（上下の空き）を出さないようにしました。画像は全体が見える縮尺（`object-fit: contain` / `height: auto`）で表示します。
- **スマートフォンの横並び行は維持**: 700px以下のコンパクトな行（サムネイルを `cover` で埋める）は、トップ・検索結果と同じく変更しません。1.3.1 の密度を保ちます。
```

---

## ビルドと確認

1. `bun x vite build` が成功すること。
2. `bun run verify:mobile` が落ちないこと（アーカイブのモバイル行が cover のままであること）。
3. 目視（cybernode.local または本番相当）:
   - `/category/ゲーム/` パソコン幅: カードが四角。アイキャッチの上下にサーフェス色の帯が無い。画像はトリミングせず全体が見える。
   - 同じページの 390px 幅: 1.3.1 と同じ横並び行。サムネは cover。カードだけ四角。
   - トップ LATEST・検索結果: 見た目不変。

## 完了報告

- 変更ファイル一覧
- `bun x vite build` の成否
- `verify:mobile` の成否
- バージョンは 1.3.2 のまま（CHANGELOG のみ 1.3.3 unreleased）であることを明記
