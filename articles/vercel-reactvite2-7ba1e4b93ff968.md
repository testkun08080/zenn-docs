---
title: "React-Vite(tailwindcss) + FastAPIをVercel上でデプロイする"
emoji: "🐒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vercel, react, vite, fastapi, python]
published: true
published_at: 2025-06-20 18:00
---


# はじめに
React-ViteとFastAPIをvercel上までデプロイするまでの流れとサンプルデータの記録です。
過去にReact-ViteとFastAPIをvercel上へデプロイする記事もあるので、単独で必要な方は以下をご覧いただければと思います。

**FastAPI on Vercel 記事**
https://zenn.dev/testkun08080/articles/vercel-fastapi-2028fa2c125270

**React-Vite(tailwindcss) on Vercel 記事**
https://zenn.dev/testkun08080/articles/vercel-reactvite-720d4ee65e3a82


#### バックエンドとフロントエンドのやり取り
バックエンドでは、api/logo/logosamples というAPIを提供しています。このAPIは、ロゴデータや関連するURLなどの情報を返します。

(https://react-vite-fast-api-vercel.vercel.app/api/logo/logoIcons)にアクセスすると、以下のようなデータが返ってきます。

```
[
  {
    "name": "fastapi",
    "src": "https://react-vite-fast-api-vercel.vercel.app/api/static/images/svg/fastapi-1.svg",
    "url": "https://fastapi.tiangolo.com"
  },
  {
    "name": "tailwind",
    "src": "https://react-vite-fast-api-vercel.vercel.app/api/static/images/svg/tailwindcss-mark.d52e9897.svg",
    "url": "https://tailwindcss.com"
  },
  {
    "name": "vercel",
    "src": "https://react-vite-fast-api-vercel.vercel.app/api/static/images/svg/vercel-icon-svgrepo-com.svg",
    "url": "https://vercel.com/"
  }
]

```

フロントエンドのトップページでは、このAPIから取得したデータを使用して、ロゴやリンクを表示するシンプルな構成となっています。
`app/src/pages/LogoSamples.jsx`で以下のようにフェッチしてただ表示しているだけです。


https://github.com/testkun08080/ReactVite-FastAPI-Vercel/blob/main/app/src/pages/LogoSamples.jsx

#### 実際にデプロイした、サンプルページ
https://react-vite-fast-api-vercel.vercel.app/

![サンプルページ](/images/vercel-reactvite2-1e6dca130d788a/ss-a.png =512x)
*サンプルページはこんな感じで見えるはずです*


#### テンプレートを使って、とりあえずデプロイしたい方

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/import/git?s=https://github.com/testkun08080/ReactVite-FastAPI-Vercel)

:::message alert
バックエンドはapiをルートにしないと、vercel上でうまく起動できなかったため、apiとしています。
※何か他の方法があると思いますが、ひとまずapiという名前にしておきます。
requirements.txtも、ルートフォルダ以下に置かないとインストールがうまくいきませんでした。
:::

## 前提条件
- npm Node.js 20+ (インストールは[こちら](https://nodejs.org/en/download/))
- Python 3.12+ (vercelの最新が3.12対応のため - 2025/6/3)
- uv (インストールは[こちら](https://docs.astral.sh/uv/getting-started/installation/))
- Vercel CLI (インストールは[こちら](https://vercel.com/docs/cli#installing-vercel-cli/))


## Vercel CLI を使ってテストからデプロイ/ホスティングまで

1. **クローン**
    ```bash
    git clone https://github.com/testkun08080/ReactVite-FastAPI-Vercel.git
    cd ReactVite-FastAPI-Vercel
   ```
2. **モジュールをインストール**
    ```bash
    cd app
    npm install
    cd ..
3. **ローカルでテスト**
    ```bash
    vercel dev
   ```
4. **vercelへデプロイ（プレビューとして）**
    ```bash
    vercel
   ```
5. **、vercelへデプロイ（プロダクトとして）**
    ```bash
    vercel --prod
   ```

## まとめ
特にweb周りに詳しくない人ですが、スムーズにバックエンドとフロントをデプロイするまでを経験できたのはいい経験になりました。
オープンソースのプロジェクトを立ち上げて、webなどを使用することにはなりそうなので、どこかで役に立ちそうです。
スマホとの連携が必要になりそうなので、reac nativeとかflutterもその際は必要になりそうです。

何か不明点、ツッコミ、不具合などあればご連絡ください。
それではまた！