---
title: 【Julia】でバーンスレイのシダを描こう
tags:
  - Julia
private: false
updated_at: '2021-12-07T00:49:38+09:00'
id: 4c1fdb8232602ffd50b7
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

Juliaでバーンスレイのシダを描きたいと思います。

▼バーンスレイのシダとは

https://ja.wikipedia.org/wiki/%E3%83%90%E3%83%BC%E3%83%B3%E3%82%BA%E3%83%AA%E3%83%BC%E3%81%AE%E3%82%B7%E3%83%80

バーンスレイのシダとは、植物の形をモデル化しようと数学的に生成されるパターンです。フラクタル図形として表され、拡大しても同じ図形が現れ続ける面白い性質があります。

出力の際、「Images」「ImageView」パッケージを利用します。

# 実装

4つの関数を定義します。

```jl
#4つの関数の定義

f1 = (x,y) -> (0., 0.16*y)
f2 = (x,y) -> (0.85*x + 0.04*y, -0.04*x + 0.85*y + 1.6)
f3 = (x,y) -> (0.2*x - 0.26*y, 0.23*x + 0.22*y + 1.6)
f4 = (x,y) -> (-0.15*x + 0.28*y, 0.26*x + 0.24*y + 0.44)
fs = [f1, f2, f3, f4]
```

上記の式を利用し、画像行列を生成します。x,y各座標を条件に従って更新していきます。

```jl
# 計算回数
num = 50000
# Canvas size (pixels)
width, height = 300, 300
image = zeros((width, height))

x, y = 1, 1
for i in 1:num
    
    # ランダムに指定の確率に従って関数が選ばれる
    
    r = rand((1:100))
    if r<=1
        f = fs[1]
    elseif r<=86
        f = fs[2]
    elseif r<=93
        f = fs[3]
    else
        f = fs[4]
    end
        
    x, y = f(x,y)

    #配置を真ん中あたり移動
    #小数を整数に変換
    ix, iy = convert(Int,round(width / 2 + x * width / 10)), convert(Int, round(y * height / 12+1))
    # Set this point of the array to 1 to mark a point in the fern
    #白ドットを打つ、縦方向
    image[ix, iy] = 1.0
end
```

可視化します。

```jl
# 全体図
using Images, ImageView

# 90度回転して出力
img = colorview(Gray, rotl90(image,1))
```

以下のようにシダの形が表示されました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/f351c866-cce8-d556-302d-0de609485e83.png)

