---
title: 【Git】複数のGitHubアカウントをプロジェクトごとに使い分ける（gh CLI + direnv）
tags:
  - Mac
  - Git
  - GitHub
  - direnv
  - GH
private: false
updated_at: '2026-06-23T13:51:56+09:00'
id: e50d5749de12ce785ebc
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

仕事と個人、あるいは複数の組織で別々の GitHub アカウントを使っていると、「このプロジェクトでは A アカウント、あっちのプロジェクトでは B アカウント」というように、ディレクトリごとにアカウントを使い分けたくなります。

`gh` CLI には `gh auth switch` で手動切り替えする機能はありますが、プロジェクトを行き来するたびに切り替えるのは地味に面倒で、切り替え忘れて間違ったアカウントで操作してしまう事故も起きがちです。

この記事では、**`cd` するだけでプロジェクトごとに `gh` のアカウントと git のコミット author が自動で切り替わる**環境を、`direnv` を使って作る方法をまとめます。

やりたいことはシンプルにこの2つです。

- `~/projects/project-a` にいるときは `account-a` で `gh` と git を操作する
- それ以外（他のプロジェクト）では普段使いの `account-main` を使う

# よくあるハマりどころ：`GH_TOKEN` 環境変数

まず最初に確認しておきたいのが、`gh` の認証状態です。

```bash
gh auth status
```

このとき、出力に `(GH_TOKEN)` と表示されているアカウントがあったら要注意です。

```
github.com
  ✓ Logged in to github.com account account-main (GH_TOKEN)
    - Active account: true
```

`GH_TOKEN`（または `GITHUB_TOKEN`）環境変数がセットされていると、`gh` はそれを**最優先**で使います。つまりこの状態だと、`gh auth switch` で別アカウントに切り替えようとしても**環境変数が勝って無視されます**。

`.zshrc` などでグローバルに `export GH_TOKEN=...` していないか確認しておきましょう。

```bash
grep -rn "GH_TOKEN\|GITHUB_TOKEN" ~/.zshrc ~/.zprofile ~/.zshenv
```

実はこの「`GH_TOKEN` が最優先される」という挙動こそが、今回の自動切り替えの肝になります。

# 作戦：グローバルはデフォルト、プロジェクトだけ上書き

`gh` には「ディレクトリごとにアカウントを変える」というネイティブ機能はありません。そこで `direnv` を組み合わせます。

`direnv` は、`.envrc` というファイルを置いたディレクトリに `cd` すると、そこに書いた環境変数を自動で読み込んでくれるツールです。ディレクトリから出ると自動でアンロードされます。

方針はこうします。

1. グローバル（`.zshrc`）の `GH_TOKEN` は普段使いの `account-main` のまま残す → **デフォルトアカウント**
2. 特定プロジェクトの `.envrc` だけで `GH_TOKEN` を別アカウントのトークンに**上書き**する

`.envrc` で `export` した環境変数はそのディレクトリ内でグローバルのものを上書きするので、「普段は account-main、このプロジェクトだけ account-b」が自然に実現できます。

# 手順

## 1. direnv のインストールと有効化

```bash
brew install direnv
```

シェルにフックを追加します（zsh の場合は `~/.zshrc` に以下を追記）。

```bash:~/.zshrc
eval "$(direnv hook zsh)"
```

追記したらシェルを開き直すか `source ~/.zshrc` で反映します。

## 2. プロジェクト専用アカウントのトークンを用意する

