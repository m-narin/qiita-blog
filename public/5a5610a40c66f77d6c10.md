---
title: '【タグ機能完全版】 ~ Rails '
tags:
  - Ruby
  - Rails
private: false
updated_at: '2021-10-06T03:05:59+09:00'
id: 5a5610a40c66f77d6c10
organization_url_name: null
slide: false
ignorePublish: false
---
#はじめに
この記事ではいわゆるタグをつけて投稿し、そのタグによって検索できる仕組みを実装しています。
前提としてTweetモデルにて投稿機能を扱う状態からタグ機能を追加していきます。

実装内容一覧
1. タグモデルの作成
2. モデルのアソシエーションの設定
3. デフォルトのタグをseedファイルで用意する
4. Tweet投稿の際、タグの投稿を可能にする
5. コントローラーでタグ配列を受け取れるようにする
6. 各Tweetにつけられたタグを表示する
7. 複数タグの検索をできるようにする
8. 新たなタグの追加をできるようにする

# 1. タグモデルの作成
まず、今回使用するモデルは以下の３つになります。

|Tweets|Tweet_tag_relations|Tags| 
|---|---|---|     
|id|id|id|
|title|tweet_id|name
|body|tag_id|

- Tweets
    - 投稿テーブル
- Tags
    - タグに関するテーブル
- Tweet_tag_relations 
    - 投稿とタグを紐づけるための中間テーブル

では、さっそくモデルを作成していきます。

Tagモデルのマイグレーションファイル生成

```
$ rails g model Tag name:string
```

Tweet_tag_relationモデルのマイグレーションファイル生成

```
$ rails g model Tweet_tag_relation tweet:references tag:references
```

DBに反映させます。

```
$ rails db:migrate
```

# 2. モデルのアソシエーションの設定

```ruby:tweet_tag_relation.rb
class TweetTagRelation < ApplicationRecord
  belongs_to :tweet
  belongs_to :tag
end
```

```tweet.rb
class Tweet < ApplicationRecord

  # 以下を追記
  #tweetsテーブルから中間テーブルに対する関連付け
  has_many :tweet_tag_relations, dependent: :destroy
  #tweetsテーブルから中間テーブルを介してTagsテーブルへの関連付け
  has_many :tags, through: :tweet_tag_relations, dependent: :destroy
end
```

```tag.rb
class Tag < ApplicationRecord

  # 以下を追記
  #Tagsテーブルから中間テーブルに対する関連付け
  has_many :tweet_tag_relations, dependent: :destroy
  #Tagsテーブルから中間テーブルを介してArticleテーブルへの関連付け
  has_many :tweets, through: :tweet_tag_relations, dependent: :destroy
end
```

# 3. デフォルトのタグをseedファイルで用意する

db/seed.rbファイル中に以下のようにデフォルトのタグを記述します。

```db/seeds.rb
Tag.create([
  { name: 'タグ1' },
  { name: 'タグ2' },
  { name: 'タグ3' },
  { name: 'タグ4' },
  { name: 'タグ5' }
])
```

以下のコマンドを打つと、tagsテーブルのレコードに初期のタグを保存できます。

```
$ rails db:seed
```

# 4. Tweet投稿の際、タグの投稿を可能にする

tweetのform_forの中で以下を追記します。

``` erb:tweets/new.html.erb
<%= form_for @tweet do |f| %>

    <!-- 以下を追記 -->
    <div class='form-group'>
        <%= f.collection_check_boxes(:tag_ids, Tag.all, :id, :name) do |tag| %>
            <div class='form-check'>
                <%= tag.label class: 'form-check-label' do %>
                    <%= tag.check_box class: 'form-check-input' %>
                    <%= tag.text %>
                <% end %>
            </div>
        <% end %>
    </div>
    <!-- ここまで -->

    <%= f.submit '送信' %>
<% end %>
```

入力フォームにタグの選択チェックボックスを表示するのに、` collection_check_boxes ` を使用します。
第一引数の`tag_ids`はタグIDのリストを渡し、複数のタグをtweetに紐づけることができます。
第二引数にはタグオブジェクトのリスト。
第三引数にチェックボックスのvalue、第四引数にタグオブジェクトのnameプロパティをラベル名に指定。

