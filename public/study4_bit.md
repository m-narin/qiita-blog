---
title: study4_bit
tags:
  - Python
  - 初心者
  - 競技プログラミング
  - 新人プログラマ応援
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
新卒エンジニア同士で実施している勉強会の第4回目の記事になります。
今回のテーマはbit演算についてです。

<!-- TODO: 記事リンク貼る -->
| 回     | テーマ | 記事リンク |
|:-----------|:------------|:------------|
| 第1️回      | 二分探索      | <coming soon>    |
| 第2回      | ソートアルゴリズム  |  <coming soon>     |
| 第3回      | 暗号化            |  <coming soon>     |
| **第4回**  | **bit演算**      |  <coming soon>    |
| 第5回      | 連想配列          | <coming soon>   |
| 第6回      | グラフ理論        |  <coming soon>   |

## 前提
今回はbit演算を用いて簡単な計算をするプログラムを作ります。bit演算はコンピュータの基本原理と深い関係があります。私たちがコンピュータを触る際は、当たり前のように足し算引き算掛け算割り算の命令を入力すると計算結果が返ってきますね。しかしコンピュータは根源的には物理的な電気信号のon, offという二つの情報しか扱っていません。これだけで四則演算をどのように実現しているのか？を考えるのが今回の目的です！
問題に取り掛かる前に、知っておきたい概念があるためその説明から入ります。

### 電気回路
コンピュータは全ての情報を電圧の高い(5v)、低い(0v)という二種類のデジタル信号のみで表しています。電気信号は回路とスイッチを使うことで流れを制御することができます。

![kairo.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/545ab640-9fd0-5082-c7e4-65f287821fce.png)

いずれの図も左側が高電圧で右の方に電流が流れているものと捉えてください。山から川が上流→下流に流れているようなイメージです。
そこに何らかの操作でスイッチをonにすることを考えます。操作とは、たとえばCtrl(command)キーと「c」を同時に入力するとコピー処理が動くといった物理的なアクションであったり、電気の入力によるスイッチの開閉等をイメージできれば良いでしょう。

一番上の図は直列回路を表します。一本の線で繋がっているため両方のスイッチがonにならないと他方に電流は流れません。これはAND回路とも言います。これはまさにCtrl(command)キーと「c」を同時に入力するとコピー処理が動く状況と対応していますね。
真ん中の図は並列回路を表します。分岐があるため少なくとも一方のスイッチがonになっていれば電流は他方に流れます。これはOR回路とも言います。
一番下の図はNOT回路です。これは高い電圧が流れてきた時は低い電圧を出力し、低い電圧が流れてきた時は高い電圧を出力します。つまり電圧の高低を反転させる働きを持ちます。内部的には下記図のよう上から常に電流を流しておいて入力がある場合は電磁石の要領でスイッチがonになるような仕組みとなっています。スイッチがonになると上から下に電流が流れるので出力に電流は流れません。逆に入力がない場合はスイッチがoffとなっていて、上からの電流は出力側に流れていきます。

![not_kairo.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/5cb953fd-dbe6-02a8-1549-71f0b3b996ec.jpeg)

この三つの回路のパターンが基本原理となります。これらを色々組み合わせることで様々な電流の流れのパターンを作り出すことができます。
様々な電流の流れのパターンは記号で規格化されています。それらをまとめると以下図のようになります。

![kairo_matome.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/dfef04d2-21c1-b464-85b4-37ec49a797ce.png)

たとえば、片方の入力のみがある場合に出力するような回路をXOR回路と言います。XORに関して上図の記号とベン図を見てみます。まずA以外の領域かつBに重なる場所は、右側の、中央の重なりの除いたBの領域になります。反対にB以外の領域かつAと重なる場所は、左側の、中央の重なりを除いたAの領域になります。つまり、A入力の反転(NOT)とB入力のAND回路、B入力の反転(NOT)とA入力のAND回路を作り、この二つの出力をOR回路に繋げば良いというわけですね。

XOR回路は下記図のようにNOT, AND, OR回路を配置することで実現できます。

![xor.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/0a4d9225-2651-cd69-db39-3cc832aa4af0.png)

### bit演算
回路によって作られる様々な電流の動きは記号化されていてプログラムで扱えるようになっています。電圧の高低はbitという単位で1セット分の情報として捉え、数字の0,1で表現されます。特に0,1からなる数列の、対応する位同士に任意の回路を当てるような計算方法をbit演算と言います。出力を見て、電流が流れた、流れなかったのような情報を再度位ごとに0,1で保持しておくイメージです。当然、一桁のみのbit演算をする場合は真理値表に相当します。

| 論理演算     | 演算子 | 例 |
|:-----------|:------------|:-----------|
| AND演算    | &  |  1010 & 1100 → 1000  |
| OR演算     | \|  |  1010 \| 1100 → 1110  |
| XOR演算    | ^  |  1010 ^ 1100 → 0110  |

