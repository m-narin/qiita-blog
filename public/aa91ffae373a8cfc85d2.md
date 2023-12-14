---
title: Google Mapを利用する  【Rails6】【Geocoder】【heroku】
tags:
  - Rails
  - Heroku
  - GoogleMapsAPI
private: false
updated_at: '2021-02-09T19:50:21+09:00'
id: aa91ffae373a8cfc85d2
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

この記事では, RailsアプリでGoogle Mapを利用する方法を書き記しています.

特徴は以下です.

1. Google MapのAPIを利用します
2. APIキーは公開防止のため, .envファイルで管理します.
3. Mapの緯度経度情報をデータベースに登録することで, 場所の投稿サイトを作ることができます.
4. 投稿された内容はMap上にピンを立てるようにします.
5. 場所を投稿すると, 自動で緯度経度を取得し, データベースに登録するようにします.
6. ピンの情報ウィンドウをクリックすると投稿の詳細ページへ飛びます

うまく動作しないときは, ディベロッパーツールのコンソールにエラー文が出力されるため参考にしましょう.

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/17328299-d5c3-3fd4-39d6-cf01a021909d.png)


## もくじ
1. Google Map APIを利用可能にする
2. MVCを用意する & その他設定
3. herokuにAPIを登録する 

## 1. Google Map APIを利用可能にする
Google MapはもちろんGoogleが発表しているサービスです. このようなサービスはAPIと呼ばれる仕組みを通して他と連携して利用することができるようになります. 具体的には, APIキーを登録します. 以下のページにて作業します.

[Google Cloud Platform](https://console.cloud.google.com/?hl=ja)

**※前提としてgoogleアカウントが必要です. また, 形式上Googleアカウントにクレジットカードを登録している必要があります.**

### プロジェクトの作成

以下の手順に従ってプロジェクトを作成します.

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/f901042c-59bc-e6cd-676c-8175f5cf28f2.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/298580d3-f984-1f94-b085-e5c0a8d3277f.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/50e7670e-9713-ed0c-e9e1-34cafbcd4f79.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/9728a6f6-5900-c61c-ea25-02af9b771dd0.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/c016f606-c95e-0d40-3291-98985014c0f0.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/c5750cfa-fdb7-dcff-229a-752c94ad54d4.png)

### APIキーの作成
以下の手順に従ってAPIキーを作成します.

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/61851226-f695-413d-64f7-7a8486b710e7.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/0ccc2de8-f16a-ebe7-f0fc-63ef27ff6706.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/5a1683ac-2c40-0ae9-015a-17953e692504.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/c7f9feef-6cfa-4798-bb8c-51093cd84f04.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/2661962a-527a-c03b-e773-3bd0befd8f9a.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/ebecd81f-98d3-054e-4fcd-fe8723e049f0.png)

<font color="RED">作成されたAPIキーは後で利用するのでどこかにメモしておきましょう！</font>

### 請求先アカウント設定
Google Map APIを利用するには, 形式上請求先アカウントを設定する必要があります. 請求情報を紐づけるだけで, 無課金設定のまま利用できます.

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/ac090ad4-0f82-390d-ac06-7668aed1fb83.png)

「お支払い」から請求先アカウントを作成します. このとき, Googleアカウントにクレジットカードを登録している必要があります.
画面の指示に従って登録しましょう.

## 2. MVCを用意する & その他設定
上記でGoogle Map APIを利用するための準備が整いました. 今度は, 自分のRailsアプリの設定をしていきます.

この記事では, 一例として以下のようにmapsテーブルを作成します.
また, indexページにて, 場所の表示と場所の投稿を両方できるようにします.

maps テーブル

| address | latitude | longitude |
|:-----------|------------:|:------------:|
| 東京駅       | 35.6812        | 139.7671         |
| ...     | ...      | ...       |

### mapsテーブル

以下のコマンドにより, address(住所), latitude(緯度), longitude(経度)を保存できるmapsテーブルを作成します.

```
$ rails g model Map address:string latitude:float longitude:float
$ rails db:migrate
```

### mapsコントローラー

mapsコントローラー作成コマンド

```
$ rails g controller maps
```

コントローラーの記述は以下のようにします.

```app/controllers/maps_controller.rb
class MapsController < ApplicationController
    def index
        @maps = Map.all
        @map = Map.new
    end

    def create
        map = Map.new(map_params)
        if map.save
            redirect_to :action => "index"
        else
            redirect_to :action => "index"
        end
    end

    def destroy
        map = Map.find(params[:id])
        map.destroy
        redirect_to action: :index
    end

    private
    def map_params
    params.require(:map).permit(:address, :latitude, :longitude)
    end
end
```

### ルーティング

routes.rbに`resources :maps`の一行を追加します.

```congif/routes.rb
resources :maps 
```

### viewファイル

