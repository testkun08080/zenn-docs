---
title: "TEST TEST"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, vite, github, 入門]
published: false
---

# はじめに
色々webで発信するにあたって無料で使えるGitHubPageを使用する手はないですよね。
ということで、レポの作成からGitHubPageとしてデプロイするまでの流れをここに記録します。
*React + Vite (+ Tailwind )を使ったページをデプロイします。*
tete

### サンプルページ
こちらで紹介する記事の最後には、おそらく以下のような画面が表示されます
![サンプルページイメージ](/images/react-github-b6fb36b54a7e31/reactvite-github.png)

以下は事前に用意したサンプルページになります。
https://testkun08080.github.io/ReactVite-On-GithubPage

## レポだけみたい方はこちら。
https://github.com/testkun08080/ReactVite-On-GithubPage


## 事前準備

- git cli(インストールは[こちら](https://cli.github.com/))
- npm Node.js 20+ (インストールは[こちら](https://nodejs.org/en/download/))


## セットアップ
以下にシェルスクリプトを使った手順と、一つ一つステップを踏んで行う手順を記載します。

- [A(シェルスクリプト使用)](#a.%E3%82%B7%E3%82%A7%E3%83%AB%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%97%E3%83%88%E3%82%92%E4%BD%BF%E3%81%86-%F0%9F%A4%96)
- [B(手動)](#b.%E6%89%8B%E5%8B%95%E3%81%A7%E4%B8%80%E3%81%A4%E3%81%9A%E3%81%A4%E8%B8%8F%E3%81%BF%E3%81%AA%E3%81%8C%E3%82%89%E3%82%84%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B-%E2%9C%8A)

## A.シェルスクリプトを使う 🤖
:::details 簡単ステップ🤖

1. **ローカルへクローンまたはダウンロード**
   ```bash
   git clone https://github.com/testkun08080/ReactVite-On-GithubPage.git
   cd ReactVite-On-GithubPage
   ```
   またはスクリプト`setup-react-vite.sh`のみダウンロード
   ```bash
   curl -o setup-react-vite.sh https://raw.githubusercontent.com/testkun08080/ReactVite-On-GithubPage/main/setup-react-vite.sh
   ```

2. **ページを作成したいリポジトリにスクリプトをコピー**
   ```bash
   cp setup-react-vite.sh /path/to/YourGitRepo/
   cd /path/to/YourGitRepo/
   ```

3. **パーミッション付与**
   ```bash
   chmod +x setup-react-vite.sh
   ```

4. **セットアップ用コマンドの実行（名前は任意）**
   フロントエンドをルート以下の別フォルダに作成したい場合
   ```
   /gitrepo
   └─ frontendapp
      └─ src
   ```

   ```bash
   # ./setup-react-vite.sh SampleProject ReactVite-On-GithubPage testkun08080
   ./setup-react-vite.sh <PROJECT_NAME> <REPO_NAME> <USER_NAME>
   ```

   もしくは新しいリポジトリのルートをフロントエンドと同じにしたい場合
   ```
   /frontendapp(gitrepo)
   └─ src
   ```
   ```bash
   # ./setup-react-vite.sh ReactVite-On-GithubPage ReactVite-On-GithubPage testkun08080
   ./setup-react-vite.sh <PROJECT_NAME> <REPO_NAME> <USER_NAME>
   ```

5. **ローカルでテスト**
   作成されたプロジェクトフォルダへ移動し、ローカルで実行してください。  
   `localhost:5173` でアクセスできるはずです。
   ```bash
   cd <PROJECT_NAME>
   npm run dev
   ```

:::

## B.手動で一つずつ踏みながらやってみる ✊

:::details 理解を深めたい方✊

1. まずターミナルを開いて、プロジェクトネームを設定します
   ```
   PROJECT_NAME=react-vite-app
   REPO_NAME=repo-name
   USER_NAME=user-name
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
    npm run dev
    ```

11. うまく起動すれば、localhost:5173/<PROJECT_NAME>へブラウザからアクセスしてみてください。
:::

## デプロイ
一応２通りの方法を記載しています。
既存のレポへの追加か、それとも新しく作るかの違いです。
- [A(新しくリポジトリを作成)](#1(a).-%E6%96%B0%E3%81%97%E3%81%8F%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B%E5%A0%B4%E5%90%88)
- [B(既存レポジトリへコミット)](#1(b).-%E6%97%A2%E5%AD%98%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E3%81%B8%E3%82%B3%E3%83%9F%E3%83%83%E3%83%88%E3%81%97%E3%81%A6%E3%83%97%E3%83%83%E3%82%B7%E3%83%A5%E3%81%99%E3%82%8B%E5%A0%B4%E5%90%88)
- ### 1(A). 新しくリポジトリを作成する場合

  1. **初期コミット:** 
    ```bash
    git init
    git add .
    git commit -m "Initial commit"
    ```
  2. **リポジトリ作成:** 
    ```bash
    gh repo create --public --source=.
    ```
  3. **プッシュ:** 
    ```bash
    git branch -M main
    git push -u origin main
    ```

- ### 1(B). 既存リポジトリへコミットしてプッシュする場合
  1. **コミットとプッシュ:**
      ```bash
      git add .
      git commit -m "add react vite app"
      git push origin main
      ```

- ### 2. アプリをデプロイする
  1. **React アプリケーションフォルダへ移動:** 
      ```bash
      cd <PROJECT_NAME>
      ```
  2. **GitHub Pages 用にビルドとデプロイ**
      事前に build も行われ、`dist` フォルダの中身のみがデプロイされます。
      ```bash
      npm run deploy
      ```
  3. **GitHub Pages の確認**
    GitHub Actions で deploy が完了しているか確認してください。  
    **Settings > Pages > Visit Site** でアプリケーションが正常に表示されるか確認してください。  
    または、以下のような GitHub Pages の URL（ご自身のプロジェクトに合わせて）で最終確認してください:
      ```bash
      https://USER_NAME.github.io/REPO_NAME/
      ```

      こちらはこのリポジトリのサンプルです。  
      [https://testkun08080.github.io/ReactVite-On-GithubPage/](https://testkun08080.github.io/ReactVite-On-GithubPage/)



## まとめ
webをただ使うだけだった僕にとって、これは良いチュートリアルとなりました。
また誰かの手助けになれば幸い。
もし間違っているところなどあればコメントください。

それではまた！