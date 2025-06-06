---
title: "UVをとりあえず触って取得する"
emoji: "🐍"
type: "tech"
topics: ["python", "UV", "github"]
published: true # trueを指定する
published_at: 2025-06-08 12:00
---


# 概要

[UV](https://docs.astral.sh/uv/)について色々と便利そうなことをまとめたものです。
とにかくpipを使っていたものとしては、高速なのでありがたい。
あとアクティベートとかも鬱陶しいのを省くことも出来るのは良い！

UV触ったことない人や、大量のモジュールをpipでインストールしたりしてる人には試す価値はあると思います。(まだ自分自身も色々なものでは試してないですけど)

#### UVレポはここ
https://github.com/astral-sh/uv


## 感想
とりあえずセットアップインストールのセットアップが早い。
localでactivate/deactivateなどの手間がない。
悪い点は今のところ無し!


## 開発環境
- mac


## インストール 💻

以下のコマンドで UV をインストールできます:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

詳細は [インストールガイド](https://docs.astral.sh/uv/getting-started/installation/) をご覧ください。


## クイックスタート

```bash
uv init heyhey-world
cd heyhey-world
uv run main.py
```


## 仮想環境とモジュールの管理 (`.toml` を使用しない場合)

### 1. 仮想環境の作成

```bash
uv venv
```

*Python のバージョンを指定する場合:*

```bash
uv venv --python 3.11
```

### 2. 仮想環境のアクティベート

```bash
source .venv/bin/activate
```

### 3. モジュールの追加

モジュールをインストール:

```bash
uv pip install requests
```

*バージョンを指定する場合:*

```bash
uv pip install 'requests==2.31.0'
```

`requirements.txt` からインストール:

```bash
uv pip install -r requirements.txt
```

### 4. モジュールの削除

```bash
uv pip uninstall requests
```

### 5. モジュールのエクスポート

```bash
uv pip freeze > requirements.txt
```

---

## 仮想環境とモジュールの管理 (`.toml` を使用する場合)

### 1. プロジェクトの初期化と仮想環境の作成

新しいプロジェクトを初期化:

```bash
uv init
uv venv
```

*カスタム名で初期化する場合:*

```bash
uv init ProjectName
```

### 2. モジュールの追加

モジュールを追加:

```bash
uv add requests
```

*バージョンを指定する場合:*

```bash
uv add 'requests==2.31.0'
```

`requirements.txt` からインストール:

```bash
uv add -r requirements.txt
```

### 3. モジュールの削除

```bash
uv remove requests
```

### 4. モジュールのエクスポート

```bash
uv pip freeze > requirements.txt
```

---

## 既存の `.toml` を使用したセットアップ

###  1. `.toml` から依存関係を同期

```bash
uv sync
```

### 2. 環境変数を指定して実行

```bash
uv run --env-file=.env main.py
```

---

とりあえずUV触る人の手間が省ければ幸い


Githubにも置いておいたので、こちらも参照頂けます。（書いてる内容はほぼ同じです）

https://github.com/testkun08080/test-uv