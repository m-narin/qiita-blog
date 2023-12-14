---
title: kaminariをカスタマイズ
tags:
  - Rails
  - Gem
  - kaminari
private: false
updated_at: '2020-04-14T02:17:32+09:00'
id: 822d8ca1647dfca7d139
organization_url_name: null
slide: false
ignorePublish: false
---
ページネーションはkaminariというgemを用いることが多いかと思います。

参考
https://qiita.com/residenti/items/1ae1e5ceb59c0729c0b9

機能面は実装済みという前提で話します。

kaminariを実装すると初めは以下のように見にくいデザインになっているかと思います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/9fcae48c-169f-a978-68d7-9bbf70a53c9e.png)


例えば、これを以下のように変えましょう！
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/c3429f12-f52b-e6e0-c57f-08001f19344d.png)

上記リンクのように、bootstrapを適用するという方法もありますが、リンクホバーなど自分でカスタマイズしたいですよね。

#kaminariのviewファイルを作成

ターミナルにて以下を打ち込んでください。

```
rails g kaminari:views default
```
これによりviewフォルダの中にkaminariのview一覧が作成されました！これはそのままにして次に進んでください！

#日本語化
kaminariのgemのデフォが英語になっているので、これを日本語環境に変える必要があります。まず、config/applicationファイルをに以下を追記します。

```config/application.rb
config.i18n.default_locale = :ja
config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}').to_s]
```
一行目は、日本語環境を適用、二行目は複数のlacaleファイル(後述)が適用されるコードになります。

全体としては以下のようになっているかと思います。

```config/application.rb
require_relative 'boot'

require 'rails/all'

# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(*Rails.groups)

module Geektwitter
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 5.1
    config.i18n.default_locale = :ja
    config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}').to_s]
    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration should go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded.
  end
end
```
これで日本語化が適用されます。

次にconfig/localesフォルダーにkaminari_ja.ymlを作成してください。(ja_ymlファイル中に例えばcreated_atの日本語化が既に書いてある場合、他の日本語化は本ページのように別のファイルを作成する必要があります。)

その中に以下をコピペします。

```config/locales/kaminari_ja.yml
ja:
  views:
    pagination:
      first: "&laquo; 最初"
      last: "最後 &raquo;"
      previous: "&lsaquo; 前へ"
      next: "次へ &rsaquo;"
      truncate: "..."
```

英語を日本語に変えました！

#カスタマイズ
次に該当のcssファイルに以下を追記しましょう。

```tweets.css
// paginate
.pagination{
  margin: auto;
  width: 50%;
  display: flex;
  justify-content: flex-start;
}
.pagination span{
  background-color: rgba(158, 158, 158, 0.4);
  text-align: center;
  width: 50px;
  border: solid 1px #344963;
  color: rgb(197, 20, 159);
  transition: .3s;
  -webkit-transform: scale(1);
  transform: scale(1);
}
.pagination span:hover{
  transition: .3s;
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
.pagination span a:hover{
  transition: .3s;
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
```
**※注意※**
**日本語環境に変えたあと、一度rails sを再起動して下さい。**

これで良い感じになったかと思います！色合いや位置の調整(width)は、各自変えてみてください！
