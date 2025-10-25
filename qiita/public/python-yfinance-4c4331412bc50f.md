---
title: 日本株3700社以上を分析。yfinance x「わが投資術」株式スクリーニングアプリを作った話（バイブコーディング）
published_at: '2025-10-27 11:30'
private: false
tags:
  - python
  - yfinance
  - react
  - docker
  - githubactions
updated_at: '2025-10-25T06:22:17.202Z'
id: null
organization_url_name: null
slide: false
---

# はじめに/作ったわけ

:::note
**バイブコーディングで作ったアプリの記事になります**
:::

いきなりですが。
海外旅行したり働き始めたりすると、日本の良さが身に染みたと感じた人は多いんじゃないでしょうか？
なんかとりあえず外で働いてみたいと思っていましたが、今はいつ戻るかと考える日々です。（とにかく温泉に入りたい）

また色々と各国を回る中で、日本企業ってアジア圏や他の国にもかなり進出してるんだなぁと実感しました。（そりゃそう）

そんなこんなで日本株に興味を持ち
昨年に[わが投資術](https://amzn.to/3IEVRkq)を購入して実践し始めました。（まだ初めて一年目なので成績はわかりません。。。が、マイナスは無し）

自分でバフェットコードや `Claude mcp-yfinance` などを利用しながらスクリーニングしてみましたが、毎回決算が出るたびに手動とチャット相手にあるのも何かなぁ。と思いまして。

じゃあ自動収集とスクリーニング用のアプリ作ってみよう(vibe coding)

そんなノリから、**日本株全銘柄を自動収集・簡易スクリーニングできる Web アプリ**を開発しました。
※現状、取得データの欠損などが見られるとことがありますが、暖かく見守っていただけると幸いです。

この記事では、粗方の構成と、実際にローカルでこれを試す方法を紹介します。
細かいコードなどはそこまで期待しないでください 😅

## 作ったもの

### 📊 [yfinance-jp-screener](https://github.com/testkun08080/yfinance-jp-screener)

<!-- ![サンプル](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/python-yfinance-4c4331412bc50f/img_sample.png) -->

![サンプル2](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/python-yfinance-4c4331412bc50f/img_sample2.png)
_CSV ファイルの読み込みと検索部分_
![検索結果](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/python-yfinance-4c4331412bc50f/search_result.png)
_検索結果(企業名はここでは伏せておきます)_

**主な機能:**

- 📈 JPX 公式データから約 3,795 銘柄を自動取得
- 🔍 財務指標による高速スクリーニング
- 📊 PBR、ROE、自己資本比率などの指標可視化
- ⚙️ GitHub Actions による自動データ収集
- 🐳 Docker 簡単デプロイ

# 「わが投資術」との出会い

[わが投資術](https://amzn.to/3IEVRkq)では、**シンプルな指標**で割安株を見つける手法が紹介されています：

- **時価総額**: 500 億以下
- **PBR**: 1 倍以下
- **PER**: 10 倍以下
- **ネットキャッシュ**: （流動資産＋投資有価証券 ×70％）－負債
- **ネットキャッシュ比率**　ネットキャッシュ/時価総額

これらの指標を**自動で取得・分析**できれば、ひとまず「わが投資術」に粗方沿ったスクリーニングができると思います。
なので、これらの基本的なものに加えて、以下のものをフィルタリング出来るようにしていきます。

### 実装済みフィルタリング項目

#### 📋 基本フィルター

![search1](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/python-yfinance-4c4331412bc50f/img_search1.png)

- **会社名検索** - テキスト検索（部分一致）
- **銘柄コード検索** - コード検索
- **時価総額** -
- **業種** - 複数選択可能（チェックボックス）
- **優先市場** - プライム/スタンダード/グロース（複数選択）
- **都道府県** - 本社所在地による絞り込み（複数選択）

#### 📊 バリュエーション指標

![search2](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/python-yfinance-4c4331412bc50f/img_search2.png)

- **PBR（株価純資産倍率）**
- **ROE（自己資本利益率）**
- **自己資本比率**
- **PER(会予)（予想株価収益率）**

#### 💹 業績・収益性指標

![search3](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/python-yfinance-4c4331412bc50f/img_search3.png)

- **売上高**
- **営業利益**
- **営業利益率**
- **当期純利益**
- **純利益率**

#### 🏛️ バランスシート指標

![search4](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/python-yfinance-4c4331412bc50f/img_search4.png)

- **負債**
- **流動負債**
- **流動資産**
- **総負債**
- **投資有価証券**

#### 💰 キャッシュ関連指標

![search5](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/python-yfinance-4c4331412bc50f/img_search5.png)

- **現金及び現金同等物**
- **ネットキャッシュ**
- **ネットキャッシュ比率**

# 技術スタック

## アーキテクチャ

```text
┌─────────────────────────────────────────────────┐
│            データソース (JPX公式 + Yahoo Finance)  │
└────────┬───────────────────────────────┬────────┘
         │                               │
         ↓                               ↓
┌────────────────────┐        ┌─────────────────────┐
│  GitHub Actions    │        │ ローカル環境          │
│  (CI/CD自動収集)    │        │ (Docker Compose).   │
│                    │        │                     │
│  Part 1-4          │        │  Python Service     │
│  → CSV Combine     │        │  → CSV生成           │
└────────┬───────────┘        └─────────┬───────────┘
         │                               │
         ↓                               ↓
┌─────────────────────────────────────────────────┐
│            CSV ファイル (Export/)                │
└────────┬────────────────────────────────────────┘
         │
         ↓
┌─────────────────────────────────────────────────┐
│         React フロントエンド (Docker/Local)      　│
│  CSV Drag & Drop → 検索・フィルタ → 結果表示        │
└─────────────────────────────────────────────────┘
```

<details><summary>詳細</summary>


```mermaid
graph TB
    subgraph "データソース"
        JPX[JPX公式データ<br/>約3,795銘柄]
        Yahoo[Yahoo Finance API<br/>yfinance]
    end

    subgraph "方法1: GitHub Actions - 自動収集パイプライン"
        StockList[Stock List Update<br/>株式リスト取得]
        Part1[Sequential Part 1<br/>stocks_1.json<br/>1-1000社]
        Part2[Sequential Part 2<br/>stocks_2.json<br/>1001-2000社]
        Part3[Sequential Part 3<br/>stocks_3.json<br/>2001-3000社]
        Part4[Sequential Part 4<br/>stocks_4.json<br/>3001-3795社]
        Combine[CSV Combine<br/>データ結合]
        CSV_GA[CSV出力<br/>Export/]
    end

    subgraph "方法2: ローカル Docker Compose"
        subgraph "Python Service"
            DataProcess[データ処理<br/>sumalize.py]
            CSV_Local[CSV生成<br/>Export/]
        end
    end

    subgraph "エクスポート済みCSV"
        CSVFiles[(CSV ファイル<br/>Export/<br/>YYYYMMDD_combined.csv)]
    end

    subgraph "完成アプリケーション (Docker/Local)"
        subgraph "React Webアプリ"
            React[React 19 + TypeScript<br/>Vite]
            Nginx[nginx<br/>静的配信]
        end

        subgraph "ブラウザ UI"
            UI[Webアプリ起動<br/>localhost:8080(PORT設定依存)]
            Upload[CSV DnD<br/>アップロード機能]
            Table[データテーブル<br/>検索・フィルタ]
            Analysis[財務分析<br/>PBR/ROE/自己資本比率]
        end
    end

    %% GitHub Actions フロー
    JPX -->|Excel→JSON変換| StockList
    StockList -->|自動トリガー| Part1
    Part1 -->|自動トリガー| Part2
    Part2 -->|自動トリガー| Part3
    Part3 -->|自動トリガー| Part4
    Part4 -->|自動トリガー| Combine

    Yahoo -->|API取得| Part1
    Yahoo -->|API取得| Part2
    Yahoo -->|API取得| Part3
    Yahoo -->|API取得| Part4

    Combine -->|CSV出力| CSV_GA
    CSV_GA -->|保存| CSVFiles

    %% ローカル Docker フロー
    JPX -->|JSON取得| DataProcess
    Yahoo -->|API取得| DataProcess
    DataProcess -->|CSV生成| CSV_Local
    CSV_Local -->|保存| CSVFiles

    %% アプリケーション起動フロー
    React -->|ビルド| Nginx
    Nginx -->|HTTP配信| UI

    %% ユーザー操作フロー (CSV → アプリ)
    CSVFiles -.->|ユーザーが手動で<br/>Drag & Drop| Upload
    Upload -->|CSVパース| Table
    Table --> Analysis

    style JPX fill:#e1f5ff
    style Yahoo fill:#e1f5ff
    style StockList fill:#fff4e6
    style Part1 fill:#f3e5f5
    style Part2 fill:#f3e5f5
    style Part3 fill:#f3e5f5
    style Part4 fill:#f3e5f5
    style Combine fill:#fff4e6
    style CSV_GA fill:#e8f5e9
    style DataProcess fill:#e8f5e9
    style CSV_Local fill:#e8f5e9
    style CSVFiles fill:#fce4ec
    style Upload fill:#ffe0b2
    style React fill:#e3f2fd
    style Nginx fill:#e3f2fd
    style UI fill:#f1f8e9
    style Table fill:#fff9c4
    style Analysis fill:#fff9c4
```

</details>

## データ収集（Python）

- **Python 3.11+**
- **yfinance**

## フロントエンド（React）

- **React 19**
- **TypeScript**
- **Vite**
- **Tailwind CSS + DaisyUI**

## インフラ

- **GitHub Actions**
- **Docker Compose**

# 開発のポイント

## 1. データ収集の自動化

### 課題: yfinance API のレート制限

約 3,795 社のデータを Github Actions で一度に取得すると、API のレート制限やタイムアウトが発生します。
（管理上も分けたかったという意図もあります。）

### 分割処理

GitHub Actions で**4 段階のワークフロー**を構築し、自動連携させました。

```yaml
# Part 1 → Part 2 → Part 3 → Part 4 → CSV結合
Sequential Stock Fetch - Part 1 (stocks_1.json: 1,000社)
  ↓ 自動トリガー
Sequential Stock Fetch - Part 2 (stocks_2.json: 1,000社)
  ↓ 自動トリガー
Sequential Stock Fetch - Part 3 (stocks_3.json: 1,000社)
  ↓ 自動トリガー
Sequential Stock Fetch - Part 4 (stocks_4.json: 795社)
  ↓ 自動トリガー
CSV Combine & Export (全データ結合)
```

### 実装コード（ワークフロー連携部分）

```yaml
name: 📊 Sequential Stock Fetch - Part 1

permissions:
  contents: write
  actions: write

on:
  workflow_dispatch:
    inputs:
      reason:
        description: "開始理由（オプション）"
        required: false
        default: "Sequential stock data collection - Part 1"
        type: string

jobs:
  fetch-stocks-1:
    runs-on: ubuntu-latest
    timeout-minutes: 120
    permissions:
      contents: write

    outputs:
      success: ${{ steps.process.outcome == 'success' }}

    steps:
      - name: 🔄 Checkout repository
        uses: actions/checkout@v4

      - name: 🐍 Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"
          cache: "pip"

      - name: 📦 Install Python dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r stock_list/requirements.txt

      - name: 📋 Show process info
        run: |
          echo "🚀 Sequential Stock Fetch - Part 1/4"
          echo "Processing file: stocks_1.json"
          echo "Reason: ${{ github.event.inputs.reason }}"
          echo "Timestamp: $(date)"
          echo "Working directory: $(pwd)"
          ls -la stock_list/

      - name: 🏃 Process stocks_1.json
        id: process
        working-directory: ./stock_list
        run: |
          echo "🚀 Starting stock data collection for stocks_1.json..."
          echo "Timestamp: $(date)"

          python sumalize.py "stocks_1.json"

          echo "✅ Part 1 completed successfully"
          echo "📄 Generated files in Export directory:"
          ls -la Export/ 2>/dev/null || echo "No files in Export directory"

      - name: Git config and pull
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git pull origin main --rebase || true

      - name: 💾 Commit changes - Part 1
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "📊 Sequential Stock Fetch - Part 1/4 完了 ($(date +'%Y年%m月%d日 %H:%M'))"
          push_options: --force

  trigger-part-2:
    needs: fetch-stocks-1
    runs-on: ubuntu-latest
    if: needs.fetch-stocks-1.outputs.success == 'true'

    steps:
      - name: 🔄 Checkout repository
        uses: actions/checkout@v4

      - name: 🚀 Trigger Part 2
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const result = await github.rest.actions.createWorkflowDispatch({
              owner: context.repo.owner,
              repo: context.repo.repo,
              workflow_id: 'stock-fetch-sequential-2.yml',
              ref: 'main',
              inputs: {
                reason: 'Auto-triggered by Part 1 completion'
              }
            });

            console.log('✅ Part 2 triggered successfully');
            console.log('Response status:', result.status);
```

## 2. データ収集について

### JPX 公式データの取得

JPX（日本取引所グループ）の公式ウェブサイトから、
上場企業の最新株式リストをダウンロードし、JSON 形式で保存します。

```python
import requests
import pandas as pd
import xlrd
from openpyxl import Workbook
import json
import logging

# ログ設定
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    handlers=[logging.StreamHandler()],
)
logger = logging.getLogger(__name__)

# ファイルのURL
url = "https://www.jpx.co.jp/markets/statistics-equities/misc/tvdivq0000001vg2-att/data_j.xls"

# ファイルをダウンロード
response = requests.get(url)

# ダウンロードしたファイルを一時的なファイルに保存
xls_file = "tickers.xls"
with open(xls_file, "wb") as f:
    f.write(response.content)

# .xlsファイルを .xlsx に変換
xlsx_file = "converted.xlsx"
workbook_xls = xlrd.open_workbook(xls_file)
sheet_xls = workbook_xls.sheet_by_index(0)

workbook_xlsx = Workbook()
sheet_xlsx = workbook_xlsx.active

# データを .xls から .xlsx に書き込む
for row in range(sheet_xls.nrows):
    for col in range(sheet_xls.ncols):
        sheet_xlsx.cell(row=row + 1, column=col + 1).value = sheet_xls.cell_value(row, col)

# .xlsx ファイルを保存
workbook_xlsx.save(xlsx_file)

# 変換された .xlsx ファイルを読み込む
data = pd.read_excel(xlsx_file)

# OR条件を使用して条件に一致する行を抽出
condition = (
    (data["市場・商品区分"] == "プライム（内国株式）")
    | (data["市場・商品区分"] == "スタンダード（内国株式）")
    | (data["市場・商品区分"] == "グロース（内国株式）")
)

filtered_df = data[condition]

# 必要な列だけを抽出
selected_df = filtered_df[["コード", "銘柄名", "市場・商品区分", "33業種区分"]]

# DataFrame を JSON 形式（リストの辞書形式）に変換
json_list = selected_df.to_dict(orient="records")

# JSONファイルに保存
with open("stocks_all.json", "w", encoding="utf-8") as f:
    json.dump(json_list, f, ensure_ascii=False, indent=2)

logger.info("JSONファイルに保存しました: stocks_all.json")

```

### ティッカーシンボルを json へ保存し直す（分割したいため）

stocks_all.json を XXXX 社ずつのファイルに分割するスクリプト

```python
import json
import math
import argparse
import sys
import logging

# ログ設定
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    handlers=[logging.StreamHandler()],
)
logger = logging.getLogger(__name__)


def split_stocks_json(input_file="stocks_all.json", chunk_size=1000):
    """
    stocks_all.jsonを指定されたサイズのチャンクに分割

    Args:
        input_file (str): 入力JSONファイル名
        chunk_size (int): 1ファイルあたりの企業数
    """
    try:
        # 元のJSONファイルを読み込み
        with open(input_file, "r", encoding="utf-8") as f:
            stock_data = json.load(f)

        total_companies = len(stock_data)
        total_files = math.ceil(total_companies / chunk_size)

        logger.info(f"総企業数: {total_companies}社")
        logger.info(f"分割ファイル数: {total_files}ファイル")
        logger.info(f"1ファイルあたり: 最大{chunk_size}社")
        logger.info("-" * 50)

        # チャンクに分割して保存
        for i in range(total_files):
            start_idx = i * chunk_size
            end_idx = min((i + 1) * chunk_size, total_companies)
            chunk_data = stock_data[start_idx:end_idx]

            # ファイル名を生成（stocks_1.json, stocks_2.json, ...）
            output_filename = f"stocks_{i + 1}.json"

            # JSON形式で保存
            with open(output_filename, "w", encoding="utf-8") as f:
                json.dump(chunk_data, f, ensure_ascii=False, indent=2)

            logger.info(
                f"✅ {output_filename}: {len(chunk_data)}社 (#{start_idx + 1}-#{end_idx})"
            )

        logger.info("-" * 50)
        logger.info(f"分割完了: {total_files}個のファイルを作成しました")

        # 各ファイルの情報を表示
        logger.info("\n作成されたファイル:")
        for i in range(total_files):
            filename = f"stocks_{i + 1}.json"
            with open(filename, "r", encoding="utf-8") as f:
                data = json.load(f)
            logger.info(f"  {filename}: {len(data)}社")

    except FileNotFoundError:
        logger.error(f"❌ エラー: {input_file}が見つかりません")
    except json.JSONDecodeError:
        logger.error(f"❌ エラー: {input_file}の形式が正しくありません")
    except Exception as e:
        logger.error(f"❌ エラー: {e}")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="日本株リストJSONファイルを指定されたサイズのチャンクに分割します",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
使用例:
  python split_stocks.py                           # stocks_all.jsonを1000社ずつに分割
  python split_stocks.py -i stocks_all.json       # stocks_all.jsonを1000社ずつに分割
  python split_stocks.py -i data.json -s 500      # data.jsonを500社ずつに分割
  python split_stocks.py --input stocks_all.json --size 2000  # 2000社ずつに分割
        """,
    )

    parser.add_argument(
        "-i",
        "--input",
        default="stocks_all.json",
        help="入力JSONファイル名 (デフォルト: stocks_all.json)",
    )

    parser.add_argument(
        "-s",
        "--size",
        type=int,
        default=1000,
        help="1ファイルあたりの企業数 (デフォルト: 1000)",
    )

    parser.add_argument("-v", "--verbose", action="store_true", help="詳細な出力を表示")

    args = parser.parse_args()

    # バリデーション
    if args.size <= 0:
        logger.error("❌ エラー: チャンクサイズは正の整数である必要があります")
        sys.exit(1)

    logger.info("=" * 60)
    logger.info("📊 stocks_all.json分割ツール")
    logger.info("=" * 60)
    logger.info(f"入力ファイル: {args.input}")
    logger.info(f"チャンクサイズ: {args.size}社")
    if args.verbose:
        logger.info("詳細モード: ON")
    logger.info("=" * 60)

    split_stocks_json(input_file=args.input, chunk_size=args.size)

```

#### yfinance での財務データ取得

メイン処理は、シンプルに読み込んだ json 野中からティッカー絞るごとに回るだけです。

```python
def main(json_filename="stocks_sample.json"):
    """メイン処理

    Args:
        json_filename (str): 処理対象のJSONファイル名
    """
    overall_start_time = time.time()
    overall_start_datetime = datetime.now()

    logger.info("=" * 80)
    logger.info(f"日本株財務データ取得プロセス開始 - 開始時刻: {overall_start_datetime.strftime('%Y-%m-%d %H:%M:%S')}")
    logger.info(f"処理対象ファイル: {json_filename}")
    logger.info("=" * 80)

    # 指定されたJSONファイルからデータを読み込み
    try:
        with open(json_filename, "r", encoding="utf-8") as f:
            stock_list = json.load(f)
        logger.info(f"{json_filename}から{len(stock_list)}社の銘柄データを読み込みました")
    except FileNotFoundError:
        logger.error(f"❌ {json_filename}ファイルが見つかりません")
        return None
    except json.JSONDecodeError:
        logger.error(f"❌ {json_filename}ファイルの形式が正しくありません")
        return None

    logger.info("=" * 60)
    logger.info("日本株財務データ取得開始")
    logger.info("=" * 60)

    results = []

    for i, stock in enumerate(stock_list, 1):
        logger.info(f"\n[{i}/{len(stock_list)}]")
        result = get_stock_data(stock)

        if result:
            results.append(result)

        # API制限回避のため少し待機
        if i < len(stock_list):
            time.sleep(2)

    # 結果をDataFrameに変換
    if results:
        df = pd.DataFrame(results)

        # 列の順序を指定
        columns_order = [
            "会社名",
            "銘柄コード",
            "業種",
            "優先市場",
            "決算月",
            "都道府県",
            "時価総額",
            "PBR",
            "売上高",
            "営業利益",
            "営業利益率",
            "当期純利益",
            "純利益率",
            "ROE",
            "自己資本比率",
            "PER(会予)",
            "負債",
            "流動負債",
            "流動資産",
            "総負債",
            "現金及び現金同等物",
            "投資有価証券",
            "ネットキャッシュ",
            "ネットキャッシュ比率",
        ]

        df = df.reindex(columns=columns_order)

        overall_end_time = time.time()
        overall_end_datetime = datetime.now()
        overall_duration = overall_end_time - overall_start_time

        # 結果を表示
        logger.info("\n" + "=" * 60)
        logger.info("取得結果サマリー")
        logger.info("=" * 60)
        logger.info(f"取得成功: {len(results)}社")
        logger.info(f"取得失敗: {len(stock_list) - len(results)}社")

        # CSVファイルに保存（Export フォルダに直接保存）
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        base_name = json_filename.replace(".json", "").replace("stocks_", "")

        filename = f"Export/japanese_stocks_data_{base_name}_{timestamp}.csv"
        df.to_csv(filename, index=False, encoding="utf-8-sig")
        logger.info(f"\nデータをCSVファイルに保存しました: {filename}")

        # データの一部を表示
        logger.info("\n取得データ（最初の3列）:")
        logger.info(f"\n{df[['会社名', '銘柄コード', '時価総額', 'PBR', 'ROE']].head()}")

        # 全体の実行時間をログ出力
        logger.info("=" * 80)
        logger.info("日本株財務データ取得プロセス完了")
        logger.info(f"開始時刻: {overall_start_datetime.strftime('%Y-%m-%d %H:%M:%S')}")
        logger.info(f"終了時刻: {overall_end_datetime.strftime('%Y-%m-%d %H:%M:%S')}")
        logger.info(f"総実行時間: {format_duration(overall_duration)}")
        logger.info(
            f"処理結果: 成功 {len(results)}社 / 失敗 {len(stock_list) - len(results)}社 / 合計 {len(stock_list)}社"
        )
        logger.info(f"平均処理時間: {format_duration(overall_duration / len(stock_list))}（1社あたり）")
        logger.info(f"保存ファイル: {filename}")
        logger.info("=" * 80)

        return df
    else:
        overall_end_time = time.time()
        overall_end_datetime = datetime.now()
        overall_duration = overall_end_time - overall_start_time

        logger.error("\n❌ データが取得できませんでした")
        logger.error("=" * 80)
        logger.error("日本株財務データ取得プロセス失敗")
        logger.error(f"開始時刻: {overall_start_datetime.strftime('%Y-%m-%d %H:%M:%S')}")
        logger.error(f"終了時刻: {overall_end_datetime.strftime('%Y-%m-%d %H:%M:%S')}")
        logger.error(f"総実行時間: {format_duration(overall_duration)}")
        logger.error("すべてのデータ取得に失敗しました")
        logger.error("=" * 80)
        return None


```

個別銘柄の財務データを取得していきます。
ティッカーシンボルごとにほしい情報を日本語に直しながら、データを作って返します。

```python
def get_stock_data(stock_info):
    code = stock_info["コード"]
    ticker_symbol = format_ticker(code)

    start_time = time.time()
    start_datetime = datetime.now()

    logger.info(f"取得中: {stock_info['銘柄名']} ({ticker_symbol})")
    logger.debug(
        f"データ取得開始: {stock_info['銘柄名']} ({ticker_symbol}) - 開始時刻: {start_datetime.strftime('%Y-%m-%d %H:%M:%S')}"
    )

    try:
        # yfinanceでティッカー作成
        ticker = yf.Ticker(ticker_symbol)

        # 基本情報取得
        info = ticker.info
        if not info:
            logger.warning(f"  ⚠️ 基本情報が取得できませんでした: {ticker_symbol}")
            return None

        # 時間を置いてAPIレート制限を回避
        time.sleep(0.5)

        # 財務諸表データ取得
        try:
            financials = ticker.financials
            balance_sheet = ticker.balance_sheet
        except Exception as e:
            logger.warning(f"  ⚠️ 財務諸表取得エラー: {e}")
            financials = pd.DataFrame()
            balance_sheet = pd.DataFrame()

        # 決算月を取得（バランスシートの最新期から）
        settlement_period = None
        if not balance_sheet.empty:
            cols = balance_sheet.columns.tolist()
            if cols:
                # 最新決算期から日付部分のみ抽出（例：2025-03-31）
                latest_period = cols[0]
                if hasattr(latest_period, "strftime"):
                    # datetimeオブジェクトの場合、日付部分のみ取得
                    settlement_period = latest_period.strftime("%Y-%m-%d")
                else:
                    # 文字列の場合、時間部分を削除
                    settlement_period = str(latest_period).split(" ")[0]

        # PER(会予)のデバッグ
        forward_pe = info.get("forwardPE", None)

        # データ収集
        result = {
            "会社名": stock_info["銘柄名"] or safe_get_value(info, "longName") or safe_get_value(info, "shortName"),
            "銘柄コード": code,
            "業種": stock_info.get("33業種区分") or safe_get_value(info, "industry") or safe_get_value(info, "sector"),
            "優先市場": stock_info.get("市場・商品区分", ""),
            "決算月": settlement_period,
            # "会計基準": None,  # yfinanceでは詳細不明 - コメントアウト
            "都道府県": get_prefecture_from_zip(safe_get_value(info, "zip")) or None,
            "時価総額": safe_get_value(info, "marketCap"),
            "PBR": safe_get_value(info, "priceToBook"),
            "PER(会予)": forward_pe,
            "ROE": safe_get_value(info, "returnOnEquity"),
            "営業利益率": safe_get_value(info, "operatingMargins"),
            "純利益率": safe_get_value(info, "profitMargins"),
        }

        # 財務諸表からのデータ取得
        if not financials.empty:
            result["売上高"] = safe_get_financial_data(ticker, "financials", "Total Revenue")
            result["営業利益"] = safe_get_financial_data(ticker, "financials", "Operating Income")
            result["当期純利益"] = safe_get_financial_data(ticker, "financials", "Net Income")
        else:
            result.update({"売上高": None, "営業利益": None, "当期純利益": None})

        if not balance_sheet.empty:
            # バランスシートデータ（test.csvの項目名に基づく、フォールバック付き）
            total_liabilities = safe_get_financial_data(
                ticker,
                "balance_sheet",
                "Total Liabilities Net Minority Interest",
                fallback_items=["Total Liab"],
            )
            current_liabilities = safe_get_financial_data(
                ticker,
                "balance_sheet",
                "Current Liabilities",
                fallback_items=["Total Current Liabilities"],
            )
            current_assets = safe_get_financial_data(
                ticker,
                "balance_sheet",
                "Current Assets",
                fallback_items=["Total Current Assets"],
            )
            total_equity = safe_get_financial_data(
                ticker,
                "balance_sheet",
                "Stockholders Equity",
                fallback_items=["Total Stockholder Equity"],
            )
            total_assets = safe_get_financial_data(ticker, "balance_sheet", "Total Assets")
            total_debt = safe_get_financial_data(ticker, "balance_sheet", "Total Debt")
            cash_and_equivalents = safe_get_financial_data(
                ticker,
                "balance_sheet",
                "Cash And Cash Equivalents",
                fallback_items=["Cash Cash Equivalents And Short Term Investments"],
            )
            investments = safe_get_financial_data(
                ticker,
                "balance_sheet",
                "Available For Sale Securities",
                fallback_items=[
                    "Short Term Investments",
                    "Investmentin Financial Assets",
                ],
            )

            result.update({
                "負債": total_liabilities,
                "流動負債": current_liabilities,
                "流動資産": current_assets,
                "総負債": total_debt,
                "現金及び現金同等物": cash_and_equivalents,
                "投資有価証券": investments,
            })

            # 自己資本比率の計算
            if total_equity and total_assets:
                result["自己資本比率"] = total_equity / total_assets
            else:
                result["自己資本比率"] = None

            # ネットキャッシュの計算（流動資産 + 投資有価証券×70% - 負債）
            net_cash = calculate_net_cash(current_assets, investments, total_liabilities)
            result["ネットキャッシュ"] = net_cash

            # デバッグ用: ネットキャッシュ計算の詳細を表示
            if any(x is not None for x in [current_assets, investments, total_liabilities]):
                inv_70 = (investments * 0.7) if investments is not None else 0
                logger.debug(
                    f"  📊 ネットキャッシュ計算: {current_assets} + {inv_70:.0f} - {total_liabilities} = {net_cash}"
                )

            # ネットキャッシュ比率の計算
            if net_cash and result["時価総額"]:
                result["ネットキャッシュ比率"] = net_cash / result["時価総額"]
            else:
                result["ネットキャッシュ比率"] = None
        else:
            result.update({
                "負債": None,
                "流動負債": None,
                "流動資産": None,
                "総負債": None,
                "現金及び現金同等物": None,
                "投資有価証券": None,
                "自己資本比率": None,
                "ネットキャッシュ": None,
                "ネットキャッシュ比率": None,
            })

        end_time = time.time()
        end_datetime = datetime.now()
        duration = end_time - start_time

        logger.info(f"  ✅ 取得完了: {result['会社名']}")
        logger.debug(
            f"データ取得完了: {result['会社名']} ({ticker_symbol}) - 終了時刻: {end_datetime.strftime('%Y-%m-%d %H:%M:%S')} - 実行時間: {format_duration(duration)}"
        )
        return result

    except Exception as e:
        end_time = time.time()
        end_datetime = datetime.now()
        duration = end_time - start_time

        logger.error(f"  ❌ エラー: {ticker_symbol} - {e}")
        logger.error(
            f"データ取得エラー: {stock_info['銘柄名']} ({ticker_symbol}) - 終了時刻: {end_datetime.strftime('%Y-%m-%d %H:%M:%S')} - 実行時間: {format_duration(duration)} - エラー: {e}"
        )
        return None


```

都道府県を知りたかったため、yfinace で郵便番号が取れるので、
郵便番号から都道府県を取得するのには、

https://github.com/GitHub30/digital-address.php

こちらを使わせていただきました。（ありがとうございます！）

```python
def get_prefecture_from_zip(zip_code):
    """郵便番号から都道府県名を取得（digital-address API使用）

    Args:
        zip_code (str): 郵便番号（ハイフンあり/なし両方対応）

    Returns:
        str: 都道府県名（例: "東京都", "大阪府"）
        None: 取得失敗時またはデータなし

    Note:
        - digital-address APIを使用してリアルタイム取得
        - 郵便番号の前処理（ハイフン・空白除去）を自動実行
        - タイムアウト設定: 10秒
    """
    try:
        if not zip_code:
            return None

        # 郵便番号の前処理（ハイフンや空白を除去）
        clean_zip = str(zip_code).replace("-", "").replace("−", "").replace(" ", "").replace("　", "")

        if len(clean_zip) < 7:  # 郵便番号として短すぎる場合
            return None

        # digital-address APIにリクエスト
        url = f"https://digital-address.app/{clean_zip}"

        response = requests.get(url, timeout=10)
        response.raise_for_status()

        data = response.json()

        if data.get("addresses") and len(data["addresses"]) > 0:
            # addressesの最初の要素からpref_nameを取得
            address = data["addresses"][0]
            prefecture = address.get("pref_name")
            logger.debug(f"  🏢 都道府県: {prefecture}")
            return prefecture

        return None

    except Exception as e:
        logger.debug(f"    郵便番号変換エラー ({zip_code}): {e}")
        return None

```

## 3. フロントエンドの実装

:::note
実際のプロジェクトコードから抜粋して記載します。
CSV のドラッグ&ドロップファイルアップロード、数十項目のフィルター、などを実装しています。
:::

### 全体の統合構造

まず、アプリケーション全体の構成を把握しましょう。
メインページ([DataPage.tsx](https://github.com/testkun08080/yfinance-jp-screener/blob/main/stock_search/src/pages/DataPage.tsx))では、ファイルアップロードと CSV ビューアを統合しています：

```typescript
// pages/DataPage.tsx（主要部分の抜粋）
export const DataPage = () => {
  const [selectedFile, setSelectedFile] = useState<CSVFile | null>(null);
  const [uploadError, setUploadError] = useState<string | null>(null);

  const handleFileUpload = (file: File) => {
    setUploadError(null);

    // アップロードされたファイルをCSVFile形式に変換
    const uploadedCSVFile: CSVFile = {
      name: file.name,
      displayName: file.name,
      size: file.size,
      lastModified: new Date(file.lastModified).toISOString(),
      url: URL.createObjectURL(file), // ブラウザでファイルを読み込むためのURL生成
    };

    setSelectedFile(uploadedCSVFile);
  };

  return (
    <div className="container mx-auto px-4 py-6">
      {/* ファイルアップロード領域 */}
      <FileUpload
        onFileSelect={handleFileUpload}
        loading={false}
        error={uploadError}
      />

      {/* CSVビューア */}
      {selectedFile ? (
        <CSVViewer file={selectedFile} />
      ) : (
        <div className="card bg-base-100 shadow-sm">
          {/* プレースホルダー表示 */}
        </div>
      )}
    </div>
  );
};
```

### 1. ドラッグ&ドロップ CSV アップロード

[FileUpload.tsx](https://github.com/testkun08080/yfinance-jp-screener/blob/main/stock_search/src/components/FileUpload.tsx)では、ドラッグ&ドロップとクリック選択の両方に対応したファイルアップロード機能を実装してます。

```typescript
// components/FileUpload.tsx
import React, { useRef } from "react";
import { CSV_FILE_CONFIG } from "../constants/csv";

interface FileUploadProps {
  onFileSelect: (file: File) => void;
  loading: boolean;
  error: string | null;
}

export const FileUpload: React.FC<FileUploadProps> = ({
  onFileSelect,
  loading,
  error,
}) => {
  const fileInputRef = useRef<HTMLInputElement>(null);

  // ファイル選択（クリック経由）
  const handleFileChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const file = event.target.files?.[0];
    if (file) {
      onFileSelect(file);
    }
  };

  // ドラッグオーバー時の処理
  const handleDragOver = (event: React.DragEvent) => {
    event.preventDefault(); // デフォルトの動作を防ぐ
  };

  // ドロップ時の処理
  const handleDrop = (event: React.DragEvent) => {
    event.preventDefault();
    const file = event.dataTransfer.files[0];

    // CSVファイルのみ受け付ける
    if (file && file.type === CSV_FILE_CONFIG.mimeType) {
      onFileSelect(file);
    }
  };

  const handleClick = () => {
    fileInputRef.current?.click();
  };

  return (
    <div className="bg-white rounded-lg shadow-sm p-6 mb-6">
      <h3 className="text-lg font-semibold text-base-content mb-4">
        CSVファイル読み込み
      </h3>

      <div
        className={`
          border-2 border-dashed rounded-lg p-8 text-center cursor-pointer
          transition-colors duration-200
          ${
            loading
              ? "border-gray-300 bg-gray-50"
              : "border-primary hover:border-primary-focus hover:bg-primary/5"
          }
        `}
        onDragOver={handleDragOver}
        onDrop={handleDrop}
        onClick={handleClick}
      >
        {/* 隠しinput要素 */}
        <input
          ref={fileInputRef}
          type="file"
          accept={CSV_FILE_CONFIG.acceptAttribute}
          onChange={handleFileChange}
          className="hidden"
          disabled={loading}
        />

        {loading ? (
          // ローディング表示
          <div className="flex flex-col items-center gap-3">
            <div className="loading loading-spinner loading-lg text-primary"></div>
            <p className="text-base-content/70">CSVファイルを読み込み中...</p>
          </div>
        ) : (
          // アップロードプロンプト
          <div className="flex flex-col items-center gap-3">
            <div className="text-4xl text-primary">📁</div>
            <div>
              <p className="text-base-content font-medium mb-1">
                CSVファイルをドラッグ&ドロップ
              </p>
              <p className="text-base-content/70 text-sm">
                または
                <span className="text-primary font-medium">
                  クリックして選択
                </span>
              </p>
            </div>
            <div className="text-xs text-base-content/50">
              対応形式: CSV ({CSV_FILE_CONFIG.extension})
            </div>
          </div>
        )}
      </div>

      {/* エラー表示 */}
      {error && (
        <div className="alert alert-error mt-4">
          <svg>...</svg>
          <span>{error}</span>
        </div>
      )}
    </div>
  );
};
```

### 2. CSV パースと数値処理

[csvParser.ts](https://github.com/testkun08080/yfinance-jp-screener/blob/main/stock_search/src/utils/csvParser.ts)では、PapaParse ライブラリを使用して CSV データを解析し、数値フィールドを適切な型に変換してます。

```typescript
// utils/csvParser.ts
import Papa from "papaparse";
import type { StockData } from "../types/stock";
import { CSV_PARSER_CONFIG, CSV_NUMERIC_FIELDS } from "../constants/csv";

export const parseCSVFile = (file: File): Promise<StockData[]> => {
  return new Promise((resolve, reject) => {
    Papa.parse(file, {
      header: CSV_PARSER_CONFIG.header, // 1行目をヘッダーとして扱う
      encoding: CSV_PARSER_CONFIG.encoding, // UTF-8エンコーディング
      skipEmptyLines: CSV_PARSER_CONFIG.skipEmptyLines, // 空行をスキップ

      // ヘッダー名の前処理
      transformHeader: (header: string) => {
        return header.trim(); // 前後の空白を除去
      },

      // 各セルの値を変換
      transform: (value: string, header: string) => {
        // 数値フィールドの場合
        if (
          CSV_NUMERIC_FIELDS.includes(
            header as (typeof CSV_NUMERIC_FIELDS)[number]
          )
        ) {
          // 単位表記を除去（倍、%、円など）
          const cleanValue = value
            .replace(/,/g, "") // カンマ除去
            .replace(/倍$/, "") // 「倍」除去
            .replace(/%$/, "") // 「%」除去
            .replace(/円$/, "") // 「円」除去
            .trim();

          const numValue = parseFloat(cleanValue);
          return isNaN(numValue) ? null : numValue;
        }

        // 文字列フィールドの場合
        return value.trim() || null;
      },

      // パース完了時
      complete: (results) => {
        if (results.errors.length > 0) {
          console.error("CSV parsing errors:", results.errors);
          reject(new Error("CSVファイルの解析中にエラーが発生しました"));
          return;
        }

        try {
          const stockData = results.data as StockData[];
          // 会社名と銘柄コードが存在するデータのみフィルタリング
          resolve(stockData.filter((row) => row.会社名 && row.銘柄コード));
        } catch (error) {
          reject(new Error("データの変換中にエラーが発生しました"));
        }
      },

      // パースエラー時
      error: (error: Error) => {
        reject(
          new Error(`CSVファイルの読み込みに失敗しました: ${error.message}`)
        );
      },
    });
  });
};

// 数値のフォーマット（日本語ロケール対応）
export const formatNumber = (value: number | null, decimals = 0): string => {
  if (value === null || value === undefined) return "-";
  return value.toLocaleString("ja-JP", {
    minimumFractionDigits: decimals,
    maximumFractionDigits: decimals,
  });
};

// 通貨フォーマット（百万円単位）
export const formatCurrency = (value: number | null): string => {
  if (value === null || value === undefined) return "-";

  // 百万円単位で表示
  return formatNumber(value / 1000000, 0);
};

// パーセンテージフォーマット
export const formatPercentage = (value: number | null): string => {
  if (value === null || value === undefined) return "-";
  return `${formatNumber(value * 100, 2)}%`;
};
```

### 3. フィルタリング機能

[useFilters.ts](https://github.com/testkun08080/yfinance-jp-screener/blob/main/stock_search/src/hooks/useFilters.ts)では、フィルター条件を管理し、useMemo でフィルタリング処理してます。

```typescript
// hooks/useFilters.ts（主要部分の抜粋）
import { useState, useMemo } from "react";
import type { StockData, SearchFilters } from "../types/stock";

const initialFilters: SearchFilters = {
  companyName: "",
  stockCode: "",
  industries: [],
  market: [],
  prefecture: [],
  marketCapMin: null,
  marketCapMax: null,
  pbrMin: null,
  pbrMax: null,
  roeMin: null,
  roeMax: null,
  // ... 計23項目のフィルター
};

export const useFilters = (data: StockData[]) => {
  const [filters, setFilters] = useState<SearchFilters>(initialFilters);
  const [sortConfig, setSortConfig] = useState<SortConfig | null>(null);

  // フィルタリング処理（useMemoで最適化）
  const filteredData = useMemo(() => {
    const filtered = data.filter((stock) => {
      // 1. 会社名フィルター（部分一致、大文字小文字区別なし）
      if (
        filters.companyName &&
        !stock.会社名?.toLowerCase().includes(filters.companyName.toLowerCase())
      ) {
        return false;
      }

      // 2. 銘柄コードフィルター（部分一致）
      if (filters.stockCode) {
        const stockCode = stock.銘柄コード || stock.コード || "";
        if (!stockCode.toString().includes(filters.stockCode)) {
          return false;
        }
      }

      // 3. 業種フィルター（複数選択対応）
      if (
        filters.industries.length > 0 &&
        !filters.industries.includes(stock.業種 || "")
      ) {
        return false;
      }

      // 4. 市場フィルター（複数選択対応）
      if (
        filters.market.length > 0 &&
        !filters.market.includes(stock.優先市場 || "")
      ) {
        return false;
      }

      // 5. 都道府県フィルター（複数選択対応）
      if (
        filters.prefecture.length > 0 &&
        !filters.prefecture.includes(stock.都道府県 || "")
      ) {
        return false;
      }

      // 6. PBR範囲フィルター（null/undefined対応）
      if (
        filters.pbrMin !== null &&
        stock.PBR !== null &&
        stock.PBR !== undefined &&
        typeof stock.PBR === "number" &&
        stock.PBR < filters.pbrMin
      ) {
        return false;
      }
      if (
        filters.pbrMax !== null &&
        stock.PBR !== null &&
        stock.PBR !== undefined &&
        typeof stock.PBR === "number" &&
        stock.PBR > filters.pbrMax
      ) {
        return false;
      }

      // 7. ROE範囲フィルター（パーセント変換対応）
      if (
        filters.roeMin !== null &&
        stock.ROE !== null &&
        stock.ROE !== undefined &&
        typeof stock.ROE === "number" &&
        stock.ROE < filters.roeMin / 100 // UIは%、データは小数なので変換
      ) {
        return false;
      }
      if (
        filters.roeMax !== null &&
        stock.ROE !== null &&
        stock.ROE !== undefined &&
        typeof stock.ROE === "number" &&
        stock.ROE > filters.roeMax / 100
      ) {
        return false;
      }

      // 8. 時価総額範囲フィルター（百万円単位からの変換）
      if (
        filters.marketCapMin !== null &&
        stock.時価総額 !== null &&
        stock.時価総額 !== undefined &&
        typeof stock.時価総額 === "number" &&
        stock.時価総額 < filters.marketCapMin * 1000000 // UI入力は百万円単位
      ) {
        return false;
      }
      if (
        filters.marketCapMax !== null &&
        stock.時価総額 !== null &&
        stock.時価総額 !== undefined &&
        typeof stock.時価総額 === "number" &&
        stock.時価総額 > filters.marketCapMax * 1000000
      ) {
        return false;
      }

      // ... 他のフィルター条件（売上高、営業利益、自己資本比率など）

      return true; // すべての条件を満たす場合のみtrue
    });

    // ソート処理
    if (sortConfig) {
      filtered.sort((a, b) => {
        const aValue = a[sortConfig.key];
        const bValue = b[sortConfig.key];

        // null値の処理（nullは最後に配置）
        if (aValue === null && bValue === null) return 0;
        if (aValue === null) return 1;
        if (bValue === null) return -1;

        let result = 0;
        if (typeof aValue === "string" && typeof bValue === "string") {
          result = aValue.localeCompare(bValue, "ja-JP"); // 日本語対応ソート
        } else if (typeof aValue === "number" && typeof bValue === "number") {
          result = aValue - bValue;
        }

        return sortConfig.direction === "desc" ? -result : result;
      });
    }

    return filtered;
  }, [data, filters, sortConfig]);

  // フィルター更新関数
  const updateFilter = (
    key: keyof SearchFilters,
    value: string | number | string[] | null
  ) => {
    const newFilters = {
      ...filters,
      [key]: value,
    };
    setFilters(newFilters);
  };

  // フィルタークリア
  const clearFilters = () => {
    setFilters(initialFilters);
  };

  return {
    filters,
    filteredData,
    sortConfig,
    updateFilter,
    clearFilters,
  };
};
```

## 4. Docker 環境の構築

### Dockerfile.fetch

シンプルにローカルにある設定を使って、コンテナ内でデータ収集をこないローカルに同期させます。

```yaml
# Python Data Collection Service Dockerfile
# 株式データ収集サービス用Dockerfile

FROM python:3.11-slim

# 作業ディレクトリ設定
WORKDIR /app

# システムパッケージ更新と必要なツールをインストール
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# Pythonの依存関係をコピーしてインストール
COPY stock_list/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 株式データ処理スクリプトをコピー
COPY stock_list/*.py ./
COPY stock_list/stocks_sample.json ./

# データディレクトリを作成
RUN mkdir -p Export

# 非rootユーザーで実行
RUN useradd -m -u 1000 stockuser && \
    chown -R stockuser:stockuser /app
USER stockuser

# 環境変数のデフォルト値を設定
ENV STOCK_FILE=stocks_sample.json
ENV CHUNK_SIZE=1000

# デフォルトコマンド（株式リスト取得 → 分割 → データ収集）
# 環境変数を使用して動的に設定
CMD ["sh", "-c", "python get_jp_stocklist.py && python split_stocks.py --input stocks_all.json --size ${CHUNK_SIZE} && python sumalize.py ${STOCK_FILE} && python combine_latest_csv.py"]
```

### Dockerfile.app

至ってシンプルなものです。
ビルドしたものをローカルでサーブするようにします。

```yaml
# React Frontend Service Dockerfile
# フロントエンドビルド・本番サービス用Dockerfile

FROM node:20-alpine AS base

# 作業ディレクトリ設定
WORKDIR /app

# 依存関係のインストール（キャッシュ最適化）
FROM base AS deps
COPY stock_search/package*.json ./
RUN npm ci

# ビルドステージ
FROM base AS builder
COPY stock_search/package*.json ./
COPY --from=deps /app/node_modules ./node_modules
COPY stock_search/ .

# TypeScriptコンパイルとViteビルド
RUN npm run build --loglevel=info

# 本番環境ステージ（nginx使用）
FROM nginx:alpine AS runner

# nginxの設定ファイルをコピー
COPY --from=builder /app/nginx.conf /etc/nginx/conf.d/default.conf

# ビルド成果物のみをコピー
COPY --from=builder /app/dist /usr/share/nginx/html

# ポート公開（nginx は 80 で待受）
EXPOSE 80

# nginx起動
CMD ["nginx", "-g", "daemon off;"]

```

### docker-compose

上記 Dockerfile をまとめた compose です。

```yaml
vservices:
  # Pythonデータ収集サービス
  python-service:
    build:
      context: .
      dockerfile: Dockerfile.fetch
    container_name: stock-data-collector
    env_file:
      - .env
    volumes:
      # 株式データディレクトリをマウント
      - ./stock_list:/app:rw
      # Exportディレクトリを共有
      - ./stock_list/Export:/app/Export:rw
    environment:
      - PYTHONUNBUFFERED=1
      - STOCK_FILE=${STOCK_FILE:-stocks_sample.json}
      - CHUNK_SIZE=${CHUNK_SIZE:-1000}
    restart: "no"

  # React フロントエンドサービス
  frontend-service:
    build:
      context: .
      dockerfile: Dockerfile.app
    container_name: stock-frontend
    env_file:
      - .env
    ports:
      - "${PORT}:80"
    environment:
      - NODE_ENV=${NODE_ENV:-production}
    command: >
      sh -c "echo 'Frontend running on http://localhost:${PORT}' && nginx -g 'daemon off;'"
```

# 使い方

## クイックスタート（Docker）

### 事前インストールが必要なもの

- [Docker](https://docs.docker.com/get-started/get-docker/)

```bash
# リポジトリをクローン
git clone https://github.com/testkun08080/yfinance-jp-screener.git
cd yfinance-jp-screener

cp .env.example .env
# STOCK_FILEはデフォルトでは"stocks_sample.json"になっています。 必ず全て取得したい場合は"stocks_all.json"へ変えて下さい

# Docker起動（データ収集 → ビルド → プレビュー）

# これでpythonとフロントエンドが起動します（pythonは裏で動き続けるので、終わったらstock_list/Export以下にあるcsvを使って下さい）
docker-compose up

# # 📦 Python データ収集ビルド・実行
# docker-compose build python-service
# docker-compose run --rm python-service

# # 🌐 フロントエンドビルド・起動
# docker-compose build frontend-service
# docker-compose up frontend-service

# ブラウザでアクセス(環境変数のPORT番号によります)
open http://localhost:8000

```

**初回起動時の注意:**
データ収集に約 4 時間かかります（約 3,795 社）。

## ローカル環境での実行

### 事前インストールが必要なもの

- [UV](https://docs.astral.sh/uv/getting-started/installation/)
- [nodejs](https://nodejs.org/en/download)

#### データ取得環境のセットアップ

```bash
# 1. リポジトリをクローン
git clone https://github.com/yourusername/yfinance-jp-screener.git
cd yfinance-jp-screener/stock_list

# 2. Python環境のセットアップ（uvを使用）
uv sync

# 3. 株式リスト取得（初回のみ）
uv run get_jp_stocklist.py

# 4. データ取得を実行
uv run sumalize.py stocks_sample.json   #ダウンロードテスト用

#===約1000社ずつダウンロード(推奨)===
# uv run sumalize.py stocks_1.json
# uv run sumalize.py stocks_2.json
# uv run sumalize.py stocks_3.json
# uv run sumalize.py stocks_4.json

#===すべての銘柄を対象にしたダウロード===
# uv run sumalize.py stocks_all.json

# 6. CSV結合
uv run combine_latest_csv.py
```

#### フロントエンド環境のセットアップ

```bash
# 1. フロントエンドディレクトリへ移動
cd ../stock_search

# 2. 依存関係をインストール
npm install

# 3. 開発サーバー起動
npm run dev
# http://localhost:5173/ にアクセス

# または、本番ビルド後のプレビュー
npm run build
npm run preview
# http://localhost:4173/ にアクセス

```

---

### GitHub Actions での自動収集

:::note
**重要**: GitHub Actions を使用する前に、以下の設定が必要です：

リポジトリの **Settings** → **Actions** → **General** に移動

- ✅ **"Read and write permissions"** を選択
- ✅ **"Allow GitHub Actions to create and approve pull requests"** にチェック

この設定により、ワークフローが生成した CSV ファイルをリポジトリにコミットできます。
:::

**実行手順:**

1. リポジトリを**プライベート**でフォーク
2. Settings → Actions で上記権限を設定
3. Actions → "Stock List Update" を実行して新しいティッカーシンボルリストを取得
4. Actions → "📊 Sequential Stock Fetch - Part 1" を実行
5. 自動連鎖実行（Part 1 → Part 2 → Part 3 → Part 4 → CSV Combine）
6. 約 3〜4 時間後に全データ収集完了
7. `stock_list/Export/YYYYMMDD_combined.csv` に結合済み CSV が生成される

**ワークフロー構成:**

- `stock-list-update.yml` - JPX 株式リスト更新用
- `stock-fetch-sequential-1~4.yml` - 4 段階データ収集（各 120 分タイムアウト）
- `csv-combine-export.yml` - CSV 結合処理

# 重要な注意事項

## ⚠️ データの取り扱いについて

このプロジェクトは**個人利用・研究・教育目的**でのみ使用してください。

:::note alert
**データの取り扱いは以下を参照ください**

1. Yahoo! Finance Terms of Service
   https://legal.yahoo.com/us/en/yahoo/terms/otos/index.html

2. Yahoo! Developer API Terms of Use
   https://policies.yahoo.com/us/en/yahoo/terms/product-atos/apiforydn/index.htm

3. Yahoo! 権利関係ページ
   https://legal.yahoo.com/us/en/yahoo/permissions/requests/index.html

:::

- API のレート制限を守り、過度なリクエストは避けてください
- 取得したデータの正確性は保証されません

## 参考リンク

- [yfinance GitHub Repository](https://github.com/ranaroussi/yfinance)
- [Yahoo Finance](https://finance.yahoo.com/)
- [日本取引所グループ（JPX）](https://www.jpx.co.jp/)
- [わが投資術（Amazon）](https://amzn.to/3IEVRkq)

---

## バイブコーディングを行う前にやったこと

大体の方々が同じやり方だと思いますが、僕はこんな感じです。

- ざっくりの流れを AI(多分何でも良い)に渡す
- 質問形式ですり合わせする
- ファイルにまとめてもらって、Claude に渡す
- Claude に渡すときに、もう一度読解させて、すり合わせる
- 実行

## その他

もしこの記事が役立ったら、[コーヒ一杯ほど](https://buymeacoffee.com/testkun08080)もらえると最高です

最後までお読みいただきありがとうございます。
何かおかしいとか、もっとこうした方がいいとかあれば issue やコメントでお待ちしています。

sakana-ai が書いていた**EDINET**を使用したデータセットを作るいい感じのレポも見つけたので、これを yfinance の代わりとするのものいいのかなぁとも思います。（おそらくこれなら、データの開示については問題ない）
時間見つけたら、やってみようかなぁと思います。

興味がある方は、以下のレポをご覧ください。
https://github.com/SakanaAI/edinet2dataset

それでは 🙏

**免責事項:**
投資判断は自己責任でお願いします。
