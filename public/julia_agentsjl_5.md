---
title: 【Julia】Agents.jlを利用したマルチエージェントシミュレーション⑤森林火災
tags:
  - Julia
  - agents.jl
private: false
updated_at: '2022-10-31T22:52:29+09:00'
id: a975ddca8c04748544eb
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに
今回はAgents.jlの森林火災の例題を見ていきます。セルオートマトンで森林火災の様子を離散的に表現しています。

![forest.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/f2da889d-fef2-e623-fa59-f10fe262b9d2.gif)

▼URL

https://juliadynamics.github.io/AgentsExampleZoo.jl/dev/examples/forest_fire

# Agentの設定

```julia:
using Agents, Random
using InteractiveDynamics
using CairoMakie

@agent Automata GridAgent{2} begin end
```

セルオートマトンはAgentを必要とせず、空間やモデルに焦点を当てます。今後モデルステップを実行するために、ここではダミーエージェントを定義しています。

# モデルの定義

```julia:
function forest_fire(; density = 0.7, griddims = (100, 100), seed = 2)
    space = GridSpace(griddims; periodic = false, metric = :manhattan)
    rng = Random.MersenneTwister(seed)

    # Empty = 0, Green = 1, Burning = 2, Burnt = 3
    forest = ABM(Automata, space; rng, properties = (trees = zeros(Int, griddims),))
    for I in CartesianIndices(forest.trees)
        # 左端の木のおよそ7割を火状態にする
        if rand(forest.rng) < density
            forest.trees[I] = I[1] == 1 ? 2 : 1
        end
    end
    return forest
end

forest = forest_fire()
```

グリッド空間にtreesという2次元配列を作成します。それがグリッド空間のMapを表していて、0~3の4種類の数字がそれぞれの空間の状態を表しています。`metric = :manhattan`は東西南北4方向を隣接している距離と見做すことを表しています。

※GridSpaceSingleは利用できなかったので、例題と異なりますがGridSpaceを利用しています。

# ステップ関数

trees配列の各要素を更新していきます。

```julia:
function tree_step!(forest)
    # Find trees that are burning (coded as 2)
    for I in findall(isequal(2), forest.trees)
        for idx in nearby_positions(I.I, forest)
            # If a neighbor is Green (1), set it on fire (2)
            if forest.trees[idx...] == 1
                forest.trees[idx...] = 2
            end
        end
        # Finally, any burning tree is burnt out (2)
        forest.trees[I] = 3
    end
end
```

まずtrees配列から火状態の要素を探します。次に`nearby_positions`メソッドを利用し、火状態の木の四方近隣に木があったら火状態に変更します。また、火状態の木を炎症後の状態にします。


# ステップの実行

ステップを任意の数だけ実行し火状態の木をカウントします。

1step実行
```julia:
Agents.step!(forest, dummystep, tree_step!, 1)
count(t == 3 for t in forest.trees)
```
-> 70

10step実行
```julia:
Agents.step!(forest, dummystep, tree_step!, 10)
count(t == 3 for t in forest.trees)
```
-> 560

# データの収集

火状態の木の割合を計測する関数を作成し、シミュレーションを実行します。その計測結果をdataframeに格納して表示します。

```julia:
forest = forest_fire(griddims = (20, 20))
burnt_percentage(f) = count(t == 3 for t in f.trees) / prod(size(f.trees))
mdata = [burnt_percentage]

_, data = run!(forest, dummystep, tree_step!, 10; mdata)
data
```

# 画像を作成

各セルの値に応じて色をつけて出力します。

```julia:
forest = forest_fire()
Agents.step!(forest, dummystep, tree_step!, 1)

plotkwargs = (
    add_colorbar = false,
    heatarray = :trees,
    heatkwargs = (
        colorrange = (0, 3),
        colormap = cgrad([:white, :green, :red, :darkred]; categorical = true),
    ),
)
fig, _ = abmplot(forest; plotkwargs...)
fig
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/0c8c0527-feaf-f5ed-5d9b-9ee2b7138b94.png)


# アニメーションを作成

```julia:
forest = forest_fire(density = 0.7, seed = 10)
abmvideo(
    "forest.gif",
    forest,
    dummystep,
    tree_step!;
    as = 0,
    framerate = 5,
    frames = 20,
    spf = 5,
    title = "Forest Fire",
    plotkwargs...,
)
```

5x20step分のframeを作成してアニメーションを動画に保存します。

![forest.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/f2da889d-fef2-e623-fa59-f10fe262b9d2.gif)

# おわりに
セルオートマトンの森林火災をagents.jlで実装しました。空間の定義や火災が広がるロジックを定義して、視覚化する流れを理解できる例題だと思います。色々ロジックやパラメータを変えたりしてみてください。
