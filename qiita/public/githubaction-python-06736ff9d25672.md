---
title: 官報とは。python/UV/Docker/GitHubActionsで通知システムを勢いだけで構築してみたが。
tags:
  - Python
  - RSS
  - UV
  - Docker
  - GitHubActions
private: false
updated_at: '2025-07-28T19:42:28+09:00'
id: 5bd3049ddc4eb6ac0b9d
organization_url_name: null
slide: false
ignorePublish: false
---

![官報通知ぼっと](https://github.com/testkun08080/kanpo-rss/blob/main/temp/%E5%AE%98%E5%A0%B1bot.png?raw=true)

# はじめに（なぜに作った？）
参院選が終わってまた色々と変わりそうだけど、若い年代の投票率が増えたのは良い傾向だなぁと思っているこの頃です。
移民問題、帰化人などなど騒がれていますが、そこで興味持ったのが官報。
全く日常で使わないものですけど、一旦購読してみようかなと思った日、、RSSもなければ通知だけXアカウントも見つからず。

となれば、
とりあえず作っちゃえということで…

## 目指す全体の構成
 1. **官報スクレイパー GitHub Action を作る**
    汎用的なスクレイピングアクションの作成(公式ページをスクレイピングして情報を抽出する（PDFリンクと、書く官報の目次タイトルとリンク）)
 2. **GitHub ActionsでRSSファイルを自動生成**
    毎日8:35分ごろに動作するRSS作成ワークフローとレポの作成(簡易ビュー用のgithub pageを含む)
 3. **作られたRSSを元に投稿するBotを作成**
    簡単なワークフローを作成して、投稿するボット

これらをベースに、以下作成したものです。
特に新しいことは何もないですが、`uv` on `docker` や pythonとgithubactionの組み合わせなどを参考にしたい方などは参考になると思います。


## 作ったもの

1. ✅ **官報スクレイパー GitHub Actions**  
　👉 <https://github.com/testkun08080/action-kanpo>

2. 📡 **非公式 官報RSS/簡易webページ含む**  
　👉 <https://github.com/testkun08080/kanpo-rss>

3. 🐤 **Xボット（自動ポスト）**  
　👉 <https://github.com/testkun08080/kanpo-tweet>


## この記事でわかること
- uvをDockerで使って、それをGitHub actions上で
- GotHub actions をマーケットへ公開
- GitHub actions x python でRSS作成
- GitHub pageをAll handsとSlackで作成してみる
- GitHub actions を通して、X上へ自動投稿する

## 開発環境

- 💻 **macOS Sequoia 15.5**
- 🧑‍💻 **VS Code**
- 🐚 **zsh 5.9 (arm64-apple-darwin24.0)**


# 1. 官報スクレイパー GitHub Action を作る
毎朝更新される官報（インターネット版）から、  
当日もしくは、指定された日付から、PDFとそのタイトル、目次情報をスクレイピングして取得し、  
GitHub Actionsのステップで扱える形（`outputs`）に整えて渡すのが目的です。

*とりあえず、サンプルのデータ見てみたいという方は、以下レポをご覧ください。*
https://github.com/testkun08080/action-kanpo

## フロー概要

1. 日付指定やPDFのダウンロード可否を引数でハンドルできる、pythonスクリプトの作成
2. `uv`と`docker`を使用した`action.yml` の作成
3. `GITHUB_OUTPUT` 経由で各ステップに値を渡す


#### アクションを通して自動DL/作成されるファイルイメージ
![イメージ](https://raw.githubusercontent.com/testkun08080/action-kanpo/refs/heads/main/docs/%E5%87%BA%E5%8A%9B%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%A4%E3%83%A1%E3%83%BC%E3%82%B8.png)

<details><summary>出力マークダウンサンプル</summary>

![サンプル官報マークダウン](https://github.com/testkun08080/action-kanpo/blob/main/kanpo/2025-07-11/%E5%AE%98%E5%A0%B1_2025-07-11.md)
</details>


### 1.pythonスクリプトの作成
公式サイトを確認して、指定された`90日以内`の日付とPDFをダウンロードするかどうかなどの入力に応じて、ローカルでも動くように以下のように取得します。
また、出力用に`GITHUB_OUTPUT`へも対応します。

#### コード
https://github.com/testkun08080/action-kanpo/blob/main/fetch_kanpo.py

<details><summary>入力と出力部分</summary>

```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="官報PDF自動取得ツール")
    parser.add_argument("--dlpdf", type=str2bool, nargs="?", const=True, default=False, help="PDFをダウンロードするか")
    parser.add_argument("--date", type=str, help="対象日付 (例: 2025-07-03)")
    args = parser.parse_args()

    kanpo_found = False
    pdf_infos = []
    toc_infos = []

    try:
        if args.date:
            try:
                target_date = datetime.strptime(args.date, "%Y-%m-%d")
            except ValueError:
                logging.error("❌ 日付の形式が不正です。例: YYYY-MM-DD")
                logging.error(f"入力された日付: {args.date}")
                exit(1)
        else:
            target_date = datetime.now(ZoneInfo("Asia/Tokyo"))

        fetcher = KanpoFetcher(target_date=target_date)

        if args.dlpdf:
            logging.info("🚀 本番モード: 官報PDFをダウンロードして、目次情報も取得します")
            kanpo_found, pdf_infos, toc_infos = fetcher.run()
        else:
            logging.info("🔍 確認モード: 官報のPDFリンクと、目次情報のみを取得します")
            kanpo_found, pdf_infos, toc_infos = fetcher.test_run()

        if kanpo_found:
            logging.info(f"📄 kanpo_found: {kanpo_found}")
            logging.info("🎉 取得成功")
        else:
            logging.warning("⚠️ 官報が見つかりませんでした")

        if len(pdf_infos) > 0:
            logging.info(f"📄 PDFリンク数: {len(pdf_infos)}")
            # for pdf in pdf_infos:
            #     logging.info(f"PDF情報:{pdf}")

        if len(toc_infos) > 0:
            logging.info(f"📄 目次数: {len(toc_infos)}")
            # for top in toc_infos:
            #     logging.info(f"目次情報:{top}")

    except KeyboardInterrupt:
        logging.error("👋 中断されました")
    except Exception as e:
        logging.error(f"❌ 実行時エラー: {e}")

    # Make outputs
    if "GITHUB_OUTPUT" in os.environ:
        output_path = os.environ.get("GITHUB_OUTPUT")
        logging.info("GITHUB_OUTPUT 環境変数が設定されています。出力を行います。")
        with open(output_path, "a") as fh:
            print(f"kanpo_found={kanpo_found}", file=fh)
            print(f"pdf_infos={pdf_infos}", file=fh)
            print(f"toc_infos={toc_infos}", file=fh)

        if output_path:
            logging.info(f"GITHUB_OUTPUT のパス: {output_path}")
            try:
                with open(output_path, "r") as f:
                    content = f.read()
                logging.info("GITHUB_OUTPUT ファイルの中身:")
                logging.info(content)
            except Exception as e:
                logging.error(f"ファイル読み込みエラー: {e}")
        else:
            logging.warning("GITHUB_OUTPUT 環境変数が設定されていません。")
    else:
        logging.info("ローカルで起動しているのか、GITHUB_OUTPUT 環境変数が設定されていません。")
        result = {"kanpo_found": kanpo_found, "pdf_infos": pdf_infos, "toc_infos": toc_infos}
        print(json.dumps(result, ensure_ascii=False))
```

</details>

#### ローカルでのテストサンプル
https://github.com/testkun08080/action-kanpo/tree/main?tab=readme-ov-file#%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E3%83%86%E3%82%B9%E3%83%88


### `uv`と`docker`を使用した`action.yml` の作成
引数の設定と、dockerfileとしてを使用して動くように指定します。
また、マーケット用の設定もいくつか。

```yml
name: "fetch-kanpo-action"
description: "官報の更新がないかをチェックする GitHub Actionです。主に官報のメインページを参照し、指定した日付の官報が発行されているかの確認を行います。 また、フラグ管理でダウンロードすることも可能です"
author: "testkun08080"

branding:
  icon: "cpu"
  color: "blue"

inputs:
  dlpdf:
    description: "PDFをダウンロードするか"
    required: false
    default: "false"
  date:
    description: "対象日付 (例: 2025-07-03), 指定しない場合は当日の日付を使用"
    required: false

outputs:
  kanpou_found:
    description: '官報が見つかったかどうか'
  pdf_infos:
    description: 'PDFの情報リスト'
  toc_infos:
    description: '目次の情報リスト'

runs:
  using: "docker"
  image: "Dockerfile"
  args:
    - "--dlpdf"
    - "${{ inputs.dlpdf }}"
    - "--date"
    - "${{ inputs.date }}"
```

#### Dockerfile
引数をGitHubActionsから渡して、uvをdocker上で動かして、やる際はおそらく以下のようになるはずです。（そもそもuv on docker in githubactions というのをやりたかっただけです。。。）

```Dockerfile
# Use a Python image with uv pre-installed
FROM ghcr.io/astral-sh/uv:python3.12-bookworm-slim

COPY fetch_kanpo.py /fetch_kanpo.py
COPY pyproject.toml /pyproject.toml
RUN uv sync

# 🐛 デバッグ追加
RUN echo "📂 中身:" && ls -al

ENTRYPOINT ["uv", "run", "/fetch_kanpo.py"]
```
#### 使用想定ワークフローサンプル
<details><summary>使用想定ワークフローサンプル</summary>

```yml
name: 毎日官報発行チェッカー

on:
  schedule:
    - cron: "45 23 * * *" # 毎日8:45 JST
  workflow_dispatch:

jobs:
  fetch-kanpo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: 官報チェック(DL無し)
        id: fetch-kanpo-step
        uses: testkun08080/action-kanpo@main
        with:
          dlpdf: "false" # PDFをダウンロードしない
          # date: "2025-07-03" # 日付を指定する場合はここに入力 ない場合は当日の日付を使用
    outputs:
      kanpo_found: ${{ steps.fetch-kanpo-step.outputs.kanpo_found }}
      pdf_infos: ${{ steps.fetch-kanpo-step.outputs.pdf_infos }}

  show-fetch-kanpo-results:
    needs: fetch-kanpo
    runs-on: ubuntu-latest
    steps:
      - name: 結果を表示
        run: |
          echo "官報取得成功 or 失敗: ${{needs.fetch-kanpo.outputs.kanpo_found}}"
          echo "取得PDF数: ${{needs.fetch-kanpo.outputs.pdf_infos}}"

  fetch-dl-kanpo:
    needs: fetch-kanpo
    if: needs.fetch-kanpo.outputs.kanpo_found == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 官報チェック(DLあり)
        id: fetch-dl-kanpo-step
        uses: testkun08080/action-kanpo@main
        with:
          dlpdf: "true" # PDFをダウンロードする
          # date: "2025-07-03" # 日付を指定する場合はここに入力 ない場合は当日の日付を使用
      - name: 自動コミット
        uses: stefanzweifel/git-auto-commit-action@v6
        with:
          commit_message: 官報のダウンロード結果の自動コミット
    outputs:
      kanpo_found: ${{ steps.fetch-dl-kanpo-step.outputs.kanpo_found}}
      pdf_infos: ${{ steps.fetch-dl-kanpo-step.outputs.pdf_infos }}

  show-fetch-dl-kanpo-results:
    needs: fetch-dl-kanpo
    runs-on: ubuntu-latest
    steps:
      - name: 結果を表示
        run: |
          echo "官報取得成功 or 失敗: ${{needs.fetch-dl-kanpo.outputs.kanpo_found}}"
          echo "取得PDF数: ${{needs.fetch-dl-kanpo.outputs.pdf_infos}}"
```
</details>


# 2. GitHub ActionsでRSSファイルを自動生成(簡易webビューワーも)
上記１で作成したアクションを流用して、出力結果を元にRSS用のファイルを作成します。
RSSなどは作成したことない筆者ですが、何か不適切あればご連絡ください。

*とりあえず、サンプルのデータ見てみたいという方は、以下レポをご覧ください。*
https://github.com/testkun08080/kanpo-rss

官報の情報をもとに、RSSフィード（`feed.xml`と`feed_toc.xml`）を自動生成・更新します。

## ワークフローの仕組み

1. `action-kanpo` で最新情報を取得
2. `updated == true` のときのみRSSを更新
3. フィード情報 を `main` ブランチにコミット
4. RSSが更新されたのをキャッチして、GitHub Pagesで簡易ビューワーのデプロイ

### RSS作成ワークフロー
時間が絶妙にズレるので、少しずらして実行するなどをしていますが、うまくピッタリには動きません。
https://github.com/testkun08080/kanpo-rss/blob/main/.github/workflows/generate-rss.yml

### 簡易ビューワー
RSSだけあっても、微妙かなぁと思い。。。
まだクレジットが微妙に残っていた`All Hands` でどこまで全部やってくれるかなと実験してみました。

#### ひとまずallhands と slackを繋げる
公式を見つつ連携します。
https://docs.all-hands.dev/usage/cloud/slack-installation

あとは以下のようなコマンドテキストでslack経由で送ってあげます。
```
vite react をベースとして、daisy UIをモジュールとして使い。
官報のRSSをリンクをwebサイト上で簡単に見れる様なGitHub pageを作成してください。
タイトルなどからタグ分けを行い、フィルタリングできるように。
また、カレンダーなどを使って絞り込みができるように。
```

これでこんな感じのwebが作成されました。（簡易ビューワーとしては十分かと）
![ビューワー](https://raw.githubusercontent.com/testkun08080/kanpo-rss/refs/heads/main/temp/%E3%83%92%E3%82%99%E3%83%A5%E3%83%BC%E3%83%AF%E3%83%BC.png)

https://testkun08080.github.io/kanpo-rss/

実際には、大まかに作られた後にアイコンやレイアウトなどは変えていますが触ったのは10行ぐらいです。


#### 自動デプロイワークフロー
自動デプロイにも備えて、以下のようなワークフローを設定
※pushだと自動デプロイがキャッチしなかったので、ワークフローの完了で実行するようにしています。
https://github.com/testkun08080/kanpo-rss/blob/main/.github/workflows/deploy-viewer.yml

---


# 3. RSSを使ってに投稿するBotを作成

上記２で作成したRSSファイルを流用して、Xへ投稿します。
ひとまずシンプルなRSS（`feed.xml`）を使用することにします。
`feed_toc.xml`を使うと大量のポストになりそうなので。

*とりあえず、サンプルのデータ見てみたいという方は、以下レポをご覧ください。*
https://github.com/testkun08080/kanpo-tweet


## ワークフローの仕組み

1. 適当なポスト用Xアカウントの作成
2. RSSフィードをpythonとGitHubActionsで定期取得（月-金）
3. 判定結果を元に、ポスト

## 実際のワークフロー
https://github.com/testkun08080/kanpo-tweet/blob/main/.github/workflows/kanpo-tweet.yml

デフォルトには、更新する時間間隔とfeed用のURLを設定。
実装のポストするアクション達はこちらを拝借させていただいています。

`azu/rss-to-twitter@v2`
https://github.com/azu/rss-to-twitter

`noweh/post-tweet-v2-action@v1.0`
https://github.com/noweh/post-tweet-v2-action

1. Xアカウントの作成
   以下ブログを参考にさせて頂きました。
   （特に詰まることはないと思いますが、アカウント申請用の文面が少し長いな。と思ったぐらいです。）
   https://efcl.info/2023/05/28/rss-to-twitter/
   

2. pythonでxmlをパースして、itemの公開日時と入力された時間幅を比較して、新しければでitemを返すスクリプト
    ```python
    import sys
    import feedparser
    import time
    from datetime import datetime, timedelta, timezone
    import os
    import logging
    import json


    def main():
        logging.basicConfig(level=logging.INFO)

        rss_url = sys.argv[1]
        minutes = int(sys.argv[2])
        window = datetime.now(timezone.utc) - timedelta(minutes=minutes)

        logging.info(f"RSS URL: {rss_url}")
        logging.info(f"チェック時間幅: {minutes}分前 = {window.isoformat()}以降")

        feed = feedparser.parse(rss_url)
        updated_entries = []

        for entry in feed.entries:
            if hasattr(entry, "published_parsed"):
                logging.info(f"公開日時: {entry.published}")
                pub_dt = datetime(*entry.published_parsed[:6], tzinfo=timezone.utc)
            else:
                continue  # pubDateがないitemはスキップ

            if pub_dt >= window:
                updated_entries.append({
                    "title": entry.get("title", ""),
                    "link": entry.get("link", ""),
                    "pubDate": pub_dt.strftime("%Y-%m-%d %H:%M:%S, GMT"),
                })

        updated = bool(updated_entries)

        # Make outputs
        if "GITHUB_OUTPUT" in os.environ:
            output_path = os.environ["GITHUB_OUTPUT"]
            logging.info("GITHUB_OUTPUT 環境変数が設定されています。出力を行います。")
            with open(output_path, "a") as fh:
                print(f"updated={'true' if updated else 'false'}", file=fh)
                print(f"entries={json.dumps(updated_entries, ensure_ascii=False)}", file=fh)

            logging.info(f"GITHUB_OUTPUT のパス: {output_path}")
            try:
                with open(output_path, "r") as f:
                    content = f.read()
                logging.info("GITHUB_OUTPUT ファイルの中身:")
                logging.info(content)
            except Exception as e:
                logging.error(f"ファイル読み込みエラー: {e}")
        else:
            logging.info("ローカルで起動しているか、GITHUB_OUTPUT 環境変数が未設定です。標準出力します。")
            result = {"updated": updated, "entries": updated_entries}
            print(json.dumps(result, ensure_ascii=False, indent=2))


    if __name__ == "__main__":
        main()
   ```

3. 返された値を確認して、それごとにポストします。
   

#### 投稿例

![ポスト例](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/githubaction-python-06736ff9d25672/ss-a.png)

---


## まとめ、感想
ここまでご覧頂き有難うございました！
とりあえず、熱が冷めないうちに作ってみたものの。。これで非公式官報通知の仕組みは作れたはずです。。
体感的にAIエージェントガンガン使ってコードは適当に人間がわかる範囲に直すぐらいで作り終えました。（多分実務でなら3日程度...修正含めて）
あとは、既存のものを流用。
謎のとりあえずやってみよう感で作りましたが、どこかで役に立てばいいなぁ〜という感じです。

一通りの流れは多分今後のプロジェクトを広める時に使えるかなぁ〜と思うところです。
それでは。

# 参考リンク
- 官報インターネット版: https://kanpou.npb.go.jp/
- GitHub Actions: https://docs.github.com/actions
- RSS仕様: https://www.rssboard.org/rss-specification
- feedparser: https://pythonhosted.org/feedparser/
