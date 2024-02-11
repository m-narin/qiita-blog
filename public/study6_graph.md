---
title: 新卒エンジニア勉強会-グラフ理論
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
新卒エンジニア同士で実施している勉強会の第6回目の記事になります。
今回のテーマはグラフ理論についてです。

<!-- TODO: 記事リンク貼る -->
| 回     | テーマ | 記事リンク |
|:-----------|:------------|:------------|
| 第1️回      | 二分探索      | <coming soon>    |
| 第2回      | ソートアルゴリズム  |  <coming soon>     |
| 第3回      | 暗号化           |  <coming soon>     |
| 第4回      | bit演算          |  <coming soon>    |
| 第5回      | 連想配列      | <coming soon>   |
| **第6回**  | **グラフ理論**   |  <coming soon>   |

## 前提
グラフ理論とはネットワークや組織、経路のようなグラフ構造を扱うための学問です。

![graph.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/394dc9a0-55a4-38c4-8f27-a3be3eb4d42b.png)

グラフ構造は、ノード(点)とエッジ(線)から成るデータ構造です。ある頂点に何本の線が繋がっているかを次数と言います。

古くからは数学的なアプローチによる研究が行われてきました。
起源は、オイラーという数学者がケーニヒスベルクにかかる7つの橋を全て渡って元に戻ってこられるか、という問題を解くことから始まりました。これは一筆書きと同じ意味を指します。

![river.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/32cafc43-ff1a-7104-87bc-69b6b74013e0.png)

![river_graph.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/d685b68b-2ebc-3c2f-fa12-684d54e90329.png)

結論としてこれらの橋は一筆書きできません。一筆書きができる条件は以下のように求められています。

- すべての頂点の次数が偶数
- 次数が奇数である頂点の数が2で、残りの頂点の次数は全て偶数

現代では、コンピュータリソースの発展により大規模なグラフ構造=ネットワークを計算できるようになってきました。世の中には、ネットワーク構造でモデル化できる事柄がたくさんあります。

- 路線図、乗り換え案内
- カーナビ、経路探索
- インターネット、検索
- SNS、フォロー
- ...

これらのネットワークを考察するにあたっては、グラフ構造をコンピュータが計算できるデータ構造として定義する必要があります。点と線、そしてそのつながりをどのように表現できるのかを見ていきましょう。

## 問題1
M頂点、N辺の無向グラフが与えられます。無向とは線に向きが存在しないことを意味します。

自分自身より頂点番号が小さい隣接頂点がちょうど1つ存在する頂点の個数はいくつかを出力してください。

```shell:input
5 6 // M, N
1 2 // 頂点1と頂点2が線で結ばれている。以下同様。
1 3
3 2
5 2
4 2
4 5
```

上記のテストケースは以下ように理解します。
まず、頂点数が5なので5つの点を用意します。

![graph_test1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/ab6039ab-80c2-3e88-4110-811ced45031a.png)

線の数が6本あるので、6行分の線の情報があります。2行目は頂点1と頂点2が線で結ばれていることを表します。

![graph_test2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/996d8237-5f0c-c44d-30f2-a847d8dc9e24.png)

以下同様に6行分の線を書くと最終的に以下ようなグラフになります。

![graph_test3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/d85ffb3e-c19d-5e19-c079-c2cdc272dbcd.png)

「自分自身より頂点番号が小さい隣接頂点がちょうど1つ存在する頂点の個数」を次のように数えます。
まず1の頂点に注目すると、自分より頂点番号が小さいものが存在しないので対象外です。
2の頂点を見ると、繋がっている点として1があるので自分より頂点番号が小さい点が一つだけあることになり題意に当てはまります。
3の頂点を見ると、1,2と繋がっていて自分より小さい点2つと繋がっているので題意に当てはまりません。
4の頂点を見ると、2,5と繋がっていて自分より小さい点が一つだけあるので、題意に当てはまります。
5の頂点を見ると、2,4と繋がっていて自分より小さい点2つと繋がっているので題意に当てはまりません。

