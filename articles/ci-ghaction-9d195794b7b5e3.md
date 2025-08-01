---
title: "ZennからQiitaとTwitterへの投稿を自動化してみる"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rss, twitter, zenn, qiita, githubactions]
published: true
published_at: 2025-06-28 15:00 # 未来の日時を指定する
---

# はじめに
Zennでは一ヶ月前ぐらいから描き始めたのですが、qiitaと自分のtwitterにも連動させて投稿しようかなと思っていたところです。
やはり先人の方たちはもう同じようなことを考えられて、便利なものを作成されておりました。
ここでは引用祭りにはなってしまいますが、先人たちが作られた以下二つのgtihub actionを使わせていただきながら紹介したいと思います。

https://github.com/azu/rss-to-twitter

https://github.com/C-Naoki/zenn-qiita-sync

## 対象読者
- TwitterにもZennの投稿を連動させたい方
- QiitaにもZennの投稿と連動して投稿したい方

## 記事を読むメリット
- 一つのレポジトリからTwitterとQiitaへ投稿可能
- Github Actionの作成と配布の参考になる

## RSSと利用したZennとTwitterの連携投稿
こちらも非常わかりやすく説明されているため、以下リンクを参照していただきながらセットアップしてみてください。
https://github.com/C-Naoki/zenn-qiita-sync

## ZennとQiitaの連携投稿
以下に既に、わかりやすく解説されていますので是非ご参照ください。
https://github.com/C-Naoki/zenn-qiita-sync

## 自分で一からアクションを作成して配布したい場合
上記二つで自分がやりたいことは出来たのですが、
Mediumにも投稿を考えている為、以下のテンプレートが自分用にカスタムする際に参考になりました。
https://github.com/actions/typescript-action

またこちらも非常によくまとめられていて感謝です！
https://zenn.dev/farstep/books/learn-github-actions


## 参考文献

https://efcl.info/2023/05/28/rss-to-twitter/

https://zenn.dev/naoki0103/articles/zenn-qiita-sync-workflow

https://zenn.dev/ot07/articles/zenn-qiita-article-centralized

## まとめ
ここまでお読みいただきありがとうございます。
色々と調べる中で、便利なものを作っていただいている方には感謝しかありません。
この場でもう一度お礼を申し上げます。
また、👍と思った方は上記の方々へスターや👍をよろしくお願いいたします!