app/views/maps/**index.html.erb** を作成し, テンプレートとして以下を記述します.

```erb:index.html.erb
<h2>Google map</h2>

<input id="address" type="textbox" value="">
<input type="button" value="地図を検索" onclick="codeAddress()">
<div id="display">緯度経度が表示されるよ！</div>

<div id='map'></div>

<style>
    #map {
        height: 400px;
        width: 400px;
    }
</style>

<script>
    let map

    const display = document.getElementById('display')

    // mapの表示関数 
    function initMap() {
        geocoder = new google.maps.Geocoder()

        // mapの初期位置, 縮尺を定義
        map = new google.maps.Map(document.getElementById('map'), {
            center: {
                lat: 35.6458437,
                lng: 139.7046171
            },
            zoom: 12,
        });

        // mapsテーブルにあるそれぞれのレコードをmap上に表示 
        <% @maps.each do |m| %>
            (function(){
            var contentString = "住所：<%= m.address %>"; 

            // マーカーを立てる
            var marker = new google.maps.Marker({
                position:{lat: <%= m.latitude %>, lng: <%= m.longitude %>},
                map: map,
                title: contentString
            });

            // 情報ウィンドウ(吹き出し)の定義
            // 投稿の詳細ページへのリンクも
            var infowindow = new google.maps.InfoWindow({
            position: {lat: <%= m.latitude %>, lng: <%= m.longitude %>},
            content: "<a href='<%= map_url(m.id) %>' target='_blank'><%= m.address %></a>"
            });

            // クリックしたときに情報ウィンドウを表示
            marker.addListener('click', function() {
            infowindow.open(map, marker);
            });

            })();
        <% end %>
    }

    let geocoder

    // 地図検索関数
    function codeAddress() {
        let inputAddress = document.getElementById('address').value;

        geocoder.geocode({
            'address': inputAddress
        }, function (results, status) {
            if (status == 'OK') {
                map.setCenter(results[0].geometry.location);
                var marker = new google.maps.Marker({
                    map: map,
                    position: results[0].geometry.location
                });

            display.textContent = "検索結果：" + results[ 0 ].geometry.location
            } else {
                alert('該当する結果がありませんでした：' + status);
            }
        });
    }
</script>

<script
    src="https://maps.googleapis.com/maps/api/js?key=<%= ENV['GOOGLE_MAP_API_KEY'] %>&callback=initMap"  
    async defer>
</script>

<h3>場所投稿フォーム</h3>
<%= form_for(@map, :url => { controller:'maps', action:'create'})do |f| %>
    
    <p>
    <%= f.label :address %>
    <%= f.text_field :address, size: "50x1" %>
    </p>

    <%= f.submit "送信"%>
<% end %>

<h3>場所一覧</h3>
<% @maps.each do |t| %>
    <p>住所 : <%= t.address %></p>
    <p>緯度 : <%= t.latitude %></p>
    <p>経度 : <%= t.longitude %></p>
    <p><%= link_to "削除する", map_path(t.id), method: :delete %></p>
    <hr>
<% end %>
```

投稿詳細(show)ページでmapを表示する場合は, 上記を参考に自分で作ってみてください！！

### その他設定

#### Gemファイル & 環境変数の設定

Gemfileに以下二つのgemを追記します

```Gemfile
gem 'geocoder'
gem 'dotenv-rails'
```

その後, bundle installします

```
$ bundle install
```

geocoderは住所(場所)から緯度経度を取得するGemです.
dotenv-railsは.envファイル(後述)を利用するためのGemです.

#### Map.rb(モデルファイル)

map.rbファイルは以下のように記述します.

```app/models/map.rb
class Map < ApplicationRecord
    geocoded_by :address
    after_validation :geocode
end
```

addressが投稿されたら, 自動で緯度経度もデータベースに登録してくれます.

#### .envファイル

ルートディレクトリーに.envファイルを作成します.

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/92738f6c-6c86-beb8-2174-26d2e586325c.png)

これは, リリースするときに一般に公開したくない類のもの(APIキー)等を管理する用途で用います.

.envファイルに`GOOGLE_MAP_API_KEY=xxx`の一行を追記します.

この`GOOGLE_MAP_API_KEY=xxx`の
右辺は先ほどメモしておいたGoogleのAPIキーを記述します.

```.env
GOOGLE_MAP_API_KEY=ここに自分のAPIキーを書く
```

これは, maps/index.html.erbの

```erb:maps/index.html.erb
<script
    src="https://maps.googleapis.com/maps/api/js?key=<%= ENV['GOOGLE_MAP_API_KEY'] %>&callback=initMap"  
    async defer>
</script>
```

で用いられます.

公開しないように .gitignoreファイルに以下一行を記述します.

```.gitignore
/.env
```

## 3. herokuにAPIを登録する

herokuにリリースするとき, GoogleのAPIキーをherokuにも登録する必要があります. 以下のxxxを自分のAPIキーに変えて, 実行することを覚えておいてください！

```
$ heroku config:set GOOGLE_MAP_API_KEY=xxxxxxxxxx
```

