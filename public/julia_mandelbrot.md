---
title: 【Julia】でマンデブルロ集合を描こう
tags:
  - Julia
  - マンデルブロ集合
private: false
updated_at: '2021-12-07T00:42:25+09:00'
id: 6f0b321206a4f198a859
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

Juliaでマンデルブロ集合を描きたいと思います。

▼マンデルブロ集合とは

https://ja.wikipedia.org/wiki/%E3%83%9E%E3%83%B3%E3%83%87%E3%83%AB%E3%83%96%E3%83%AD%E9%9B%86%E5%90%88

マンデルブロ集合とは複素数列において、n → ∞で無限大に発散しない条件を満たす複素数の集合のことです。フラクタル図形として表され、拡大しても同じ図形が現れ続ける面白い性質があります。コンピュータや言語の性能を評価する際に扱われることもあるそうです。

出力の際、「Images」「ImageView」パッケージを利用します。

# 実装

マンデルブロ集合の関数を定義します。

```jl
# (kx,ky)はウィンドウ座標系 (cx,cy)は複素平面上の座標
function mandelbrot(arr, width, height, xmin, ymin, xcoef, ycoef, maxIt)
    # 各ウィンドウ座標を複素座標に変換
    for ky in 1:height
        cy = ycoef*(height-ky) + ymin
        for kx in 1:width
            cx = xcoef*kx + xmin
            c = complex(cx,cy)
            z = complex(0.0, 0.0)
            flag = true
            
            count = 0
            
            # 複素平面の計算
            for i in 1:maxIt
                count = i
                z = z * z + c
                
                if abs(z) >= 2.0
                    flag = false
                    break
                end
            end
            
            # 発散したタイミングがcountに代入される
            if flag
                arr[:,ky, kx] .= ( 255.0, 255.0, 255.0 )　#色 juliaのImageライブラリはfloat型でRGBを記述
            else
                if count <= 1
                    b_color = 0 #急速な発散は黒に、濃淡
                elseif count <= 3
                    b_color = 60
                elseif count <= 5
                    b_color = 150
                else
                    b_color = 255
                end
                arr[:,ky, kx] .= ( 0, 0, b_color)

            end
        end
    end
end
```

可視化をします。

```jl
#全体を見るには次の2行を活かし，下の2行をコメントアウト
xmin, xmax, ymin, ymax = -3.0, 1.0, -1.5, 1.5
WIDTH  = 800 ; HEIGHT = 600 # propotional to (x,y) range



#一部を見るには次の2行を活かし，上の2行をコメントアウト
# xmin, xmax, ymin, ymax = 0.1, 0.5, 0.4, 0.8
# WIDTH  = 800 ; HEIGHT = 800 # propotional to (x,y) range

xwidth = xmax - xmin
ywidth = ymax - ymin
maxIt = 256

x_coeff = xwidth/WIDTH
y_coeff = ywidth/HEIGHT

# JuliaのRGB表示、Float型、Indexの順番に注意
array = zeros(Float64, (3,HEIGHT, WIDTH))  # 3 => RGB array
size(array)
mandelbrot( array, WIDTH, HEIGHT, xmin, ymin, x_coeff, y_coeff, maxIt )
```

```jl
# Juliaのパッケージ、Images ImageViewを使う
# RGBの3次元配列はFloat型でないといけない、

using Images, ImageView
img = colorview(RGB, array)
```

以下のようにマンデルブロ集合を得られました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/5d2f739d-5d53-b58b-0cdd-9028f8dcdcd4.png)

また、一部のみ出力してみることもできます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/44a4f344-e8c4-a8f8-5d53-71a5cf6bc0a1.png)

