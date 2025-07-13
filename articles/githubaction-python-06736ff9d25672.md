---
title: "官報に少し興味を持ってRSSを作るために、GitHub Actionsでスクレイピングするものを作った件(UV on Docker in githubactions)"
emoji: "📖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, githubactions ,rss, uv]
published: false
---


# なぜに作った？
官報というものに少し興味を持って調べたところ、まぁ日頃知らないような内容が公開されているわけです。
ほぼ日常を生きる中では全く必要にならないと思いますが、RSSとかtwitterのアカウントで告知ぐらいしているかな？と思って調べましたが、🈚️。
やる事は比較的簡単なはずなので、RSSを作ってtweetするボットも作ってみようかなと思ったのがキッカケです。

ここでは、汎用的なGitHub Actionsを作った流れと、内容について記述します。

#### 基本構造
- 毎日8:35分ごろに動くワークフロー
- pythonで公式ページをスクレイピングして情報を抽出する（PDFリンクと、書く官報の目次タイトルとリンク）
- pythonだけで良かったけど、UV on docker in githubactions をしてみたかったので、構造はそんな感じ。

時間がくれば、GitHubActionsを使った、RSS作成や、自動Tweetも記述します。

*とりあえず、サンプルのデータ見てみたいという方は、以下レポをご覧ください。*
https://github.com/testkun08080/action-kanpo


#### アクションを通して自動DL/作成されるファイルイメージ
![イメージ](https://raw.githubusercontent.com/testkun08080/action-kanpo/refs/heads/main/docs/%E5%87%BA%E5%8A%9B%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%A4%E3%83%A1%E3%83%BC%E3%82%B8.png)

:::details 出力マークダウンサンプル
![サンプル官報マークダウン](https://github.com/testkun08080/action-kanpo/blob/main/kanpo/2025-07-11/%E5%AE%98%E5%A0%B1_2025-07-11.md)
:::


## この記事の流れ
1. 官報って？
2. 公式ページをスクレイピングしてPDF情報と目次情報を抽出する
3. GitHub Actionsを使って、毎日動くように設定してみる

### 開発環境
- macOS Sequoia 15.5
- VsCode
- zsh 5.9 (arm64-apple-darwin24.0)

## 官報って？


## 公式ページをスクレイピングしてPDF情報と目次情報を抽出する

## GitHub Actionsを使って、毎日動くように設定してみる


## まとめ、感想
とりあえず、これで汎用的なactionは作れたはずです。
謎のとりあえずやってみよう感で作りましたが、どこかで役に立つことを願います。
RSSを作成した話と、Tweetも後々記述します。

## 参考文献
- 公式
  https://www.kanpo.go.jp/