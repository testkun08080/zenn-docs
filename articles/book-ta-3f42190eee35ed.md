---
title: "そろそろテクニカルアーティスト10年目/役に立った書籍やwebをどうぞ"
emoji: "📚"
type: "idea"
topics: [テクニカルアーティスト, ゲーム, CG, 書籍, キャリア]
published: false
published_at: 2025-XX-XX XX:XX
---

# はじめに
気づけばTechnical Artistとして役10年目を迎えました。
この10年間で読んできた本やwebから、特に役立ったもの、いくつか紹介したいと思います。
少し古いものなどがありますが、ツール特化の内容以外は現在でも有効かと思います。
（古い本の改正版なども出ていますので、そちらを紹介しています）


### 私のざっくり経歴

:::details 経歴
アーティストとして入社
👇
シェーダーアーティスト/テクニカルアーティスト
👇
テクニカルアーティスト
👇
フリーランス(ゲームデバイスとソフトの両方を開発中)
:::

これからTechnical Artistを目指す方、既に現場で活躍されている方、どちらにも参考になれば幸いです。
（経歴に関連した内容を多く含みますが、ご了承ください。）

# そもそも、Technical Artistとは?
簡単に言うと、アーティストとエンジニアの橋渡しをする役割です。
僕が会社入ったところは結構微妙な定義でした。（今でもまぁまぁ定義は微妙かと/もはや定義なんて🙂‍↔️）

**美術的な感性と技術的な知識を持ち、パイプライン構築やツール開発、最適化などを担当している人。**

というところに落ち着くと思います。

が、国を跨いだりするときっぱり分かれていて、
うちの会社ではこうだけど、他の会社（国）ではそうではないなんてことはあり得ます。

僕なりの結論としては、

**上記の内容を理解でき、つつも、どんな分野でも飛び込んで理解し、解決まで持ち込める人**

です。

---




# 基礎・入門編

### 1. コンピュータグラフィックス関連

テクニカルアーティストスタートキット 改訂版
https://amzn.to/4hmo6l3

UnrealEngine4マテリアルデザイン入門
https://amzn.to/4ogePNE

Physically Based Rendering 4th Edition 日本語版
https://amzn.to/4q27HpF


リアルタイムレンダリング 第4版 (Real Time Rendering Fourth Edition 日本語版) 
https://amzn.to/4nHfaZx


ライティング & レンダリング 第3版
https://amzn.to/4q1bmnH


Processing:ビジュアルデザイナーとアーティストのためのプログラミング入門
https://amzn.to/4nQhJsE


カラーコレクションハンドブック第2版
https://amzn.to/48VGZsB


Houdini ビジュアルエフェクトの教科書
https://amzn.to/3WsCGNZ

ゲーム制作者になるための3Dグラフィックス技術 改訂3版
https://amzn.to/4mWSjYW


「電波と光」のことが一冊でまるごとわかる
https://amzn.to/4of9cza

「電波」のキホン　周波数帯の再割り当てで注目される電波の世界 (イチバンやさしい理工系)
https://amzn.to/48mhuAD

「電子回路」のキホン (イチバンやさしい理工系)
https://amzn.to/4q7rmEQ

「レンズ」のキホン (イチバンやさしい理工系)
https://amzn.to/3INHdaL

「半導体」のキホン　人類の情報力を拡張する「半導体」のすべて (イチバンやさしい理工系)
https://amzn.to/3IRr0Bq

「通信」のキホン　いつでもどこでも超高速でつながる世界 (イチバンやさしい理工系)
https://amzn.to/4pY6cZL

電子部品 超入門: 抵抗・コイル・コンデンサの実務で役立つ基礎知識
https://www.amazon.co.jp/gp/product/B0929F97FH/ref=kinw_myk_ro_title


## web

shadertoy
https://www.shadertoy.com/

thebookofshaders
https://thebookofshaders.com/11/?lan=jp

**Real-Time Rendering, Fourth Edition**
https://www.amazon.co.jp/dp/1138627003

リアルタイムレンダリングのバイブル的存在。
分厚くて最初は intimidating ですが、辞書的に使うだけでも価値があります。
レンダリングパイプライン、シェーダー、ライティングなど、CGの基礎を網羅的に学べる一冊。

**Game Engine Architecture, Third Edition**
https://www.amazon.co.jp/dp/1138035459

ゲームエンジンの内部構造を理解するのに最適。
TAとして「なぜこういう仕様なのか」を理解するために読みました。
エンジニアとのコミュニケーションが格段に楽になります。

### 2. シェーダープログラミング

**The Book of Shaders**
https://thebookofshaders.com/

オンラインで無料で読めるシェーダー入門書。
視覚的にわかりやすく、実際にブラウザで試しながら学べるのが素晴らしい。
GLSLベースですが、考え方はHLSLにも応用できます。

**Unity シェーダープログラミングの教科書**
https://www.amazon.co.jp/dp/XXXXXXXXXX

Unity特化ですが、シェーダーの基礎から応用まで日本語でわかりやすく解説。
Unityを使っている方には特におすすめです。

---

# プログラミング・スクリプティング

### 3. Python関連

**退屈なことはPythonにやらせよう**
https://www.amazon.co.jp/dp/4873117682

TAの仕事の半分は「面倒な作業の自動化」です。
この本はまさにそのためのバイブル。
DCCツールのスクリプティングにも応用できる知識が満載。

**Effective Python 第2版**
https://www.amazon.co.jp/dp/4873119170

Pythonをある程度書ける人向け。
より良いコードを書くためのベストプラクティス集。
Maya、Houdini、Blenderなどでスクリプトを書く際に役立ちます。

### 4. コード品質・設計

**リーダブルコード**
https://www.amazon.co.jp/dp/4873115655

