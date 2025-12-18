---
title: "FastMCP Cloud にデプロイした Amazon Product API MCP を Claude から呼ぶまで"
emoji: "🛒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [claude, fastmcp, python, amazon, mcp]
published: false
---

## はじめに

MCP は適当に自分用にローカルで作成しては消しとばす、などを繰り返してましたが...
とうとう、fastmcp cloud に作ってみたものを乗せてみることにしました。

この記事では、Amazon Product API を使用できる mcp サーバーを FastMCPCloud 上にデプロイし、
ヘッダーでクライアントのキーなどを渡して動くまでを書きます。
クライアントからの設定の例は Claude code と Claude desktop にしていますが、ヘッダーなどが指定できればどこからでも使えるはずです。

実際のコード部分にはあまり触れませんが、やっていることはシンプルです。
Amazon Product API のラッパーを使って、mcp ようにさらにツールとして呼び出せるようにラップし。
尚且つ、キーデータなどはヘッダーを通して受け取れる様にしているだけです。

これらを FastMCPCloud（`https://amazon-product-ad-api.fastmcp.app/mcp`）に、デプロイし、
Claude Code と Claude Desktop からヘッダーで認証情報を指定して接続する方法を説明します。

:::message alert
現在デプロイしているサービスを止める予定はありませんが、悪意あるアクセスや無料枠を超えそうになれば非公開にします。
また、ヘッダーで何もしていなければ、デフォルトの値で動きます。（一応信用してますので、悪用しないで下さい。）
:::

## Claude desktop で試した結果

こんな感じで見えるはずです。

https://x.com/testkun08080/status/2001087697805566210

まず Amazon Product API だけでは、KindleUnlimited かどうかを判定する術がなさそうなので、動画では結果にしたいして Claude が頑張ってアイテムの詳細などから探そうとしていますが、結果はまぁ微妙です 😅
が、まぁざっくりやりたい事はできたので、一旦よしとします。

## FastMCP Cloud へのデプロイ手順

### 前提条件

- Amazon Product Advertising API の認証情報を持っていること（持っていなくても出来ますが、テスト用の範囲でお試しください）
- FastMCP Cloud アカウントがあること

### デプロイの流れ

#### ステップ 1: FastMCP プロジェクトの作成

まずシンプルに fastmcp を使って mcp をローカルで試します。
動くことが確認できたら MCP インスペクターを使って、テストして置くと尚良いかもしれません。

https://modelcontextprotocol.io/docs/tools/inspector

終わったら、適当にレポジトリを作成して頂いて、プッシュしておきます。

#### ステップ 2: FastMCP Cloud にデプロイ

それでは FastMCP Cloud へデプロイしていきます。
cli を使って出来るそうですが、今回はブラウザを使って行います。
※ここで作成していない人は、アカウントを作成してください。

https://fastmcp.cloud/testkun/welcome/server-setup

![レポ選択](/images/fastmcp-amazonproduct/ss-2.png)

画像のようにご自身のレポジトリを選択します。

![デプロイ設定](/images/fastmcp-amazonproduct/ss-3.png)

あとは、ローカルで試していたメインのファイルを選択するようにエントリーポイントとお好きな名前を設定して下さい。
おそらくビルドは地味に時間がかかるので、気長に待ちましょう。

## FastMCP Cloud てテストしてみる

ビルド完成したら、FastMCP Cloud 上でのインスペクターを使ってテストできます。

![FastMCPCloudInspector](/images/fastmcp-amazonproduct/ss-6.png)

チャットから、こんかなじで気軽にツールのテストも行えます

![FastMCPCloudChat](/images/fastmcp-amazonproduct/ss-7.png)

## Python からの接続とテスト

まず、Python から FastMCP Cloud へ動いているかどうかテストしてみます。

### テストスクリプト

以下のスクリプトで接続をテストできます：

:::details FastMCP Cloud サーバーのテストスクリプト

