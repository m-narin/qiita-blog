---
title: 新卒エンジニア勉強会-連想配列
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
新卒エンジニア同士で実施している勉強会の第5回目の記事になります。
今回のテーマは連想配列についてです。

<!-- TODO: 記事リンク貼る -->
| 回     | テーマ | 記事リンク |
|:-----------|:------------|:------------|
| 第1️回      | 二分探索      | <coming soon>    |
| 第2回      | ソートアルゴリズム  |  <coming soon>     |
| 第3回      | 暗号化           |  <coming soon>     |
| 第4回      | bit演算          |  <coming soon>    |
| **第5回**  | **連想配列**      | <coming soon>   |
| 第6回      | グラフ理論        |  <coming soon>   |

## 前提
連想配列とは、キーバリュー型のデータ構造のことを指します。Pythonで言うと辞書型のデータ構造のことです。

```py:python
dict_person = {'name' : 'tanaka', 'age' : 25, 'email' : 'a@gmail.com'}
```

これは値に対して一意のキーを割り当てたデータ構造であり、`dict_person['age']`のように任意のキー名の値を取り出すことができます。連想配列はデータに名前を付けて扱いやすくできるメリットがありますが、実は内部的にO(1)の計算量で情報を取得することができるメリットもあります。通常の配列データから任意の値を取得する場合は検索するために全探索や二分探索といった操作が必要となり計算量が発生します。連動配列はハッシュ関数を内部的に使用することで検索せずとも一発で欲しい情報を取り出すことができるのです。

ハッシュ関数とは、なんらかのデータの集合を重複のない数字に変換する関数です。連想配列の場合、キー集合に対してハッシュ関数を実行します。そして変換された数字を、配列のインデックスとして利用することで一発で欲しい情報のアドレスをGETできるというわけです。

![hash.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/4f35f856-385d-3579-4480-f2ed4bbb84e8.png)

## 問題1
ハッシュ関数には様々な実装方法がありますが、まずは簡単なハッシュ関数を作りましょう。
テストケースの3つのデータを、長さ8の配列に格納することを考えます。データをどのindexに格納するか決めるためのハッシュ関数を作ります。以下の対応表をもとに、inputの各アルファベットを数字に変換して、和をとってください。この数字(ハッシュ値)を、さらに8で割ったあまりを出力してください。これが格納先のindexになります。

```py:python
{
  "A": 1,
  "B": 2,
  "C": 3,
  "D": 4,
  "E": 5,
  "F": 6,
  "G": 7,
  "H": 8,
  "I": 9,
  "J": 10,
  "K": 11,
  "L": 12,
  "M": 13,
  "N": 14,
  "O": 15,
  "P": 16,
  "Q": 17,
  "R": 18,
  "S": 19,
  "T": 20,
  "U": 21,
  "V": 22,
  "W": 23,
  "X": 24,
  "Y": 25,
  "Z": 26
}
```

なお、以下の条件に従ってください。
- 組み込みメソッドは使わないこと。つまり、分岐、繰り返し処理を自分で書くこと。
  - Array.findなどは使わない
- 解法を調べるためにchatgpt、インターネットは使用しないこと
  - 文法を調べることのみ可
- 計算量は問いません

### テストケース
```shell:input1
SATO
```

```shell:output1
7
```

```shell:input2
TANAKA
```

```shell:output2
0
```

```shell:input3
WATANABE
```

```shell:output3
3
```

### コマンドラインからの入力を受け取る方法

```py:python
# 入力
name = str(input())
```

### 解答例

<details><summary>クリックすると解答例が見られます</summary>

