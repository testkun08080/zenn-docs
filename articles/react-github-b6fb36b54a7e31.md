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
*React + Vite (+ Tailwind )を使ったページをデプロイします。*

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

- [A(シェルスクリプト使用)](#%E8%A6%8B%E5%87%BA%E3%81%971%E3%81%A7%E3%81%99)
- [B(手動)](#%E8%A6%8B%E5%87%BA%E3%81%971%E3%81%A7%E3%81%99)

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
一応２通りの方法を記載しています。
既存のレポへの追加か、それとも新しく作るかの違いです。
- [A(新しくリポジトリを作成)](#%E8%A6%8B%E5%87%BA%E3%81%971%E3%81%A7%E3%81%99)
- [B(既存レポジトリへコミット)](#%E8%A6%8B%E5%87%BA%E3%81%971%E3%81%A7%E3%81%99)

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