結果として題意を満たす頂点は2,4の2つなので回答は2となります。

なお、以下の条件に従ってください。
- テストケースは全て満たすこと
- 組み込みメソッドは使わないこと。つまり、分岐、繰り返し処理を自分で書くこと。
  - Array.findなどは使わない
- 解法を調べるためにchatgpt、インターネットは使用しないこと
  - 文法を調べることのみ可
- 計算量は問いません

### テストケース
```shell:input1
5 6 
1 2 
1 3
3 2
5 2
4 2
4 5
```

```shell:output1
2
```

```shell:input2
2 1
1 2
```

```shell:output2
1
```

```shell:input3
7 18
7 2
1 6
5 2
1 3
7 6
5 3
5 6
5 4
1 7
2 6
3 4
5 1
4 7
4 6
5 7
3 2
4 2
1 4
```

```shell:output3
0
```

### コマンドラインからの入力を受け取る方法

```py:python
# 入力
# M頂点、N辺の数を受け取る
M, N = map(int, input().split())

# N辺分の頂点の結びを受け取る
for i in range(N):
    a, b = map(int,input().split())
```

### 解答例

<details><summary>クリックすると解答例が見られます</summary>

グラフ構造の表現でよく用いられるデータ構造は隣接行列、隣接リストがあります。

隣接行列を用いた方法

```py:
# 入力
M, N = map(int, input().split())

# 隣接行列の初期化
adjacency_matrix = [[0 for _ in range(M)] for _ in range(M)]

for i in range(N):
    a, b = map(int,input().split())
    adjacency_matrix[a-1][b-1] = 1
    adjacency_matrix[b-1][a-1] = 1

count = 0
for i in range(M):
    smaller_neighbors = 0
    for j in range(i):
        if adjacency_matrix[i][j] == 1:
            smaller_neighbors += 1
    if smaller_neighbors == 1:
        count += 1

print(count)
```

```py:テストケース1の隣接行列
[
 [0, 1, 1, 0, 0], 
 [1, 0, 1, 1, 1], 
 [1, 1, 0, 0, 0],
 [0, 1, 0, 0, 1],
 [0, 1, 0, 1, 0]
]
```

隣接行列はグラフ構造を二次元配列で表現し、繋がっているか否かを0,1で表します。例えば、1行目の配列は頂点1が2,3と繋がっていることを表します。テストケースの隣接行列を作成したら、隣接行列を頂点ごとにループで回し、自分自身より小さい頂点を繋がっている線をカウントします。それがちょうど一つだけ存在する場合にカウントし、結果を出力します。

隣接リストを用いた方法
```py:
# 入力
M, N = map(int, input().split())

# 隣接リストの初期化
adjacency_list = [[] for _ in range(M)]

for i in range(N):
    a, b = map(int,input().split())
    adjacency_list[a-1].append(b)
    adjacency_list[b-1].append(a)

count = 0
for i in range(M):
    smaller_neighbors = 0
    for j in adjacency_list[i]:
        if j < i+1:
            smaller_neighbors += 1
    if smaller_neighbors == 1:
        count += 1

print(count)
```

```py:テストケース1の隣接リスト
[[2, 3], [1, 3, 5, 4], [1, 2], [2, 5], [2, 4]]
```

隣接リストもグラフ構造を二次元配列で表現し、インデックスを頂点をみなして繋がっている頂点を配列として持ちます。例えば、1つ目の要素=頂点1は、2,3の頂点と繋がっていることを表します。テストケースの隣接リストを作成したら、隣接リストをループで回し、各頂点自分自身より小さい頂点を繋がっている線をカウントします。それがちょうど一つだけ存在する場合にカウントし、結果を出力します。

