---
title: Dockerfile内でCPUを条件分岐する方法
tags:
  - ShellScript
  - Docker
private: false
updated_at: '2024-06-29T22:53:21+09:00'
id: dd87b140cd84dd2fe837
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

こちらの記事では、Dockerfile 内で CPU を条件分岐する方法について書きます。

筆者は以前、M1 チップの Macbook で開発を進める際、プロジェクトの Dockerfile の環境構築がうまくできない事象に遭遇しました。
というのも、そのプロジェクトではそれまでは intel チップを前提として環境構築していたわけですが、そのまま M1 チップに適用できないようなのです。調べていくと CPU のアーキテクチャごとに、インストールする必要があるパッケージや設定が異なる場合があることが分かりました。

暫定対応として、ローカル環境のみ Dockerfile を書き換えることで開発自体を進めることはできます。しかし、管理が面倒なので、環境の差異を吸収できるような一元的な Dockerfile を作りたいという要望がありました。それを実現するために、ShellScript を使って CPU を判定し、処理を分岐することを考えました。

# 書き方

以下のように書くことができます。

```dockerfile:Dockerfile
# 共通のパッケージをインストール
RUN apt-get update && apt-get install -y curl

# アーキテクチャごとのパッケージをインストール
RUN if [ "$(uname -m)" = "aarch64" ]; then \
    apt-get install -y package-for-arm; \
elif [ "$(uname -m)" = "x86_64" ]; then \
    apt-get install -y package-for-x86; \
fi
```

RUN 命令は、Dockerfile 内でシェルコマンドを実行するための命令です。`uname -m`コマンドは現在のマシンのアーキテクチャを返します。`aarch64`は ARM64、つまり M1 Mac であることを判定できます。`x86_64`は Intel/AMD の 64 ビットアーキテクチャを指します。

このように Dockerfile 内では、ShellScript を用いてマシンのアーキテクチャに応じた条件分岐を書くことができます。
