---
title: "初webフルスタックアプリ作って月額サービス狙ったけどやめた話"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, vite, サブスク, stripe, フルスタック]
published: false
---

# はじめに

昔会社をやめて、1 年間ぐらい旅行していた時に YouTube をやっていた時期がありました（なんでや）
その当時、商品価格を他通貨に換算して一つ一つ画像を作っていました。
その当時も、恐らく今も、色々サービスを探しても無かったので便利なアプリ作ってみようというのがきっかけです。

また一応フロントエンドとバックエンドやデータベースなどの一連の流れをこの開発で経験できると踏んだからです。

https://currency-converter-board.com/

# どんなアプリなのか？

![ss1](/images/service-web/ss1.png)

こんな感じで、シンプルな通貨換算された結果を元に画像やウィジェットを作成できるものです。

![ss2](/images/service-web/ss2.png)

![ss3](/images/service-web/ss3.png)

主な機能としては：

- Wise API を使ったリアルタイム為替レート取得（数多くの通貨対応）
- ドラッグ&ドロップで通貨の並び順をカスタマイズ
- 背景色やテキスト色などのカスタマイズ
- PNG 画像としてエクスポート
- iframe 埋め込みコード生成

## 使用した技術スタック

:::details 技術スタック

### フロントエンド

- **React 19** + **Vite 6**
- **TailwindCSS 4** + **DaisyUI 5**: スタイリング
- **@dnd-kit**: ドラッグ&ドロップ機能
- **html-to-image**: Canvas 描画と PNG エクスポート

### バックエンド

- **FastAPI** (Python)
- **PostgreSQL**: ユーザー情報や使用履歴の保存
- **Alembic**: データベースマイグレーション
- **Google OAuth 2.0** + **JWT**: 認証機能

### その他

- **Wise API**: 為替レート取得
- **Stripe**: 決済機能（実装済みだが現在は無料モデル）
- **Vercel**: フロントエンドデプロイ
- **Railway**: バックエンド・データベースデプロイ

:::

:::details アーキテクチャ図

```mermaid
graph TB
    subgraph "フロントエンド (Vercel)"
        A[React 19 + Vite 6]
        B[TailwindCSS 4 + DaisyUI 5]
        C[@dnd-kit]
        D[html-to-image]
        E[i18next]
        A --> B
        A --> C
        A --> D
        A --> E
    end

    subgraph "バックエンド (Railway)"
        F[FastAPI Python]
        G[Google OAuth 2.0 + JWT]
        H[Alembic]
        F --> G
        F --> H
    end

    subgraph "データベース (Railway)"
        I[(PostgreSQL)]
        J[Users Table]
        K[Usage History Table]
        I --> J
        I --> K
    end

    subgraph "外部API"
        L[Wise API<br/>為替レート取得]
        M[Stripe<br/>決済機能]
        N[Google OAuth<br/>認証]
    end

    A -->|API Request| F
    F -->|認証| G
    G -->|認証| N
    F -->|データ保存/取得| I
    F -->|為替レート取得| L
    F -->|決済処理| M
    D -->|PNG Export| A
    C -->|ドラッグ&ドロップ| A
```

:::

# せっかくだから月額サービス化してみたけども辞めた

作っている時は、月額 3 とか 5 ドルぐらいと無料バージョンでいいかなぁ。とか考えてましたが辞めました。

なんでか？

- 初サービスだし、まずはユーザーがどれくらい来てくれるのか実験したかった
- 画像生成はフロントエンドでやってるので、知恵を働かせれば簡単にダウンロードできる笑
- 法律周り（特定商品取引法）などについて知識が無いので少しビビった（準備するものがあったというところが大きい）
- 有料サービスと無料サービスのボーダーを何処にするべきか迷った

これらの点から有料化はやめました。
※Stripe の統合自体は実装済みなので、将来的に有料化する場合は比較的簡単に切り替えるようには残してます。

# 開発の流れ

このアプリの構成ともう少し噛み砕いた内容を備忘録的に書いておきます。

## 1. バックエンド：通貨データの取得と処理

