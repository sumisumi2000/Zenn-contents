---
title: "boolean 型のカラムに presence のバリデーションをしようとして失敗した話"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Ruby, Rails, バリデーション]
published: false
---

# はじめに

```ruby
class Wish < ApplicationRecord
  validates :granted, presence: true
end
```

Rails の開発をしていて `boolean` 型のカラム（`granted`）に `null` が入らないよう、上記のようなバリデーションを設定していました。
しかし、これでは `granted` が `false` の場合、バリデーションエラーとなってしまいました。

```bash
ActiveRecord::RecordInvalid: バリデーションに失敗しました: Grantedを入力してください
```

今回の記事はこの問題の解説と解決方法をまとめたものです。

# 動作環境

- Ruby 3.2.2
- Rails 7.1.3.2
- PostgreSQL 14.12

# `presence`

[Rails ガイド](https://railsguides.jp/active_record_validations.html#presence)には以下のように書かれていました。

> このヘルパーは、指定された属性が空（empty）でないことを確認します。値が nil や空文字でない、つまり空でもなければホワイトスペースでもないことを確認するために、内部で `Object#blank?` メソッドを使っています。（中略）
> false.blank?は常に true なので、真偽値に対してこのメソッドを使うと正しい結果が得られません。<br>
> Rails ガイド -> Active Reocrd バリデーション -> 2. バリデーションヘルパー -> 2.9 `presence`

どうやら `Object#blank?` メソッドの返り値が `true` だとバリデーションに失敗するようですね。
（ `presence: true` の場合）
というわけで Rails コンソールで確かめてみます。

```ruby
[1] pry(main)> false.blank?
=> true
[2] pry(main)> nil.blank?
=> true
```

つまり、 `false.blank?` の返り値が `true` なので、 `presence: true` のバリデーションが失敗しているということですね。
これでは `nil` だけでなく `false` の値も許容しないバリデーションとなっているので、意図しない挙動となってしまっています。

### `Object.blank?` メソッド

このメソッドは Ruby の `Object` クラスには存在せず、Rails の **Active Support** による拡張機能のひとつのようです。

https://railsguides.jp/active_support_core_extensions.html#blank-questionmark%E3%81%A8present-questionmark

では `false` のクラスである `FalseClass` が継承しているクラスを `ancestors` メソッドで確認してみましょう。

```ruby
[1] pry(main)> false.class
=> FalseClass
[2] pry(main)> FalseClass.ancestors
=> [ActiveSupport::ToJsonWithActiveSupportEncoder,
 FalseClass,
 JSON::Ext::Generator::GeneratorMethods::FalseClass,
 MessagePack::CoreExt,
 ActiveSupport::Dependencies::RequireDependency,
 ActiveSupport::ToJsonWithActiveSupportEncoder,
 Object,
 PP::ObjectMixin,
 ActiveSupport::Tryable,
 JSON::Ext::Generator::GeneratorMethods::Object,
 DEBUGGER__::TrapInterceptor,
 Kernel,
 BasicObject]
```

`false` は `FalseClass` のインスタンスであり、 `Object` クラスを継承しているので `blank?` メソッドが使用できるというわけですね。

:::details ancestors メソッド

`ancestors` メソッドは親クラスとインクルードしているモジュールを配列にして返してくれる `Module` クラスのインスタンスメソッドです。

https://docs.ruby-lang.org/ja/3.2/method/Module/i/ancestors.html

```ruby
[1] pry(main)> FalseClass.class
=> Class
[2] pry(main)> Class.ancestors
=> [ActiveSupport::DescendantsTracker::ReloadedClassesFiltering,
 Class,
 Module,
 Module::Concerning,
 ActiveSupport::Dependencies::RequireDependency,
 ActiveSupport::ToJsonWithActiveSupportEncoder,
 Object,
 PP::ObjectMixin,
 ActiveSupport::Tryable,
 JSON::Ext::Generator::GeneratorMethods::Object,
 DEBUGGER__::TrapInterceptor,
 Kernel,
```

`FalseClass` は `Class` クラスのインスタンスであり、 `Class` クラスは `Module` クラスを継承しているので、 `ancestors` メソッドが使用できるということですね。
ちなみに、 `class` メソッドは `Class` クラスのメソッドではなく、 `Object` クラスのインスタンスメソッドです。

https://docs.ruby-lang.org/ja/3.2/method/Object/i/class.html

:::

# 解決方法

こちらも [Rails ガイド](https://railsguides.jp/active_record_validations.html#presence)に記載されていました。

> 真偽値の存在をチェックしたい場合は、以下のいずれかを使う必要があります。
>
> ```ruby
> # 値は true か false でなければならない
> validates :boolean_field_name, inclusion: [true, false]
> # 値は nil であってはならない、すなわち true か false でなければならない
> validates :boolean_field_name, exclusion: [nil]
> ```
>
> これらのバリデーションのいずれかを使うことで、値が決して nil にならないようにできます。nil があると、ほとんどの場合 NULL 値になります。<br>
> Rails ガイド -> Active Reocrd バリデーション -> 2. バリデーションヘルパー -> 2.9 `presence`

### `inclusion`

```diff ruby
class Wish < ApplicationRecord
- validates :granted, presence: true
+ validates :granted, granted: [true, false]
end
```

`inclusion` を使用することで `granted` には `true` or `false` が含まれているかを検証してくれます。

https://railsguides.jp/active_record_validations.html#inclusion

### `exclulsion`

```diff ruby
class Wish < ApplicationRecord
- validates :granted, presence: true
+ validates :granted, exclusion: [nil]
end
```

`exclusion` は `inclusion` と逆で、 `granted` には `nil` が**含まれていない**ことを検証してくれます。

https://railsguides.jp/active_record_validations.html#exclusion

##### 補足

PostgreSQL の `boolean` 型には `true`, `false`, `unknown`（SQL でいう `null`）の 3 つの状態しか保持しないようです。
なので上記の２種類のバリデーションのどちらでも対応できるということですね。

https://www.postgresql.org/docs/14/datatype-boolean.html

# おわりに

今回のバリデーションの目的はデータベースに存在する `boolean` 型のカラムに `null` が入らないようにバリデーションを設定するのが目的なので、個人的には `exclusion` の方が直感的だなと感じました。

お気づきの点があればコメントいただけると幸いです。
最後までお読みいただき、ありがとうございました。

# 参考記事

https://railsguides.jp/active_record_validations.html#presence
https://api.rubyonrails.org/v7.1/classes/Object.html#method-i-blank-3F
https://docs.ruby-lang.org/ja/3.2/class/Object.html
https://thoughtbot.com/blog/how-to-validate-the-presence-of-a-boolean-field-in-a-rails-model