この問題でグラフ構造の2種類の表現方法を学びました。それぞれには特徴があります。計算量を考えると、隣接行列はO(N^2)、隣接リストはO(N+M)となるので隣接リストの方が効率的となります。しかし、隣接行列は情報量が多い分、できることの幅が広い表現形式です。例えば、線にコストが発生するような情報(電車の運賃など)を扱いたい場合、任意の数字を各マスに置くことで表現することができます。これを特に距離行列を言ったりします。また、行列形式なので色々な数学演算が可能となります。

</details>

## 問題2
グラフ構造を応用して簡単なカーナビを作りましょう。
家から温泉まで行くには以下のような経路が存在します。

![graph_question2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/9ba60725-28e1-fd32-9686-674a13b2b617.png)

それぞれの線には距離が存在し、0と1の距離は7となります。距離情報を踏まえて家から温泉までの最短経路を求めてください。

家から温泉までたどり着く経路には例えば0→3→4があり、距離は9です。しかし、0→2→1→4の経路を辿る場合距離は7となり、実はこちらが最短経路となります。

なお、入力形式は以下とします。M頂点、N辺の重み付き無向グラフが与えられます。
始点Sから終点Eまでの最短経路を求めます。

```shell:input
5 7 0 4 // M, N, S, E
0 1 7 // 頂点0と頂点1が距離7の線で結ばれている。以下同様。
0 2 4
0 3 3 
1 2 1
1 4 2
2 4 6
3 4 6
```

なお、以下の条件に従ってください。
- テストケースは全て満たすこと
- 組み込みメソッドは使わないこと。つまり、分岐、繰り返し処理を自分で書くこと。
  - Array.findなどは使わない
- 解法を調べるためにchatgpt、インターネットは使用しないこと
  - 文法を調べることのみ可
- 計算量は問いません

```shell:input1
5 7 0 4
0 1 7
0 2 4
0 3 3 
1 2 1
1 4 2
2 4 6
3 4 6
```

```shell:output1
[0, 2, 1, 4]
7
```

```shell:input2
4 3 0 3
0 1 3
1 2 4
2 3 5
```

```shell:output2
[0, 1, 2, 3]
12
```

```shell:input3
6 9 0 5
0 1 3
0 2 2
1 2 1
1 3 2
2 3 3
2 4 2
3 5 1
4 5 2
1 4 3
```

```shell:output3
[0, 2, 4, 5]
6
```

### コマンドラインからの入力を受け取る方法

```py:python
# 入力
M, N, S, E = map(int, input().split())

# N辺分の頂点の結びと距離を受け取る
for i in range(N):
    a, b, c = map(int,input().split())
```

### 解答例

<details><summary>クリックすると解答例が見られます</summary>

```py:
# 入力
M, N, S, E = map(int, input().split())

# 隣接行列の初期化
INF = float('inf')
adjacency_matrix = [[INF for _ in range(M)] for _ in range(M)]

# 距離行列の作成
for i in range(M):
    adjacency_matrix[i][i] = 0

for i in range(N):
    a, b, c = map(int,input().split())
    adjacency_matrix[a][b] = c
    adjacency_matrix[b][a] = c

# ダイクストラ法
def dijkstra(S, E, adjacency_matrix):
    INF = float('inf')
    dist = [INF] * M
    prev = [None] * M
    visited = [False] * M
    dist[S] = 0

    for _ in range(M):
        min_dist = INF
        for v in range(M):
            if not visited[v] and dist[v] < min_dist:
                min_dist = dist[v]
                current_minimum_node = v
        visited[current_minimum_node] = True

        for neighbor in range(M):
            if not visited[neighbor] and dist[current_minimum_node] + adjacency_matrix[current_minimum_node][neighbor] < dist[neighbor]:
                dist[neighbor] = dist[current_minimum_node] + adjacency_matrix[current_minimum_node][neighbor]
                prev[neighbor] = current_minimum_node

    path = [E]
    while path[-1] != S:
        path.append(prev[path[-1]])
    path = path[::-1]

    return path, dist[E]

path, min_dist = dijkstra(S, E, adjacency_matrix)

print(path)
print(min_dist)
```

