---
title: 【Python】画像処理入門
tags:
  - 'Python'
  - '初心者'
  - '初学者向け'
  - '画像処理'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
こちらの記事では、初学者向けにPythonを使った画像処理に触れていきたいと思います！
コンピュータはどのように画像を扱っているのか？を実感できる内容を意識して書きました。

自分のPC(ローカル環境)上で実行していきます。
具体的には、自分で作成したファイル中にPython言語を記述していき、これを実行させます。

# 準備
以下二つを利用していきます。まだの方がいたら、先に環境構築の章を取り組んでいきましょう！

・Anaconda
・VSCode

# 画像処理

## Python実行フォルダー、ファイルの準備
今回は分かりやすいようにデスクトップ上にフォルダーを作成します。

### step1
デスクトップの何もないところで、
右クリック -> 新規作成 -> フォルダーで新規フォルダーをデスクトップ上に作成します。名前はpython_practiceとしておきましょう！

![python1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/eb4f57d7-b4e9-e2c3-5b45-15f7f5b87973.png)

![python2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/94cc1b5c-5c4c-589f-6b99-4201dc661911.png)

### step2
次にデスクトップ上のpython_practiceをVscodeで開きます。
まず、VSCodeのアプリを起動します。機種によって微妙に見た目が違いますが、大体以下のような黒い背景のエディターが立ち上がっています。

![python3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/4db0a3f8-0d55-bd92-0ee8-0e975782f93b.png)

左クリックでpython_practiceを掴み、Vscode上にドラッグ&ドロップしてください。

![python4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/7c47a105-a20c-c363-3cd8-87958ea85b85.png)

そうすると、以下のようにVSCode内でpython_practiceを開くことができました。

![python5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/344bc95f-e1b1-4226-44fd-23fac7618350.png)

### step3
今度は、今後記述していくPythonファイルを作成していきます。

![python6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/ffb068e7-b581-837f-3a8f-aa28c1cf454e.png)

「PYTHON_PRACRICE」と書いてある箇所の下のスペース上で右クリックして「New File」を選択してください。ファイル名はsample1.pyにします。

同様にsample2.pyとsample3.pyとsample4.pyも作成します。
以下のように4つのファイルが作成できれいればOKです。

![python7.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/c571ed79-ef7c-9b3b-abfe-3f9c642057b6.png)

### step4
処理したい画像ファイルを用意します。

本記事では以下の画像を例に用います。自分の好きな画像をダウンロードして利用してみてもOKです。

![python8.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/60faee8c-6d54-22d4-8926-9ee0ffc71379.png)

画像処理に利用する画像を先ほど作ったpython_practiceフォルダー内にそのまま置きます。今記事で用いるファイル名はhuman.pngにします。

![python9.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/6aba46de-8310-aa4f-2c7f-a46819f5da30.png)

以上で準備完了です！これから画像処理のコードを記述していきますが、まず、用いるライブラリや周辺知識を説明します。

## ライブラリ

![python10.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/4cf6ed2b-5415-09e4-a6e8-55984efe466e.jpeg)

ライブラリとは、「繰り返し使う機能がまとまっている工具箱」の様なもので、簡単に利用することができます。
今回使うライブラリは以下です

①OpenCV
コンピュータビジョンのライブラリ。
画像を白黒にするなどの画像編集・加工ができる。

②Matplotlib
グラフ描画のために使われるライブラリ。画像の出力用に用いる。

③Numpy
計算処理に便利なライブラリ。しかも高速。

## コンピュータはどのように画像を扱っているの？
画像データは全て数字でコンピュータ内では扱われます。
赤，緑，青の各色を0～255の値で表すことで、それらの組み合わせによって色が決まります。

![python11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/4906bd68-f611-e1f4-0727-7657c55a36ad.png)

それぞれの画素(Pixel)の色がRGBで表されています。

![python12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/bbd6427c-3313-6be2-20bb-bbe77393fddd.png)

これらは、一般的には以下画像のように【縦座標 x 横座標 x 色情報】の3つの情報を扱う3次元配列の形式のデータとして内部では利用されることになります。

