---
title: ログインなしでもいいね機能を利用できるようにする
tags:
  - Rails
  - いいね
  - login
private: false
updated_at: '2020-05-06T23:54:09+09:00'
id: c4e310e2501a5dd58268
organization_url_name: null
slide: false
ignorePublish: false
---
#目的
今回はログインなしでも閲覧者がいいね機能を利用できるように実装します。しかし、大きな注意点が1つあります。それはユーザーが無限回いいねを押せてしまうという点です。これでは正当な評価ができませんよね。ログイン機能があれば、それぞれのログインユーザーがそれぞれのツイートに対していいね1つのみ、という実装ができます。これに近い制限をかける必要性が出てきます。ここでは、例としてIPアドレスで制限する方法を実装します。

**※一通り投稿機能は実装できている前提で話します。上記のようないいね機能をゼロから作っていきます。**

※IPアドレスとは何か？については、以下のページをご覧ください。
https://qiita.com/MandoNarin/items/69f8e3e68ad834b62f6b

#概要
1. Likeモデル(テーブル)を作成する
2. アソシエーション
3. ルーティング
4. 投稿詳細ページ
5. likesコントローラー
6. エラー文
7. まとめ

#1. Likeモデルの作成
以下をターミナルで実行してください

```
rails g model like tweet:references
```
tweet:referenceのおかげで、like.rbにはすでに```belongs_to :tweet```の記述が、likesテーブルにはすでに```tweet_id```が入っています(migrationファイル中には明記されないので注意してください)。次にlikesのmigrationが作成されていると思うので、以下のようにipアドレスのカラム(string型)を追加してください。

```ruby
class CreateLikes < ActiveRecord::Migration[5.1]
  def change
    create_table :likes do |t|
      t.references :tweet, foreign_key: true
      t.string :ip 
      t.timestamps
    end
  end
end
```

この状態で```rails db:migrate```を実行してテーブルを作成しましょう。

#2. アソシエーション
tweet.rbに以下を追加しましょう

```models/tweet.rb
has_many :likes, dependent: :destroy
```

TweetとLikeの1対多の関係を実現しています。

**※参考**
関係データベースとは？
https://qiita.com/MandoNarin/items/7bd624801940c999d667

#3. ルーティング
ルーティングに以下を追加してください。

```config/routes.rb
  resources :tweets do
    resources :likes, only: [:create]
  end
```
それぞれの投稿のURLの中に、いいね機能を入れ子にしています。

#4. 投稿詳細ページ
以下のいいねボタンを任意の場所に置いてください。

```views/show.html.erb
<%= button_to 'いいね', tweet_likes_path(@tweet) %>
```
これにより、@tweetのレコード情報がlikesコントローラーのcreateアクション(後述)に送信されます。

#5. likesコントローラー
まず、likesコントローラーを作成しましょう。
ターミナル(コマンドプロンプト)にて以下を実行して下さい。

```
rails g controller likes
```
次に、likesコントローラーのcreateアクションを以下のように書いてください。

```controllers/likes_controller.rb
def create
  @tweet = Tweet.find(params[:tweet_id])
  @alreadylike = Like.find_by(ip: request.remote_ip, tweet_id: params[:tweet_id])
  if @alreadylike
     redirect_back(fallback_location: root_path)
     flash[:notice] = "すでにいいねしています"
  else
     @like = Like.create(tweet_id: params[:tweet_id], ip: request.remote_ip)
     redirect_back(fallback_location: root_path)
  end
end
```
まず、@tweetに投稿詳細ページのtweetレコードを代入します。また、@alreadylikeには、Likeのレコード中から、送信者のipアドレス(```request_remote.ip```)と、投稿詳細ページのtweet_idの組み合わせを持つものがあれば代入します。
@alreadylikeが存在すれば(そのipアドレスでそのtweetをすでにいいねしていれば)、ただ元のページに戻るだけ、存在しなければ(そのIPアドレスでそのtweetをいいねしていなければ)、新たにいいねを作成する、といった内容になっています。

#6. エラー文
上記において、```flash[:notice] = "すでにいいねしています"```という記述があるかと思います。これは、その名の通り、likeのcreateアクション実行後、すでにいいねしている場合にredirect先のページで表示されるものです。showページの任意の場所に以下のエラー文を追加しましょう。

```views/show.html.erb
<%= flash[:notice] %>
```
#7. まとめ
ここではIPアドレスでいいねを制限するという仕様を作りましたが、IPアドレスでは完全ではないです。しかし少なくとも、1人がその場で無限回いいねが押せてしまうのを防ぐことはできていると言えます。
**※ipアドレスはローカル環境(開発環境)では全てデフォルトの```::1```が取得されるので、注意してください。**
