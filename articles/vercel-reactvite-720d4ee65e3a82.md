---
title: "React-Vite(tailwindcss)をVercel上でデプロイする"
emoji: "🏄‍♂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vercel, react, vite, tailwindcss]
published: true
published_at: 2025-06-16 08:00
---


#　はじめに
React-Viteをvercel上までデプロイするまでの流れとサンプルデータの記録です。
前回、FastAPIをvercel上にホスティングを行いましたが、そちらも気になる方は以下をご覧ください。

記事サンプル
https://fast-api-vercel-murex.vercel.app/

vercelこれから触る人や、FastAPI使って作ったけどホスティングどこにしようかと試している人向けかと思います。
vercelは無料でかなり使えますので、ぜひお試しください。（configの書き方さえわかればかなり簡単だと思います）


#### サンプルページ
https://fast-api-vercel-murex.vercel.app/

![サンプルページ](/images/vercel-fastapi-2028fa2c125270/ss-a.png =512x)
*サンプルページはこんな感じで見えるはずです*
![サンプルページAPI](/images/vercel-fastapi-2028fa2c125270/ss-b.png =512x)
*API用ドキュメント*

## 感想
無料で簡単にデプロイできるvercelはありがあたい限り。
自分の職種がら全く触ってこなかった分野なので、まぁまぁ楽しいです。
恐らくゲーム会社や映像業界でも、独自のwebサービスなどを社内で持っていると思いますが、
触っておくことで実験的なものを簡単に広めたりできるのでいいと思います。
（パッと思いつくのは、何かしらの巨大なプロセスの監視用とかには最悪役に立つはずです...）

## 開発環境
- macOS Sequoia 15.5
- VsCode
- zsh 5.9 (arm64-apple-darwin24.0)

#### テンプレートを使って、とりあえずデプロイしたい方

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/import/git?s=https://github.com/testkun08080/ReactVite-vercel)

#### サンプルページ
https://react-vite-vercel-ten.vercel.app/


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
    cd ReactVite-vercel
    npm install
   ```
3. **ローカル起動**
   以下コマンドを叩いて、ブラウザで<http://localhost:3000>確認できるはずです。
    ```bash
    npm run dev
   ```

## Vercel CLI を使ってテストからデプロイ

1. **ローカル起動**
   以下コマンドを叩いて、ルートを選ぶ際に'app'と入力してください。
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

3. **、vercelへデプロイ（プロダクトとして）**
    ```bash
    vercel --prod
   ```

## まとめ
とにかく素早くデプロイできるサービスは非常にありがたい。
恐らく今後、オープンソースのプロジェクトで必要な際はを立ち上げるときは使うと思います。
まあ他のredenr/etc...も触っていこうと思います。
近々、React-Viteのvercel上でデプロイや、React-Vite with FastAPIも記事にしようと思います。

それではまた！