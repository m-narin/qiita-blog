---
title: ページトップボタン
tags:
  - HTML
  - CSS
private: false
updated_at: '2020-04-14T01:37:45+09:00'
id: d98a9a625bd119cc263e
organization_url_name: null
slide: false
ignorePublish: false
---
以下のようなページトップボタン(ページの右下当たりにあって、クリックすると一番上に戻ってくるやつ)を実装します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/adf5e81e-46bd-e45a-b741-515a3c4c191a.png)

まず、ボタンを置きたいhtmlに以下をコピペ

```index.html.erb(など)
<p id="pagetop"><a href="#">↑ </a></p>
```

これがページトップの指令になります。

次に以下を該当cssファイルにコピペしましょう。

```tweets.css
/*ページトップ設定*/
#pagetop {
  margin-right: 10px;
	clear: both;
	padding-bottom: 40px;
}
#pagetop a {
	color: #FFF;		/*文字色*/
	font-size: 20px;	/*文字サイズ*/
	background: grey;	/*背景色*/
	text-decoration: none;
	text-align: center;
	display: block;
	float: right;
	border-radius: 30px;	/*角丸のサイズ*/
	width: 50px;
	line-height: 50px;
}
```

良い感じになるかと思います！
