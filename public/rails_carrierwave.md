---
title: '【Rails】音声投稿機能 ~Cloudinary, CarrierWave'
tags:
  - Rails
  - cloudinary
  - carrierwave
private: false
updated_at: '2020-10-27T23:32:56+09:00'
id: 79b5d787187a829fe214
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
この記事では音声ファイルの投稿機能を実装します。

Cloudinary(外部ストレージ)を使います。
フリープランで月1GBまで利用できます。

# 前提
Tweetモデルにて、テキストでの投稿機能は一通りできている

#まず、、、
Cloudinaryにアカウントを登録してください！
メルアドにて本登録も忘れずに！

https://cloudinary.com/

上記のアカウント取得後、以下の作業に入ってください！

### audioカラムの追加

tweetsテーブルにstring型のaudioカラムを追加します。

```:ターミナル
$ rails generate migration AddAudioToTweets audio:string
```

``` :ターミナル
$ rails db:migrate
```

このaudioカラムは音声ファイルのURL(住所)が書かれます。音声ファイル自体は別の場所に保存されます。

### Index.html.erb

audioファイルを再生するコードを書きます

```erb:app/views/tweets/index.html.erb
    <% @tweets.each do |t| %>

        <%= audio_tag t.audio_url, :controls => true if t.audio? %>

    <% end %>
```

```:controls => true```はコントローラパネルを表示します。
また、audio_tagには以下のオプションがあります。

``:autoplay`` 自動再生
``:loop`` ループ再生

### ストロングパラメーター 

audioパラメータを受け取れるようにストロングパラメーターに記述します。

```ruby:tweets_controller.rb
# 割愛

private
def tweet_params
  params.require(:tweet).permit(:body, :audio)
end
```

### new.html.erb

fileをuploadするコードを書きます。

```erb:app/views/tweets/new.html.erb

<%= form_for @tweet do |f| %>

  <div class="field">
    <%= f.label :audio %>
    <%= f.file_field :audio %>
  </div>

  <%= f.submit "Tweetする" %>
<% end %>

```

### gemの追加
今回はcloudinaryと、carrierwaveというgemを使っていきます。Gemfileの一番下に以下を追加しましょう。

```ruby:Gemfile
gem 'carrierwave'
gem 'cloudinary'
```

```:ターミナル
$ bundle install
```

