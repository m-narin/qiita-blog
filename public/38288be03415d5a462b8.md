---
title: Impressionistを用いてPV数(閲覧数)取得~Rails
tags:
  - Rails
  - impressionist
private: false
updated_at: '2021-06-12T10:45:49+09:00'
id: 38288be03415d5a462b8
organization_url_name: null
slide: false
ignorePublish: false
---
# 目標

この記事ではImpressionistというgemを用い、投稿のPV(page view)数取得とPV数ランキングを実装します。

[詳しくはこちら~](https://github.com/charlotte-ruby/impressionist)

# 開発環境
・Ruby: 2.6.2
・Rails: 6.0.3
・OS: windows

# 前提

Tweetモデルにて、投稿機能一式作成済み。

# 実装

### 1.`impressionist`を導入

gemfileに下記一行を追加します。

```ruby:Gemfile
gem 'impressionist'
```

```terminal:ターミナル
$ bundle install
```

次にPV数をカウントするテーブルを作成します。

```terminal:ターミナル
$ rails g impressionist
```

```terminal:ターミナル
$ rails db:migrate
```

以下のようなimperssionsテーブルができます。

```ruby:schema.rb
create_table "impressions", force: :cascade do |t|
  t.string "impressionable_type"
  t.integer "impressionable_id"
  t.integer "user_id"
  t.string "controller_name"
  t.string "action_name"
  t.string "view_name"
  t.string "request_hash"
  t.string "ip_address"
  t.string "session_hash"
  t.text "message"
  t.text "referrer"
  t.text "params"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
  t.index ["controller_name", "action_name", "ip_address"], name: "controlleraction_ip_index"
  t.index ["controller_name", "action_name", "request_hash"], name: "controlleraction_request_index"
  t.index ["controller_name", "action_name", "session_hash"], name: "controlleraction_session_index"
  t.index ["impressionable_type", "impressionable_id", "ip_address"], name: "poly_ip_index"
  t.index ["impressionable_type", "impressionable_id", "params"], name: "poly_params_request_index"
  t.index ["impressionable_type", "impressionable_id", "request_hash"], name: "poly_request_index"
  t.index ["impressionable_type", "impressionable_id", "session_hash"], name: "poly_session_index"
  t.index ["impressionable_type", "message", "impressionable_id"], name: "impressionable_type_message_index"
  t.index ["user_id"], name: "index_impressions_on_user_id"
end
```

### 2.Tweetsテーブルにカウント数のカラムを追加

```terminal:ターミナル
$ rails g migration AddImpressionsCountToTweets impressions_count:integer
```

以下のmigrationファイルに```default: 0```を追記します。

```ruby:~_add_impressions_count_to_tweets.rb
class AddImpressionsCountToTweets < ActiveRecord::Migration[6.0]
  def change
    # 「default: 0」を追記
    add_column :tweets, :impressions_count, :integer, default: 0
  end
end
```

```terminal:ターミナル
$ rails db:migrate
```

### 3.モデルを編集

```ruby:tweet.rb
# 追記
is_impressionable counter_cache: true
```

`is_impressionable`
➡︎ Tweetモデルで`impressionist`を使用できるようにします。

`counter_cache: true`
➡︎ impressions_countカラムがupdateされるようにします。

### 4.コントローラーを編集

```ruby:tweets_controller.rb

def index
  @tweets = Tweet.all
  @rank_tweets = Tweet.order(impressions_count: 'DESC') # ソート機能を追加
end

def show
  @tweet = User.find(params[:id])
  impressionist(@tweet, nil, unique: [:ip_address]) # 追記
end
```

`Tweet.order(impressions_count: 'DESC')`
➡︎ tweet一覧をPV数の多い順に並び替える。

`impressionist(@user, nil, unique: [:ip_address])`
➡︎ tweet詳細ページにアクセスするとPV数が1つ増える。

**※自主的にPV数を伸ばす事が出来ないように、今回はip_addressにてPV数をカウントします。**

**※rails sしてlocalhostで試す場合、ip_addressはデフォルトの::1が入るため、PV数は1を超えません。しかしデプロイ後はきちんと動作することが確認できていますので、ご安心ください。**

### 5.ビューを編集

```erb:app/tweets/index.html.erb

<h3> 投稿一覧 </h3>
<% @tweets.each do |t| %>
  <%= t.~ %>
  
  # PV数
  <%= t.impressions_count %>
<% end %>


<h3> PV数ランキング </h3>
<% @rank_tweets.each do |t| %>
  <%= t.~ %>
  
  # PV数
  <%= t.impressions_count %>
<% end %>

```
