---
title: "FastAPIをVercel上でデプロイする"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vercel, FastAPI, python]
published: true
published_at: 2025-06-16 08:00
---


# はじめに
FastAPIをvercel上までデプロイするまでの流れとサンプルデータの記録です。
vercelこれから触る人や、FastAPI使って作ったけどデプロイ先をどこにしようかと試している人向けかと思います。
vercelは無料でかなり使えますので、ぜひお試しください。（configの書き方さえわかればかなり簡単だと思います）

#### 実際にデプロイした、サンプルページ
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


#### テンプレート
こちらからVercel上へクローンして簡単にデプロイすることも可能です。
[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/import/git?s=https://github.com/testkun08080/FastAPI-vercel)
:::message
GitとVercelの連携が必要です
:::


## 事前インストール必要なもの

- Python 3.12 (vercelの最新が3.12対応のため - 2025/6/3)
- uv (インストールは[こちら](https://docs.astral.sh/uv/getting-started/installation/))
- Vercel CLI (インストールは[こちら](https://vercel.com/docs/cli#installing-vercel-cli/))

#### 注意

:::message alert
バックエンドはapiをルートにしないと、vercel上でうまく起動できなかったため、apiとしています。
※何か他の方法があると思いますが、ひとまずapiという名前にしておきます。
requirements.txtも、ルートフォルダ以下に置かないとインストールがうまくいきませんでした。
:::

## ローカル上でセットアップから起動

1. **ローカルへクローンする**
    ```bash
    git clone https://github.com/testkun08080/FastAPI-vercel.git
   ```

3. **仮想環境の構築**
    ```bash
    cd FastAPI-vercel
    uv venv -p 3.12
    uv pip install -r requirements.txt
   ```
   
4. **起動**
    ```bash
    uv run -m api.app.main
   ```

## Pytestを使ったテスト
1. **テスト用にモジュールをインストール**
    ```bash
    uv pip install -r requirements_test.txt
   ```
1. **Pytest実行**
    ```bash
    uv run pytest --html=report.html --self-contained-html --log-level=INFO
   ```


## Vercel CLI を使ってテストからデプロイ

1. **ローカルでテスト**
    ```bash
    vercel dev
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