---
title: "チャチャっとクイック、FastAPI"
emoji: "🐍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [FastAPI, python, docker]
published: true # trueを指定する
published_at: 2025-06-11 11:11
---


# 概要
とりあえず、FastAPIでチャチャっと何か作るときに素早く立ち上げたい。
そんなときに、多分便利なshellスクリプトと手順をまとめたものです。


## 感想
イメージのビルドとかに時間は多少かかるものの、全て揃っていればとりあえず2分未満でDoker上で動くまでは行くはず。（個人環境でテスト）


## 開発環境
- macOS Sequoia 15.5
- VsCode
- zsh 5.9 (arm64-apple-darwin24.0)


## 前提条件

システムに以下がインストールされていることを確認してください:

- Python 3.12
- uv pip (インストールは[こちら](https://docs.astral.sh/uv/getting-started/installation/))
  
Docker上で起動するには:
- Dcoker (インストールは[こちら](https://github.com/docker/docker-install))
  
---

## 使い方

### このリポジトリをクローンするか、シェルスクリプトをダウンロードする

```bash
git clone https://github.com/testkun08080/FastAPI-QuickSetup.git
cd FastAPI-QuickSetup
```

または以下スクリプトを直接ダウンロードしてください:

> [!NOTE]
> ファイルをダウンロードするには、リンクを右クリックして「名前を付けてリンク先を保存」などのオプションを選択してください。

[quickFastAPI.sh をダウンロード](https://raw.githubusercontent.com/testkun08080/FastAPI-QuickSetup/main/quickFastAPI.sh)  
[setupDocker.sh をダウンロード](https://raw.githubusercontent.com/testkun08080/FastAPI-QuickSetup/main/setupDocker.sh) 


*うまくいかない場合や直接レポをみたい方はこちらを。*
https://github.com/testkun08080/FastAPI-QuickSetup


### ローカルに環境を構築
1. スクリプトを実行して、ローカル環境に FastAPI プロジェクトをセットアップします:
    ```bash
    ./quickFastAPI.sh
    ```
2. ターミナルでプロジェクト名を入力するように求められます:
    ```bash
    Setting up a FastAPI project...
    Enter the project name: <PROJECT_NAME>
    ```
3. FastAPI サーバーをすぐに起動するかどうか尋ねられます:
    ```bash
    Would you like to start the FastAPI server now? (y/n): 
    ```
4. 後で手動でサーバーを起動するには、以下のコマンドを使用します:
    ```bash
    cd <PROJECT_NAME>
    ./run.sh
    ```
5. アプリケーションがローカルで起動したら、以下の URL をブラウザで開いてアクセスします:
   http://127.0.0.1:8000/


### Docker でアプリケーションをビルド・実行
1. Docker セットアップスクリプトに実行権限を付与し、実行:
    ```bash
    chmod +x setupDocker.sh
    ./setupDocker.sh
    ```
2. プロンプトが表示されたら、Docker ビルドに使用するディレクトリを番号で選択:
    ```bash
    Select a directory from the list below:
    1) <PROJECT_NAME>/
    ```
3. セットアップが正常に完了すると、次のようなメッセージが表示されます:
    ```bash
    Docker setup complete!
    Your FastAPI application is now running at: http://localhost:8000
    ```
    こちらにアクセスできるはずです。（できない場合は他に同じポートが使われていないか確認してください。）
    http://localhost:8000


## 最後に
UVを含んだイメージをDocker上でやるのと小さなpythonイメージを使うのはどちらがいいのか。
サイズで的には、pythonXXX-slimとかが良さそうだけど。
P.S. これが誰かに役立つことを願って、、、

*もっと高速でFastAPIの環境揃えられるというやり方あれば教えてください！*