```python
from fastmcp import Client
from fastmcp.client.transports import StreamableHttpTransport
from fastmcp.client.auth import OAuth

# FastMCP Cloud サーバーの URL
MCP_URL = "https://amazon-product-ad-api.fastmcp.app/mcp"

# 認証情報（直接指定）
HEADERS = {
    "aws_access_key_id": "YOUR_ACCESS_KEY_ID",
    "aws_secret_access_key": "YOUR_SECRET_ACCESS_KEY",
    "aws_associate_tag": "YOUR_ASSOCIATE_TAG",
    "amazon_marketplace": "JP",
}

# OAuth 認証を設定
oauth_auth = OAuth(mcp_url=MCP_URL, additional_client_metadata={"token_endpoint_auth_method": "client_secret_post"})

# クライアントを作成
client = Client(transport=StreamableHttpTransport(MCP_URL, headers=HEADERS, auth=oauth_auth))


async def test_connection():
    """MCP server 接続テスト"""
    print("=" * 60)
    print("接続テスト開始...")
    print("=" * 60)
    try:
        async with client:
            await client.ping()
            print("✅ 接続成功!")
            return True
    except Exception as e:
        print(f"❌ 接続失敗: {e}")
        return False


async def main():
    """メイン処理"""
    print(f"📍 テスト URL: {MCP_URL}\n")

    # 認証情報をマスクして表示
    masked_headers = {}
    for key, value in HEADERS.items():
        if "key" in key.lower() or "secret" in key.lower():
            masked_value = value[:4] + "***" if value and len(value) > 4 else "***"
            masked_headers[key] = masked_value
        else:
            masked_headers[key] = value
    print("認証情報:", masked_headers, "\n")

    # Test connection
    result = await test_connection()
    if not result:
        print("\n❌ 接続失敗. テストを中止します.")
        return

    # 結果
    print("\n" + "=" * 60)
    print("テスト結果:", "✅ 成功" if result else "❌ 失敗")
    print("=" * 60)


if __name__ == "__main__":
    asyncio.run(main())

```

:::

## Claude Desktop からの接続（Mac）...手動コマンド設定

アコーディオン(タイトル)
:::message alert
Claude Desktop を開くたびに、認証を促されます。
ブラウザをクリックしたり、アクティブにしないと Claude が Desktop がフリーズした様になる可能性があります。
うまくいかない場合は、一度アプリを落として立ち上げるなどをして見てください。
:::

### 設定ファイルの場所

mac の場合、設定ファイルは以下にあるかと思います：

```bash
~/Library/Application Support/Claude/claude_desktop_config.json
```

※開発者モードを ON にしていない人は、してみてください。

### 設定方法

Claude Desktop から FastMCP Cloud サーバーにヘッダーで認証情報を渡すには、`mcp-remote` パッケージを使用します。

設定ファイルに以下を追加します：

```json
{
  "mcpServers": {
    "amazon-product-ad-api": {
      "command": "npx",
      "type": "streamable-http",
      "args": [
        "mcp-remote@latest",
        "https://amazon-product-ad-api.fastmcp.app/mcp",
        "--header",
        "aws_access_key_id: YOUR_ACCESS_KEY_ID",
        "--header",
        "aws_secret_access_key: YOUR_SECRET_ACCESS_KEY",
        "--header",
        "aws_associate_tag: YOUR_ASSOCIATE_TAG",
        "--header",
        "amazon_marketplace: JP"
      ]
    }
  }
}
```

**重要なポイント**:

- `YOUR_ACCESS_KEY_ID`、`YOUR_SECRET_ACCESS_KEY`、`YOUR_ASSOCIATE_TAG` を自分の認証情報に置き換えてください
- `amazon_marketplace` は日本の場合 `JP`、アメリカの場合 `US` などを指定します
- `mcp-remote@latest` により、最新の mcp-remote パッケージが自動的に使用されます

## Claude Desktop からの接続（Mac）...mcp-remote を使用

現在はまだベータ版ですが、一応これからもいけるはずです。
ただしヘッダーやメタも設定できそうにないので、デフォルトの値で動きます。（一応信用してますので、悪用しないで下さい。）
テスト的に触ってみたいぐらいの感じは、これで十分です。

https://modelcontextprotocol.io/docs/develop/connect-remote-servers

認証を促すブラウザが開くと思いますので、指示に従って fastmcpcloud で認証します。
※開発者モードを ON にしていない人は、してみてください。

## Claude cli からの接続

### コマンドで追加

ターミナルから以下のようにコマンドを打ちます。

```bash
claude mcp add --transport http amazon-product-api-test https://amazon-product-ad-api.fastmcp.app/mcpp \
  --header "aws_access_key_id: YOUR_ACCESS_KEY_ID" \
  --header "aws_secret_access_key: YOUR_SECRET_ACCESS_KEY" \
  --header "aws_associate_tag: YOUR_ASSOCIATE_TAG" \
  --header "amazon_marketplace: JP"
```

```bash
claude
/mcp amazon-product-api-test
```

などを打つと認証しますか？解かれると思うので、実行すれば以下のようが画面がブラウザで開きます。
FastMCPCloud アカウントが必要です。

![認証イメージ](/images/fastmcp-amazonproduct/ss-1.png)

