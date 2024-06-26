---
title: 新卒エンジニア勉強会-暗号化
tags:
  - Python
  - 初心者
  - 競技プログラミング
  - 新人プログラマ応援
private: false
updated_at: '2024-05-20T13:35:39+09:00'
id: 4de301502f1050355846
organization_url_name: null
slide: false
ignorePublish: false
---

:::note warn
こちらの勉強会は、本形式にまとめています。
https://zenn.dev/mandenaren/books/algorithm-study
:::

## はじめに

新卒エンジニア同士で実施している勉強会の第 3 回目の記事になります。
今回のテーマは暗号化についてです。

| 回                               | テーマ                                                                        |
| :------------------------------- | :---------------------------------------------------------------------------- |
| 第 1 回                          | [二分探索](https://qiita.com/MandoNarin/items/50b645309fe272325333)           |
| 第 2 回                          | [ソートアルゴリズム](https://qiita.com/MandoNarin/items/92990e6ef985d7eb9702) |
| <font color="Red">第 3 回</font> | [暗号化](https://qiita.com/MandoNarin/items/4de301502f1050355846)             |
| 第 4 回                          | [bit 演算](https://qiita.com/MandoNarin/items/aff39666dbf63960ea68)           |
| 第 5 回                          | [連想配列](https://qiita.com/MandoNarin/items/711d958a16f7294a0441)           |
| 第 6 回                          | [グラフ理論](https://qiita.com/MandoNarin/items/9976ecff2ecd0521f8b6)         |

## 問題

あなたはハッカーです。脆弱なサイトから、ユーザーの入力したパスワードを傍受することができました。
しかしパスワードは RSA 暗号で暗号化されているので本当の中身はまだ分かりません。公開鍵は取得済みです。

公開鍵 n,e と、暗号化されたパスワード c が以下のように与えられます。

```shell:input
901 # n
5 # e
836 # c
```

以下の手順に従って暗号文を復号(解読)してください。

1. n は素数 p, q の積です。n = p × q となる p, q を見つける
2. e _ d を (p - 1) _ (q - 1)で割った余りが 1 となるような d を見つける。ただし、d < (p - 1) \* (q - 1)である。
3. c の d 乗を n で割った余りが元の文となる

なお、以下の条件に従ってください。

- 全てのテストケースで暗号が解読できる保証はありません
- 組み込みメソッドは使わないこと。つまり、分岐、繰り返し処理を自分で書くこと。
  - Array.find などは使わない
- 解法を調べるために chatgpt、インターネットは使用しないこと
  - 文法を調べることのみ可
- 計算量は問いません

### テストケース

```shell:input1
901
5
836
```

```shell:output1
777
```

```shell:input2
2773
17
1647
```

```shell:output2
2024
```

```shell:input3
23763750720320755089986836889386752333148454401954967562246067964867289138953959780717079688179130465650799885381128444602378117801374965735319583220896780571277261256156846877426142522785396049700319360493746460388891291877334647769138315576265478137249565382290203874694134519392957422540203022062882075405911121664718704644321479425687111651838911360686725629604293945223524169767648631213310792471748148155314286839441622349839299969847466395263222429344465646709824625925965427682282530676386023451803005481500038573372924827423756149573698159403684701896723624641100816859773212094543180200907204545388240139873
65537
8415193396543311353484421939238676093551842164971427185704097190178024351893340295680735321393780311840244869986682879096195012272166629614593460647812271305235527777754835140274392570370347880248411379950593107597810919502273451084589323176477491051476512881744527931527774798075893681139907243050509632488215183600512761950585193887831590494038281760727275445162394107244799292585041038475269525888152844146387779398177114276753330862352263395304480337880761057765477558514448524077641571808843866618899695005802755036106106076084395503279687316594954590030543862355124606471965783095127625163775336406697681524837
```

```shell:output3
893
```

### コマンドラインからの入力を受け取る方法

```py:python
# 入力
n = int(input())
e = int(input())
c = int(input())
```

### 解答例

<details><summary>クリックすると解答例が見られます</summary>

```py:
# 入力
n = int(input())
e = int(input())
c = int(input())

def decrypt_rsa(n, e, c):
    # pとqを見つける（素因数分解。一つでも素数が見つかれば、自動的にもう片方も一つに定まる）
    p, q = None, None
    for i in range(2, n):
        if n % i == 0:
            p = i
            q = n // i
            break

    if not p or not q:
        return "pとqが見つかりませんでした。"

    # dを見つける
    phi = (p - 1) * (q - 1)
    d = None
    for i in range(1, phi):
        if (i * e) % phi == 1:
            d = i
            break

    if not d:
        return "dが見つかりませんでした。"

    # 暗号文cを復号する
    M = None
    for _ in range(d):
        M = (c ** d) % n

    return M

# 復号されたパスワードを計算
decrypted = decrypt_rsa(n, e, c)

print(decrypted)
```

繰り返し処理を用いて、それぞれの指示に従ったプログラムを書きます。
テストケースは 3 つありますが、3 つとも解読できたでしょうか？

3 つ目の暗号は timeout になるはずです。これが解けたら世界のセキュリティが崩壊します。。。

</details>

## 解説

RSA 暗号は公開鍵暗号方式の一種となります。
なんとなく、公開鍵、秘密鍵という単語は聞いたことのある方も多いとは思いますが、上で見たように数式によって成立しているんですね。

:::note warn
上記の数式でなぜ元の文が復号できるのかの理論的な背景はこちらの記事に詳しいです。(数学の証明になります)
https://qiita.com/reika727/items/215d23bf18e21e3cbc52
:::

攻撃者の視点に立てば、p と q が分かれば暗号を解読することができます。
しかし、3 つ目のテストケースは、617 桁の途方もなく大きい数字(ちなみに無量大数は 68 桁)なので、素因数分解のプログラムが timeout してしまいます。
実際にこれくらいのサイズの公開鍵が SSL 等で利用されています。
また現代では、素因数分解のアルゴリズムはしらみ潰しに調べる O(n)の方法しか見つかっていません。

つまり、巨大な数字の素因数分解の困難さによって RSA 暗号のセキュリティが保証されているのです。これを RSA 仮定と言います。

RSA 暗号は、あくまで現代の技術では解読できないという前提のもと成立している暗号化技術です。もし将来的に量子コンピュータの発展や、新たな素数の法則、新たな素因数分解のアルゴリズムが見つかってしまえば、現代のセキュリティが崩壊してしまう恐れがあることも理解しておく必要があります。

改めて、順を追って RSA 暗号の暗号化と復号化を見ていきましょう。

### step1 公開鍵(n,e)を用意する

まずサーバー側の立場に立って公開鍵を作ります。

1. 互いに素な p,q を任意に選びます。「互いに素」は、共通の約数が 1 以外に存在しない数字のペアを意味します。
2. n = pq とします。
3. (p - 1) \* (q - 1)と互いに素な自然数 e を任意に選びます。
4. e _ d を(p - 1) _ (q - 1)で割った余りが 1 となる自然数 d を任意に選びます。

例えば以下のような数字を選びます。(簡単のため小さい数字を使います。)

1. p = 7、q = 5 とする。
2. n = 7 \* 5 = 35 となる。
3. (p - 1) _ (q - 1) = 6 _ ４ = 24 となるので、e ＝ 11 とする。
4. 11 \* d を 24 で割った余りが 1 となる d を探し、d = 83 とする。(913 ÷ 24 = 38...あまり 1)

この n = 35, e = 11 を公開鍵として公開します。
p = 7, q = 5, d = 83 は秘密鍵となります。

### step2 元の文を暗号化する

次にユーザーの立場に立って、公開鍵を利用して元の文を暗号化します。

1. 元の文 m を e 乗を n で割った余りが暗号文 c となります。

具体的に元の文 m = 12 として計算してみます。

1. c = 12 の 11 乗を 35 で割った余り = 3

よって暗号文 c = 3 がサーバー側に送信されます。

### step3 暗号文を復号する

受信者であるサーバー側の視点に立って、暗号文を復号します。

1. c の d 乗を n で割った余りが元の文となります。

具体的に、当てはめて計算してみます。

1. 3 の 83 乗を 35 で割ったあまり = 12

このように復号することができました。

今までの流れを図示すると以下のようになります。

![スクリーンショット 2024-01-01 1.27.11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/1023910a-c923-20db-797c-2ef303366b4f.png)

n=pq が途方もなく大きいおかげで、攻撃者から通信を保護することができるというわけですね ✨

## おわりに

勉強会第 3 回の内容として、暗号化をテーマに学びました。

普段利用する https(ssl)や ssh も、RSA 暗号のような暗号化技術が基になってプロトコルが制定されています。
なんとなく聞いたことのある公開鍵や秘密鍵の正体がこの記事を通して理解できたと思います。
また一つ、教養が深まりましたね 🙌

次回の勉強会は[bit 演算](https://qiita.com/MandoNarin/items/aff39666dbf63960ea68)についてです。