```py:
# 入力
name = str(input())

mapping = {
  "A": 1,
  "B": 2,
  "C": 3,
  "D": 4,
  "E": 5,
  "F": 6,
  "G": 7,
  "H": 8,
  "I": 9,
  "J": 10,
  "K": 11,
  "L": 12,
  "M": 13,
  "N": 14,
  "O": 15,
  "P": 16,
  "Q": 17,
  "R": 18,
  "S": 19,
  "T": 20,
  "U": 21,
  "V": 22,
  "W": 23,
  "X": 24,
  "Y": 25,
  "Z": 26
}

sum = 0
for char in name:
    sum += mapping[char]

index = sum % 8
    
print(index)
```
繰り返し処理を用いて文字列から文字を一つずつ取り出し、対応する数字を加算していきます。合計値を8で割った余りを出力します。
三つのテストケースの場合、たまたま8で割った余りを求めるアルゴリズムを使えば7, 0, 3のように重複のない数字に変換することができました。

</details>

## 問題2
問題1で作ったハッシュ関数を使って、以下のような首都の連想配列を自作します。

```py:python
capital = {
  "JAPAN": "Tokyo",
  "USA": "Washington D.C.",
  "SPAIN": "Madrid"
}
```

手順は以下です。
1. 長さ8の配列を初期化する
2. 値を代入するメソッドsetを作成する
  a. keyとvalueを引数にとる
  b. keyを問題1で作成したhash関数にかけて、indexに変換する
  c. indexを指定して配列へ値を代入する
3. 値を取得するメソッドgetを作成する
  a. keyを引数にとる
  b. keyを同じくhash関数にかけてindexを得る
  c. indexを指定して配列から値を取得する

なお、以下の条件に従ってください。
- 組み込みメソッドは使わないこと。つまり、分岐、繰り返し処理を自分で書くこと。
  - Array.findなどは使わない
- 解法を調べるためにchatgpt、インターネットは使用しないこと
  - 文法を調べることのみ可
- 計算量は問いません

### テストケース

まずは以下を実行して値を代入してください。

```python:
set("JAPAN", "Tokyo")
set("USA", "Washington D.C.")
set("SPAIN", "Madrid")
```

その後、get関数を実行して値を取得できることを確認しましょう。

```shell:code
get("JAPAN")
get("USA")
get("SPAIN")
```

```shell:output1
Tokyo
Washington D.C.
Madrid
```

### コードの最後に以下のメソッドを実行します。

```py:python

set("JAPAN", "Tokyo")
set("USA", "Washington D.C.")
set("SPAIN", "Madrid")

get("JAPAN")
get("USA")
get("SPAIN")
```

### 解答例

<details><summary>クリックすると解答例が見られます</summary>

```py:
array = [0] * 8

mapping = {
  "A": 1,
  "B": 2,
  "C": 3,
  "D": 4,
  "E": 5,
  "F": 6,
  "G": 7,
  "H": 8,
  "I": 9,
  "J": 10,
  "K": 11,
  "L": 12,
  "M": 13,
  "N": 14,
  "O": 15,
  "P": 16,
  "Q": 17,
  "R": 18,
  "S": 19,
  "T": 20,
  "U": 21,
  "V": 22,
  "W": 23,
  "X": 24,
  "Y": 25,
  "Z": 26
}

def hash(key):
  sum = 0
  for char in key:
      sum += mapping[char]

  index = sum % 8
  return index

def set(key, value):
    index = hash(key)
    array[index] = value

def get(key):
    index = hash(key)
    print(array[index])

set("JAPAN", "Tokyo")
set("USA", "Washington D.C.")
set("SPAIN", "Madrid")

get("JAPAN")
get("USA")
get("SPAIN")
```
指示通りにset, get関数を実装し、出力結果も無事各首都を出力することができました。ちなみに今回のテストケースでは`array = [0, 'Washington D.C.', 'Tokyo', 'Madrid', 0, 0, 0, 0]`となります。キー名にハッシュ関数を実行し、キー名に対応する数字に変換します。この数字は0~7の間を取りますが、今回はたまたま重複がありませんでした。そのため、配列のindexとして利用できるようになり、O(1)の計算量、つまり一発で値を取り出すことができたというわけです。

</details>

衝突の話
## 問題3

## 解説


## おわりに
勉強会第5回の内容として、連想配列をテーマに学びました。

<!-- TODO: 記事リンク貼る -->
次回の勉強会はグラフ理論についてです。

