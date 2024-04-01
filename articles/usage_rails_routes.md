---
title: "rails routes コマンドの使い方"
emoji: "🛤️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails"]
published: true
---

# はじめに

みなさん、Rails のルーティングを確認したい時ってどうしてますか？
`resources` は便利ですけど、実際にルーティングを記述するわけではないので、どんなルーティングが出来ているか確認したいことがあると思います。
そんな時によく使う `rails routes` コマンドについてまとめてみました。

# rails routes コマンドとは

ターミナルでルーティング一覧を確認できるコマンドです。
`config/routes.rb` ファイルに記載された順番でルーティング出力されます。

## 使い方

以下のような `config/routes.rb` ファイルがあるとします。

```
Rails.application.routes.draw do
  root 'welcome#index'

  get 'about', to: 'pages#about'
  get 'contact', to: 'pages#contact'

  resources :articles do
    resources :comments
  end

  namespace :admin do
    resources :dashboard
  end

  get 'search', to: 'search#index'
end

```

ターミナルで以下のコマンドを実行します。

```
bin/rails routes
```

実行結果（一部省略）

```
                Prefix Verb   URI Pattern                                         Controller#Action
                  root GET    /                                                   welcome#index
                 about GET    /about(.:format)                                    pages#about
               contact GET    /contact(.:format)                                  pages#contact
      article_comments GET    /articles/:article_id/comments(.:format)            comments#index
                       POST   /articles/:article_id/comments(.:format)            comments#create
   new_article_comment GET    /articles/:article_id/comments/new(.:format)        comments#new
  edit_article_comment GET    /articles/:article_id/comments/:id/edit(.:format)   comments#edit
       article_comment GET    /articles/:article_id/comments/:id(.:format)        comments#show
                       PATCH  /articles/:article_id/comments/:id(.:format)        comments#update
                       PUT    /articles/:article_id/comments/:id(.:format)        comments#update
                       DELETE /articles/:article_id/comments/:id(.:format)        comments#destroy
              articles GET    /articles(.:format)                                 articles#index
                       POST   /articles(.:format)                                 articles#create
           new_article GET    /articles/new(.:format)                             articles#new
          edit_article GET    /articles/:id/edit(.:format)                        articles#edit
               article GET    /articles/:id(.:format)                             articles#show
                       PATCH  /articles/:id(.:format)                             articles#update
                       PUT    /articles/:id(.:format)                             articles#update
                       DELETE /articles/:id(.:format)                             articles#destroy
 admin_dashboard_index GET    /admin/dashboard(.:format)                          admin/dashboard#index
                       POST   /admin/dashboard(.:format)                          admin/dashboard#create
   new_admin_dashboard GET    /admin/dashboard/new(.:format)                      admin/dashboard#new
  edit_admin_dashboard GET    /admin/dashboard/:id/edit(.:format)                 admin/dashboard#edit
       admin_dashboard GET    /admin/dashboard/:id(.:format)                      admin/dashboard#show
                       PATCH  /admin/dashboard/:id(.:format)                      admin/dashboard#update
                       PUT    /admin/dashboard/:id(.:format)                      admin/dashboard#update
                       DELETE /admin/dashboard/:id(.:format)                      admin/dashboard#destroy
                search GET    /search(.:format)                                   search#index
```

これでルーティングの一覧が出力されました。
１番上に Prefix や Verb などの単語が並んでいるので１つずつ見ていきます。

### Prefix

prefix とは日本語で接頭辞、接頭語という意味です。
ルーティングの名前のようなもので、 `_path` や `_url` などのヘルパーメソッドの頭につけて使うことができます。

例えば、`articles_path` は `articles`コントローラの `index` アクションを実行する `/articles` を参照します。

Prefix は基本的に自動で生成されますが `:as` オプションで明示的に指定することもできます。
https://railsguides.jp/routing.html#%E5%90%8D%E5%89%8D%E4%BB%98%E3%81%8D%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0

### Verb