### 2進法とは？
0,1を数列として並べたものをbit列と言い、これに位の概念を乗せることで**数**として解釈することができるようになります。

普段私たちは10進法で物事を数えています。10進法は0~9の十種類の数字を使用し、1から数えて十個目の数字で桁上がりが発生し10となります。11, 12,,, 19, 20, ,, 99のように増加し、桁上がりで100(百), 1000(千)となっていきます。桁上がり直後の数字同士を比較すると、数の大きさは十倍ずつ増えていきます。

2進法も同様に考えることができます。2進法は0,1の二種類の数字を使用し、1から数えて二個目の数字で桁上がりが発生し10となります。その次は11, 100, 101, 110, 111, 1000,,,のように表記していきます。桁上がり直後の数字同士を比較すると、数の大きさは二倍ずつ増えていきます。
10進数における235は、100 x 2 + 10 x 3 + 1 x 5のように表現できます。位の位置に応じて10のN乗の数と対応しているわけです。
同様に2進数の場合も、下記画像のように位の位置に応じて2のN乗の数と対応しています。

![full_adder.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/9b8ff607-7813-41ee-2ee5-a24231f1d8b6.png)

| 2進数  | 10進数 |
|:------|:-------|
| 0    | 0  |
| 1    | 1  |
| 10   | 2  |
| 11   | 3  |
| 100  | 4  |
| 101  | 5  |
| 110  | 6  |
| 111  | 7  |
| 1000 | 8  |
| 1001 | 9  |
| 1010 | 10 |

bit列を左右に移動させる演算をシフト演算と言います。

| 論理演算     | 演算子 | 例 |
|:-----------|:------------|:-----------|
| 左シフト演算    | << |  1010 << 2 → 101000  |
| 右シフト演算    | >> |  1010 >> 2 → 10  |

これにより、移動した分だけ位を増減させることができます。つまり、2進数においてシフトした分だけ2の倍数増加したり、割られたりします。

## 問題1
1桁の2進数の数字2つが以下のように与えられます。

```shell:input
1,0
```
入力された二つの数字の和に関して、2進法で計算した場合の繰り上がりと一桁目を順に出力してください。

なお、以下の条件に従ってください。
- bit演算子を使うこと
- 計算量は問いません

### テストケース

テストケースは以下四つです。例えば入力が1,1だった場合、1+1で繰り上がりが発生します。そのため繰り上がり部分は1, 一桁目は0となります。入力が0,1だった場合、繰り上がりが発生せず、一桁目は1となります。

| input | output |
|:------|:-------|
| 0,0   | 0 0    |
| 0,1   | 0 1    |
| 1,0   | 0 1    |
| 1,1   | 1 0    |


### コマンドラインからの入力を受け取る方法

```py:python
# 入力
first, second = list(map(int, input().split(',')))
```

### 解答例

<details><summary>クリックすると解答例が見られます</summary>

```py:
# 入力
first, second = list(map(int, input().split(',')))

# 半加算器
def half_adder(a, b):
    return a & b, a ^ b

carry, sum = half_adder(first, second)
print(carry, cum)
```
入力された二つの数字から、繰り上がりと一桁目、それぞれどのbit演算(回路)を当てれば良いかを考えます。
繰り上がりは、入力値が両方1の場合のみ1となるのでAND演算と対応しています。1桁目は、入力値の片方のみが1となっている場合に1となるのでXOR演算と対応しています。(XOR回路も元々はNOT, AND, OR回路の組み合わせでできていましたね。)

この問題では1bitの足し算に焦点を当てた回路を作りました。これは半加算器と呼ばれています。

</details>

## 問題2
問題1で1桁目の足し算ができるようになりました。今度は2桁目の計算をしたいです。
2桁目に計算する2進数の数字2つA,Bと、下からの繰り上がりCが以下のように与えられます。

```shell:input
0,1,1 # A,B,C
```
2桁目の計算結果に関して、3桁目の繰り上がりと2桁目の数字の計算結果を出力してください。

なお、以下の条件に従ってください。
- bit演算子を使うこと
- 計算量は問いません

### テストケース
テストケースは以下6つです。例えば入力が1,1,0の場合、下位からの繰り上がりは0であるため、結果として1,1の足し算(=半加算器)をすればよく、3桁目の繰り上がりは1, 2桁目の数は0となります。入力が0,1,1の場合、下位からの繰り上がりが存在するため2桁目では繰り上がりと片方の入力値同士で1,1の足し算をすることになります。そのため、3桁目の繰り上がりは1, 2桁目の数は0となります。入力が0,0,1の場合、下位からの繰り上がり分が2桁目の数字として落ち着きます。

| input | output |
|:------|:-------|
| 0,0,0 | 0 0    |
| 0,1,0 | 0 1    |
| 1,1,0 | 1 0    |
| 0,0,1 | 0 1    |
| 0,1,1 | 1 0    |
| 1,1,1 | 1 1    |

### コマンドラインからの入力を受け取る方法