最短経路問題を解くにあたって、ダイクストラ法として知られるアルゴリズムを実装します。まず、距離行列を作成します。

```py:テストケース1の距離行列
[
 [0, 7, 4, 3, inf],
 [7, 0, 1, inf, 2],
 [4, 1, 0, inf, 6],
 [3, inf, inf, 0, 5],
 [inf, 2, 6, 5, 0]
]
```
1行目では頂点1が2,3,4にそれぞれの距離で繋がっていることを表します。また、自分自身との間の距離は0, 直接移動できない頂点同士の距離は無限大として置きます。
そして、指定のグラフ(距離行列)に関して、始点から終点までの最短距離をダイクストラ法で解きます。
ダイクストラ法は以下のようなアルゴリズムとなります。
まず変数に関して、
`visited`: 訪れた頂点
`prev`: 各頂点に最短経路で到達する前のノードを格納する配列
`dist`: 各頂点に到達する最短距離を格納する配列
のように定義します。

メインループの中で以下の処理を行います。
1. 未訪問の頂点の中で最短距離が最小となる点を見つけて、visitedを更新する。
2. 上記の点に隣接している未訪問の頂点のうち、距離が現在の`dist`より小さい場合、distとprevを更新する。

具体的な値を見ていきましょう。まず、初期状態では、
`visited`: [False, False, False, False, False]
`prev`: [None, None, None, None, None]
`dist`: [0, inf, inf, inf, inf]
このようになります。

ループ1では、未訪問の最短距離が最小となる頂点は0が選ばれます。0に隣接している未訪問の頂点1,2,3への距離は、いずれも現在の`dist`より小さいので更新されます。
`visited`: [True, False, False, False, False]
`prev`: [None, 0, 0, 0, None]
`dist`: [0, 7, 4, 3, inf]

ループ2では、未訪問の最短距離が最小となる頂点は3が選ばれます。3に隣接している未訪問の頂点4への距離は、現在の`dist[4]`より小さいので更新されます。
`visited`: [True, False, False, True, False]
`prev`: [None, 0, 0, 0, 3]
`dist`: [0, 7, 4, 3, 9]

ループ3では、未訪問の最短距離が最小となる頂点は2が選ばれます。2に隣接している未訪問の頂点は1,4があり、1の場合のみ現在の`dist[1]`より小さいので更新されます。
`visited`: [True, False, True, True, False]
`prev`: [None, 2, 0, 0, 3]
`dist`: [0, 5, 4, 3, 9]

ループ4では、未訪問の最短距離が最小となる頂点は1が選ばれます。1に隣接している未訪問の頂点は4があり、現在の`dist[1]`より小さいので更新されます。
`visited`: [True, True, True, True, False]
`prev`: [None, 2, 0, 0, 1]
`dist`: [0, 5, 4, 3, 7]

ループ5では、未訪問の最短距離が最小となる頂点は4が選ばれます。4に隣接している未訪問の頂点はないので更新されません。
`visited`: [True, True, True, True, True]
`prev`: [None, 2, 0, 0, 1]
`dist`: [0, 5, 4, 3, 7]

これで全ての点が訪問済みとなりループは終了します。最終的な`dist`は始点から各頂点への最短距離を、`prev`は各頂点に最短経路で到達する前のノードを表します。

最短経路を求める際は、終点から辿っていきます。まず、終点4の前のノードは`prev[4]`: 1となります。頂点1の前のノードは`prev[1]`: 2となります。頂点2の前のノードは`prev[2]`: 0となります。つまり、4←1←2←0のように最短経路が選ばれたということになります。

</details>

## おわりに
勉強会第6回の内容として、グラフ理論をテーマに学びました。
