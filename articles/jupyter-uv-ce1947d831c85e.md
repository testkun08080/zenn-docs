---
title: "UVを使ってJupyter NotebookをVSCodeでセットアップしてみよう!"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [jupyternotebook, uv, python, vscode]
published: true
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

2. `UV`を使って、`jupyernote`のインストール
    ```zsh
    uv add --dev ipykernel
    uv run ipython kernel install --user --env VIRTUAL_ENV $(pwd)/.venv --name=KERNEL_NAME
    ```

    もしくは、サーバーとしてローカル起動したい場合は

    ```
    uv run --with jupyter jupyter lab
    ```

3. ipynbファイルを作成
   `>Create: New Jupyter Notebook`と入力して作成
    ![Create: New Jupyter Notebook](/images/jupyter-uv-ce1947d831c85e/ss-a.png)


4. VS Code 上でカーネルの選択
   ![カーネルの選択](/images/jupyter-uv-ce1947d831c85e/ss-b.png)

   次に、2の工程で作成したカーネルが表示されていれば、それを選択します。

   ![カーネルの選択](/images/jupyter-uv-ce1947d831c85e/ss-d.png)

   :::message
   ない場合は、
   他のカーネルを選択 > Jupyert Kaenrl > 自分の作成したカーネルを選択
   ![カーネルの選択](/images/jupyter-uv-ce1947d831c85e/ss-e.png)
   :::

5. 起動
   あとは、noteに適当なスクリプトを入力して起動テストするだけです
   ![起動テスト](/images/jupyter-uv-ce1947d831c85e/ss-c.png =512x)

   以下のスニペットでどのvenvが有効なのか確認できるはずです。
   ```python
   import sys

   print(f"Python version: {sys.version}")
   print(f"Virtual environment: {sys.prefix}")
   ```
   ![環境確認](/images/jupyter-uv-ce1947d831c85e/ss-f.png)

### その他コマンド集

- ローカルのjupyterカーネルチェック
  ```zsh
  uv run jupyter kernelspec list
  ```
- カーネルの削除
  ```zsh
   uv run jupyter kernelspec uninstall KERNEL_NAME
  ```

## 参考文献
https://docs.astral.sh/uv/guides/integration/jupyter/#creating-a-kernel


## まとめ
一秒でも誰かの時間が節約できれば幸いです!
👍いただけるとモチベ上がります！