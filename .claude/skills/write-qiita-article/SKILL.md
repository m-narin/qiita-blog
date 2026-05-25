---
name: write-qiita-article
description: Qiita記事を新規作成し、公開・同期までを一貫して実行する。task.md/memo.mdなどの素材を一般化して記事化し、npx qiita newで雛形を作り、commit/pushしてCI完了後にqiita pullで同期する。
---

# Qiita記事作成スキル

`task.md` / `memo.md` などの素材を元に Qiita 記事を新規作成し、公開・ローカル同期までを一気通貫で実行する。

## 引数

- 任意。引数なしで呼び出された場合は、リポジトリ直下の `task.md` を読んで指示に従う。
- 引数で素材ファイルやテーマが渡された場合はそれを優先する。

## 前提

- リポジトリは Qiita CLI ベースのブログ（`qiita.config.json` が存在する）
- master/main ブランチへの push で GitHub Actions により Qiita に自動公開される構成
- `README.md` の手順に従う:
  - 新規作成: `npx qiita new <記事のファイルベース名>`
  - 同期: `npx qiita pull`

## 手順

### 1. 素材の確認

- `task.md` を読み、指示内容（タイトル候補、参照する素材、配置先など）を把握する
- 素材として `memo.md` や `sample-project/` 配下が指定されていれば、それらも読む
- 既存記事のフォーマット（フロントマター、タグの粒度、文体）を `public/` 配下の最近の記事 1〜2 本から確認する

### 2. 一般化が必要な箇所を洗い出す

社内 LT などの素材を記事化する場合、以下を一般化する:

- 社名・プロダクト名・チーム名（例: 固有プロダクト名 → 「あるプロジェクト」）
- 特徴的なネーミングを含む環境名・ブランチ名・チャネル名
- 具体的な実行回数・件数などの統計値（推測されうるもの）
- 内部 Slack チャネル名、内部 URL
- メンバー名・個人を特定しうる情報

技術的な内容（コード例、設定例、構文）は教育的価値があるので残す。

判断に迷うものは `AskUserQuestion` でユーザーに確認する。

### 3. タイトルとファイルベース名を決定

- タイトルは記事内容を端的に表すものを提案
- 既存記事のタイトル傾向に合わせる（例: `【XX】〜` のような prefix を使っているか）
- ファイルベース名はスネークケースの英数字（例: `claude_skill_story_point`）
- 複数案がある場合は `AskUserQuestion` でユーザーに選んでもらう

### 4. Qiita CLI で雛形を作成

```bash
npx qiita new <記事のファイルベース名>
```

- `public/<記事のファイルベース名>.md` が生成される
- 生成された雛形のフロントマターは以下のような形:

```yaml
---
title: <ファイルベース名>  # 後で書き換える
tags:
  - ''
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
```

### 5. 記事本文の作成

- `Write` ツールで雛形を上書きする前に、必ず一度 `Read` する（Write の制約）
- フロントマターを更新:
  - `title`: 決定したタイトル
  - `tags`: 5 個程度。既存記事のタグ粒度に合わせる
  - その他のフィールドは触らない（id/updated_at は CI で自動付与される）
- 本文は素材を再構成して書く。素材の構造に引きずられすぎず、読者目線で読みやすい流れにする

### 6. ユーザー確認

- 記事を書いたら、プレビューや内容確認をユーザーに促す
- 修正指示があれば反映し、commit/push の許可を得る

### 7. Commit & Push

`git add` は対象ファイルを明示的に指定する（`memo.md` `task.md` `sample-project/` などをうっかり含めないため）。

```bash
git add public/<記事のファイルベース名>.md
git commit -m "<コミットメッセージ>"
git push
```

コミットメッセージは既存のスタイル（日本語、短め）に合わせる。**Co-Authored-By トレーラーは付けない**（ユーザー設定）。

### 8. CI 完了を待機して同期

push 後、GitHub Actions により Qiita へ自動公開され、その結果がリポジトリにコミットされて戻ってくる（フロントマターに `id` / `updated_at` が付与される）。CI の完了を待つ必要があるため、約 20 秒待機してから pull する。

```bash
sleep 20 && git pull && npx qiita pull
```

- `git pull`: CI が付与した `id` / `updated_at` 入りのフロントマターをローカルに取り込む
- `npx qiita pull`: Qiita 側の状態とローカルを同期

待機しても CI が完了していなかった場合、`git pull` 時に追加コミットが取り込めない（後で取りこぼす）ため、`git pull` 後に新しいコミットがあるかを念のため確認し、なければもう一度 `git pull` する。

### 9. 完了報告

- 作成したファイルパス
- Qiita 記事 ID（`git pull` で取得できた場合）
- `updated_at`

を報告して終了する。

## 既存記事の修正フロー

新規作成ではなく既存記事を修正する場合も、編集後は同じ「commit → push → 20 秒待機 → `git pull && npx qiita pull`」のフローを実行する。タイトル変更でも本文修正でも同様。

## アンチパターン

- `git add .` や `git add -A` で広く追加する（作業用ファイルが混入する）
- `Co-Authored-By` トレーラーを commit に付ける
- 待機なしで `git pull` を実行する（CI が間に合わず取りこぼす）
- 素材の固有名詞を残したまま公開する
- 既存記事のタイトル傾向（例: `【XX】〜`）を無視する
