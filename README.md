# Zenn CLI

* [📘 How to use](https://zenn.dev/zenn/articles/zenn-cli-guide)

# Commands 📝

1. create new article
    ```
    npx zenn new:article
    npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨
    ```

2. create new book
    ```
    npx zenn new:book
    ```
    
3. preview
    ```
    npx zenn preview
    ```


---
# 見出し1です
## 見出し2だね
### 見出し3かな
#### 見出し4デス
---

- こんちわ!
- おはようございます!
  - こんばんわ!
  * こんばんにちわ!
---

1. 一番目
2. 二番目
---

[アンカーテキスト](#%E8%A6%8B%E5%87%BA%E3%81%971%E3%81%A7%E3%81%99)

---

![](/images/test/heyhey.png)
![](https://storage.googleapis.com/zenn-user-upload/avatar/9965dabc76.jpeg =250x)
![Altテキスト-Zenn](https://storage.googleapis.com/zenn-user-upload/avatar/9965dabc76.jpeg =150x)
![](https://storage.googleapis.com/zenn-user-upload/avatar/9965dabc76.jpeg =50x)
*Zenn*

[![](https://storage.googleapis.com/zenn-user-upload/avatar/9965dabc76.jpeg =50x)](https://zenn.dev/)
*Zenn webページ(キャプションが中央に来ない)*




---

| タスク1 | タスク２ | タスク３ |
| ---- | ---- | ---- |
| 終わらない | 終わったはず | 忘れた |
| 😅 | 知らない | まじ？ |

---

```js
シンプルコード
```

```js:hey.hey
コード+ファイル名
```

```diff js:hey.hey
@@ -4,6 +4,5 @@
+    const foo = bar.baz([1, 2, 3]) + 1;
-    let foo = bar.baz([1, 2, 3]);
```

$$
e^{i\theta} e^{-i\theta} = (\cos\theta + i\sin\theta)e^{-i\theta}
$$

---

> 引用文..etc


> 脚注の例[^1]です。インライン^[脚注２はインライン]で書くこともできます。

[^1]: 脚注の内容その1（別々に書いたほうが可読性が高い）

---


*イタリック🇮🇹*
**太字**
~~打ち消し線~~
インラインで`python hey.py`を挿入する

<!-- TODO: ◯◯について追加しなければならない -->


:::message
ワーニングっぽいメッセージ
:::

:::message alert
⚠️警告っぽいメッセージ
:::

:::details アコーディオン(タイトル)
ここへコメント
:::

::::details タイトル
:::message
ネストされた要素
:::
::::

https://github.com/testkun08080


# ポストのURLだけの行（前後に改行が必要です）
https://twitter.com/jack/status/20

# x.comドメインの場合
https://x.com/jack/status/20

# YouTubeのURLだけの行（前後に改行が必要です）
![](https://youtu.be/l0GN40EL1VU?si=Ba7Ulv9sVqkn5KDe)
https://youtu.be/l0GN40EL1VU?si=Ba7Ulv9sVqkn5KDe
*ずっと聴いていられる*

# GitHubのファイルURLまたはパーマリンクだけの行（前後に改行が必要です）
https://github.com/testkun08080/testkun08080/blob/main/README.md


# コードの開始行と終了行を指定
https://github.com/testkun08080/testkun08080/blob/main/README.md#L1-L6

# コードの開始行のみ指定
https://github.com/testkun08080/testkun08080/blob/main/README.md#L6



@[gist](GistのページURL)

@[codepen](ページのURL)

@[slideshare](スライドのkey)

@[speakerdeck](スライドのID)


例:
@[speakerdeck](4f926da9cb4cd0001f00a1ff)
@[speakerdeck](4f926da9cb4cd0001f00a1ff?slide=24)


@[docswell](スライドのURL)
# もしくは
@[docswell](埋め込み用のURL)

例:
@[docswell](https://www.docswell.com/s/ku-suke/LK7J5V-hello-docswell)
@[docswell](https://www.docswell.com/s/ku-suke/LK7J5V-hello-docswell#p13)
@[docswell](https://www.docswell.com/s/ku-suke/LK7J5V-hello-docswell/13)
@[docswell](https://www.docswell.com/slide/LK7J5V/embed)


@[jsfiddle](ページのURL)


@[codesandbox](embed用のURL)


@[stackblitz](embed用のURL)


@[figma](ファイルまたはプロトタイプのURL)


@[blueprintue](ページのURL)

例：
@[blueprintue](https://blueprintue.com/render/0ovgynk-/)


```mermaid
graph TB
    A[Hard edge] -->|Link text| B(Round edge)
    B --> C{Decision}
    C -->|One| D[Result one]
    C -->|Two| E[Result two]
```


```mermaid
graph LR
   a --> b & c--> d
```