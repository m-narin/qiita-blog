---
title: Indexページにいいね機能をつける
tags:
  - Rails
  - いいね
  - index
private: false
updated_at: '2020-06-11T22:39:55+09:00'
id: 850cbcd13f10e047b46b
organization_url_name: null
slide: false
ignorePublish: false
---
#目的
ここでは、indexページにいいね機能を付ける方法とその仕組みを解説したいと思います。railsにおけるいいね機能を解説してくれている多くの記事はshowページのみに対応しています。それをindexページで使う場合、注意点が1つあるのです。

**※showページにおけるいいね機能は実装できている前提で話します。
参考**
https://qiita.com/nojinoji/items/2c66499848d882c31ffa

#概要
1. indexページ
2. 解説

#1. indexページ
indexページにいいね機能を付けるときは、任意の場所に下記の真ん中の5行を追記してください。多くの場合はeach doの中に入るはずです。

```views/tweets/index.html.erb
<% @tweets.each do |t| %>
・
・
  <% if current_user.already_liked?(t) %>
    <%= button_to 'いいねを取り消す', tweet_like_path(id: t.id, tweet_id: t.id), method: :delete %>
  <% else %>
    <%= button_to 'いいね', tweet_likes_path(id: t.id, tweet_id: t.id) %>
  <% end %>
・
・
<% end %>
```

#2. 解説
showページに書かれたいいね機能とindexページに書かれたいいね機能の大きな違いは、パラメーターとして送られる変数の違いです。

**showページ**
```<%= button_to 'いいねを取り消す', tweet_like_path(@tweet) %>```

**indexページ**
```<%= button_to 'いいねを取り消す', tweet_like_path(id: t.id, tweet_id: t.id) %>```

likeモデルは、tweet_idを保存する必要があります。showページにおいてはURLの中にtweet_idがすでに埋め込まれているので、@tweetを送信してあげれば、tweet_idのパラメーターを送ることができます。

しかし、indexページでは、tweet_idはeach doで書かれた```t```から引っ張ってくる必要があります。これを次のように実現します。
①```id: t.id```で本来URLに埋め込まれるidの部分にt.idを代入する
②```tweet_id: t.id```でパラメーターのtweet_idを指定してあげる。

このように、2段階の工程として2つのパラメータを記述してあげれば、とりあえず辻津を合わせることができます。

①の記述がないと例えばlikeのdestroyアクションで下記のようなエラーが出ます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/c634d3a1-f71c-fada-0975-c893e8a3798c.png)
これは、indexページのURLの中にtweet/**:id**の記述がないという意味です。destroyアクションはtweetのidがURL中に存在しないとエラーを起こす仕様になっているんですね～。これは多分、ルーティングでtweetの中にlikeを入れ子にしていることが関係してるんだと思います。

(ちなみにlikeのcreateアクションはpathの部分を```(t)```のように書いても動きます。この違いは何でなんでしょう？それとcommentをindexページで扱う際にも同様のことに気を付ける必要がありそうな？)
