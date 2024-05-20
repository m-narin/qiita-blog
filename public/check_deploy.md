---
title: featureブランチに最新のmainが取り込まれているかチェックする方法
tags:
  - Git
  - GitHub
  - CloudBuild
  - CICD
  - GoogleCloud
private: false
updated_at: '2024-05-20T19:58:14+09:00'
id: b0c889c1699621ff67da
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

こちらの記事では、Git コマンドを使って feature ブランチに最新の main ブランチの変更が取り込まれているかチェックする方法を図解入りで書きます。

# 背景

関わっているプロジェクトで CloudBuild を使った CICD を構成していました。開発フローは main ブランチ から feature ブランチを切って開発を進めています。

そこで、staging 環境上で QA をするために、feature ブランチを指定してデプロイすることがありました。
しかし、最新の main ブランチの内容を feature ブランチに取り込んでいないと、予期せぬ不具合が起きることがあり、feature ブランチを常に Update branch(=最新の main ブランチを取り込む)する必要がありました。

GitHub 上では、main ブランチに保護ルールを設定することで、feature ブランチが最新の main ブランチの変更を取り込んでいないと main ブランチにマージできないように設定することはできます。しかし、ブランチ指定デプロイをする場合は、CloudBuild 内の処理でそれと同様の内容を実現する必要があります。

# 処理

以下の cloudbuild.yaml のステップを使用して、feature ブランチが最新の main ブランチの変更を取り込んでいるかを確認します。

```yaml:cloudbuild.yaml
steps:
- name: 'gcr.io/cloud-builders/git'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    git remote set-url origin https://$$GITHUB_PERSONAL_ACCESS_TOKEN@github.com/organization/project.git
    git fetch --unshallow
    if [ $(git rev-parse origin/main) != $(git merge-base origin/$BRANCH_NAME origin/main) ]; then
      echo "最新のmainがブランチに取り込まれていません。Update branchしてください。"
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

リモートリポジトリの main ブランチの最新コミットハッシュを取得します。
`git rev-parse origin/main`

CloudBuild 実行時に指定したリモートリポジトリの $BRANCH_NAME ブランチと main ブランチの共通の祖先のコミットハッシュを取得します。
`git merge-base origin/$BRANCH_NAME origin/main`

この両者が一致していなければ最新の main が feature ブランチに取り込まれていないと判断されるわけですが、図解すると以下のようになります。

一つ目は一致していないパターンです。
![不一致パターン.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/5692fdd4-8514-7dee-abca-59ebbdedcc5a.png)

main ブランチから feature ブランチが切られ、開発が進んでいます。feature ブランチは main の古いコミットから分岐していますね。
最新の main ブランチのコミットが feature ブランチには取り込まれていないのでビルドを中止します。

二つ目は一致しているパターンです。
![一致パターン.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/02a9b8cc-2c9f-bb50-72cc-29e2edf0b96a.png)

main ブランチから feature ブランチへのマージコミット(=Update branch)が追加されています。main ブランチの最新コミットが feature ブランチとの分岐点となっています。
つまり、最新の main ブランチが feature ブランチに取り込まれていることを表すのでビルドを続けます。

このような処理を追加することで、最新の main ブランチが feature ブランチに取り込まれているかどうかを判定することができます。

# まとめ

この記事では、Git コマンドを使って最新の main ブランチが feature ブランチに取り込まれているかどうかを判定する方法を書きました。

皆さんのお役に立てれば幸いです。
