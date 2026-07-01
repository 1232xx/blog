# GitHub Pages 公開メモ

このリポジトリでブログを GitHub Pages に載せるときの手順を、後から見返しやすいようにまとめたメモです。

## まず最初に

このテンプレートは Astro + pnpm 前提です。

- `node` が入っていること
- `pnpm` が使えること

今回の環境では `pnpm dev` を実行すると `pnpm is not recognized` になりました。これは Astro の不具合ではなく、Node.js / pnpm が端末から見えていないのが原因です。

Windows なら、まず Node.js LTS を入れて、端末を開き直してから確認します。

```bash
node -v
pnpm -v
```

どちらかが出なければ、そこを先に直します。

## ローカル起動

1. 依存関係を入れる

   ```bash
   pnpm install
   ```

2. 開発サーバーを起動する

   ```bash
   pnpm dev
   ```

3. 公開前に本番ビルドを確認する

   ```bash
   pnpm build
   pnpm preview
   ```

## GitHub Pages に公開する

このテーマは `astro.config.ts` の `site` 設定を使っています。さらに GitHub Pages の URL 形式に合わせて `base` を合わせる必要があります。

### 1. 公開先の種類を決める

#### User / Organization site

- URL 例: `https://username.github.io/`
- リポジトリ名は通常 `username.github.io`
- `base` は基本的に不要

#### Project site

- URL 例: `https://username.github.io/repo-name/`
- `base` に `/repo-name/` を入れる

### 2. `astro.config.ts` を合わせる

`site` は本番 URL にする。

例:

```ts
export default defineConfig({
  site: 'https://username.github.io/repo-name/',
  base: '/repo-name/',
  // ...
})
```

Project site で `base` を入れ忘れると、CSS や画像のパスがずれて表示が崩れやすいです。

### 3. `src/config.ts` も本番 URL に合わせる

このテーマでは `themeConfig.site.website` が RSS や canonical などに使われます。

```ts
site: {
  website: 'https://username.github.io/repo-name/',
  title: 'CHIRI',
  author: 'YOUR_NAME',
  description: '...',
  language: 'ja-JP'
}
```

### 4. GitHub Pages の公開方法を選ぶ

おすすめは GitHub Actions で自動デプロイです。

GitHub のリポジトリ設定で Pages を開き、Source を `GitHub Actions` にします。

そのうえで、`main` に push されたら `pnpm build` して `dist/` を Pages にデプロイする workflow を置きます。

必要な流れは次の通りです。

1. `pnpm install`
2. `pnpm build`
3. `dist/` を GitHub Pages に公開

すでにこのリポジトリには `validate` 用の CI はありますが、Pages へ配信する workflow は別で必要です。

### 5. 目安の workflow

GitHub Actions では、だいたい次の部品を使います。

- `actions/checkout`
- `pnpm/action-setup`
- `actions/setup-node`
- `actions/configure-pages`
- `actions/upload-pages-artifact`
- `actions/deploy-pages`

手元で作るなら、`pnpm build` のあとに `dist/` を Pages に送る形にします。

## 新しい記事を追加する

### いちばん簡単な方法

```bash
pnpm new "My First Post"
```

すると `src/content/posts/my-first-post.md` が作られます。

### 下書きにしたいとき

タイトルの先頭に `_` を付けます。

```bash
pnpm new "_draft post"
```

この場合は `src/content/posts/_draft-post.md` が作られます。

`src/pages/[...slug].astro` では、先頭が `_` の投稿は公開ルートから除外されています。

### 手で作るときの形

`src/content/posts/` に `.md` か `.mdx` ファイルを追加します。

最低限の frontmatter はこれです。

```md
---
title: 'My First Post'
pubDate: '2026-07-01'
---
```

このテーマの投稿 schema は次の3つを見ています。

- `title`
- `pubDate`
- `image` という任意項目

### MDX も使える

`.mdx` にすると、Astro コンポーネントを記事内で使えます。

例:

```mdx
---
title: 'Using MDX'
pubDate: '2026-07-01'
---

import Callout from '@/components/examples/Callout.astro'

<Callout />
```

### 画像の置き場所

記事ごとの画像は `src/content/posts/_assets/` のように、記事の近くに置いて参照できます。

例:

```md
![caption](./_assets/example.webp)
```

### 公開される条件

- ファイルが `src/content/posts/` にある
- `title` と `pubDate` が入っている
- ファイル名の先頭が `_` ではない

この条件を満たすと、一覧と個別ページに自動で出ます。

## つまずきやすいところ

- `pnpm` が認識されない
  - Node.js LTS を入れる
  - 端末を開き直す
  - `corepack` か pnpm の PATH を確認する
- GitHub Pages で CSS が崩れる
  - `astro.config.ts` の `base` を確認する
- RSS や canonical の URL が変
  - `src/config.ts` の `themeConfig.site.website` を確認する
- 記事が公開されない
  - ファイル名の先頭が `_` になっていないか確認する

## ざっくり手順まとめ

1. Node.js と pnpm を入れる
2. `pnpm install`
3. `pnpm dev` で確認する
4. `src/config.ts` と `astro.config.ts` を GitHub Pages 用に直す
5. GitHub Actions で `dist/` を Pages にデプロイする
6. 記事は `pnpm new "title"` か `src/content/posts/*.md` で追加する