[carrierwave](https://github.com/carrierwaveuploader/carrierwave)
[cloudinary](https://cloudinary.com/documentation/rails_integration)

それぞれのgemを簡単に説明すると
carrierwave : railsにてファイルのuploadができるようにするためのgem
cloudinary : 外部ストレージサービスのcloudinaryが利用できるようにするためのgem

### アップローダーの作成
CarrierWaveのジェネレーターでアップローダーを作成します。以下のコマンドをターミナルに打ち込みましょう。

```:ターミナル
$ rails g uploader Audio
```

### モデルの修正
app/models/tweet.rbを以下のように修正します。

```ruby:app/models/tweet.rb
class Tweet < ApplicationRecord

# 追記ここから
  mount_uploader :audio, AudioUploader
# 追記ここまで

end
```

```mount_uploader :audio, AudioUploader```は音声ファイルを指定の場所に保存することを表します。

次に保存場所を指定するアップローダの設定です。

app/uploaders/image_uploader.rbの6~8行目を変更しましょう。

```ruby:app/uploaders/audio_uploader.rb
  # Choose what kind of storage to use for this uploader:
  storage :file
  # storage :fog
```
上記を下図のように変えます。

```ruby:app/uploader/audio_uploader.rb
  # Choose what kind of storage to use for this uploader:
  if Rails.env.production?
    include Cloudinary::CarrierWave
    CarrierWave.configure do |config|
      config.cache_storage = :file
    end
  else
    storage :file
  end
  # storage :fog
```

cloudinaryは外部のストレージサービスです。本番環境(リリース後=production)ではcloudinaryに画像が保存され、それ以外(開発環境=ローカル)では自分のpcに保存されます。(publicフォルダー内に保存されます)

### APIキーの非公開

Cloudinaryの各アカウントには「Cloud name」、「API Key」、「API Secret」が付与されています。

[Cloudinaryマイページ](https://cloudinary.com/console/welcome)

<img width="1138" alt="Cloudinary_Management_Console_-_Dashboard_png.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/202861/85d4bf4e-ec96-553a-d50d-6fce0f8e4489.png">

Cloudinaryのアカウントを外部に漏らさないためにセキュリティを強化します。

ここではdotenv-railsというgemを使っていきます。Gemfileの一番下に以下を追加しましょう。

```:Gemfile
gem 'dotenv-rails'
```

```:ターミナル
$ bundle install
```

と打ち込んでインストールします。

次に`.env`というファイルを**アプリケーションディレクトリ(appやdbやGemfileがあるディレクトリ)**に自分で作成します。
以下の図のようになっていればオッケーです。
※vendorフォルダーの中ではないので注意してください。

<img width="1434" alt="スクリーンショット_2018-10-17_2_03_28.png" src="https://qiita-image-store.s3.amazonaws.com/0/301142/9c267a60-4ba5-7281-447a-f2504e59d892.png">

次に作成した`.env`ファイルに以下を入力します。



```:.env
CLOUD_NAME=q0w9e8r7t6yu5  #←この値は人によって違います！！
CLOUDINARY_API_KEY=123456789012345 #←この値は人によって違います！！
CLOUDINARY_API_SECRET=1a2s3d4f5g6h7j8k9l0a1s2d4f5g6h1q #←この値は人によって違います！！
```

ここで「=」の後のそれぞれの値は先ほどの[Cloudinaryのマイページ](https://cloudinary.com/console/welcome)で取得したキーに書き換えてください（自分が取得したキーは絶対他言しないように！！また数字は個々人によって変わります）。
また、書き換える際は、**コメントアウト部分を削除**してください！

最後に隠しておきたいデータを定義した.envファイルを公開しないようにします。

`.gitignore`に下記を追加します。
<font color="red">**※もし`.gitignore`ファイルがない場合はアプリケーションディレクトリーにて作りましょう！**</font>

```:.gitignore
# 省略

/.env
```

これでOKです！

###APIキーの利用

最後の手順です！
まずはconfigフォルダにcloudinary.ymlファイルを作成してください。

<img width="932" alt="スクリーンショット_2019_10_20_1_11.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/202861/0cd104d2-332b-f681-380a-1f0d39a2e7ab.png">

config/cloudinary.ymlに以下のようにそのままコピペしてください。

```ruby:config/cloudinary.yml

development:
  cloud_name: <%= ENV['CLOUD_NAME'] %>
  api_key: <%= ENV['CLOUDINARY_API_KEY'] %>
  api_secret: <%= ENV['CLOUDINARY_API_SECRET'] %>
  enhance_image_tag: true
  static_file_support: false
production:
  cloud_name: <%= ENV['CLOUD_NAME'] %>
  api_key: <%= ENV['CLOUDINARY_API_KEY'] %>
  api_secret: <%= ENV['CLOUDINARY_API_SECRET'] %>
  enhance_image_tag: true
  static_file_support: false
test:
  cloud_name: <%= ENV['CLOUD_NAME'] %>
  api_key: <%= ENV['CLOUDINARY_API_KEY'] %>
  api_secret: <%= ENV['CLOUDINARY_API_SECRET'] %>
  enhance_image_tag: true
  static_file_support: false
```

これでAPIキーを非公開にしたまま利用することができました。

これにて、音声ファイルの投稿はできるようになりました。お疲れ様です(^^)/

#公式ドキュメント

[carrierwave](https://github.com/carrierwaveuploader/carrierwave)
[cloudinary](https://cloudinary.com/documentation/rails_integration)
[dotenv-rails](https://github.com/bkeepers/dotenv)
