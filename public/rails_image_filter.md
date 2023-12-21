---
title: 画像有り投稿を検索する(画像有フィルター)
tags:
  - Rails
private: false
updated_at: '2020-04-14T02:45:50+09:00'
id: 039d9a37ce06a9986007
organization_url_name: null
slide: false
ignorePublish: false
---
#実装したいこと
画像が投稿されている投稿に絞って一覧表示できるようにします。(imageカラムの中身が存在するレコード一覧を取ってくる)

#indexページ
indexページの適当な場所に以下を追記します。

```tweets/index.html.erb
  <%= form_tag({controller:"tweets",action:"index"}, method: :get) do %>
  画像有りフィルター<%= radio_button_tag("image", "image") %>
  <%= submit_tag '🔎'  %>
  <% end %>
```

これは以下のようになります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/d7d267c3-d629-f7cb-2125-2755cf839015.png)

ラジオボタンをクリックして、送信すると、tweetsコントローラーのindexアクションに「image」というパラメーターが送られます。それを、コントローラーにて後述のような処理をすることで、画像有り投稿の一覧を取得し表示できます。

#コントローラー
tweetsコントローラーのindexアクションに以下のように書きます。

```tweets_controller.rb
def index
  if params[:image]
     @tweets = Tweet.where.not(image: nil)
  else
     @tweets = Tweet.all
  end
end
```
ややこしいですが、**imageカラムの中身が存在しないもの以外のレコード**を取ってこいという命令になってます。(カラムの中身が存在するものを取ってこいというストレートな書き方の情報が、意外と見当たらないんですよね～)

また、検索機能などと組み合わせる場合は、if文が入れ子になります。もしくは、あまりおすすめしませんが、検索機能付きのif文の下に追記する形でも問題なく動くっちゃ動きます。
以下参考

```tweets_contoroller.rb
def index
    if params[:search]
      @tweets = Tweet.where("content LIKE ? ",'%' + params[:search] + '%')
    else
      @tweets= Tweet.all
    end

    if params[:image]
        @tweets = Tweet.where.not(image: nil)
    end
end
```

