---
title: 【GitHub Actions】同時デプロイして動作確認を効率化
tags:
  - Git
  - GitHub
  - GitHubActions
  - CICD
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

こちらの記事では、GitHub Actions にてテスト環境（ステージングなど）に複数の開発内容を同時デプロイし、動作確認を効率化する方法について書きます。

# 背景

関わっていたプロジェクトでは、開発した内容を個別にステージング環境にデプロイし動作確認する開発フローがありました。
まず、ステージング環境を利用するエンジニアは Slack で使用しますと宣言をします。そして動作確認が完了したら完了しましたと報告し、他のエンジニアに環境を開放します。

しかし、このやり方だと環境占有問題が発生し、待ち時間が発生してしまいます。特に、差し込みで緊急な改修が必要となった場合や、複数機能を同時に開発していたりする場合、環境の競合が発生すると動作確認の待ちが容易に発生します。また、昨今であれば AI により開発速度が向上したため、開発フローにおける動作確認の比重も高まってきています。

さらには、ステージング環境上で Biz の方々による動作レビューを依頼するケースもあるため、時間調整コストも発生します。

このように、動作確認周りの開発フローが効率悪いよねという課題感を持っており、サイクルタイムで言えばレビューから merge までの時間に改善の余地が多分に存在していました。

# 解決方針

解決方針はシンプルに複数の内容をまとめて deploy できるようにすれば良いのでは？という発想から下記の点が満たせれば良いと条件を決めました。
動作確認 フェーズ の生産性向上を期待しています。

- 待ちを気にしないで開発内容を deploy し動作確認できる
- 複数機能まとめて deploy できる
  - 対象は PR から取得する

## 方法

最初に、これから説明する便利ツールを設定することにより、開発者はたったこれだけのステップで上記の機能を享受できるようになります。
実際の処理は GitHub Actions にて設定していきます（後述）。

### step1 動作確認対象の PR に専用 label をつける

TODO: 画像

### step2 専用ワークフローを 実行

TODO: 画像

これを実行することで、専用 label のついた PR（Open or Draft）の内容がまとめて stg-2 に deploy されます。

## 専用ワークフロー

作成するワークフローはこちらになります。label 名は「deploy/staging」とします。

```yaml:multi-deploy-staging.yaml
name: Multi deploy staging

on:
  workflow_dispatch:

jobs:
  update-deploy-branch:
    runs-on: ubuntu-22.04-arm64
    concurrency:
      group: update-multi-deploy-staging
      cancel-in-progress: true
    permissions:
      contents: write
      pull-requests: read
    outputs:
      sha: ${{ steps.get-sha.outputs.sha }}

    steps:
      - name: Checkout full repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set Git config
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Reset deploy/staging branch to develop
        run: |
          git fetch origin develop
          git checkout -B deploy/stgging origin/develop

      - name: List open PRs with label deploy/staging
        run: |
          gh pr list \
            --state=open \
            --label=deploy/staging \
            --json number \
            --jq '.[].number' \
            > pr_lists.json
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Debug Show raw pr_lists.json
        run: |
          echo "=== raw pr_lists.json ==="
          cat pr_lists.json || echo "(none)"
          echo "=== End raw ==="

      - name: Merge each labeled PR
        run: |
          if [ ! -s pr_lists.json ]; then
            echo "No deploy/staging PRs found, skipping merge step."
            exit 0
          fi
          while IFS= read -r pr; do
            echo "Merging PR #$pr…"
            git fetch origin pull/"$pr"/head:pr-"$pr"
            if git merge --no-ff pr-"$pr" -m "Merge PR #$pr"; then
              echo "merged #$pr"
            else
              echo "conflict in #$pr, aborting" >&2
              git merge --abort
              exit 1
            fi
          done < pr_lists.json

      - name: Push to deploy/staging
        run: |
          git push origin deploy/staging --force
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Get current SHA
        id: get-sha
        run: echo "sha=$(git rev-parse HEAD)" >> $GITHUB_OUTPUT

  # ${{ needs.update-deploy-branch.outputs.sha }}で「deploy/staging」ブランチの最新コミットハッシュが取得できます。
  # この後は各々のdeployワークフローを設定する必要があります。
  build:
    needs: update-deploy-branch
    # 各自追記

  migrate:
    needs: update-deploy-branch
    # 各自追記

  deploy:
    needs: update-deploy-branch
    # 各自追記

```

## 専用ワークフロー解説

### 1. List open PRs with label deploy/staging

GitHub CLI を利用し、専用 label のついた Open PR 一覧を取得します。

TODO: 画像

### 2. Reset deploy/staging branch to develop

最新の develop を基準に branch = deploy/staging を作成します。
この deploy/staging ブランチに各 PR の内容を反映していきます。
前回実行時の deploy/staging の内容が残っているため、最新 develop からの差分に合わせるようにします。

### 3. Merge each labeled PR

専用 label のついた PR の差分を、current branch である deploy/staging に取り込んでいきます。
PR 番号から PR の HEAD（最新コミット）を参照し、local のブランチに保存します。

下記のようなログが出ています。
`refs/pull/1520/head -> pr-1520`

PR ごとの差分が競合する場合も考慮する必要があり、もし conflict が発生するようなら処理停止します。
こうなったら従来通り個別 deploy するしかなさそうですが、自チームの運用では 150 回ほど実行され一度もこの状況は発生しませんでした。

### 4. Push to deploy/staging

タイトルの通り、deploy/staging ブランチを push します。

### 5. Get current SHA

deploy/staging ブランチの の最新 commit hash を取得します。

その後、取得された commit hash を用いたりしながら、各自 deploy ワークフローに渡してあげます。

## 特徴と効果

こちらのワークフローを利用することにより、他人の実行を気にせずにいつでも自分の修正内容を deploy し動作確認できるようになります。
group と cancel-in-progress により既に実行中のものはキャンセルし、常に最新のワークフローを実行します。
最新のワークフローは label がついている限り全ての PR を deploy するため、他人の動作確認を邪魔せずに済みます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/7e7399f4-a8ca-4a19-b9e2-b5165f19cf10.png)

上記画像では開発者 A が PR#1 の内容を先に deploy し動作確認していますが、その後開発者 B が PR#2 の内容を動作確認すべく multi-deploy-staging を実行します。そうすると、PR#1 + PR#2 の内容が deploy されるため、開発者 A は PR#1 の内容を引き続き動作確認でき、開発者 B も PR#2 を動作確認できるようになります。

このようにして下記の条件を満足することができ、

- 待ちを気にしないで開発内容を deploy し動作確認できる
- 複数機能まとめて deploy できる
  - 対象は PR から取得する

# まとめ

TODO: まとめ

皆さんのお役に立てれば幸いです。