![python13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/0e75ee19-501a-89ea-954d-88834cb09a87.png)

## ライブラリを使って画像表示
OpenCVとmatplotlibを使って画像を表示します。それと同時に実行方法も見ていきましょう。

### step1
VSCodeでsample1.pyを開き、以下コードを丸々コピペします。
コピペしたら、「Ctrl(Command) + S」で保存します。
詳しいコードの説明はコメントアウト部分に譲ります。

```python:sample1.py
# ライブラリのインポート
# matplotlibのインポート。今後pltという名前で用いる。
from matplotlib import pyplot as plt

# OpenCVのインポート
import cv2

# 画像ファイルの読み込み
# 同じ階層にあるため、ファイル名のみ記述
filename = "human.png"

# 画像配列の生成
orig = cv2.imread(filename)

# RGBの順に整形
src = cv2.cvtColor(orig, cv2.COLOR_BGR2RGB)

# 配列の形を表示(たて座標 x 横座標 x 3色の値)
print(src.shape)

# 画像の表示
plt.imshow(src)
plt.show()
```

### step2
上記のPythonファイルを実行します。いくつか方法はありますが、以下一例です。

Windowsの方は、「Anaconda prompt」
Macの方は、「ターミナル」を起動します。
これらは、コマンド(命令)を入力して実行してくれるものです。

Anacondaには、MatplotlibとNumpyがデフォルトで入っていますが、OpenCVは入っていないので、まずこちらをダウンロードします。
以下を入力して「Enter」を押します。

```sh:ターミナル
pip install opencv-python
```

すると、以下画面のようにOpenCVがダウンロードされました。

![python14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/e6322289-34ea-66de-6df8-6a3dce51e2b4.png)

次に、デスクトップ上に作った「python_practice」の階層に移動し、sample1.pyファイルを実行します。以下を一行ずつ順番に入力して「Enter」を押していきましょう！ `cd`は`change directly`の略で、今いる階層を移動するコマンドです。

```sh:ターミナル
cd Desktop
cd python_practice
python sample1.py
```

上記のようにsample1.pyを実行出来たら、以下のように配列の形の出力と、画像表示用のWindowが出てきます。このようにして、ローカル環境でPythonファイルを実行していくことができます。

![python15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/f91468dd-2b9f-a4bb-67e3-0842139709f3.png)

また、新たにターミナルに入力するには、画像windowsを閉じるか、「Ctrl(command) + C」を入力します。

すると、以下のように入力できる状態に戻りました。

![python16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/a3476755-fbd6-0542-7347-7c70fb8978c6.png)

## 画像を加工する

### blur(ぼかし)
OpenCVを用いると、blur(ぼかし)を簡単にかけることができます。

sample2.pyファイルに以下を丸々記入して保存します。

```python:sample2.py
# ライブラリのインポート
# matplotlibのインポート。今後pltという名前で用いる。
from matplotlib import pyplot as plt

# OpenCVのインポート
import cv2

# numpyのインポート
import numpy as np

# 画像ファイルの読み込み
# 同じ階層にあるため、ファイル名のみ記述
filename = "human.png"

# 画像配列の生成
orig = cv2.imread(filename)

# RGBの順に整形
src = cv2.cvtColor(orig, cv2.COLOR_BGR2RGB)

# OpenCVのblurメソッドを利用し、ぼかしをつける
blurred = cv2.blur(src, (20, 20))

# ぼかし画像の表示
plt.imshow(blurred)
plt.show()
```

前回と同様、ターミナル上でsample2.pyファイルを実行します。python_practiceの階層の場所で以下を入力して「Enter」を押します。

```sh:ターミナル
python sample2.py
```

ぼかし画像は出てきましたか？？(^^)/

![python17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/fb800f28-2899-cf6e-b0bb-1ee6c93d1be1.png)

### 色の反転
色の反転は、numpyで計算を行います。

sample3.pyファイルに以下を丸々記入して保存します。

