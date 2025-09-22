---
title: "Reactでポストプロセスを触ってカスタムシェーダーの作り方をざっくり理解してみる"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, postprocess, shader ]
published: false
---



# はじめに
もう九月か。。。
独り立ちして約１年ぐらい経とうとしている中、ポートフォリオページがまだ`Under Maintenance`なので、
重い腰を上げて、ポートフォリオページを作成することにしました。
そこで、大好きなシェーダーをウェブにはもりもり使おうと思い立ったのが、きっかけです。
（ゲームだとそうは行かないので）

さて、ここでは以下モジュールですでに用意されている一般的？なポストを実装するまでの工程と、
カスタムシェーダーを使ってワールドノーマルを表示数だけの工程を記します。
なぜに、ワールドノーマルかというと、ディファードレンダリングで慣れている僕は、やっぱり使いやすいワールドノーマルが欲しくなるんです。
それと、やっぱ3D空間でベクトルとの連携を考えると、まぁまずカスタムシェーダーを作る最初の一歩としては無難なのかなと思ったのです。

## `SuperClaude_Framework`とは
Claude Codeを “拡張”して開発作業を進めやすくするためのフレームワークです。
ここでなんだかんだ言うよりインストールして頂いて、触っていただくのが早いです。

以下レポです。

https://github.com/SuperClaude-Org/SuperClaude_Framework

## `claude-code-slack-bot`をClaude Codeに紹介してもらう
どちらもにも同じ質問をしてみて解説してもらいます。

### 通常 Claude Code

#### コマンド
```zsh
このプロジェクトを説明してほしい
```

#### 結果
```zsh
⏺ このプロジェクトは、Slack内で Claude Code の AI
  コーディングアシスタント機能を提供する TypeScript ベースの Slack ボットです。

  主要機能

  作業ディレクトリ管理
  - ベースディレクトリ設定（例：/Users/username/Code/）で短いプロジェクト名使用可能
  - チャンネルごとのデフォルト作業ディレクトリ
  - スレッド単位での作業ディレクトリ上書き機能

  リアルタイム タスク追跡
  - Claude の計画プロセスをフォーマットされたタスクリストとして表示
  - 作業進行に応じたリアルタイム ステータス更新
  - 優先度インジケーター（🔴 高、🟡 中、🟢 低）

  ファイルアップロード対応
  - 画像（JPG、PNG、GIF、WebP）、テキストファイル、コードファイル対応
  - テキストファイルはプロンプトに直接埋め込み
  - 画像は Claude が Read ツールで分析

  高度なメッセージ処理
  - ストリーミング レスポンス
  - ツール使用の豊富なフォーマット
  - ステータス インジケーター（🤔 思考中、⚙️ 作業中、✅ 完了）

  MCP (Model Context Protocol) 統合
  - 外部 MCP サーバーによる Claude 機能拡張
  - ファイルシステムアクセス、GitHub API、データベース接続、Web検索など

  アーキテクチャ

  主要コンポーネント：
  - slack-handler.ts - Slack イベント処理
  - claude-handler.ts - Claude Code SDK 統合
  - working-directory-manager.ts - ディレクトリ管理
  - todo-manager.ts - タスク追跡
  - mcp-manager.ts - MCP サーバー管理

  このボットにより、Slack ワークスペース内で直接 AI によるコーディング支援、ファイル
  分析、コードレビュー、プロジェクト管理が可能になります。
```


### Super Claude

#### コマンド
```zsh
/sc:explain このプロジェクトを説明してほしい
```

