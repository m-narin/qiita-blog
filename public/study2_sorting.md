---
title: 新卒エンジニア勉強会-ソートアルゴリズム
tags:
  - Python
  - 初心者
  - 競技プログラミング
  - 新人プログラマ応援
private: false
updated_at: '2024-05-20T13:35:39+09:00'
id: 92990e6ef985d7eb9702
organization_url_name: null
slide: false
ignorePublish: false
---

:::note warn
こちらの勉強会は、本形式にまとめています。
https://zenn.dev/mandenaren/books/algorithm-study
:::

## はじめに

新卒エンジニア同士で実施している勉強会の第 2 回目の記事になります。
今回のテーマはソートアルゴリズムについてです。

| 回                               | テーマ                                                                        |
| :------------------------------- | :---------------------------------------------------------------------------- |
| 第 1 回                          | [二分探索](https://qiita.com/MandoNarin/items/50b645309fe272325333)           |
| <font color="Red">第 2 回</font> | [ソートアルゴリズム](https://qiita.com/MandoNarin/items/92990e6ef985d7eb9702) |
| 第 3 回                          | [暗号化](https://qiita.com/MandoNarin/items/4de301502f1050355846)             |
| 第 4 回                          | [bit 演算](https://qiita.com/MandoNarin/items/aff39666dbf63960ea68)           |
| 第 5 回                          | [連想配列](https://qiita.com/MandoNarin/items/711d958a16f7294a0441)           |
| 第 6 回                          | [グラフ理論](https://qiita.com/MandoNarin/items/9976ecff2ecd0521f8b6)         |

## 問題

次の数列を昇順に並べ替えるプログラムを 2 通り以上考えてください。

```shell:input
5,3,2,4,1,6,9,10,8,7
```

なお、以下の条件に従ってください。

- テストケースは全て満たすこと
- 組み込みメソッドは使わないこと。つまり、分岐、繰り返し処理を自分で書くこと。
  - Array.find などは使わない
- 解法を調べるために chatgpt、インターネットは使用しないこと
  - 文法を調べることのみ可
- 計算量は問いません

### テストケース

```shell:input1
5,3,2,4,1,6,9,8,7,10
```

```shell:output1
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

```shell:input2
482,395,749,319,584,738,293,784,492,378
```

```shell:output2
[293, 319, 378, 395, 482, 492, 584, 738, 749, 784]
```

### コマンドラインからの入力を受け取る方法

```py:python
# 入力
numbers = list(map(int, input().split(',')))
```

### 解答例

<details><summary>クリックすると解答例が見られます</summary>

```py:バブルソート
# 入力
numbers = list(map(int, input().split(',')))

def bubble_sort(numbers):
  for i in range(len(numbers)):
    for j in range(len(numbers) - 1):
      if numbers[j] > numbers[j + 1]:
        numbers[j], numbers[j + 1] = numbers[j + 1], numbers[j]
  return numbers

sorted_numbers = bubble_sort(numbers)

print(sorted_numbers)
```

二重ループを使い、要素を並べ替えていきます。
一回目の外側のループで、最も大きい要素が一番右に浮かび上がってきます。
同様に全ての要素に関して、最大の要素が最後尾に固定されていくことを繰り返しながら並び替えていきます。

![sort.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/6e431fa8-2b55-231b-5f92-8db0cfbaa083.png)

```py:選択ソート
# 入力
numbers = list(map(int, input().split(',')))

def selection_sort(numbers):
  for i in range(len(numbers)):
    min_index = i
    for j in range(i+1, len(numbers)):
      if numbers[j] < numbers[min_index]:
        min_index = j
    numbers[i], numbers[min_index] = numbers[min_index], numbers[i]
  return numbers

sorted_numbers = selection_sort(numbers)

print(sorted_numbers)
```

未ソート部分から最小値（または最大値）を選び、それを未ソート部の最初の位置と交換していくアルゴリズムです。

![sentaku.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/f399377a-cb70-3a3f-2868-e11e1a0fe9a4.png)

</details>

## 解説

解答例には 2 通りのソートアルゴリズムを載せましたが、他にも色々な方法があります。
詳細は以下の記事に譲ります。
https://qiita.com/drken/items/44c60118ab3703f7727f

それぞれのアルゴリズムにおいて計算量は元々の数列の状態に応じて変わってくる性質があります。
そのため、実際には条件に応じて様々なソートアルゴリズムを組み合わせたものが利用されています。

例えば、マージソートを挿入ソートを組み合わせた[ティムソート](https://ja.wikipedia.org/wiki/%E3%83%86%E3%82%A3%E3%83%A0%E3%82%BD%E3%83%BC%E3%83%88)は Java, Android, Swift, Rust で採用されているみたいです。

また、明確なソースは見つけられなかったのですが C や Ruby はクイックソートをベースに作られているとの記述がいくつかありました。

## おわりに

勉強会第 2 回の内容として、ソートアルゴリズムをテーマに学びました。

エンジニアリングをしていると、頻繁にソートする場面があります。
日付順に並べ替えたり、五十音順に並べ替えるといった操作をよくしますね。
その背景には、何らかのソートアルゴリズムが動いているはずです。

無味乾燥に.sort メソッドを使うのではなく、背景を想像しながら利用できるとまた楽しさが増すことでしょう。

次回の勉強会は[暗号化](https://qiita.com/MandoNarin/items/4de301502f1050355846)についてです。
