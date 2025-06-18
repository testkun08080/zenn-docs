---
title: "UVを使ってJupyternotebookをVS codeでセットアップする"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [jupyternotebook, uv, python, vscode]
published: false
---

# はじめに
`UV`を使い始めたばかりのものなんですが、`jupyternotebook(.ipynb)`を`VS Code`で使いたい時が来たので備忘録として残します。
公式ページを引用して作成しましたが、何かあればこちらの[公式ページ](https://docs.astral.sh/uv/guides/integration/jupyter/#creating-a-kernel)をご覧ください。

## 開発環境
- macOS Sequoia 15.5
- VS Code
- zsh 5.9 (arm64-apple-darwin24.0)


## 前提条件

システムに以下がインストールされていることを確認してください:

- Python
- uv pip (インストールは[こちら](https://docs.astral.sh/uv/getting-started/installation/))

:::message
uvについてはこちらも参考にしてみてください
https://zenn.dev/testkun08080/articles/python-uv-99ae614a1a4f13
:::

---

## ローカルでのセットアップ

1. プロジェクトフォルダの作成
   ```
   uv init projectname
   cd projectname
   ```

2. UVをjupyernote のインストール
    ```zsh
    uv add --dev ipykernel
    uv run ipython kernel install --user --env VIRTUAL_ENV $(pwd)/.venv --name=project
    ```

3. ipynbファイルを作成
   ```

   ```

4. VS code 上でカーネルの選択
   ![Altテキスト-Zenn](https://storage.googleapis.com/zenn-user-upload/avatar/9965dabc76.jpeg =150x)


5. 起動
    ![Altテキスト-Zenn](https://storage.googleapis.com/zenn-user-upload/avatar/9965dabc76.jpeg =150x)



## まとめ
一秒でも誰かの時間が節約できれば幸いです
何かミスなどがあれば、コメントください〜

では！
