---
title: "ReactViteをGitHubPageにデプロイするまでの流れ"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, vite, github]
published: false
---


# はじめに
色々webで発信するにあたって無料で使えるGitHubPageを使用する手はないですよね。
ということで、レポの作成からGitHubPageとしてデプロイするまでの流れをここに記録します。

※React + Vite ( + Tailwind )を使ったページをデプロイします。

##　記事をすっ飛ばしてレポだけ見せろ。という方はこちら。
https://github.com/testkun08080/ReactVite-On-GithubPage
*サンプルページは[こちら](https://testkun08080.github.io/ReactVite-On-GithubPage/)*

## 事前準備

- git cli(インストールは[こちら](https://cli.github.com/))
- npm Node.js 20+ (インストールは[こちら](https://nodejs.org/en/download/))


## セットアップ
基本的には、レポに入れておいた[setup-react-vite.sh](https://github.com/testkun08080/ReactVite-On-GithubPage/blob/main/setup-react-vite.sh)をダウンロードしてて、下記で説明する様にすれば、問題なくデプロイできるはずです。
以下にシェルスクリプトを使った手順と、一つ一つステップを踏んで行う手順を記載します。

[手順A(シェルスクリプト使用)](#%E8%A6%8B%E5%87%BA%E3%81%971%E3%81%A7%E3%81%99)
[手順B(手動)](#%E8%A6%8B%E5%87%BA%E3%81%971%E3%81%A7%E3%81%99)

## A.シェルスクリプトを使う 🤖
:::details 簡単ステップ🤖

1. **ダウンロードシェルコマンドスクリプト:** 
  React-ViteをベースとしたTailwindを適用するwebアプリケーションのテンプレートを作成します。
  さらに、GitHubPageへのデプロイも兼ねた設定も行います。
   ```bash
    setup-react-vite.sh
   ```
1. **パーミッション付与用のコマンド:** 
   ```bash
    chmod +x setup-react-vite.sh
   ```
2. **セットアップ用コマンドの実行(名前はお好みで)**
    必ず、フロントエンドのフォルダ名と、作るレポジトリの名前、Githubのユーザー名を入れてください
    ```bash
   #./setup-react-vite.sh SampleProject ReactVite-On-GithubPage testkun08080
    ./setup-react-vite.sh <PROJECT_NAME> <REPO_NAME> <USER_NAME>
   ```
3. **ローカルでテスト**
    作成し終わったら、以下コマンドでローカル実行してみてください。
    localhost:5173/<PROJECT_NAME>でアクセスできるはずです。
    ```bash
    npm run dev
   ```
:::

## B.手動で一つずつ踏みながらやってみる ✊

:::details 理解を深めたい方✊

1. まずターミナルを開いて、プロジェクトネームを設定します
   ```
   PROJECT_NAME=${1:-react-vite-app}
   REPO_NAME=${2:-repo-name}
   USER_NAME=${3:-user-name}
   ```

2. 自分の GitHubページのURL設定
   ```
   HOMEPAGE_URL="https://${USER_NAME}.github.io/${REPO_NAME}/"
   ```

3. React-ViteProjectの設定
   ```
   npm create vite@latest "$PROJECT_NAME" -- --template react
   ```

4. プロジェクトへ移動し、モジュールをインストール
   ```
   cd "$PROJECT_NAME"
   npm install
   npm install react-router-dom
   npm install gh-pages --save-dev
   ```

5. ホームページをpackage.jsonへ追加します
   ```
   jq --arg homepage "$HOMEPAGE_URL" \
   '.homepage = $homepage | .scripts += {"predeploy": "npm run build", "deploy": "gh-pages -d dist"}' \
   package.json > package.tmp && mv package.tmp package.json
   ```

6. Tailwind CSSをインストール
   ```
   npm install -D tailwindcss @tailwindcss/vite postcss autoprefixer
   npx tailwindcss init
   ```

7. Tailwind の設定ファイルを変更
   ```
   cat <<EOL > tailwind.config.js
    /** @type {import('tailwindcss').Config} */
    export default {
    content: [
        "./index.html",
        "./src/**/*.{js,jsx,ts,tsx,html}",
    ],
    theme: {
        extend: {},
    },
    plugins: [],
    };
    EOL
   ```

8. Tailwind CSSをindex.css へ追加
   ```
   cat <<EOL > src/index.css
   @import "tailwindcss";
   EOL
   ```

9. tailwindcssをプラグインとしてviteへ追加
    ```
    cat <<EOL > vite.config.js
    import { defineConfig } from 'vite'
    import react from '@vitejs/plugin-react'
    import tailwindcss from '@tailwindcss/vite'

    // https://vite.dev/config/
    export default defineConfig({
    plugins: [
        react(),
        tailwindcss(),
    ],
    base: /$REPO_NAME/
    })
    EOL
    ```
    
10. 起動!!!
    ```
    npm run debv
    ```

11. うまく起動すれば、localhost:5173/<PROJECT_NAME>へブラウザからアクセスしてみてください。

:::




## デプロイ
1. **初期コミット:** 
   ```bash
    git init
    git add .
    git commit -m "Initial commit"
2. **レポジトリ作成:** 
   手動でレポジトリを作成、もしくは以下Github CLIを使用
   ```bash
   gh repo create --public --source=.
   ```
3. **プッシュ:** 
   ```bash
    git branch -M main 
    git push -u origin main
   ```
4. **Reactアプリケーションフォルダへ移動:** 
   ```bash
    cd <Your_App_Folder>
   ```
5. **GitHubPage用にビルドとデプロイ**
    事前にbuildも行われて、distフォルダの中身のみがデプロイされずはずです
    ```bash
    npm run deploy
   ```
6. **GitHubPageの確認**
   
    Git Actionsでdeployが完了しているかどうかも確認してください。<br>
   Settings > Pages > Visit Site で アプリケーションが正常に表示されるか確認してみてください。<br>
   もしくは、以下のような　github page URL(適宜、ご自分のプロジェクトに合わせて)に行って最終確認をして下さい。

    ```bash
    https://USER_NAME.github.io/REPO_NAME/
   ```
   
   こちらは、このレポジトリのサンプルです。
   (https://testkun08080.github.io/ReactVite-On-GithubPage/)


## まとめ
会社員勤めでwebをただ使うだけだった僕にとって、これは良いチュートリアルとなりました。
また誰かの手助けになれば幸い。
もし間違っているところなどあればコメントください。

それではまた！