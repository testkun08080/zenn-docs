---
title: "官報自動通知X Twitter(X)の改修。python/GitHubActionsで"
emoji: "👨‍🔧"
type: "tech"
topics: [python, githubactions, rss]
published: true
---

![官報通知ぼっと](/images/githubaction-python-4e1d593f4286b1/a.jpg)
*Grok作*

# はじめに
前回、[官報通知システム](https://zenn.dev/testkun08080/articles/githubaction-python-06736ff9d25672)という記事を書いてみたものです。
いざパッと作ってみたものの、、
ツイート部分があまりうまくいかないのと、少しrssの内容をまとめてポストしたくなったので、また記事にしてみてました。

## 官報通知システム（関連レポ）

1. ✅ **官報スクレイパー GitHub Actions**  
　👉 <https://github.com/testkun08080/action-kanpo>

2. 📡 **非公式 官報RSS/簡易webページ含む**  
　👉 <https://github.com/testkun08080/kanpo-rss>

3. 🐤 **Xボット（自動ポスト）**  
　👉 <https://github.com/testkun08080/kanpo-tweet>


## この記事でわかること
- GitHub actions を通して、X上へ自動投稿する
- tweepyライブラリを使用したツイート

## 開発環境

- 💻 **macOS Sequoia 15.5**
- 🧑‍💻 **VS Code**
- 🐚 **zsh 5.9 (arm64-apple-darwin24.0)**


## RSSを使ってX(Twitter)に投稿するBotを作成（改修）

基本的にはRSSを元に既存のActionsを使われていただいていましたが、
ポストへ返信する形で投稿するようにしたかったため、自分でpython スクリプトを作成しました。

*とりあえず、サンプルのデータ見てみたいという方は、以下レポをご覧ください。*
https://github.com/testkun08080/kanpo-tweet

### 改善後のワークフロー (post-feed-to-x.yml)
現在のシステムでは以下ワークフローを運用しています
基本的には、RSSの簡易版と詳細版を二つフェッチして、最新状態を確認して自作のスクリプト経由でポストするという流れです。
見つからなかった場合のみに、`noweh/post-tweet-v2-action@v1.0`を使って、**見つからなかったです。無かったですすみません。**というポストをします。

#### 平日の動作スケジュール
```yml
schedule:
  - cron: "30 23 * * 0-4" # 月曜〜金曜の JST 8:30
```
約10分程度遅れて実行されることが多いです。
（なので、この時間でちょうどいいと判断しています）


```yml
name: Knapo Tweet Post(RSS Feed to X)

on:
  workflow_dispatch:
    inputs:
      rss_url:
        description: "RSSフィードのURL"
        required: true
        default: "https://raw.githubusercontent.com/testkun08080/kanpo-rss/refs/heads/main/feed.xml"
      rss_toc_url:
        description: "RSS_詳細版フィードのURL"
        required: true
        default: "https://raw.githubusercontent.com/testkun08080/kanpo-rss/refs/heads/main/feed_toc.xml"
      minutes:
        description: "何分以内に更新されたアイテムを投稿するか"
        required: true
        default: "720" # 12時間

  schedule:
    - cron: "30 23 * * 0-4" # 月曜〜金曜の JST 8:30 (遅延を考慮して 30分に設定)だいたい10分遅延する


jobs:
  check_rss_and_post_feeds:
    runs-on: ubuntu-latest
    outputs:
      updated: ${{ steps.check_and_post.outputs.updated }}
    env:
      X_API_KEY: ${{ secrets.TWITTER_APIKEY }}
      X_API_SECRET: ${{ secrets.TWITTER_APIKEY_SECRET }}
      X_ACCESS_TOKEN: ${{ secrets.TWITTER_ACCESS_TOKEN }}
      X_ACCESS_TOKEN_SECRET: ${{ secrets.TWITTER_ACCESS_TOKEN_SECRET }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: pip install feedparser tweepy

      - name: Check RSS and Post feed to X
        id: check_and_post
        env:
          RSS_URL: ${{ inputs.rss_url || 'https://raw.githubusercontent.com/testkun08080/kanpo-rss/refs/heads/main/feed.xml' }}
          RSS_TOC_URL: ${{ inputs.rss_toc_url || 'https://raw.githubusercontent.com/testkun08080/kanpo-rss/refs/heads/main/feed_toc.xml' }}
          MINUTES: ${{ inputs.minutes || '720' }}
        run: |
          echo "▶️ RSS_URL: $RSS_URL"
          echo "▶️ RSS_TOC_URL: $RSS_TOC_URL"  
          echo "⏱ MINUTES: $MINUTES"
          python scripts/check_rss_and_posting.py "$RSS_URL" "$RSS_TOC_URL" "$MINUTES"

      - name: Show Results
        run: |
          echo "✅ updated: ${{ steps.check_and_post.outputs.updated }}"
          echo "📝 entries:"
          echo '${{ steps.check_and_post.outputs.entries }}'
  
  re_check_rss_and_post_feeds:
    needs: check_rss_and_post_feeds
    if: needs.check_rss_and_post_feeds.outputs.updated == 'false'
    runs-on: ubuntu-latest
    outputs:
      updated: ${{ steps.check_and_post.outputs.updated }}
    env:
      X_API_KEY: ${{ secrets.TWITTER_APIKEY }}
      X_API_SECRET: ${{ secrets.TWITTER_APIKEY_SECRET }}
      X_ACCESS_TOKEN: ${{ secrets.TWITTER_ACCESS_TOKEN }}
      X_ACCESS_TOKEN_SECRET: ${{ secrets.TWITTER_ACCESS_TOKEN_SECRET }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: pip install feedparser tweepy

      - name: Check RSS and Post feed to X
        id: check_and_post
        env:
          RSS_URL: ${{ inputs.rss_url || 'https://raw.githubusercontent.com/testkun08080/kanpo-rss/refs/heads/main/feed.xml' }}
          RSS_TOC_URL: ${{ inputs.rss_toc_url || 'https://raw.githubusercontent.com/testkun08080/kanpo-rss/refs/heads/main/feed_toc.xml' }}
          MINUTES: ${{ inputs.minutes || '720' }}
        run: |
          echo "▶️ RSS_URL: $RSS_URL"
          echo "▶️ RSS_TOC_URL: $RSS_TOC_URL"  
          echo "⏱ MINUTES: $MINUTES"
          python scripts/check_rss_and_posting.py "$RSS_URL" "$RSS_TOC_URL" "$MINUTES"

      - name: Show Results
        run: |
          echo "✅ updated: ${{ steps.check_and_post.outputs.updated }}"
          echo "📝 entries:"
          echo '${{ steps.check_and_post.outputs.entries }}'
          
  twitter-non-post:
    needs: re_check_rss_and_post_feeds
    if: needs.re_check_rss_and_post_feeds.outputs.updated == 'false'
    runs-on: ubuntu-latest
    steps:
      - uses: noweh/post-tweet-v2-action@v1.0
        with:
          message: "現時刻では官報の更新が見つかりません。\nhttps://www.kanpo.go.jp/"
          consumer-key: ${{ secrets.TWITTER_APIKEY }}
          consumer-secret: ${{ secrets.TWITTER_APIKEY_SECRET }}
          access-token: ${{ secrets.TWITTER_ACCESS_TOKEN }}
          access-token-secret: ${{ secrets.TWITTER_ACCESS_TOKEN_SECRET }}
```

### カスタム投稿スクリプトの詳細
基本構成は、

- RSS達をフェッチ
- キーのチェック
- 本誌、号外などの情報をポスト
- 本誌、号外の詳細をフィルタリングしてリプライとしてポストする(x APIの制限によりオミット中)
- Gemini でまとめを作成させる (これもあんま意味ないのでオミット中)
- `GITHUB_OUTPUT`へリターン値を入れて終了

https://github.com/testkun08080/kanpo-tweet/blob/main/scripts/check_rss_and_posting.py

#### 主要機能
全部コピペでここに載せるのはなんなので、とりあえず目立った部分のみ載せておきます。

1. **Tweepy によるX投稿**
`tweepy`を使ったシンプルな投稿です。(リターンとしてtweet_idを返すだけです。)
```python
def post_to_x(text, in_reply_to_tweet_id=None):
    """
    テキストをX（旧Twitter）に投稿します。tweepy（Twitter API v2）を使用します。
    in_reply_to_tweet_idが指定された場合は、リプライとして投稿します。
    投稿に成功した場合はツイートIDを返し、失敗した場合はNoneを返します。

    Args:
        text (str): 投稿するツイートの本文。
        in_reply_to_tweet_id (Optional[int]): リプライ先のツイートID（省略可）。

    Returns:
        Optional[int]: 投稿されたツイートのID。失敗した場合はNone。
    """

    logging.info(f":-------------------Tweet内容:-------------------")
    logging.info(text)

    # bearer_token = os.environ.get("BEARER_TOKEN")
    api_key = os.environ.get("X_API_KEY")
    api_secret = os.environ.get("X_API_SECRET")
    access_token = os.environ.get("X_ACCESS_TOKEN")
    access_token_secret = os.environ.get("X_ACCESS_TOKEN_SECRET")
    if not (api_key and api_secret and access_token and access_token_secret):
        logging.error("Twitter APIの認証情報が環境変数に設定されていません。")
        return None

    client = tweepy.Client(
        consumer_key=api_key,
        consumer_secret=api_secret,
        access_token=access_token,
        access_token_secret=access_token_secret,
        wait_on_rate_limit=True,
    )

    tweet_id = None  # 先に定義しておくことで finally でも参照可能

    try:
        if in_reply_to_tweet_id:
            response = client.create_tweet(
                text=text,
                in_reply_to_tweet_id=in_reply_to_tweet_id,
            )
        else:
            response = client.create_tweet(text=text)

        tweet_id = response.data["id"]

    except Exception as e:
        logging.error(f"Tweetに失敗 tweet: {e}")
        tweet_id = None

    finally:
        logging.info(f"Tweet ID: {tweet_id}")
        logging.info(f":-------------------Tweet内容End:-------------------")

    return tweet_id
```

2. **メイン投稿処理**
基本構成はすでにいった通りで、コメントアウトしてある部分は主に、APIの制限により今はつかていません。
XのAPIもう少しポストできるようにならないものか。。。（黙ってサブスクに入れというイーロンの声が聞こえてくる）
(現状文字の制限も厳しいため、現状本誌と号外などの簡易ポストのみにしています)

```python
def main():
    """
    RSSフィードをチェックし、指定した時間幅内に更新されたエントリを抽出してX（旧Twitter）に投稿します。
    また、関連する要約情報があればリプライとして投稿します。GitHub Actions用の出力も行います。

    コマンドライン引数:
        rss_url (str): チェック対象のRSSフィードURL。
        rss_toc_url (str): 関連情報取得用のRSSフィードURL。
        minutes (int): 何分前からの更新をチェックするか。

    環境変数:
        BEARER_TOKEN: X Bearerトークン。
        X_API_KEY: X APIキー。
        X_API_SECRET: X APIキーシークレット。
        X_ACCESS_TOKEN: Xアクセストークン。
        X_ACCESS_TOKEN_SECRET: Xアクセストークンシークレット。
        GITHUB_OUTPUT: GitHub Actions用の出力ファイルパス（オプション）。

    処理内容:
        1. RSSフィードから指定時間幅内の新規エントリを抽出。
        2. 新規エントリがあればXに投稿し、関連する要約情報があればリプライとして投稿。
        3. GitHub Actions用に更新有無とエントリ情報を出力。

    例外:
        Twitter APIの認証情報が不足している場合は投稿をスキップします。
        新規エントリがない場合は警告を出力します。
    """
    logging.basicConfig(level=logging.INFO)

    rss_url = sys.argv[1]
    rss_toc_url = sys.argv[2]
    minutes = int(sys.argv[3])
    diff_time = datetime.now(timezone.utc) - timedelta(minutes=minutes)

    logging.info(f"RSS URL: {rss_url}")
    logging.info(f"RSS_toc URL: {rss_toc_url}")
    logging.info(f"チェック時間幅: {minutes}分前 = {diff_time.isoformat()}以降")

    feed = feedparser.parse(rss_url)
    feed_toc = feedparser.parse(rss_toc_url)
    updated_entries = []
    updated_toc_entries = []

    # RSSフィードのエントリをチェックして、更新必要分だけにフィルタリングする
    for entry in feed.entries:
        if hasattr(entry, "published_parsed"):
            pub_dt = datetime(*entry.published_parsed[:6], tzinfo=timezone.utc)
        else:
            continue  # Skip items without pubDate

        if pub_dt >= diff_time:
            updated_entries.append({
                "title": entry.get("title", ""),
                "link": entry.get("link", ""),
                "summary": entry.get("summary", ""),
                "pubDate": pub_dt.strftime("%Y-%m-%d %H:%M:%S, GMT"),
            })

    updated = bool(updated_entries)

    # RSS_TOC(詳細版)もフィードのエントリをチェックして、更新必要分だけにフィルタリングする
    for entry in feed_toc.entries:
        if hasattr(entry, "published_parsed"):
            pub_dt = datetime(*entry.published_parsed[:6], tzinfo=timezone.utc)
        else:
            continue  # Skip items without pubDate

        if pub_dt >= diff_time:
            updated_toc_entries.append({
                "title": entry.get("title", ""),
                "link": entry.get("link", ""),
                "summary": entry.get("summary", ""),
                "pubDate": pub_dt.strftime("%Y-%m-%d %H:%M:%S, GMT"),
                "categories": [tag["term"] for tag in entry.get("tags", [])],
            })
    logging.info(f"更新されたRSSフィードのエントリ数: {len(updated_entries)}")
    logging.info(f"更新されたRSS_TOCのエントリ数: {len(updated_toc_entries)}")

    # --- X (Twitter) posting ---
    base_tags = ["#官報", "#官報通知"]
    required_env = ["X_API_KEY", "X_API_SECRET", "X_ACCESS_TOKEN", "X_ACCESS_TOKEN_SECRET"]
    if all(os.environ.get(env_key) for env_key in required_env):
        if updated:
            for entry in updated_entries:
                # ツイート内容を作成
                # extra = "各項目のリンクなどは以下リプライをご覧ください.."
                extra = f"以下RSSビューワーwebで項目ごとで見ることも可能です。。。\n{RSS_VIEWER_URL}"
                tweet_text = f"📚{entry['title']}\n{entry['link']}\n\n{' '.join(base_tags)}\n\n{extra}"
                tweet_id = post_to_x(tweet_text)
                if tweet_id is None:
                    continue

                # # feed_tocから関連情報を探してリプライ(with gemini)
                # batch_text = ""
                # serch_entries = [e for e in updated_toc_entries if entry["title"] in e.get("summary", "")]
                # for toc_entry in serch_entries:
                #     summary = toc_entry.get("summary", "")
                #     categories = toc_entry.get("categories", [])

                #     if entry["title"] not in summary:
                #         logging.info(f"{entry['title']} not in {summary}")
                #         continue

                #     reply_title = toc_entry.get("title", "")
                #     reply_link = toc_entry.get("link", "")

                #     if categories:
                #         categories_tags = " ".join([f"#{cat}" for cat in categories])
                #         entry_text = f"✐{reply_title}\n{reply_link}\n{categories_tags}\n\n"
                #     else:
                #         entry_text = f"✐{reply_title}\n{reply_link}\n\n"

                #     batch_text += entry_text

                # print("batch_text", batch_text)

                # template = "簡潔なまとめ\n\nタグ"
                # base_prompt = f"以下のものは、官報のリンクとタグをまとめたものです。\nこれをツイートに治るように159文字以内で、tweet用に簡潔にまとめてください。\nまた、{template}のテンプレートに必ず沿った形で出力してください。"
                # ping_prompt = f"{base_prompt}\n{tweet_text}\n{batch_text}"

                # gemini_res = ping_to_gemini(ping_prompt)

                # # feed_tocから関連情報を探してリプライ
                # batch_text = ""
                # serch_entries = [e for e in updated_toc_entries if entry["title"] in e.get("summary", "")]
                # for toc_entry in serch_entries:
                #     summary = toc_entry.get("summary", "")
                #     categories = toc_entry.get("categories", [])

                #     if entry["title"] not in summary:
                #         logging.info(f"{entry['title']} not in {summary}")
                #         continue

                #     reply_title = toc_entry.get("title", "")
                #     reply_link = toc_entry.get("link", "")

                #     if categories:
                #         categories_tags = " ".join([f"#{cat}" for cat in categories])
                #         entry_text = f"✐{reply_title}\n{reply_link}\n{categories_tags}\n\n"
                #     else:
                #         entry_text = f"✐{reply_title}\n{reply_link}\n\n"

                #     # このentryを追加したら文字数制限を超えるか？
                #     if count_tweet_length(batch_text + entry_text) > MAX_TWEET_LENGTH:
                #         post_to_x(batch_text.strip(), in_reply_to_tweet_id=tweet_id)
                #         time.sleep(1)
                #         batch_text = entry_text
                #     else:
                #         batch_text += entry_text

                # # 最後に残っていたら投稿
                # if batch_text:
                #     post_to_x(batch_text.strip(), in_reply_to_tweet_id=tweet_id)

                # リプライの間隔を空ける
                time.sleep(2)
        else:
            logging.warning("RSSフィードのアップデートが見つかりません.")
    else:
        logging.warning("Twwitter APIの認証情報が不足しています。投稿をスキップします。")

    # 結果の出力
    if "GITHUB_OUTPUT" in os.environ:
        output_path = os.environ["GITHUB_OUTPUT"]
        with open(output_path, "a") as fh:
            print(f"updated={'true' if updated else 'false'}", file=fh)
            print(f"entries={json.dumps(updated_entries, ensure_ascii=False)}", file=fh)
    else:
        result = {"updated": updated, "entries": updated_entries}
        print(json.dumps(result, ensure_ascii=False, indent=2))


```

3. **ツイート文字数制御**
オミットしてますが、文字数を159文字（日本語だと）に収めたかったので、今関数もclaudeに作ってもらいました。
```python
def count_tweet_length(text: str) -> int:
    """Twitter仕様に準拠した文字数カウント（URL=23文字）"""
    url_regex = re.compile(r"https?://\S+")
    adjusted_text = text
    for url_match in url_regex.finditer(text):
        url = url_match.group(0)
        adjusted_text = adjusted_text.replace(url, "X" * TWEET_URL_LENGTH, 1)
    return len(adjusted_text)
```


#### 投稿例

実際の投稿では以下のような形で[ポスト](https://x.com/dailykanpo/status/1956138606331891785?s=61)されます：
![ポスト例](/images/githubaction-python-4e1d593f4286b1/b.png)

## 💬 補足
- 本tweetは非公式のものであり、正確性を保証するものではありません。
- 利用に関して問題があれば[Issue](https://github.com/testkun08080/kanpo-tweet/issues)からご連絡ください。
- バグ報告や機能リクエスト、プルリクエストは大歓迎です。

# 参考リンク
- 官報インターネット版: https://kanpou.npb.go.jp/
- GitHub Actions: https://docs.github.com/actions
- RSS仕様: https://www.rssboard.org/rss-specification
- feedparser: https://pythonhosted.org/feedparser/
- tweepy: https://docs.tweepy.org/
- X (Twitter) API v2: https://developer.twitter.com/en/docs/twitter-api