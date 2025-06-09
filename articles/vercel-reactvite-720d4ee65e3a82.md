---
title: "React-Vite(tailwindcss)をVercel上でデプロイする"
emoji: "🏄‍♂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vercel, react, vite, tailwindcss]
published: false
---


# はじめに
React-Viteをvercel上までデプロイするまでの流れとサンプルデータの記録です。
前回、FastAPIをvercel上にデプロイを行いましたが、そちらも気になる方は以下をご覧ください。

**FastAPI on Vercel 記事**
https://zenn.dev/testkun08080/articles/vercel-fastapi-2028fa2c125270


vercelは無料でかなり使えますので、ぜひお試しください。（configの書き方さえわかればかなり簡単だと思います）


#### 実際にデプロイした、サンプルページ
https://react-vite-vercel-ten.vercel.app/

![サンプルページ](/images/vercel-reactvite-720d4ee65e3a82/ss-a.png =512x)
*サンプルページはこんな感じで見えるはずです*


## 感想
高速でwebアプリをデプロイできるので、テストでちょっとやってみようという方には良いと思います。

## 開発環境
- macOS Sequoia 15.5
- VsCode
- zsh 5.9 (arm64-apple-darwin24.0)

#### テンプレートを使って、とりあえずデプロイしたい方

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/import/git?s=https://github.com/testkun08080/ReactVite-vercel)


## 事前インストール必要なもの

- npm Node.js 20+ (インストールは[こちら](https://nodejs.org/en/download/))
- Vercel CLI (インストールは[こちら](https://vercel.com/docs/cli#installing-vercel-cli/))

## ローカル上でセットアップから起動

1. **ローカルへクローンする**
    ```bash
    git clone https://github.com/testkun08080/ReactVite-vercel.git
   ```

2. **仮想環境の構築**
    ```bash
    cd ReactVite-vercel/app
    npm install
   ```
3. **ローカル起動**
   以下コマンドを叩いて、ブラウザで<http://localhost:3000>確認できるはずです。
    ```bash
    npm run dev
   ```

## Vercel CLI を使ってテストからデプロイ

1. **ローカル起動**
   app階層にターミナルが移動していること路確認してください。
   あとはお好きなプロジェクト名などに適宜変えてください。
   以下のように順調に進めば、ブラウザで<http://localhost:3000>確認できるはずです。
    ```bash
    vercel dev
   ```
   以下ターミナルのサンプル表示
    ```bash
    YOURUSERNAME ReactVIte-vercel % vercel dev
    Vercel CLI 42.3.0
    ? Set up and develop “~/Git/ReactVIte-vercel”? yes
    ? Which scope should contain your project? testkun08080's projects
    ? Link to existing project? no
    ? What’s your project’s name? react-vite-vercel
    ? In which directory is your code located? ./app
    Local settings detected in vercel.json:
    Auto-detected Project Settings (Vite): 
    - Build Command: vite build
   - Development Command: vite --port $PORT
   - Install Command: `yarn install`, `pnpm install`, `npm install`, or `bun install`
   - Output Directory: dist
   ? Want to modify these settings? no
   🔗  Linked to testkun08080s-projects/react-vite-vercel (created .vercel)
   > Running Dev Command “vite --port $PORT”

     VITE v6.3.5  ready in 389 ms

     ➜  Local:   http://localhost:3000/
     ➜  Network: use --host to expose
     ➜  press h + enter to show help
   > Ready! Available at http://localhost:3000
      ```

2. **vercelへデプロイ（プレビューとして）**
    ```bash
    vercel
   ```

   ![vercel上](/images/vercel-reactvite-720d4ee65e3a82/ss-b.png =512x)
   *vercel上ではこんな感じで見えるはずです*

3. **、vercelへデプロイ（プロダクトとして）**
    ```bash
    vercel --prod
   ```

## まとめ
特に困ることはなく、ローカルでreact-viteのアプリを作って、あとはvercel cliに任せれば、いい感じにデプロイまでやってくれます。
次はReact-Vite with FastAPIも記事にしようと思います。

それではまた！