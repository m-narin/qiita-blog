---
title: 'シンプル星評価機能 ~ Rails '
tags:
  - HTML
  - CSS
  - Rails
private: false
updated_at: '2021-06-28T11:58:05+09:00'
id: abb9c956b238bceb4a6e
organization_url_name: null
slide: false
ignorePublish: false
---
# 星評価機能

このページでは投稿の星評価機能をシンプルに実装します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/ef3dcce3-8372-f69b-d025-d9eac8922264.png)


１．integer型のstarカラム追加
２．for文で番号の数だけ☆を表示
３．CSSで簡単にデザインを実装

投稿用のテーブルはtweetsテーブルを例に挙げています。

### １．integer型のstarカラム追加

```sh:cmd
$ rails g migration AddStarToTweets star:integer
$ rails db:migrate
```

### ２．for文で番号の数だけ★を表示

※過去に投稿していたレコードのstarカラムはnilになっているので、for文で扱うときにエラーが起こります。これを回避するために一旦過去レコードを削除しましょう。

```sh:cmd
$ rails c
> Tweet.destroy_all
> exit
```

次に、新規投稿ページと一覧表示ページを以下のように記述します。

```erb:views/new.html.erb
<%= form_for(@tweet, :url => { controller:'tweets', action:'create'})do |f| %>

<!-- 中略 -->

<div class="star-field">
    <input id="star5" type="radio" name="tweet[star]" value="5" />
    <label for="star5">★</label>
    <input id="star4" type="radio" name="tweet[star]" value="4" />
    <label for="star4">★</label>
    <input id="star3" type="radio" name="tweet[star]" value="3" />
    <label for="star3">★</label>
    <input id="star2" type="radio" name="tweet[star]" value="2" />
    <label for="star2">★</label>
    <input id="star1" type="radio" name="tweet[star]" value="1" />
    <label for="star1">★</label>
</div>

<!-- 中略 -->

<% end %>

```

ラジオボタンを利用しています。どれか一つが選択されたら、該当の数字が送られます。

```erb:views/tweets/index.html.erb
  <% @tweets.each do |t| %>
      
    <!-- 中略 -->

    <div class="star">
      <% if t.star? %>
        <% for i in 1..t.star do %>
          ★
        <% end %>
      <% end %>
    </div>

  <!-- 中略 -->

  <% end %>
</div>
```

もし`t.star`が存在していたら、t.starの数だけfor文を回して★を表示しています。

### ３．CSSで簡単にデザインを実装
一例として以下のように実装できます。
自分に好みにしていきましょう。

```tweets.scss
// 新規投稿ページ
// https://www.will3in.co.jp/frontend-blog/article/5star-rating-on-contact-form7
.star-field{
    display: flex;
    flex-direction: row-reverse;
    justify-content: right;
}
.star-field input[type='radio']{
    display: none;
}
.star-field label{
    font-size: 50px;
    padding-right: 10px;
    color: #ccc;
    font-size: 30px;
    cursor: pointer;
} 
.star-field label:hover,
.star-field label:hover ~ label,
.star-field input[type='radio']:checked ~ label{
    color: rgb(255, 145, 0);
}

// 一覧表示ページ
.star{
  font-size: 50px;
  color: rgb(255, 145, 0);
}
```
