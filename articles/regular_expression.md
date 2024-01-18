---
title: "正規表現におけるメタキャラクタまとめ" # 記事のタイトル
emoji: "🖍️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["正規表現", "Linux"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# 正規表現とは
* 正規表現とは**条件に合致する文字列集合を表現するための記法**である。
* 正規表現で特別な意味を持つ記号を**メタキャラクタ**(**メタ文字**) という。
* 正規表現には**基本正規表現**と使えるメタ文字を増やした**拡張正規表現**がある。
* 同じパターンでも書き方が異なる場合がある。

### 使用例
* 文字列の検索と置換
  * 特定のパターンに一致する文字列を検索できる。
* フォームの入力検証
  * メールアドレス、電話番号、郵便番号などの形式を検証できる。
* パスワードの強度認証
  * 大文字・小文字・数字・特殊文字の組み合わせを検証できる。

これら以外にもログ解析やデータのフィルタリングなど様々な場面で活用することができる。

# 文字としてマッチするメタキャラクタ
| メタキャラクタ | 意味 | 図解例 | 図解例にマッチする文字列の例 |
| ---- | ---- | ---- | ---- |
| `.` | 任意の1文字 | `t.st` ![](/images/t.st.png) `.txt` ![](/images/.txt.png) | `test` `tzst` `tast` <br><br><br><br><br><br> `.txt` `stxt` |
| `[ ]` | [ ]の中に含まれる、いずれかの1文字 | `g[rly]ow` ![](/images/g[rly]ow.png) `a[b-fB-F]g` ![](https://storage.googleapis.com/zenn-user-upload/188778065b35-20240110.png) `test[1-5]` ![](https://storage.googleapis.com/zenn-user-upload/8b79aeb060a6-20240110.png) | `grow` `glow` `gyow` <br><br><br><br><br><br><br><br><br> `abg` `aEg` <br><br><br><br><br><br><br><br> `test1` `test4` |
| `[^ ]` | [ ]の中に含まれない、いずれかの1文字 | `g[^rly]ow` ![](https://storage.googleapis.com/zenn-user-upload/7a68a9e0be35-20240110.png) `a[^b-fB-F]g` ![](https://storage.googleapis.com/zenn-user-upload/7d211bca0f9c-20240110.png) `test[^1-5]` ![](https://storage.googleapis.com/zenn-user-upload/29f8b50c5c66-20240110.png) | `gaow` `gdow` `gfow` <br><br><br><br><br><br><br><br><br> `akg` `aRg` <br><br><br><br><br><br><br> `test8` |
| `\` | 直後のメタ文字の意味を打ち消す | `\.txt` ![](https://storage.googleapis.com/zenn-user-upload/f5283b4dea26-20240110.png) | `.txt` |

- 直前に`/`を置いてメタ文字の意味を打ち消すこと「**エスケープする**」と言う。

# 参考資料
https://regexper.com/
https://www.debuggex.com/
[新しいLinuxの教科書](https://www.amazon.co.jp/%E6%96%B0%E3%81%97%E3%81%84Linux%E3%81%AE%E6%95%99%E7%A7%91%E6%9B%B8-%E4%B8%89%E5%AE%85-%E8%8B%B1%E6%98%8E/dp/4797380942/ref=sr_1_1?adgrpid=117229375656&hvadid=655144332605&hvdev=c&hvqmt=e&hvtargid=kwd-1152146940662&hydadcr=21814_13461165&jp-ad-ap=0&keywords=%E6%96%B0%E3%81%97%E3%81%84linux%E3%81%AE%E6%95%99%E7%A7%91%E6%9B%B8&qid=1688622508&sr=8-1)