切り替えたいアカウント（ここでは `account-b`）でログインした状態で、[Personal Access Token (classic)](https://github.com/settings/tokens) を発行します。

スコープは用途に合わせますが、`gh` で PR 操作や CI 連携まで行うなら以下あたりを付けておくと無難です。

- `repo`
- `workflow`
- `read:org`
- `project`

発行された `ghp_xxxxxxxxxxxx` を控えておきます。

## 3. プロジェクトに `.envrc` を置く

切り替えたいプロジェクトの直下に `.envrc` を作成します。

```bash:~/projects/project-a/.envrc
# このプロジェクトは account-b で操作する（グローバルの account-main を上書き）
export GH_TOKEN=ghp_xxxxxxxxxxxx
```

`.envrc` を初めて置いた（または編集した）ときは、安全のため direnv が許可を求めてきます。一度だけ許可します。

```bash
cd ~/projects/project-a
direnv allow
```

これで、このディレクトリにいる間だけ `GH_TOKEN` が `account-b` のトークンに切り替わります。

## 4. トークンを Git 管理から除外する

`.envrc` にはトークンという秘密情報が入るので、**絶対にコミットしてはいけません**。

チームで共有しているリポジトリの場合、リポジトリの `.gitignore` を勝手に書き換えたくないこともあります。そういうときは、自分の環境全体で効く**グローバル gitignore** に追加しておくと、どのリポジトリでも自動的に無視されて便利です。

```bash:~/.config/git/ignore
.envrc
.direnv/
```

`~/.config/git/ignore` は git が標準で参照してくれるパスです（macOS / Linux の場合）。ちゃんと無視されているか確認しておきましょう。

```bash
cd ~/projects/project-a
git check-ignore .envrc   # .envrc と表示されれば無視されている
```

## 5. コミット author もプロジェクトごとに変える

`gh` のアカウントだけでなく、git のコミット author（`user.name` / `user.email`）も分けたいケースは多いです。これは direnv ではなく、**リポジトリローカルの git 設定**で対応するのが確実です。

切り替えたいプロジェクトの中で、`--local` を付けて設定します。

```bash
cd ~/projects/project-a
git config --local user.name  "account-b"
git config --local user.email "you@project.example.com"
```

`--local` なのでこのリポジトリ内だけに効き、他のプロジェクトはグローバル設定（`account-main`）のままです。

> **メモ:** GitHub にコミットを `account-b` 名義として正しく表示させるには、`user.email` を `account-b` アカウントに登録済みのメールにする必要があります。メールを公開したくない場合は、GitHub の [メール設定](https://github.com/settings/emails) で確認できる `12345678+account-b@users.noreply.github.com` 形式の noreply アドレスを使うとよいです。

# 動作確認

最後に、ちゃんと切り替わっているか確認します。

プロジェクト内（`account-b` になっているはず）：

```bash
cd ~/projects/project-a
gh api user --jq '.login'   # => account-b
git config user.name        # => account-b
git config user.email       # => you@project.example.com
```

プロジェクトの外（普段使いの `account-main` のまま）：

```bash
cd ~/other-project
gh api user --jq '.login'   # => account-main
```

`cd` で行き来するだけで、`gh` のアカウントも git の author も自動で切り替わるようになりました 🎉

# おまけ：AI コーディングツールと相性が良い

この仕組みは、Claude Code や Cursor のような **AI コーディングツール経由で開発するときに特に効いてきます**。

これらのツールはターミナルでコマンドを実行できるので、`gh` CLI が使える環境なら AI に「この PR の内容を見て」「レビューコメントを確認して」と頼むだけで、AI が `gh` を叩いて PR の差分やコメントを直接取得し、文脈を踏まえた作業をしてくれます。

```bash
gh pr view 123              # PR の概要・本文を表示
gh pr diff 123             # PR の差分を表示
gh pr view 123 --comments  # レビューコメントを取得
```

このとき、`.envrc` でプロジェクトごとに `gh` のアカウントが切り替わっていれば、**AI もそのプロジェクトのアカウントの権限で `gh` を実行**してくれます。つまり、

- プロジェクト A を開いて AI に作業させると `account-b` の権限で PR を参照・操作
- プロジェクト B では `account-main` の権限で操作

というように、人間が手動で切り替えなくても、AI が「今いるディレクトリの正しいアカウント」で GitHub を操作してくれるわけです。AI に毎回「どのアカウントで操作して」と指示する必要がなく、アカウントの取り違え事故も防げます。

# まとめ

- `GH_TOKEN` 環境変数は `gh` の認証で**最優先**される。この性質を逆手に取る
- グローバルの `GH_TOKEN` は普段使いアカウントの**デフォルト**として残す
- 切り替えたいプロジェクトにだけ `.envrc` を置き、`direnv` で `GH_TOKEN` を**上書き**する
- コミット author は `git config --local` でリポジトリごとに設定する
- `.envrc` は**グローバル gitignore** で無視してトークン流出を防ぐ

手動の `gh auth switch` と違って切り替え忘れが起きないので、複数アカウントを日常的に行き来する人にはかなり快適になります。

# おわりに

複数の GitHub アカウントをプロジェクトごとに自動で使い分ける方法を、`gh` CLI と `direnv` の組み合わせで紹介しました。

トークンの有効期限が切れると `gh` が `Bad credentials`（401）を返すようになるので、その場合は新しいトークンを発行して `.envrc` の1行を差し替えるだけで復活します。

こちらの記事が皆さんのお役に立てれば幸いです。