#### 結果
```zsh
> /sc:explain is running… このプロジェクトを説明してほしい
  ⎿  Allowed 4 tools for this command

⏺ このプロジェクトについて詳しく分析して説明いたします。

⏺ Update Todos
  ⎿  ☐ プロジェクト構造を分析    
     ☐ 主要コンポーネントを特定
     ☐ アーキテクチャと機能を解説
     ☐ 技術スタックと設定を説明

⏺ Read(package.json)
  ⎿  Read 29 lines (ctrl+r to expand)

⏺ List(~/Git/claude-code-slack-bot)
  ⎿  Listed 26 paths (ctrl+r to expand)

⏺ Read(src)
  ⎿  Error: EISDIR: illegal operation on a directory, read

⏺ List(src)
  ⎿  Listed 16 paths (ctrl+r to expand)

⏺ Read(src/index.ts)
  ⎿  Read 58 lines (ctrl+r to expand)

⏺ Update Todos
  ⎿  ☒ プロジェクト構造を分析    
     ☐ 主要コンポーネントを特定
     ☐ アーキテクチャと機能を解説
     ☐ 技術スタックと設定を説明

⏺ Read(src/slack-handler.ts)
  ⎿  Read 50 lines (ctrl+r to expand)

⏺ Read(src/claude-handler.ts)
  ⎿  Read 50 lines (ctrl+r to expand)

⏺ Read(src/config.ts)
  ⎿  Read 34 lines (ctrl+r to expand)

⏺ Update Todos
  ⎿  ☒ プロジェクト構造を分析    
     ☒ 主要コンポーネントを特定
     ☐ アーキテクチャと機能を解説
     ☐ 技術スタックと設定を説明

⏺ Claude Code Slack Bot プロジェクト説明

  このプロジェクトは Claude Code SDK を活用した Slack Bot で、Slack ワークスペース内で AI
  によるコーディング支援を提供するTypeScript ベースのアプリケーションです。

  🏗️ プロジェクト概要

  主要目的

  - リアルタイム AI アシスタンス: Slack チャネルや DM で Claude Code の機能を利用
  - ファイル分析: 画像やコードファイルのアップロードと分析
  - 作業ディレクトリ管理: プロジェクト固有のコンテキスト維持
  - タスク追跡: リアルタイムでのタスクリストと進捗表示

  🔧 技術アーキテクチャ

  コアコンポーネント

  1. エントリーポイント (src/index.ts)

  - アプリケーション初期化とシステム起動
  - 環境設定の検証
  - Slack App、Claude Handler、MCP Manager の初期化

  2. Slack イベント処理 (src/slack-handler.ts)

  - メッセージイベント、メンション、ファイルアップロードの処理
  - Socket Mode によるリアルタイム通信
  - スレッド管理とコンテキスト維持

  3. Claude Code 統合 (src/claude-handler.ts)

  - Claude Code SDK との通信
  - ストリーミングレスポンス処理
  - セッション管理（ユーザー/チャネル/スレッド別）

  4. 作業ディレクトリ管理 (src/working-directory-manager.ts)

  - チャネル固有のデフォルトディレクトリ
  - スレッドによる一時的なオーバーライド
  - 相対パス/絶対パスの解決

  5. ファイル処理 (src/file-handler.ts)

  - 画像、テキスト、コードファイルのサポート
  - セキュアなダウンロードと一時保存
  - ファイルサイズ制限（50MB）と自動クリーンアップ

  6. タスク管理 (src/todo-manager.ts)

  - Claude の計画プロセスのリアルタイム表示
  - 優先度レベル（🔴 高、🟡 中、🟢 低）
  - 進捗状況の動的更新

  7. MCP サーバー管理 (src/mcp-manager.ts)

  - Model Context Protocol サーバーの設定と管理
  - 外部ツールとの統合（ファイルシステム、GitHub API等）
  - 動的設定再読み込み

  🌟 主要機能

  1. インテリジェントな作業ディレクトリ管理

  階層: スレッド固有 > チャネルデフォルト > DM 固有
  設定: `cwd project-name` または `cwd /absolute/path`
  サポート: 相対パス・絶対パス両対応

  2. リアルタイムタスク追跡

  📋 タスクリスト

  🔄 進行中:
  🔴 認証システムの分析

  ⏳ 待機中:
  🟡 OAuth フローの実装
  🟢 エラーハンドリングの追加

  進捗: 1/3 タスク完了 (33%)

  3. 高度なメッセージ処理

  - ストリーミングレスポンス: リアルタイムでメッセージ更新
  - ツール使用の可視化: ファイル編集、bash コマンド等の表示
  - ステータス表示: 🤔 考え中、⚙️ 作業中、✅ 完了

  4. MCP 統合

  - 外部ツール拡張: ファイルシステム、GitHub、データベース連携
  - マルチサーバー対応: stdio、SSE、HTTP プロトコル
  - 自動設定: mcp-servers.json から自動読み込み

  5. セキュリティと権限管理

  - Permission Server: Slack コンテキストでの安全な実行
  - ファイル検証: タイプとサイズのバリデーション
  - 一時ファイル管理: 自動クリーンアップ

⏺ Update Todos
  ⎿  ☒ プロジェクト構造を分析    
     ☒ 主要コンポーネントを特定
     ☒ アーキテクチャと機能を解説
     ☐ 技術スタックと設定を説明

⏺ 🛠️ 技術スタック

  フロントエンド（Slack）

  - @slack/bolt: Slack App 開発フレームワーク
  - Socket Mode: リアルタイムイベント処理
  - ファイルアップロード: 画像・コード・ドキュメント対応

  バックエンド（Node.js）

  - TypeScript 5.8.3: 型安全性とモダンJS
  - @anthropic-ai/claude-code 1.0.35: Claude Code SDK
  - @modelcontextprotocol/sdk 1.13.2: MCP サーバー統合
  - node-fetch 3.3.2: HTTP クライアント
  - dotenv 16.6.0: 環境変数管理

  開発環境

  "scripts": {
    "start": "tsx src/index.ts",     // 本番実行
    "dev": "tsx watch src/index.ts", // 開発モード
    "build": "tsc",                  // TypeScript コンパイル
    "prod": "node dist/index.js"     // プロダクション
  }

  ⚙️ 環境設定

  必須環境変数

  # Slack App
  SLACK_BOT_TOKEN=xoxb-your-bot-token
  SLACK_APP_TOKEN=xapp-your-app-token
  SLACK_SIGNING_SECRET=your-signing-secret

  # Claude Code
  ANTHROPIC_API_KEY=your-anthropic-api-key

  オプション設定

  # 作業ディレクトリ
  BASE_DIRECTORY=/Users/username/Code/

  # サードパーティプロバイダー
  CLAUDE_CODE_USE_BEDROCK=1
  CLAUDE_CODE_USE_VERTEX=1

  # 開発
  DEBUG=true

  Slack App 権限

  必須権限:
    - app_mentions:read    # メンション読み取り
    - channels:history     # チャネル履歴
    - chat:write          # メッセージ送信
    - im:history          # DM履歴
    - users:read          # ユーザー情報
    - reactions:write     # リアクション追加

  イベント:
    - app_mention         # メンション時
    - message.im          # DM受信時
    - member_joined_channel # チャネル追加時

  🚀 使用例

  チャネルセットアップ

  1. ボットをチャネルに追加
  2. ウェルカムメッセージで作業ディレクトリ設定を要求
  3. `cwd project-name` でデフォルト設定
  4. `@ClaudeBot 認証について手伝って` で利用開始

  ファイル分析

  [画像/コードファイルをアップロード]
  このスクリーンショットを分析して改善案を提案して

  MCP サーバー管理

  # 設定済みサーバー確認
  User: mcp
  Bot: 🔧 MCP Servers:
       • filesystem (stdio)
       • github (stdio)

  # 設定再読み込み
  User: mcp reload
  Bot: ✅ MCP設定を再読み込みしました

⏺ Update Todos
  ⎿  ☒ プロジェクト構造を分析    
     ☒ 主要コンポーネントを特定
     ☒ アーキテクチャと機能を解説
     ☒ 技術スタックと設定を説明

⏺ 💡 主要設計思想

  1. セッションベースの会話管理

  - ユーザー、チャネル、スレッド毎の独立したコンテキスト
  - 継続的な対話の一貫性を保証
  - メモリ効率的なセッション管理

  2. 階層的作業ディレクトリ

  優先順位: スレッド設定 > チャネルデフォルト > DM設定
  柔軟性: 相対パス・絶対パス両対応
  智能解決: ベースディレクトリ + プロジェクト名

  3. リアルタイム透明性

  - Claude の思考プロセスを可視化
  - 進行中タスクのライブ更新
  - エラー発生時の詳細なフィードバック

  4. セキュリティファースト

  - 環境変数による機密情報管理
  - ファイルタイプ・サイズの厳格な検証
  - 一時ファイルの自動クリーンアップ
  - Permission Server による安全な実行制御

  このプロジェクトは、開発者の生産性向上を目的とした高度な AI アシスタントシステムで、Slack
  の使いやすさと Claude Code の強力な AI 機能を統合した実用的なソリューションです。
```

### そもそもclaude-code-slack-botの活用シーン
スマホからとりあえずコマンドできるので、

- ご飯中
- 外出中
- お茶・コーヒ休憩

などなど。
Claude Codeが時間を奪います。
ただし、エラーやうまくいかないところがあるので、その内ローカルで修正してPRを出そうかと思いってます。

## まとめ
体感時間としては、通常の方が早いけど、細かさで言うとSuper Claudeの方がまあ軍配が上がる。
是非触っていない方は触って頂きたい。

`体調崩さないようにAIツールとは向き合っていきたい`


### 関連リンク
- [SuperClaude Framework](https://github.com/SuperClaude-Org/SuperClaude_Framework)
- [claude-code-slack-bot](https://github.com/mpociot/claude-code-slack-bot)