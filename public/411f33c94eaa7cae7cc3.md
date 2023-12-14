---
title: 簡易パスワード機能(削除制限)
tags:
  - Rails
private: false
updated_at: '2020-04-17T16:22:37+09:00'
id: 411f33c94eaa7cae7cc3
organization_url_name: null
slide: false
ignorePublish: false
---
#簡易パスワード機能とは？
webサイト作成にあたって、ログイン機能なしで投稿機能を作りたいという要求もあるかと思います。しかし、誰でも自由に投稿できてしまうと不都合が生じます。削除、編集機能を付ける場合、勝手に他人の投稿をいじることができてしまうためです。したがって、これらの機能は投稿した人のみに制限する必要があります。ここでは、簡易パスワードを投稿者が設定し、ロックをかける仕様を実装したいと思います。一例として投稿(Tweet)に関して、削除キー(半角数字4桁)を設定していきます。
**※投稿機能は一通り実装できている前提で話します。ルーティングはresourcesなどで作っておいてください！**

#概要
1. Tweetテーブルにdeletekeyカラム(string)を追加する
2. modelで、バリデーション(deletekeyのルール)を設定する
3. 投稿フォームにdeletekeyの項目を追加する
4. 投稿詳細ページに、deletekeyの入力フォームと削除ボタンをセットで設定する
5. tweetsコントローラーのdestroyアクションにて、削除ボタンからパラメーターで送信されてきたkeyと、設定されたdeletekey(レコードに保存されているdeletekey)が一致したら、削除するという条件分岐を書く
6. エラー文

#1. deletekeyカラムの追加

ターミナル(コマンドプロンプト)上で以下を実施。(Tweetsテーブルは、必要があれば自分のテーブル名に変更してください)
**※データ型は必ずstring型にしてください。formで送信されるデータは、たとえ数字でも文字列型(string含む)だからです**

```
rails generate migration AddDeletekeyToTweets deletekey:string
```

```rails db:migrate```を忘れずに。これによりカラムを追加できました。

#2. modelでvalidationを設定する

tweet.rbに以下を追加

```models/tweet.rb
validates :deletekey, length: { is: 4}, presence: true
```
長さは4文字、未入力はダメ、というルールを設定します。これは任意にルールを設定してもOKです。

#3. 投稿フォーム
投稿フォーム(newページ)に以下を追加してください。(もちろんform_forとsubmitの間です)

```views/tweets/new.html.erb
<%= f.label :"削除キー(半角数字4桁)" %>
<%= f.number_field :deletekey, :size=> "4" %>
```

長さ4文字で、先ほど作ったdeletekeyカラムに保存できるようにしました。(number_fieldですが、文字列型(string型含む)で送られます。)

※tweetsコントローラーにて、ストロングパラメーターを設定している場合は、privateの中にdeletekeyカラムを追加するのを忘れずに。

#4. 投稿詳細ページ
deletekeyの入力フォームと削除ボタンをセットで実装します。削除ボタンを置きたい場所に以下を追加してください。

```views/tweets/show.html.erb
<%= form_tag({controller:"tweets",action:"destroy"}, method: :delete) do %>
  <%= number_field_tag :key, @word, placeholder: "削除キー(半角数字4桁)" %>
  <%= submit_tag '削除する' %>
<% end %>
```

これにより、tweetsコントローラーのdestroyアクションに```key```という名前で入力内容(半角数字4桁)が送信されます。

※途中の@wordというのは、form_tagにplaceholder(フォーム中に薄く表示される情報)を置くために仕方なく便宜的に書いているだけです。

#5. tweetsコントローラー
destroyアクションを以下のように書いてください。

```tweets_controller.rb
def destroy
  @tweet = Tweet.find(params[:id])
  if @tweet.deletekey == params[:key]
    @tweet.destroy
    redirect_to action: "index"
  else
    redirect_back(fallback_location: root_path)
    flash[:notice] = "削除キーが違います"
  end
end
```
まず、@tweetに投稿詳細ページのtweetレコードを代入します。そのレコードに保存されているdeletekeyがparamsで送られてきた```key```の中身と等しければ(等しいかどうかを判断するときは```==```を書きましょう)、tweet削除、等しくなければ、削除せず投稿詳細ページに戻る、という内容になっています。

※ちなみにもしデータ型が異なる場合、演算子(一致を表す```==```や、大小関係の<>など)で比較ができません。なので、formで送られている文字列型に統一する必要があったわけです。

等しくない場合、エラー文を表示できると親切ですよね。

#6. エラー文
上記のコントローラーにて、```flash[:notice] = "削除キーが違います"```という記述があったかと思います。これは、destroyアクションを実行してdeletekeyが一致しない場合、投稿詳細ページにエラー文を表示してくれます。削除ボタンの下あたりに以下を追加しましょう

```views/tweets/show.html.erb
<%= flash[:notice] %>
```

これにて、良い感じになったかと思います。
