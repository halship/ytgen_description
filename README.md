# Youtube説明文ジェネレータ

## 説明

Youtubeの配信、動画投稿時に設定する説明文を生成するツールです。
現時点では以下の項目を設定できます。

- ハッシュタグ
- 動画概要
- プレイするゲーム
- 使用したツール
- 使用した素材

エクスポートボタンでJSON形式のファイルをダウンロードできます。

インポートボタンでエクスポートしたJSON形式ファイルをインポートすることができます。
これを使用することにより、毎時使用したいテンプレートをローカルに保存し使用することができます。

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```sh
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```