```py:python
# 入力
A, B, C = list(map(int, input().split(',')))
```

### 解答例

<details><summary>クリックすると解答例が見られます</summary>

```py:
# 入力
A, B, C = list(map(int, input().split(',')))

# 半加算器
def half_adder(a, b):
    return a & b, a ^ b

# 全加算器
def full_adder(a, b, c):
    carry1, sum1 = half_adder(a, b)
    carry2, sum2 = half_adder(sum1, c)
    carry = carry1 | carry2
    return carry, sum2

carry, sum = full_adder(A, B, C)
print(carry, sum)
```
まず半加算器を用いて入力値AとBの加算を行い、その結果の和sum1と繰り上がりcarry1を得ます。次に、sum1と下位からの繰り上がりCを半加算器で加算し、その結果の和sum2と繰り上がりcarry2を得ます。最後に、carry1とcarry2のどちらかが1であれば繰り上がりが発生するため、これらをOR演算して最終的な繰り上がりcarryを得ます。

このように、下位からの繰り上がりを考慮した加算器を全加算器と呼びます。

この全加算器を下位から繰り返し再帰的に利用していくことで、2進数での足し算を作ることができます。

これを図示すると下記のようになります。
![full_adder.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/9b8ff607-7813-41ee-2ee5-a24231f1d8b6.png)

</details>

## その他のトピック
これまでの問題は2進数での足し算を考えてきました。では引き算や掛け算、割り算はどのように実現されるのでしょうか？

### 引き算
まず引き算についてです。一旦仕組みをイメージしやすくするために10進数で考えてみます。例えば8 - 6という計算を見ていきます。

引き算を回路で表現するために、昔の人は足し算の原理を利用することを発想しました。 8 - 6は当たり前ですが2となりますね。これを8 + (-6)で計算すると考え、引く方の-6を何らか表現できれば良いということになります。

ここで補数という考え方を導入します。補数とはある数の桁が１つ増える為に必要な数のことです。6の場合、補数は4です。6 + 4 = 10のように桁が増える相方の数をイメージしてください。5の補数は5、3の補数は7です。唐突ですが、引く方の6の補数 = 4と8を足すと12となります。そして12の10の位を無視し、1の位だけを注目すれば確かに、2であるため、8 - 6 = 2と同じ結果になりますね。このように補数を利用すれば、足し算の要領で引き算を実現することができます。

補数の足し算を利用するとなぜ引き算の結果が得られるのかは、不思議なようで当たり前でもあります。以下の式変形を見てください。
8 - 6 = 8 + (-6) = 8 + (10 - 6) - 10 = 8 + 4 - 10 = 2
確かに、補数を使って引き算を求めることができますね。

コンピュータの引き算は、実はこのような原理で動いています。
2進数でも同じで引く方の数の補数さえ分かれば、足し算の原理を利用し、最上位を無視して引き算を実現することができます。
2進数における補数は、実は以下のように機械的に求めることができます。

1. 全ビットを反転させる（1と0を反転させる）
2. 1を加算する

例えば0101(10進数における5)の場合、全ビット反転で1010、ここに1を加算して1011(10進数における11)となります。
0101 + 1011を計算すると、確かに桁が増えて10000(10進数における16)となります。

改めて、8 - 6 を2進数で考えてみましょう。2進数で表記すると、1000 - 110となります。110の補数は全ビット反転で001、ここに1を足して010(10進数における2)となります。
そして、1000に010を足すと1010、最上位を無視して10(10進数における2)となりました。確かに正しい計算結果が得られていますね。

このようにコンピュータの引き算も、補数という概念と足し算の原理を組み合わせることで実現することができます。

### 掛け算と割り算
掛け算と割り算は極端な話、足し算と引き算をひたすら繰り返せば実現することができてしまいます。とはいえ、2の倍数分の変化がある場合はシフト演算を使って簡単に表現することもできます。例えば、0110(10進数における6)を2倍するには、左に1桁ずらして右端に0を埋めます。そうすると1100(10進数における12)となります。
これは、位置に応じて2進数の位の大きさが2の倍数で対応しているからですね。2の2乗 = 4倍したい場合は左に2つ分シフト演算することで求めることができます。

## おわりに
勉強会第4回の内容として、bit演算をテーマに学びました。

物理的な回路の原理は電圧のon, offの流れのパターンを作り出すことができます。これに2進法の数字を対応させ、位の概念を乗せることで計算処理を作ることができます。2進数を基に全加算器を利用して足し算を実現し、補数を利用して引き算を実現できます。掛け算割り算は、足し算引き算の繰り返しで計算可能ですが、シフト演算を使うと簡単に2の倍数分の乗除を実現できたりします。

今回のテーマを通して、コンピュータはどのように計算を可能にしているのか？のイメージが持てるようになったかと思います。普段私たちが当たり前のように利用しているコンピュータの裏側を少しは覗けたのではないでしょうか？

<!-- TODO: 記事リンク貼る -->
次回の勉強会は連想配列についてです。