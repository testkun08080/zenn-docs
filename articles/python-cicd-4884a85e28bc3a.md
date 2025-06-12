---
title: "pyinstallerとGithub Actions(CI/CD)組んでみる"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, cicd, pyinstaller, githubactions, qt]
published: true
published_at: 2025-06-22 15:00
---


# はじめに
Pyinstallerとqt5を使って様々なスタジオへいろんなものをバンドして配布する機会がありました。
今回は、Pyinstallerを使ったパッケージ作成(windows/mac)までの工程をチュートリアル形式でここに残したいと思います。
また、今回はCI/CDにはGitHub Actionsを使って行います。

### チュートリアルの流れ
1. UVを使って、ローカルでPyQt5を使った簡単アプリの起動
2. GitHub Actionsのワークフローを作成(DL)
3. タグの作成とリリース

### サンプルレポ
https://github.com/testkun08080/pyinstaller-CICD

## 開発環境
- macOS Sequoia 15.5
- VsCode
- zsh 5.9 (arm64-apple-darwin24.0)


## 前提条件

システムに以下がインストールされていることを確認してください:

- Python 3.10
- uv pip (インストールは[こちら](https://docs.astral.sh/uv/getting-started/installation/))

:::message
uvについてはこちらも参考にしてみてください
https://zenn.dev/testkun08080/articles/python-uv-99ae614a1a4f13
:::

---

## ローカルでのセットアップ

1. プロジェクトフォルダの作成
   ```
   mkdir pyinstaller-CICD
   cd pyinstaller-CICD
   ```

2. UVを使ってプロジェクトの作成
    ```zsh
    uv init -p 3.10
    uv add pyqt5 pyinstaller
    ```
3. `app.py`の作成
   ```
   cat <<EOL > app.py
   import sys
   from PyQt5.QtWidgets import QApplication, QWidget, QLabel, QVBoxLayout

   class HelloWindow(QWidget):
      def __init__(self):
         super().__init__()
         self.setWindowTitle("Hello PyQt5")
         self.setGeometry(100, 100, 300, 100)

         layout = QVBoxLayout()
         label = QLabel("Hello from PyQt5!")
         layout.addWidget(label)
         self.setLayout(layout)


   if __name__ == "__main__":
      app = QApplication(sys.argv)
      window = HelloWindow()
      window.show()
      sys.exit(app.exec_())
   EOL
   ```

4. ローカル起動テスト
   ```zsh
   uv run app.py
   ```
5. ローカルパッケージ作成テスト
   ```zsh
   uv run pyinstaller app.py
   ```
6. パッケージ起動のテスト
   ```
   uv run dist/app/app 
   ```

## GitHub ActionsでCI/CDを組む

1. `.github/workflows/pyinstaller-build.yml`のダウンロード  
   ```zsh
   # 保存先ディレクトリ
   TARGET_DIR=".github/workflows"

   # 保存先ディレクトリを作成（存在しない場合のみ）
   mkdir -p "$TARGET_DIR"

   # ダウンロードするファイルの正しいURL
   FILE_URL="https://raw.githubusercontent.com/testkun08080/pyinstaller-CICD/main/.github/workflows/pyinstaller-build.yml"

   # ファイルをダウンロード
   curl -o "$TARGET_DIR/pyinstaller-build.yml" "$FILE_URL"

   # 結果の確認
   if [ -f "$TARGET_DIR/pyinstaller-build.yml" ]; then
      echo "ファイルが正常に保存されました: $TARGET_DIR/pyinstaller-build.yml"
   else
      echo "ダウンロードに失敗しました。"
      exit 1
   fi
   ```

2. `project.toml`へ設定を追加
   ```
   cat <<EOF >> pyproject.toml
   [tool.uv]
   environments = [
      "sys_platform == 'win32'",
      "sys_platform == 'darwin'",
   ]
   constraint-dependencies = [
      "pyqt5-qt5<=5.15.2; sys_platform == 'win32'",
      "pyqt5-qt5>=5.15.17; sys_platform == 'darwin'",
   ]
   EOF
   ```

3. 新しいレポジトリの作成
   ```
   gh repo create username/pyinstaller-CICD --public --source=. --remote=origin
   ```

4. コミットとプッシュ
   ```bash
   git init .
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin git@github.com:testkun08080/pyinstaller-CICD.git
   git push -u origin main
   ```

5. タグの作成
   ```
   git tag -a v1.0.0 -m "Release version 1.0.0"
   ```
6. リリース
   ```
   git push origin v1.0.0
   ```
7. ご自身のリリースページへ行って以下の様にリリースされていれば成功です。

    ![リリースイメージ](https://raw.githubusercontent.com/testkun08080/pyinstaller-CICD/refs/heads/main/docs/sample-release.png =512x)

   このレポのリリースページは[こんな感じ](https://github.com/testkun08080/pyinstaller-CICD/releases)です。


### テストでパッケージを作成しまくって、消す際に役立ったコマンド集

- タグ確認
  ```
  git tag
  git ls-remote --tags origin
  ```

- タグ消し
  ```
  git tag -d $(git tag)
  git push origin --delete $(git tag -l)
  ```

- リリース確認
  ```
  gh release list
  git ls-remote --tags origin
  ```

- リリース消去
  ```
  gh release list | awk '{print $1}' | xargs -n1 gh release delete -y
  ```


## まとめ
Actionsのワークフローを組むときに、uvの設定ファイルで少し手こずってしまいました。
`win_amd64`とsys_platformには書いていましたが、正解は`win32`でした。(🤔)
pyinstaller　と GitHubActionsを使う人に役立てたら幸いです。
何かミスなどがあれば、コメントください〜

では！