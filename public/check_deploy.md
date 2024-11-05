---
title: featureブランチに最新のdevelopが取り込まれているかチェックする方法
tags:
  - Git
  - GitHub
  - CloudBuild
  - CICD
  - GoogleCloud
private: false
updated_at: '2024-05-20T22:55:05+09:00'
id: b0c889c1699621ff67da
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

こちらの記事では、Git コマンドを使って feature ブランチに最新の develop ブランチの変更が取り込まれているかチェックする方法を図解入りで書きます。

# 背景

関わっていたプロジェクトで CloudBuild を使った CICD を構成していました。開発フローは develop ブランチ から feature ブランチを切って開発を進めています。

そこで、staging 環境上で QA をするために、feature ブランチを指定してデプロイすることがありました。
しかし、最新の develop ブランチの内容を feature ブランチに取り込んでいないと、予期せぬ不具合が起きることがあります。
つまり、feature ブランチはデプロイする際には、最新の develop ブランチを取り込んでいる(=Update branch)必要がありました。

GitHub 上では、develop ブランチに保護ルールを設定することで、feature ブランチが最新の develop ブランチの変更を取り込んでいないと develop ブランチにマージできないように設定することはできます。しかし、ブランチ指定デプロイをする場合は、CloudBuild 内の処理でそれと同様の内容を実現する必要があります。

# 処理

以下の cloudbuild.yaml のステップを使用して、feature ブランチが最新の develop ブランチの変更を取り込んでいるかを確認します。

```yaml:cloudbuild.yaml
steps:
- name: 'gcr.io/cloud-builders/git'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    git remote set-url origin https://$$GITHUB_PERSONAL_ACCESS_TOKEN@github.com/organization/project.git
    git fetch --unshallow
    if [ $(git rev-parse origin/develop) != $(git merge-base origin/$BRANCH_NAME origin/develop) ]; then
      echo "最新のdevelopがブランチに取り込まれていません。Update branchしてください。"
      exit 1
    fi
  secretEnv: ['GITHUB_PERSONAL_ACCESS_TOKEN']
  dir: '$_REPO_NAME'
```

それぞれのコードの説明です。
まずは、リモートリポジトリの URL を設定します。認証には個人アクセストークンを使用しています。
`git remote set-url origin https://$$GITHUB_PERSONAL_ACCESS_TOKEN@github.com/organization/project.git`

次に、リポジトリの履歴を全て取得します。
`git fetch --unshallow`

時間がかかる場合は、以下のように履歴の長さを指定することもできます。
`git fetch --depth=100`
筆者が関わったプロジェクトでは、`--unshallow`では 30s、`--depth=100`で 10s くらいの処理時間がかかっていました。

リモートリポジトリの develop ブランチの最新コミットハッシュを取得します。
`git rev-parse origin/develop`

CloudBuild 実行時に指定したリモートリポジトリの $BRANCH_NAME ブランチと develop ブランチの共通の祖先のコミットハッシュを取得します。
`git merge-base origin/$BRANCH_NAME origin/develop`

この両者が一致していなければ最新の develop が feature ブランチに取り込まれていないと判断されるわけですが、図解すると以下のようになります。

一つ目は一致していないパターンです。
![不一致パターン.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/fd22ea30-52b6-6408-16e6-72931c04b2e1.png)

develop ブランチから feature ブランチが切られ、開発が進んでいます。feature ブランチは develop の古いコミットから分岐していますね。
最新の develop ブランチのコミットが feature ブランチには取り込まれていないのでビルドを中止します。

二つ目は一致しているパターンです。
![一致パターン.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/9dc44558-0929-31ea-9891-8a39c218b064.png)

develop ブランチから feature ブランチへのマージコミット(=Update branch)が追加されています。develop ブランチの最新コミットが feature ブランチとの分岐点となっています。
つまり、最新の develop ブランチが feature ブランチに取り込まれていることを表すのでビルドを続けます。

このような処理を追加することで、最新の develop ブランチが feature ブランチに取り込まれているかどうかを判定することができます。

# まとめ

この記事では、Git コマンドを使って最新の develop ブランチが feature ブランチに取り込まれているかどうかを判定する方法を書きました。

皆さんのお役に立てれば幸いです。
