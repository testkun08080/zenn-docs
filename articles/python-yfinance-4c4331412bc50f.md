---
title: "yfinanceで日本株3700社を分析！「わが投資術」株式スクリーニングアプリを作った話"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "yfinance", "react", "docker", "githubactions"]
published: false
---

## はじめに/作ったわけ

海外で働き始めて、日本の良さが身に染みた。（海外に住んだ人なら、8割そうなんじゃないかなぁと）
👇
日本企業のもの結構外で見つけるなぁ。
👇
[わが投資術](https://amzn.to/3IEVRkq)という本を昨年？見つけて、参考に。
👇
お金貯めてても仕方ないし、良い企業探して実践してみよう。
👇
せっかくだから、アプリ作って公開してみよう

そんなノリから、**日本株全銘柄を自動収集・分析できるWebアプリ**を開発しました。

この記事では、開発の経緯と技術的な実装について紹介します。

注意点として、yfinanceのデータの二時配布は禁止されています。
詳しくは、yfinanceの利用規約をご覧ください。

## 作ったもの

### 📊 [waga-toushijutsu](https://github.com/testkun08080/waga-toushijutsu)

![システム概要図](/images/python-yfinance-4c4331412bc50f/system-overview.png)

**主な機能:**

- 📈 JPX公式データから約3,795銘柄を自動取得
- 🔍 財務指標による高速スクリーニング
- 📊 PBR、ROE、自己資本比率などの指標可視化
- ⚙️ GitHub Actionsによる完全自動データ収集
- 🐳 Docker環境での簡単デプロイ


### 「わが投資術」との出会い

[わが投資術](https://amzn.to/3IEVRkq)では、**シンプルな指標**で割安株を見つける手法が紹介されています：

- **PBR（株価純資産倍率）**: 1倍以下が割安の目安
- **ROE（自己資本利益率）**: 経営効率の指標
- **自己資本比率**: 財務健全性の指標
- **ネットキャッシュ**: 現金から負債を引いた実質的な資産

これらの指標を**自動で取得・分析**できれば、個人投資家でも効率的に銘柄選定できると考えました。

## 技術スタック

### アーキテクチャ

```text
┌─────────────────────────────────────────────────┐
│         GitHub Actions (データ収集)              │
│  JPX公式 → yfinance API → CSV生成 → Combine     │
└─────────────────┬───────────────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────────────┐
│              Docker Compose                      │
│  ┌──────────────┐      ┌──────────────────────┐ │
│  │ Python       │      │ React + nginx        │ │
│  │ (データ処理) │ ───→ │ (フロントエンド)      │ │
│  └──────────────┘      └──────────────────────┘ │
└─────────────────────────────────────────────────┘
```

### バックエンド（Python）

| 技術 | 用途 |
|------|------|
| **Python 3.11+** | データ処理 |
| **yfinance** | Yahoo Financeからデータ取得 |
| **pandas** | データ整形・分析 |
| **Docker** | 実行環境の統一 |

### フロントエンド（React）

| 技術 | 用途 |
|------|------|
| **React 19** | UIフレームワーク |
| **TypeScript** | 型安全性 |
| **Vite** | 高速ビルドツール |
| **Tailwind CSS + DaisyUI** | スタイリング |
| **Papa Parse** | CSV解析 |

### インフラ（GitHub Actions + Docker）

| 技術 | 用途 |
|------|------|
| **GitHub Actions** | 自動データ収集（7ワークフロー） |
| **Docker Compose** | マルチコンテナ管理 |
| **nginx** | 静的ファイル配信 |

## 開発のポイント

### 1. データ収集の自動化

#### 課題: yfinance APIのレート制限

約3,700社のデータを一度に取得すると、APIのレート制限やタイムアウトが発生します。

#### 解決策: Sequential実行による分割処理

GitHub Actionsで**4段階のワークフロー**を構築し、自動連携させました。

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

**メリット:**

- ✅ タイムアウト回避（各120分以内）
- ✅ レート制限の回避
- ✅ エラー時の部分的再実行が可能
- ✅ 進捗の可視化

#### 実装コード（ワークフロー連携部分）

```yaml
# stock-fetch-sequential-1.yml
name: "📊 Sequential Stock Fetch - Part 1"
on:
  workflow_dispatch:

jobs:
  fetch-part-1:
    runs-on: ubuntu-latest
    timeout-minutes: 120
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          cd stock_list
          pip install -r requirements.txt

      - name: Fetch stock data (Part 1)
        run: |
          cd stock_list
          python sumalize.py stocks_1.json

      - name: Commit results
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add stock_list/Export/
          git commit -m "📊 Part 1 completed"
          git push

      # 次のワークフローを自動トリガー
      - name: Trigger Part 2
        uses: peter-evans/repository-dispatch@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          event-type: start-fetch-part-2
```

### 2. データ処理の効率化

#### JPX公式データの活用

```python
# get_jp_stocklist.py（抜粋）
import pandas as pd
import requests
from io import BytesIO

def fetch_jpx_stock_list():
    """JPX公式から最新の株式リストを取得"""
    url = "https://www.jpx.co.jp/markets/statistics-equities/misc/tvdivq0000001vg2-att/data_j.xls"

    response = requests.get(url)
    df = pd.read_excel(BytesIO(response.content))

    # 必要なカラムのみ抽出
    stocks = df[['コード', '銘柄名', '市場・商品区分', '33業種区分']].to_dict('records')

    # JSONで保存
    with open('stocks_all.json', 'w', encoding='utf-8') as f:
        json.dump(stocks, f, ensure_ascii=False, indent=2)

    return stocks
```

#### yfinanceでの財務データ取得

```python
# sumalize.py（抜粋）
import yfinance as yf
import pandas as pd
from typing import Dict, Any

def fetch_stock_data(ticker: str) -> Dict[str, Any]:
    """個別銘柄の財務データを取得"""
    try:
        stock = yf.Ticker(f"{ticker}.T")  # 東証銘柄には.Tサフィックス
        info = stock.info

        return {
            '銘柄コード': ticker,
            '会社名': info.get('longName', 'N/A'),
            '時価総額': info.get('marketCap', None),
            'PBR': info.get('priceToBook', None),
            'ROE': info.get('returnOnEquity', None),
            '自己資本比率': info.get('debtToEquity', None),
            '営業利益率': info.get('operatingMargins', None),
            # ... その他の指標
        }
    except Exception as e:
        print(f"Error fetching {ticker}: {e}")
        return None

def process_stock_list(json_file: str):
    """株式リストを処理"""
    with open(json_file, 'r', encoding='utf-8') as f:
        stocks = json.load(f)

    results = []
    for stock in stocks:
        data = fetch_stock_data(stock['コード'])
        if data:
            results.append(data)
        time.sleep(1)  # レート制限対策

    # CSVで保存
    df = pd.DataFrame(results)
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    df.to_csv(f'Export/japanese_stocks_data_{timestamp}.csv',
              index=False, encoding='utf-8-sig')
```

### 3. フロントエンドの実装

#### 動的CSVパース

```typescript
// csvParser.ts
import Papa from 'papaparse';

export interface StockData {
  [key: string]: string | number;
}

export const parseCSV = (csvText: string): StockData[] => {
  const result = Papa.parse<StockData>(csvText, {
    header: true,
    dynamicTyping: true,
    skipEmptyLines: true,
    encoding: 'UTF-8',
  });

  return result.data;
};

// 日本語金融データのフォーマット
export const formatValue = (value: any, columnName: string): string => {
  if (value === null || value === undefined) return 'N/A';

  // パーセンテージ
  if (columnName.includes('率') || columnName.includes('ROE')) {
    return `${(value * 100).toFixed(2)}%`;
  }

  // 金額（億円単位）
  if (columnName.includes('時価総額') || columnName.includes('売上高')) {
    return `${(value / 100000000).toFixed(2)}億円`;
  }

  // 倍率
  if (columnName.includes('PBR') || columnName.includes('PER')) {
    return `${value.toFixed(2)}倍`;
  }

  return value.toString();
};
```

#### リアルタイム検索・フィルタリング

```typescript
// useFilters.ts
import { useMemo, useState } from 'react';
import { StockData } from '../types/stock';

export const useFilters = (data: StockData[]) => {
  const [searchTerm, setSearchTerm] = useState('');
  const [pbrMax, setPbrMax] = useState<number | null>(null);
  const [roeMin, setRoeMin] = useState<number | null>(null);

  const filteredData = useMemo(() => {
    return data.filter(stock => {
      // 検索フィルタ
      const matchesSearch = searchTerm === '' ||
        Object.values(stock).some(value =>
          String(value).toLowerCase().includes(searchTerm.toLowerCase())
        );

      // PBRフィルタ
      const matchesPBR = pbrMax === null ||
        (stock.PBR && Number(stock.PBR) <= pbrMax);

      // ROEフィルタ
      const matchesROE = roeMin === null ||
        (stock.ROE && Number(stock.ROE) >= roeMin);

      return matchesSearch && matchesPBR && matchesROE;
    });
  }, [data, searchTerm, pbrMax, roeMin]);

  return {
    filteredData,
    searchTerm,
    setSearchTerm,
    pbrMax,
    setPbrMax,
    roeMin,
    setRoeMin,
  };
};
```

### 4. Docker環境の構築

#### docker-compose.yml

```yaml
version: '3.8'

services:
  # データ収集サービス
  python-service:
    build:
      context: ./stock_list
      dockerfile: ../Dockerfile.fetch
    environment:
      - STOCK_FILE=stocks_1.json
    volumes:
      - stock-data:/app/Export
    command: >
      sh -c "
        python get_jp_stocklist.py &&
        python split_stocks.py --input stocks_all.json --size 1000 &&
        python sumalize.py ${STOCK_FILE} &&
        python combine_latest_csv.py
      "

  # フロントエンドサービス
  frontend-service:
    build:
      context: ./stock_search
      dockerfile: ../Dockerfile.app
    ports:
      - "8080:80"
    volumes:
      - stock-data:/usr/share/nginx/html/csv:ro
    depends_on:
      - python-service
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  stock-data:
    driver: local

networks:
  stock-network:
    driver: bridge
```

## 使い方

### クイックスタート（Docker）

```bash
# リポジトリをクローン
git clone https://github.com/testkun08080/waga-toushijutsu.git
cd waga-toushijutsu

# Docker Composeで起動
docker-compose up --build

# ブラウザでアクセス
open http://localhost:8080
```

**初回起動時の注意:**
データ収集に約4時間かかります（約3,700社）。

### ローカル環境での実行

```bash
# データ収集
cd stock_list
pip install -r requirements.txt
python sumalize.py stocks_sample.json  # テスト用（数十社）

# フロントエンド
cd stock_search
npm install
npm run build
npm run preview
```

### GitHub Actionsでの自動収集

1. リポジトリを**プライベート**でフォーク
2. Actions → "Sequential Stock Fetch - Part 1" を実行
3. 約6〜8時間後に全データ収集完了
4. `stock_list/Export/` に結合済みCSVが生成される

## ハマったポイント

### 1. yfinance APIのレート制限

**問題:**
連続でAPIリクエストを送ると429エラー（Too Many Requests）が発生。

**解決策:**

```python
import time
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
def fetch_stock_data(ticker: str):
    stock = yf.Ticker(f"{ticker}.T")
    time.sleep(1)  # 1秒待機
    return stock.info
```

### 2. GitHub Actionsのタイムアウト

**問題:**
3,700社を一度に処理すると6時間制限を超える。

**解決策:**
ワークフローを4分割し、順次実行する仕組みを構築。

### 3. 日本語データの文字化け

**問題:**
CSVをExcelで開くと文字化けする。

**解決策:**

```python
# UTF-8 BOMで保存
df.to_csv('output.csv', encoding='utf-8-sig', index=False)
```

## 今後の展望

### 機能追加予定

- [ ] **スクリーニング条件のプリセット機能**
  - 「わが投資術」推奨の条件を1クリックで適用
- [ ] **チャート表示機能**
  - 株価推移のビジュアライゼーション
- [ ] **ポートフォリオ管理機能**
  - 保有銘柄の一覧管理と損益計算
- [ ] **アラート機能**
  - 条件に合致した銘柄の通知

### 技術的改善

- [ ] **データ更新の効率化**
  - 差分更新による処理時間短縮
- [ ] **キャッシュ機能**
  - ブラウザキャッシュによる表示高速化
- [ ] **モバイル対応強化**
  - レスポンシブデザインの改善

## まとめ

この記事では、yfinanceとReactを使った日本株スクリーニングアプリの開発について紹介しました。

**ポイント:**

- ✅ GitHub Actionsで完全自動化
- ✅ Sequential実行でAPI制限を回避
- ✅ Docker環境で誰でも簡単に実行可能
- ✅ 「わが投資術」の考え方を実装

個人投資家が**無料で**銘柄分析できる環境を作ることができました。

### リポジトリ

<https://github.com/testkun08080/waga-toushijutsu>

興味のある方はぜひスターやフォークをお願いします！

## 使用技術・ライブラリ

このプロジェクトで使用している主要な技術とライブラリを紹介します。

### バックエンド（Python）

| ライブラリ | バージョン | 用途 |
|-----------|-----------|------|
| **yfinance** | 0.2.x | Yahoo Finance APIからの財務データ取得 |
| **pandas** | 2.x | データ整形・分析・CSV操作 |
| **requests** | 2.x | HTTP通信（JPXデータ取得） |
| **openpyxl** | 3.x | Excelファイル（.xls）の読み込み |
| **tenacity** | 8.x | リトライ処理・エラーハンドリング |

### フロントエンド（React）

| ライブラリ | バージョン | 用途 |
|-----------|-----------|------|
| **React** | 19.x | UIフレームワーク |
| **TypeScript** | 5.x | 型安全性の確保 |
| **Vite** | 6.x | 高速ビルドツール・開発サーバー |
| **Papa Parse** | 5.x | CSVパース（日本語対応） |
| **Tailwind CSS** | 3.x | ユーティリティファーストCSS |
| **DaisyUI** | 4.x | Tailwind CSSコンポーネント |

### インフラ・DevOps

| 技術 | 用途 |
|------|------|
| **Docker** | コンテナ化・環境統一 |
| **Docker Compose** | マルチコンテナオーケストレーション |
| **nginx** | 静的ファイル配信・プロキシサーバー |
| **GitHub Actions** | CI/CD・自動データ収集 |
| **uv** | Python高速パッケージマネージャー |

### その他

- **Git** - バージョン管理
- **JPX公式データ** - 株式リスト取得元
- **Yahoo Finance API** - 財務データ取得元（yfinance経由）

## 重要な注意事項

### ⚠️ データの取り扱いについて

このプロジェクトは**個人利用・研究・教育目的**でのみ使用してください。

#### データ二次配布の禁止

**Yahoo Financeから取得したデータは二次配布が禁止されています。**

- ❌ 取得したCSVファイルをGitHubで公開しない
- ❌ 取得したデータを他のユーザーと共有しない
- ❌ 商用目的での利用は禁止
- ✅ 個人の投資研究・学習目的のみ使用可
- ✅ プライベートリポジトリでの使用を推奨

#### 推奨される使用方法

1. **プライベートリポジトリでフォーク**
   - パブリックリポジトリでの使用は避ける
   - データファイルは必ず`.gitignore`に追加

2. **ローカル環境での実行**
   - データ収集は各自の環境で実行
   - CSVファイルはローカル保存のみ

3. **GitHub Actionsの利用**
   - プライベートリポジトリでワークフロー実行
   - 生成されたデータファイルはコミットしない設定を推奨

#### Yahoo Finance利用規約の遵守

- データは**Yahoo! Japanの利用規約**に従って使用してください
- APIのレート制限を守り、過度なリクエストは避けてください
- 取得したデータの正確性は保証されません

### 本プロジェクトのライセンス

- **コード**: MIT License（非商用前提）
- **データ**: Yahoo! Japan利用規約に従うこと
- **yfinance**: Apache License 2.0

詳細は[LICENSE](https://github.com/testkun08080/waga-toushijutsu/blob/main/LICENSE)をご確認ください。

## 参考リンク

- [yfinance GitHub Repository](https://github.com/ranaroussi/yfinance)
- [Yahoo Finance](https://finance.yahoo.com/)
- [日本取引所グループ（JPX）](https://www.jpx.co.jp/)
- [わが投資術（Amazon）](https://amzn.to/3IEVRkq)

---

**免責事項:**
本記事およびアプリケーションは教育・研究目的で作成されたものです。投資判断は自己責任でお願いします。
