---
title: Pyhton3の標準入出力まとめ+応用
tags:
  - Python
private: false
updated_at: '2022-03-13T00:00:00+09:00'
id: f03d1087c8e0f9ab7b17
organization_url_name: null
slide: false
ignorePublish: false
---
# 標準入力

## 一つの行から、一つの数字を変数に

```py:入力値
100
```

```py:取得方法
N = int(input())
print(N) => 100
```

## 一つの行から、複数の文字列を変数に

※左辺が1変数の場合、配列になる

```py:入力値
m n
```

```py:取得方法
M, N = input().split()
print(M, N) => m n
```

## 一つの行から、複数の数値を変数に

```py:入力値
3 200
```

```py:取得方法~変数ver
M, N = map(int, input().split())
print(M, N) => 3 200
```


```py:取得方法~配列ver
M = list(map(int, input().split())
print(M) => [3,200]
```

## 入力行数を利用し、行分の数値入力を1次元配列に

```py:入力値
3
200
300
400
```

```py:取得方法
N = int(input())
S = [int(input()) for i in range(N)]
print(S) => [200,300,400]
```

## 入力行数を利用し、各行複数の数値入力を2次元配列に

```py:入力
3
100 200 20
100 20 20
500 20 20
```

```py:取得方法
N = int(input())

# リスト内包表記
W = [list(map(int, input().split())) for i in range(N)] 
# 間に半角スペースが無い場合、split()を除けばOK

print(W) => [[100,200,20],[100,20,20],[500,20,20]]
```

## 入力行数を利用し、各行複数のつながった文字入力を2次元配列に

```py:入力
3
abc
def
ghi
```

```py:取得方法
N = int(input())

# リスト内包表記
W = [list(input()) for i in range(N)] 
# 間に半角スペースがある場合、split()をつければOK
print(W) => [['a', 'b', 'c'], ['d', 'e', 'f'], ['g', 'h', 'i']]
```

## 縦方向に並んだ数値列を、指定行分区切って2次元配列に

```py:入力
3
10
20
30
40
50
60
70
80
90
```

```py:取得方法
N = int(input())

# リスト内包表記
S = [[int(input()) for i in range(N)] for i in range(N)]
print(S) => [[10,20,30],[40,50,60],[70,80,90]]
```

# 出力方法

## 一つの行に半角スペースで複数の回答を出す

※`end=""`は末尾にも半角スペースが入るので注意

```py:出力
1 2 3
```

```py:方法
# 回答配列を作成する
ans = [1,2,3] 

# 末尾か否かで分岐
for i in range(len(ans)):
    if i+1 == len(ans):
        print(ans[i])
    else:
        print(ans[i], end=" ")
```

# その他の便利技

## 2次元配列を転置

```
W = [[1,2,3],[4,5,6],[7,8,9]]
W_T = [list(x) for x in zip(*W)]

print(W_T) => [[1, 4, 7], [2, 5, 8], [3, 6, 9]]
```

## 2次元配列各要素早見表

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/06dc15cd-badb-5db9-3124-48fc607f5596.png)
