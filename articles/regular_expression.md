---
title: "正規表現におけるメタキャラクタまとめ(チートシート)"
emoji: "📄" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # te| idea: アイデア記事
topics: ["正規表現", "Linux"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに

Linux の grep コマンドを学習している際に、正規表現を初めて知りました。
今後のテキスト処理を効率的に行うために基本的なメタキャラクタと使い方をまとめてみました。

# 正規表現とは

- 正規表現とは**条件に合致する文字列集合を表現するための記法**である。
- 正規表現で特別な意味を持つ記号を**メタキャラクタ**(**メタ文字**) という。
- 正規表現には**基本正規表現**と使えるメタ文字を増やした**拡張正規表現**がある。
- 同じパターンでも書き方が異なる場合がある。

### 使用例

- 文字列の検索と置換
  - 特定のパターンに一致する文字列を検索できる。
- フォームの入力検証
  - メールアドレス、電話番号、郵便番号などの形式を検証できる。
- パスワードの強度認証
  - 大文字・小文字・数字・特殊文字の組み合わせを検証できる。

これら以外にもログ解析やデータのフィルタリングなど様々な場面で活用することができる。

# 文字としてマッチするメタキャラクタ

| 基本/拡張 |                  意味                  |                                                                                 図解例                                                                                  | 図解例にマッチする文字列の例                                                                                           |
| :-------: | :------------------------------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------------------------- |
|    `.`    |             任意の 1 文字              |                                   `t.st` ![](/images/regular_expression/image1.png) `.txt` ![](/images/regular_expression/image2.png)                                   | `test` `tzst` `tast` <br><br><br><br><br><br> `.txt` `stxt`                                                            |
|   `[ ]`   |  [ ]の中に含まれる、いずれかの 1 文字  |  `g[rly]ow` ![](/images/regular_expression/image3.png) `a[b-fB-F]g` ![](/images/regular_expression/image4.png) `test[1-5]` ![](/images/regular_expression/image5.png)   | `grow` `glow` `gyow` <br><br><br><br><br><br><br><br><br> `abg` `aEg` <br><br><br><br><br><br><br><br> `test1` `test4` |
|  `[^ ]`   | [ ]の中に含まれない、いずれかの 1 文字 | `g[^rly]ow` ![](/images/regular_expression/image6.png) `a[^b-fB-F]g` ![](/images/regular_expression/image7.png) `test[^1-5]` ![](/images/regular_expression/image8.png) | `gaow` `gdow` `gfow` <br><br><br><br><br><br><br><br><br> `akg` `aRg` <br><br><br><br><br><br><br> `test8`             |
|    `\`    |     直後のメタ文字の意味を打ち消す     |                                                           `\.txt` ![](/images/regular_expression/image9.png)                                                            | `.txt`                                                                                                                 |

- 直前に`\`を置いてメタ文字の意味を打ち消すこと「**エスケープする**」と言う。

# 位置にマッチするメタキャラクタ

| 基本/拡張 | 意味 |                        図解例                        | 図解例にマッチする文字列の例   |
| :-------: | :--: | :--------------------------------------------------: | :----------------------------- |
|    `^`    | 行頭 | `^https` ![](/images/regular_expression/image10.png) | `httpstest` `https://zenn.dev` |
|    `$`    | 行末 |  `png$` ![](/images/regular_expression/image11.png)  | `image.png` `testpng`          |

# 繰り返しを指定するメタキャラクタ

他の正規表現の後に置いて、**直前の正規表現が一定回数繰り返される**ことを意味する。
| 基本/拡張 | 意味 | 図解例 | 図解例にマッチする文字列の例 |
| :----: | :---: | :----: | :---- |
| `*` | 0 回以上の繰り返し | `Ye*ah` ![](/images/regular_expression/image12.png) `Y[ea]*h` ![](/images/regular_expression/image13.png) | `Yeah` `Yeeeeeah` `Yah` <br><br><br><br><br><br><br> `Yeah` `Yeeaeah` `Yaaaaah` |
| なし/`+` | 1 回以上の繰り返し | `Ye+ah` ![](/images/regular_expression/image14.png) | `Yeah` `Yeeeah` |
| なし/`?` | 0 回または 1 回の繰り返し | `Ye?ah` ![](/images/regular_expression/image15.png) | `Yah` `Yeah` |

- 任意の 1 文字を表す`.`と 0 回以上の繰り返しを表す`*`を組み合わせて`.*`とするとあらゆる文字列にマッチする。
- たとえば「`image`で始まって`png`で終わる」文字列は`^image.*png$`で表すことができる。

### 繰り返し回数の指定をするメタキャラクタ

繰り返しの回数を数値で指定するには`{}`を用いて表現できる。
| 基本/拡張 | 意味 |
| :----: | :---: |
| `\{m,n\}`/`{m,n}` | m 回以上 n 回以下の繰り返し |
| `\{m\}`/`{m}` | ちょうど m 回の繰り返し |
| `\{m,\}`/`m,` | m 回以上の繰り返し |

# その他のメタキャラクタ

|  基本/拡張  |                意味                |                            図解例                             | 図解例にマッチする文字列の例 |
| :---------: | :--------------------------------: | :-----------------------------------------------------------: | :--------------------------- |
| `\(\)`/`()` |           グループ化する           |   `(Test){2,}` ![](/images/regular_expression/image16.png)    | `TestTest` `TestTestTest`    |
|  なし/`\|`  | 複数の正規表現を OR 条件で連結する | `(My\|Your)house` ![](/images/regular_expression/image17.png) | `Myhouse` `Yourhouse`        |

# 終わりに

ご指摘があればコメントいただけると幸いです。
最後までお読みいただきありがとうございました。

# 参考資料

書籍：[新しい Linux の教科書](https://www.amazon.co.jp/%E6%96%B0%E3%81%97%E3%81%84Linux%E3%81%AE%E6%95%99%E7%A7%91%E6%9B%B8-%E4%B8%89%E5%AE%85-%E8%8B%B1%E6%98%8E/dp/4797380942/ref=sr_1_1?adgrpid=117229375656&hvadid=655144332605&hvdev=c&hvqmt=e&hvtargid=kwd-1152146940662&hydad\cr=21814_13461165&jp-ad-ap=0&keywords=%E6%96%B0%E3%81%97%E3%81%84linux%E3%81%AE%E6%95%99%E7%A7%91%E6%9B%B8&qid=1688622508&sr=8-1)

https://www.megasoft.co.jp/mifes/seiki/meta.html

- この記事ではまとめきれなかったメタ文字の一覧が確認できます。

https://regexper.com/

- 正規表現のパターンを入力すると図で分かりやすく表してくれます。

https://www.debuggex.com/

- 入力したパターンが一致するかどうかをテストすることができます。