HTTP メソッド（GET や POST など）のことです。
URL が同じでも HTTP メソッドが異なる場合は別のルーティングとなります。

### URI Pattern

リクエストされた URI[^1]パターンと一致するパターンを表します。
ブラウザの検索バーに表示されている URL と一致します。
`:id` や `:article_id` には任意の値が入ります。詳しくは後述します。

### Controller#Action

そのルーティングに対応するコントローラとアクションのことです。

例えば、 `comments#show` というのは `comments` コントローラの `show` アクションを実行するよって意味です。

## オプション

これでルーティングを確認することはできたと思います。
しかし、 `rails routes` コマンドを実行するとすべてのルーティングが出力されます。

なので、ルーティングが増えていくと、確認したいルーティングを探すのが少し面倒です。
そんな時のために `routes` コマンドには便利なオプションがあります。

### -c

指定したコントローラに対応するルーティングだけを出力します。
引数にはさまざまな方法でコントローラを指定できます。

```
bin/rails routes -c comments
bin/rails routes -c admin/dashboard
bin/rails routes -c Comments
bin/rails routes -c CommentsController
```

### -g

Prefix、HTTP verb、URI Pattern のいずれかに部分マッチするルーティングが出力されます。
「POST メソッドのルーティングだけを確認したい」などの特定の HTTP メソッドで絞り込みたい時や、 `id` を持つルーティングだけを確認したいときに使いました。

```
bin/rails routes -g POST
bin/rails routes -g id
```

### -E

出力するフォーマットを以下のように切り替えます。

```
--[ Route 1 ]---------------------------------------------
Prefix            | root
Verb              | GET
URI               | /
Controller#Action | welcome#index
--[ Route 2 ]---------------------------------------------
Prefix            | about
Verb              | GET
URI               | /about(.:format)
Controller#Action | pages#about
--[ Route 3 ]---------------------------------------------
Prefix            | contact
Verb              | GET
URI               | /contact(.:format)
Controller#Action | pages#contact
```

## :id とは

URI Pattern の部分に `:id` や `:article_id` などが含まれるルーティングがあります。
これらには任意の値が入ります。なので `:id` などを自分で明示的に指定する必要があります。
具体的には、ヘルパーメソッドの引数に `:id` が推測できるようなデータを指定します。

指定方法は以下の２つがあります。

1. `:id` に対応するデータを直接引数に指定する。
   以下のような記述が可能です。

```
# 5 という数字は適当なものです。実際には params から取得したものを使用したりします。
article_path(id: 5)
article_path(5)
ariticle_path(@article.id)
```

2. モデルのインスタンスを引数に指定する。
   もし `@article` が `:id` を持つモデルのインスタンスなら、`article_path(@article)` というように渡すことができます。
   この場合はヘルパーメソッドがインスタンスから `:id` に当たる部分を自動的に推測してくれます。
   （もし、 `article_path` とすると `:id`に当たる部分を自動的に推測できないので、 `UrlGenerationError` が発生します）

# おわりに

理解が間違っている点などあれば、コメントいただけると幸いです。
最後までお読みいただきありがとうございました。

## 補足

この記事では詳しく触れていないですが、ターミナルで `rails routes` コマンドを実行する以外にもルーティングを確認する方法があります。
ブラウザで http://localhost:3000/rails/info/routes にアクセスすると、以下の画像のようにルーティング一覧が表示されます。

![](/images/usages_rails_routes/routes_sample.png)

僕はあまり使わないので割愛させていただきます。
ブラウザの方で便利な使い方があれば、コメントで教えていただけるとありがたいです。

[^1]: URI とはリソースの位置や名前を指定するための識別子です。

# 参考

https://railsguides.jp/routing.html#%E6%97%A2%E5%AD%98%E3%81%AE%E3%83%AB%E3%83%BC%E3%83%AB%E3%82%92%E4%B8%80%E8%A6%A7%E8%A1%A8%E7%A4%BA%E3%81%99%E3%82%8B
https://ichigick.com/urlgenerationerror-no-route-matches-missing-required-keys-id/