Wise はめちゃくちゃ日頃使用してるんですが、wise って api 基本無料で提供してるんですよね。
なので今回は[こちら](https://wise.com/ja/help/articles/2958106/wise-api%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)を為替レートと通貨の種類の取得に使わせて頂きました。

バックエンドは FastAPI で実装しており、主に以下の 4 つの機能を提供しています。

まず最初に、Wise API を使って為替レートのデータを取得します。この工程では以下の処理を行います：

### 1. 通貨リストと為替レートの取得

Wise API の `/v1/rates` エンドポイントを使用して、リアルタイムの為替レートを取得します。

```python
# Wise API へのリクエスト例
url = f"{WISE_API_URL}?source={from_currency}"
headers = {
    "Authorization": f"Bearer {WISE_API_TOKEN}",
    "Content-Type": "application/json"
}
response = requests.get(url, headers=headers)
```

**主な処理の流れ：**

- API トークンを Bearer 認証ヘッダーに付与してリクエスト
- レスポンスから対応通貨のリスト（`target`）と為替レート（`rate`）を取得
- 履歴データが必要な場合は `time` パラメータ（ISO 8601 形式）を追加
- 取得したレートを使って通貨換算（`amount * rate`）
- 通貨シンボルや国旗 SVG と組み合わせて表示用データを生成

### 2. 換算レート計算とデータ整形

取得した為替レートを元に、各通貨への換算額を計算し、フロントエンド向けにフォーマットします。

```python
def generate_currency_display_data(currency_tuples, domain, user_sorted):
    """通貨表示データを生成"""
    data = []
    for code, amount, rate, time in currency_tuples:
        flag_filename = f"{code[:2].lower()}.svg"  # 国旗アイコン
        symbol = CurrencySymbols.get_symbol(code)  # 通貨シンボル (¥, $, €など)
        data.append({
            "code": code,
            "symbol": symbol,
            "amount": "{:,.3f}".format(amount).rstrip("0").rstrip("."),
            "rate": rate,
            "time": time,
            "flag_url": flag_filename,
        })
    return data
```

**ポイント：**

- 通貨コードの先頭 2 文字を小文字にして国旗 SVG のファイル名を生成（例: `JPY` → `jp.svg`）
- `currency_symbols` ライブラリで通貨シンボルを取得
- 小数点以下の無駄なゼロを削除して見やすくフォーマット
- ユーザーのドラッグ&ドロップでの並び替え順序を維持

### 3. Google OAuth 認証

Google OAuth 2.0 を使ってユーザー認証を行い、JWT トークンを発行します。

```python
def verify_google_token(token: str) -> Optional[Dict]:
    """Google ID トークンを検証してユーザー情報を取得"""
    request = requests.Request()
    idinfo = id_token.verify_oauth2_token(
        token, request, GOOGLE_CLIENT_ID
    )

    # トークンの発行元を確認
    if idinfo["iss"] not in ["accounts.google.com", "https://accounts.google.com"]:
        return None

    # ユーザー情報を抽出
    return {
        "sub": idinfo.get("sub"),        # ユーザー ID
        "email": idinfo.get("email"),    # メールアドレス
        "name": idinfo.get("name"),      # 名前
        "picture": idinfo.get("picture") # プロフィール画像
    }
```

**認証フロー：**

1. フロントエンドで Google OAuth ログインを実行
2. Google から受け取った ID トークンをバックエンドへ送信
3. `google.oauth2.id_token` で署名を検証
4. 検証が成功したらユーザー情報を抽出し、JWT アクセストークンを生成
5. フロントエンドに JWT トークンを返却（以降のリクエストで使用）

### 4. ユーザー管理とデータベース連携

PostgreSQL を使ってユーザー情報と使用履歴を管理します。

**データベーステーブル構成：**

- **users テーブル**: ユーザー ID、メール、名前、プラン（free/paid）、Stripe 情報など
- **usage_history テーブル**: ユーザーごとの日次エクスポート回数を記録

```python
def get_or_create_user(db: Session, user_id: str, email: str, name: str, picture: str) -> User:
    """ユーザーを取得、存在しなければ新規作成"""
    user = db.query(User).filter(User.id == user_id).first()

    if not user:
        user = User(
            id=user_id,
            email=email,
            name=name,
            picture=picture,
            plan="free"  # 初期プランは無料
        )
        db.add(user)
        db.commit()
    return user
```

**使用履歴の記録：**

```python
def record_export(db: Session, user_id: str) -> Tuple[bool, str, int, int]:
    """エクスポート実行を記録"""
    today = date.today()
    usage = db.query(UsageHistory).filter(
        and_(UsageHistory.user_id == user_id, UsageHistory.usage_date == today)
    ).first()

    if usage:
        usage.export_count += 1  # カウントを増やす
    else:
        usage = UsageHistory(user_id=user_id, usage_date=today, export_count=1)
        db.add(usage)

    db.commit()
    return True
```

**現在の仕様：**

- 全ての機能が無料で利用可能（無制限）
- Stripe 統合は実装済みだが、有料プランは無効化
- 使用履歴は記録しているので、将来的に有料化する際はすぐに切り替え可能

これらの処理により、シンプルながらも拡張性のあるバックエンド構成になっています。

#### Wise API からのデータ取得

- Wise API の `/v1/rates` エンドポイントにリクエストを送信
- 指定した基準通貨（例：USD、JPY など）から換算できる全ての通貨リストを取得
- リアルタイムの為替レート、または過去の特定時刻のレート（履歴データ）を取得可能
- API は無料で利用でき、数多くの通貨ペアに対応

#### 取得したデータの整形

- 為替レートを使って各通貨への換算額を計算（金額 × レート）
- 通貨コードから国旗 SVG ファイルと通貨シンボル（¥、$、€ など）を紐付け
- 小数点以下の無駄なゼロを削除して見やすくフォーマット
- フロントエンドで使いやすい JSON 形式に整形して返却

#### ユーザー認証とデータベース管理

- Google OAuth 2.0 でユーザーのログイン認証
- ユーザー情報（ID、メール、名前、プロフィール画像）を PostgreSQL に保存
- エクスポート回数などの使用履歴を記録（将来的な有料プラン切り替えに備えた設計）
- JWT トークンを発行して、以降のリクエストで認証状態を維持

これらの処理により、バックエンドは「データの取得・加工・配信」と「ユーザー管理」という 2 つの役割を果たしています。

## 2. フロントエンド：取得データの UI 表示

バックエンドから受け取ったデータを元に、ユーザーが見やすい形で表示します。

#### 通貨換算ボードの表示

- グリッド形式で複数の通貨を一覧表示（列数は可変設定可能）
- 各通貨カードには、国旗アイコン、通貨コード、シンボル、換算額を表示
- リアルタイムで換算結果を反映（入力金額や基準通貨を変更すると即座に再計算）
- レスポンシブデザインで、PC・タブレット・スマートフォンに対応

#### 入力フォームの実装

- 金額入力欄：換算したい基準金額を入力
- 通貨セレクター：基準となる通貨を選択（検索機能付きドロップダウン）
- 時刻選択機能：過去の特定時刻のレートを取得できる履歴機能
- 入力値のデバウンス処理で、無駄な API リクエストを削減

#### 状態管理とパフォーマンス最適化

- React の `useState` と `useContext` で複雑な状態を管理
- `useMemo` と `useCallback` で不要な再レンダリングを防止
- API レスポンスのキャッシュで、同じリクエストの重複を回避
- デバウンス処理で、ユーザーの入力中は API リクエストを遅延実行

### 3. カスタマイズ：見た目と並び順の調整

ユーザーが自分好みにボードをカスタマイズできる機能を提供します。

#### ドラッグ&ドロップによる並び替え

- `@dnd-kit` ライブラリを使って、通貨カードをドラッグ&ドロップで自由に並び替え
- モーダルウィンドウで、グリッド形式のドラッグ可能な通貨一覧を表示
- 新しい通貨を検索して追加、不要な通貨を削除
- 並び順の変更は即座にプレビューに反映され、リアルタイムで確認可能

#### デザイン設定のカスタマイズ

- **レイアウト設定**：列数、カード間の余白、エクスポートサイズ（幅・高さ）
- **色設定**：背景色とテキスト色をカラーピッカーで自由に変更
- **表示・非表示の切り替え**：基準通貨の表示、タイムスタンプ、カード枠線の表示
- **表示形式**：グリッド表示かテーブル表示かを選択

#### 設定の保存とプレビュー

- 設定変更はリアルタイムでプレビューに反映
- ユーザーの並び替え順序をバックエンドに送信して、データ整形時に維持
- 設定値は URL パラメータとしてエンコードされ、共有や埋め込みに活用

### 4. エクスポート：画像生成と埋め込みコード

最後に、完成したボードを画像として保存したり、Web サイトに埋め込むためのコードを生成します。

#### Canvas による PNG 画像エクスポート

- `html-to-image` ライブラリで、HTML 要素を Canvas に描画
- 指定したサイズ（幅・高さ）で高品質な PNG 画像を生成
- エクスポート前に認証チェックと使用回数の記録
- ダウンロード完了後、任意で寄付モーダルを表示（収益化の試み）

#### iframe 埋め込みコード生成

- ユーザーの設定（通貨リスト、色、レイアウトなど）を URL パラメータとしてエンコード
- 埋め込み専用の URL を生成（例：`/embed-canvas?cols=5&width=1920&...`）
- iframe タグの HTML コードを自動生成
- ワンクリックでクリップボードにコピー可能

#### エクスポート機能の制御

- Google OAuth でログインしたユーザーのみエクスポート可能
- 使用履歴をデータベースに記録（日次カウント）
- 現在は無制限に利用可能だが、将来的に有料プランへの切り替えに対応できる設計
- Stripe 統合は実装済みで、有料化する場合は環境変数の切り替えで対応可能

これらの工程を経て、ユーザーは簡単に通貨換算ボードを作成し、画像として保存したり、自分の Web サイトに埋め込んだりできるようになっています。

# つまづいた所と打開策

- **Claude で一気に書き上げて、杜撰な所が見つかってもう一回 Cursor を使って作り直しました**  
  設計をもっと細かくして Cursor に渡しただけです。最初から設計をしっかりしておけば良かったと反省。

- **Canvas の描画とグリッド処理した順番の同期が微妙**  
  ドラッグ&ドロップで並び順を変更した後、Canvas に描画する際の順番が正しく反映されない問題がありました。状態管理を見直して解決。

- **useMemo, useEffects などの扱い方をしっかり把握しておきべき**  
  パフォーマンス最適化のために React Hooks の理解が重要だと実感。無駄な再レンダリングを防ぐために useMemo や useCallback を適切に使う必要がありました。(ちなみにまだ無駄が少し残っている....)

- **React DnD がバグるので、一番時間を費やしました**  
   `@dnd-kit` を使いましたが、window の幅などをいじると react 19 ではバグったため、地味に解決方法を見つけるのに時間がかかりました。

# これから

まずはやっぱ勢いで作ってみましたが、どれだけニーズなどがあるのかも微妙ですし探り探りやって行こうと思います。
あと SEO 対策とかそもそもパフォーマンスどうなのかとか。
入力する部分とプレビューが同じスペースで出来ないのが、微妙に操作しづらいのでそこを何とかしたい。
様子見ながら、ちょくちょく手直ししていこうと思います。

# まとめ

フルスタックアプリを一から作る経験ができたのは良かったです。
特にバックエンドとフロントエンドの連携、認証周り、サブスク API の使用など、色々学べました。
もし使ってみたい方がいれば、フィードバックいただけると嬉しいです！

## 出典・参考リンク

- [Wise API](https://wise.com/ja/help/articles/2958106/wise-api%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) - 為替レート取得に使用
- [Flag Icons](https://github.com/lipis/flag-icons) - 国旗 SVG アイコンのライブラリ
- [通貨換算ボード（本アプリ）](https://currency-converter-board.com/)
- [React](https://react.dev/) - フロントエンドフレームワーク
- [FastAPI](https://fastapi.tiangolo.com/) - バックエンドフレームワーク
- [DaisyUI](https://daisyui.com/) - TailwindCSS コンポーネントライブラリ
- [@dnd-kit](https://dndkit.com/) - ドラッグ&ドロップライブラリ
