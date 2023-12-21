---
title: 任意のページにPV数(訪問者数)を計測、表示する【rails】
tags:
  - Rails
private: false
updated_at: '2020-04-07T09:50:09+09:00'
id: 69f8e3e68ad834b62f6b
organization_url_name: null
slide: false
ignorePublish: false
---
#概要
webページを作成してリリースしたあと、大体何人見に来てくれているのか気になりますよね！以下のような「あなたは<51>人目の訪問者です」のような記述です。
<div align="center">
<img width="400" alt="PV数.JPG" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/e4f9a3a6-5c84-40f3-c50f-5831987bad3a.jpeg">
</div>

アクセス回数などで調べる方法もありますが、同一人物が何度もアクセスしたら、延べ人数として重複して数えられてしまいます。これでは、より多くの人に見てもらいたいという意味では、正確な数字が測りにくくなります。また、他のQiita記事を見ると、impressionistとうgemで計測できるとあります。しかし、これはあくまで投稿詳細ページのPV数しか計測できず(投稿のidと紐づけられているため)任意のページのPV数が取得できません。そこで、ここでは、この二つの問題を"ある程度"解決できるIPアドレスを記録するという方法で訪問者数を計測していきます。(ログイン機能があれば、User数で知ることもできますね！)

# IPアドレスとは
IPアドレスは機種ごと(パソコンやスマホ)に割り振られる固有の番号と知っている方も多いと思います。しかし、実際はやや異なるみたいです。世界で一つしかないIPアドレス(グローバルIP)はルーター(各家庭や各施設に一つあるインターネットとの出入口の役割を果たすもの)が持っており、その家庭内でのみ一意性が保たれるIPアドレス(プライベートIP)を各機種が持っているというのが一般的みたいです。後述のようにrailsでIPアドレスを取得する場合は、基本的にこのルーターのグローバルIPアドレスが取得されてしまうようです。したがって例えば、家庭内で同じWi-fiを使っている人同士は全て同一のIPアドレスとなります。何人いようが一人と計測されてしまうわけですね。もしくは、Wi-fiや機種を変えたり、インターネット環境を変えると、同一人物でも異なるIPアドレスが取得されてしまいます。このようにIPアドレスで訪問者数をカウントする場合は、"ある程度"の概算にならざるを得ないという事情があります。しかし、アクセス回数で計測する際の、製作者が気になって何度もwebページを見に来たり、利用者がPV数を誤魔化そうとその場で大量にアクセスしまくるといったPV数の不正確性に比べると、まだメリットは大きいのかなと個人的には感じています。

#ipアドレス計測用のモデル(テーブル)作成
上記の事情を理解して頂けた方は、さっそく実装に入りましょう。まず、以下のようにipアドレスを計測するモデル(テーブル)を作成します。ターミナル(コマンドプロンプト)に入力してください。(プロダクトフォルダーの階層に移動することを忘れずに！)

```
$ rails generate model See ip:string
```

ここではSeeモデルという名前で作成します。(※データ型は**string**型にしてください。ipアドレスはstring型で取得されるためです)するとmigrationファイルに

```ruby:db/migration/******.rb
class CreateSees < ActiveRecord::Migration[*.*]
  def change
    create_table :sees do |t|
      t.string :ip

      t.timestamps
    end
  end
end
```
が作成されました。以下を実行してテーブルにカラムを適用させましょう。(ターミナル)

```
$ rails db:migrate
```

これにてIPアドレスを計測できるデータベースは完成しました。

#Controllerの中身変更
次にControllerの中身に入ります。訪問者数を計測&表示したいviewに対応するアクションの中身を以下のように変えてください。ここでは例としてtweetsコントローラーのindexアクションを取り上げます。

```ruby:app/views/tweets/index.html.erb
def index
  @see = See.find_by(ip: request.remote_ip) 
    if @see 
      @tweets = Tweet.all
    else 
      @tweets = Tweet.all
      See.create(ip: request.remote_ip)
    end
end
```

``` request.remote_ip ```で前述のようにアクセス者のIPアドレスが取得できます。何をしているかというと、まずseesテーブルで、アクセスした利用者のipアドレスと等しいipのレコードがあるか探し、あれば@seeに代入します。次に、もし、@seeが存在していれば(利用者が過去アクセスしたことがあれば)そのままviewページに受け渡す変数を書くだけです。もし、@seeが空だったら(利用者が初訪問の場合は)viewページに変数が受け渡されるとともに、seesテーブルのipカラムに、新たに利用者のIPアドレスが追加され保存されます。(検索機能などを付けている場合は、以下のように条件分岐が入れ子になります！)

```ruby:app/views/tweets/index.html.erb
def index
  @see = See.find_by(ip: request.remote_ip)
    if @see 
      if params[:search]
        #部分検索
        @tweets = Tweet.where("content LIKE ? ",'%' + params[:search] + '%')
      else
        @tweets= Tweet.all
      end
    else 
      See.create(ip: request.remote_ip)
      if params[:search]
        #部分検索
        @tweets = Tweet.where("content LIKE ? ",'%' + params[:search] + '%')
      else
        @tweets= Tweet.all
      end
    end
end
```

#viewページで訪問者数を表示する
これは簡単で以下の一文を任意の場所に追加するだけです。

``` 
あなたは<<%= See.count %>>人目の訪問者です 
```
これにより、過去訪問してきた、重複の無いIPアドレスの合計数が表示されます。(厳密には前述の理由と、今の新訪問者が未カウントなので、正確性を欠きますが。)今の"新"訪問者をカウントするなら

```
あなたは<<%= See.count + 1 %>>人目の訪問者です
```
このようになりますね！

# 最後に注意点
ローカル環境で開発する場合、この``` request.remote_ip ```では、ローカルでデフォルトで用意されている"::1"というものがipアドレスとして取得されてしまいます。開発中にちゃんとできているか実験したくても、それは不可能です。言い換えると、開発環境においては、たとえ異なるパソコンでも全て同じデフォルトのIPアドレスになってしまうわけですね。ご注意ください。**リリースしてからのお楽しみで！**