```python:sample3.py
# ライブラリのインポート
# matplotlibのインポート。今後pltという名前で用いる。
from matplotlib import pyplot as plt

# OpenCVのインポート
import cv2

# numpyのインポート
import numpy as np

# 画像ファイルの読み込み
# 同じ階層にあるため、ファイル名のみ記述
filename = "human.png"

# 画像配列の生成
orig = cv2.imread(filename)

# RGBの順に整形
src = cv2.cvtColor(orig, cv2.COLOR_BGR2RGB)

# numpyを用い、画像データ配列の各要素のRGB値を255から引く。
# ブロードキャストにより全要素の(r ,g, b)それぞれの値が255から引かれる
pixels = np.array(src)
pixels = 255 - pixels

# 色反転画像の表示
plt.imshow(pixels)
plt.show()
```

同様に、以下実行していきましょう！

```sh:ターミナル
python sample3.py
```

### 画像の一部のみに変更を適用する

画像配列の一部の範囲内のRGB値を計算対象とします。
座標の範囲を指定し、繰り返し処理を利用します。

![python18.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/52d551b0-cb4c-693f-2480-eee25d622796.png)

```python:sample4.py
# ライブラリのインポート
# matplotlibのインポート。今後pltという名前で用いる。
from matplotlib import pyplot as plt

# OpenCVのインポート
import cv2

# numpyのインポート
import numpy as np

# 画像ファイルの読み込み
# 同じ階層にあるため、ファイル名のみ記述
filename = "human.png"

# 画像配列の生成
orig = cv2.imread(filename)

# RGBの順に整形
src = cv2.cvtColor(orig, cv2.COLOR_BGR2RGB)

# 範囲(left, top, right, bottom) 数値は任意
roi = (60,15,130,110)

pixels = np.array(src)


# 以下画像配列の一部の座標範囲内のRGBを計算対象とする
# 繰り返し処理を利用

# 縦座標の範囲
for y in range(roi[1],roi[3]):

  # 横座標の範囲
  for x in range(roi[0],roi[2]):

    # 各座標のピクセル値を変更
    # ブロードキャストにより(r ,g, b)それぞれの値が255から引かれる
    pixels[y][x] = 255 - pixels[y][x]

# 画像の表示
plt.imshow(pixels)
plt.show()
```

### 画像の一部のみに変更を適用する-その2
画像の一部を黒色で覆ってみましょう！
sample5.pyとして作ってみましょう！

![python19.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/1e32e20f-5951-097b-8e97-fa2573d322d6.png)

<details><summary>解答例はこちらをクリック</summary><div>

```python:sample5.py
# ライブラリのインポート
# matplotlibのインポート。今後pltという名前で用いる。
from matplotlib import pyplot as plt

# OpenCVのインポート
import cv2

# numpyのインポート
import numpy as np

# 画像ファイルの読み込み
# 同じ階層にあるため、ファイル名のみ記述
filename = "human.png"

# 画像配列の生成
orig = cv2.imread(filename)

# RGBの順に整形
src = cv2.cvtColor(orig, cv2.COLOR_BGR2RGB)

# 範囲(left, top, right, bottom) 数値は任意
roi = (60,55,120,70)

pixels = np.array(src)


# 以下画像配列の一部の座標範囲内のRGBを計算対象とする
# 繰り返し処理を利用

# 縦座標の範囲
for y in range(roi[1],roi[3]):

  # 横座標の範囲
  for x in range(roi[0],roi[2]):

    # 各座標のピクセル値を黒色に変更
    pixels[y][x] = (0, 0, 0)

# 画像の表示
plt.imshow(pixels)
plt.show()
```

</div></details>

# おわりに
この記事では、Pythonとそのライブラリを使用して画像処理について書きました。
コンピュータは画像を縦x横xRGB値という3次元配列で表現しており、プログラミングで色々処理できることが実感できたと思います。
ちなみに動画はここに時間軸が加わるので4次元配列となります。

本記事で触れた処理内容自体は基本的なものでしたが、色々な画像処理ツールはこの基本原理に基づいて作られています。
どのような画像処理アルゴリズムが使われているのだろう？と興味を持って色々調べていくと、新たな発見があるかもしれません。