コードの読みやすさに特化した名著。
チームで使うツールを作る際、この本の知識は必須です。
「自分以外の人が読むコード」を意識するきっかけになりました。

**リファクタリング 第2版**
https://www.amazon.co.jp/dp/4274224546

既存のツールやパイプラインを改善する際に何度も読み返しています。
「動くコード」から「良いコード」への進化に必要な知識が詰まっています。

---

# アート・デザイン

### 5. ビジュアル理論

**デジタルライティング&レンダリング 第3版**
https://www.amazon.co.jp/dp/XXXXXXXXXX

ライティングアーティストとの会話に必須の知識。
技術的な実装だけでなく、芸術的な理論も学べます。

**カラー&ライト**
https://www.amazon.co.jp/dp/4862460623

色彩理論とライティングの基礎。
シェーダーを書く際に「なぜこの色なのか」を理解するために読みました。
薄くて読みやすいのも◎

---

# パイプライン・ワークフロー

### 6. プロジェクト管理

**アジャイルサムライ**
https://www.amazon.co.jp/dp/4274068560

ゲーム開発のワークフローを改善するために読みました。
TAはパイプラインの設計者でもあるので、プロジェクト管理の視点は重要です。

**チームトポロジー**
https://www.amazon.co.jp/dp/4820729632

組織構造とコミュニケーションパターンについて。
TAがどこに所属すべきか、どう動くべきかのヒントが得られます。

---

# 数学・アルゴリズム

### 7. 数学基礎

**ゲームプログラミングのための3Dグラフィックス数学**
https://www.amazon.co.jp/dp/XXXXXXXXXX

ベクトル、行列、クォータニオンなど、CGに必要な数学を実践的に学べます。
「なぜこの計算が必要なのか」が理解できるようになります。

**プログラマのための線形代数**
https://www.amazon.co.jp/dp/4274065782

より深く線形代数を理解したい方向け。
シェーダーでの座標変換などを理解するのに役立ちました。

---

# 思考法・キャリア

### 8. ソフトスキル

**ソフトウェア開発者の人生マニュアル**
https://www.amazon.co.jp/dp/4822255743

技術者としてのキャリア全般について。
TAとしてどう成長していくか考える際に参考になりました。

**エンジニアリング組織論への招待**
https://www.amazon.co.jp/dp/4774196053

技術組織の中でTAとしてどう価値を出すか。
組織論の視点から自分の役割を見直すきっかけになります。

---

# 特定ツール・エンジン

### 9. Maya/Houdini/Unreal Engine関連

**Maya Python for Games and Film**
https://www.amazon.co.jp/dp/XXXXXXXXXX

Mayaでのパイプライン構築に特化。
実践的なサンプルコードが豊富で、すぐに業務に活かせます。

**Houdini for Technical Artists**
https://www.amazon.co.jp/dp/XXXXXXXXXX

Houdiniをプロシージャル的に使うための知識。
TAとしてHoudiniを学ぶ際の最初の一冊におすすめ。

**Unreal Engine Blueprint Scripting**
https://www.amazon.co.jp/dp/XXXXXXXXXX

Blueprintを使ったツール開発やパイプライン構築。
プログラマーでないアーティストとの協業に役立ちます。

---

# 最適化・パフォーマンス

### 10. パフォーマンス改善

**ゲーム最適化技法**
https://www.amazon.co.jp/dp/XXXXXXXXXX

TAの重要な仕事の一つが最適化。
プロファイリングから具体的な改善手法まで網羅されています。

**GPU Gems シリーズ**
https://developer.nvidia.com/gpugems/gpugems/contributors

オンラインで無料公開されているGPU最適化のバイブル。
シェーダー最適化やレンダリング技法の宝庫です。

---

# 番外編：読み物として面白かった本

### 11. ゲーム開発の裏側

**Masters of Doom**
https://www.amazon.co.jp/dp/4797335408

id Softwareの黎明期を描いたノンフィクション。
技術者としてのモチベーションを上げてくれる一冊。

**The Art of Game Design**
https://www.amazon.co.jp/dp/XXXXXXXXXX

ゲームデザインの本質について。
TAとして「なぜこのゲームにこの技術が必要なのか」を考える視点を養えます。

---

## まとめ/最後に
最後までお読み頂きありがとうございました！
この記事を書きながら、こんなのもあったなぁ。。
でもなんか変わっているようでやってることあんま本質的には変わってないなぁと感じました笑

あと、技術書って地味に高いので。。
会社でこれ買ってくれ！とかやって同僚とか、会社でシェアできるようにした方がいいと思います。
（個人で買うと地味に痛いので。）

今はAIがあるのでなんとかなりますが、ゲーム開発は知恵を必要とする場面が多々あると思います。
そういう知恵とか鍛えるのはこういう本などから得られるかなぁ,,,とか前向きに考えています。

今回紹介した本はあくまで私個人の経験ですが、これからTAを目指す方の参考になれば幸いです。
何か「これも読んだ方がいいよ！」というおすすめがあれば、是非コメントで教えてください。

👍いただけるとモチベ上がります！

---

### 関連リンク
- [Technical Artist Bootcamp (GDC)](https://www.gdconf.com/)
- [Tech-Artists.org](https://tech-artists.org/)
- [Houdini Documentation](https://www.sidefx.com/docs/)
- [Unreal Engine Documentation](https://docs.unrealengine.com/)

---

## 補足：Amazonアソシエイトリンクについて
この記事に含まれるAmazonリンクの一部はアソシエイトリンクです。
リンク経由でご購入いただくと→コーヒー中毒の僕の栄養分になります≠モチベが上がります
もちろん、書籍タイトルで直接検索していただいても構いません。