<img width="1226" alt="スクリーンショット 2019-11-27 16.33.11.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/451305/79bdee9b-e822-1b5c-6685-8cd46b6a507e.png">

先ほど投入したタグデータのチェックボックスが表示されていますね。


# 5. コントローラーでタグ配列を受け取れるようにする
private以下のparamsに```tags_ids:[]```を追加

```ruby:tweets_controller.rb
 class TweetsController < ApplicationController

  private

  def article_params
    params.require(:article).permit(:body, tag_ids: [])
  end
end
```

ここで先ほどタグのチェックボックスで設定したtag_idsを許可します。
複数のtag_idが渡ってくるので配列の形式で記述しています。

# 6. 各Tweetにつけられたタグを表示する

チェックしたタグを`show.html.erb`で表示してみたいと思います。

```erb:tweets/show.html.erb
<% @tweet.tags.each do |tag| %>
    <span><%= tag.name %></span>
<% end %>
```

<img width="1226" alt="スクリーンショット 2019-11-27 16.34.31.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/451305/ab4d2a06-4162-81b7-8d00-e781c0c64f20.png">

このように複数のタグを表示できれば完成です。

# 7. 複数タグの検索をできるようにする

投稿一覧ページでタグの検索用の情報を送信する項目を追記します。

```erb:views/tweets/index.html.erb

<%= form_tag({controller:"tweets",action:"index"}, method: :get) do %>
    <% Tag.all.each do |t| %>
        <li><%= check_box :tag_ids, t.name %><%= t.name %></li>
    <% end %>
    <%= submit_tag '検索' %>
<% end %>

```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/e650ef6c-bc07-b14a-0cfb-a4a44bc74389.png)

上記のようなcheck_boxで情報を送信した場合、送信された情報は以下のようになります。

```
{"tag_ids"=>{"タグ1"=>"0", "タグ2"=>"1", "タグ3"=>"1", "タグ4"=>"0", "タグ5"=>"0"}
```

コントローラーに検索アルゴリズムを記述します。上記のデータにループ処理を施し、該当のタグを持つtweetを取得するといった流れで考えることができます。やや難解ですが以下のアルゴリズムを読解することはプログラミング学習で非常にためになると思うので、ぜひ自分で理解してみてください！

### OR検索の場合

```app/controllers/tweets_controller.rb
  def index  

    @tweets = Tweet.all

    #以下を追記
    if params[:tag_ids]
      @tweets = []
      params[:tag_ids].each do |key, value|      
        @tweets += Tag.find_by(name: key).tweets if value == "1"
      end
      @tweets.uniq!
    end
    #ここまで

  end
```

### AND検索の場合

```app/controllers/tweets_controller.rb
  def index

    @tweets = Tweet.all

    # 以下を追記
    if params[:tag_ids]
      @tweets = []
      params[:tag_ids].each do |key, value|
        if value == "1"
          tag_tweets = Tag.find_by(name: key).tweets
          @tweets = @tweets.empty? ? tag_tweets : @tweets & tag_tweets
        end
      end
    end
    #ここまで

  end
```

# 8. 新たなタグの追加をできるようにする

簡易的に投稿一覧ページでタグの追加をできるようにします。

```erb:views/tweets/index.html.erb

<!-- 以下を追記 -->
<%= form_tag({controller:"tweets",action:"index"}, method: :get) do %>
    <%= text_field_tag :tag %>
    <%= submit_tag 'タグを追加' %>
<% end %>
<!-- ここまで -->

```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/c5f39d8a-f181-003e-cbeb-11f813c9c9b9.png)


上記でタグのパラメータを送信し、そのパラメータがあればindexアクション内でtagsテーブルに保存します。

```app/controllers/tweets_controller.rb
  def index

    # 以下を追記
    if params[:tag]
      Tag.create(name: params[:tag])
    end
    #ここまで

  end
```


# 最後に
これにて包括的なタグ機能を実装することができました。難しい部分もあったと思いますが、パラメータの理解やアルゴリズムにいついて勉強しがいのある項目です。頑張りましょう！