## 使い方

設定が終わったら、あとは各々のインターフェイスで、適当に質問してみてください。

```text
amazon-product-api-testを使って、任天堂のゲームを調べてください。
```

など。

## 利用可能な MCP ツール

:::details このサーバーは以下の 5 つのツールを提供しています：

### 1. `search_products`

キーワードで Amazon 商品を検索します。

**パラメータ:**

- `keyword` (str, 必須): 検索キーワード（例: "ワイヤレスイヤホン", "gaming laptop"）
- `max_results` (int, オプション): 取得する最大結果数（デフォルト: 10、最大: 10/リクエスト）
- `category` (str, オプション): カテゴリフィルター（例: "Electronics", "Books"）

**戻り値:**

- `query`: 検索クエリ
- `total_results`: 検索結果の総数
- `products`: 商品リスト（各商品には ASIN、タイトル、URL、価格、評価、画像などが含まれます）

### 2. `get_product_details`

ASIN で商品の詳細情報を取得します。

**パラメータ:**

- `asin` (str, 必須): Amazon Standard Identification Number（例: "B08N5WRWNW"）

**戻り値:**

- 商品の基本情報（ASIN、タイトル、URL、価格、評価、画像など）
- `description`: 商品説明
- `specifications`: 仕様情報（辞書形式）
- `reviews_summary`: レビューサマリー
- `top_reviews`: トップレビューのリスト

### 3. `get_product_variations`

商品のバリエーション（色、サイズなど）を取得します。

**パラメータ:**

- `asin` (str, 必須): Amazon Standard Identification Number（例: "B08N5WRWNW"）
- `languages_of_preference` (list[str], オプション): 言語コードのリスト（例: ["ja_JP", "en_US"]）

**戻り値:**

- `parent_asin`: 親商品の ASIN
- `variation_count`: バリエーション数
- `variation_dimensions`: バリエーションディメンションのリスト（例: ["Color", "Size"]）
- `variations`: バリエーション商品のリスト（各バリエーションには`variation_dimension`と`variation_value`が含まれます）

### 4. `get_browse_nodes`

ブラウズノード（カテゴリ）情報を取得します。カテゴリの階層構造や子カテゴリ、親カテゴリの情報を取得できます。

**パラメータ:**

- `browse_node_ids` (list[str], 必須): ブラウズノード ID のリスト（例: ["3210981", "2127209051"]）
- `languages_of_preference` (list[str], オプション): 言語コードのリスト（例: ["ja_JP", "en_US"]）

**戻り値:**

- `browse_nodes`: ブラウズノードのリスト（各ノードには以下が含まれます）
  - `id`: ノード ID
  - `display_name`: 表示名
  - `context_free_name`: コンテキストフリー名
  - `children`: 子ノードのリスト
  - `ancestors`: 親ノードのリスト

**よく使われるブラウズノード ID（日本）:**

- 家電＆カメラ: `3210981`
- パソコン・周辺機器: `2127209051`
- ホーム&キッチン: `3828871`
- 本: `465392`

### 5. `check_api_status`

API の設定と動作を確認します。認証情報が正しく設定されているか、API 接続が正常に動作するかを確認できます。

**パラメータ:**

- なし

**戻り値:**

- `status`: ステータス（"ok", "error", "not_configured"）
- `message`: ステータスメッセージ
- `configuration`: 設定値（API キー、ID など、機密情報はマスクされます）
- `api_test_result`: API テスト呼び出しの結果

**このツールの用途:**

- 接続時の初期チェック
- 認証情報の検証
- API 接続の診断
- トラブルシューティング

:::

## トラブルシューティング用

使えない場合は MCP インスペクターを使って、テストすると分かりやすいかもしれません。
https://modelcontextprotocol.io/docs/tools/inspector

## まとめ

Fastmcpcloud へのデプロイは高速でできましたし、特に引っかかるところはありませんでした。
ローカルで動かして使うのもいいですが、チーム内や友達などに使ってもらうとかするにはやはりサーバーへ置いておくと便利かと思います。（まぁモノによるかと思いますが）

それではよき MCP ライフを。

## 参考リンク

- [FastMCP Documentation](https://github.com/jlowin/fastmcp)
- [FastMCP Cloud](https://fastmcp.cloud/)
- [MCP Protocol Specification](https://modelcontextprotocol.io/)
- [Amazon Product Advertising API](https://webservices.amazon.co.jp/paapi5/documentation/)
- [mcp-remote](https://modelcontextprotocol.io/docs/develop/connect-remote-servers)
