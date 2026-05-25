---
title: 【Claude Code】IssueのStory Point付与を自動化するskill
tags:
  - GitHub
  - 生成AI
  - GitHubProjects
  - ClaudeCode
  - AgentSkills
private: false
updated_at: '2026-05-25T12:02:06+09:00'
id: 17c348f53614ba7e20b8
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

スクラム開発において、Issue への Story Point の付与は地味だが定常的に発生する作業です。

- 内容を読んで規模感・複雑さを評価する
- GitHub Projects の Story Point フィールドに数値を入れる
- 根拠をコメントとして残す
- Status を Estimated などに動かす

1 件ならまだしも、リファインメント前にまとめて 10 件以上裁こうとすると、それなりにスイッチングコストがかかります。本記事では、この一連の流れを **Claude Code の Skill 機能** で自動化した事例を紹介します。

# Claude Code の Skill とは

Claude Code には、特定タスクの手順をあらかじめ Markdown ファイルに定義しておき、「このスキルを使って」と呼び出せる仕組みがあります。

- 定義ファイル: `.claude/skills/<skill-name>/SKILL.md`
- フロントマターに `name` と `description` を記述する
- `description` に書いた発火条件（例: 「`issueにpoint付与して`と依頼されたら使用する」）を見て、Claude が自動的にスキルを選択する

詳細は公式ドキュメントを参照してください: [Skills - Claude Code](https://code.claude.com/docs/ja/skills)

> 仕組みの肝は「自然言語で手順を書いておくだけ」で、Claude がそれをタスク実行時のプレイブックとして参照してくれる、という点です。

# 作ったスキルの全体像

今回作ったスキルは、引数で渡された Issue 番号に対して以下を実行します。

1. Issue の内容を読む（sub issues があれば確認して対象を切り替える）
2. Story Point を判定する（規模感 × 複雑さ）
3. ユーザーに一覧で確認する
4. 承認されたら、Projects に Story Point を設定
5. 根拠を Issue コメントとして残す
6. Status を Estimated に遷移
7. 完了報告

呼び出し方は以下のような形を想定しています。

```
/story-point 1234
/story-point 1234 1235 1236
/story-point https://github.com/{OWNER}/{REPO}/issues/1234
```

# 前提

- 対象 Issue が GitHub Projects に所属していること
- `gh` CLI に `project` スコープが付与されていること
    - `gh auth status` で確認可能。なければ `gh auth refresh -s project` で追加

Projects のフィールド ID やオプション ID（Story Point フィールド、Status フィールド、Estimated（= 見積もり済み）オプション）はあらかじめ調べてスキル本文にハードコードしておきます（後述）。

# SKILL.md の中身

`.claude/skills/story-point/SKILL.md` というパスに、以下のような Markdown を配置します。`{OWNER}`、`{REPO}`、`{PROJECT_ID}` などはお使いの環境に合わせて置き換えてください。

## フロントマター

```markdown
---
name: story-point
description: GitHub IssueにStory Pointを付与し、根拠コメントの追加とStatusの遷移を行う。
disable-model-invocation: true
---
```

ポイントは `disable-model-invocation: true` を指定していることです。

このフラグを付けると、Claude が `description` を読んで自動的にスキルを発火させる挙動を抑制でき、**スラッシュコマンド (`/story-point`) からの明示的な呼び出しのみ** に限定できます。

> Story Point 付与のように「実行したいタイミングはユーザーが決めたい」「うっかり自動発火されると困る」タイプのスキルでは、このフラグを付けておくのが安全です。
>
> 逆に、自然言語の依頼から自動で発火させたいスキルでは付けない方が良いです（その場合は `description` に発火条件をしっかり書く必要があります）。

加えて、**コンテキスト消費を抑えられる** という地味に大きなメリットもあります。

通常、自動発火の対象となるスキルは、Claude がいつでも選べるように `name` / `description` などのメタデータがセッション開始時に読み込まれます。スキルの数が増えるとこのメタデータの合計がそれなりのトークン数になり、本来のタスクに使える有効コンテキストが削られていきます。

`disable-model-invocation: true` を付けたスキルはこの自動ロード対象から外れ、コマンドで明示的に呼び出されたときに初めて中身が読み込まれます。「常駐させる必要のないスキル」にこのフラグを付けて回るだけで、全体のコンテキスト効率が上がります。

`description` 自体は、コマンドの一覧表示や Claude が「何のためのスキルか」を把握するためのドキュメントとして引き続き使われます。

## 引数の設計

```markdown
## 引数

- Issue番号またはURL（スペースまたはカンマ区切りで複数指定可能）
  - 単一: `/story-point 1234`
  - 複数: `/story-point 1234 1235 1236`
  - URL: `/story-point https://github.com/{OWNER}/{REPO}/issues/1234`

URLが渡された場合は、末尾のIssue番号を抽出して使用する。
```

普段の運用では「この PR に書いてる Issue まとめて頼む」みたいなパターンが多いので、URL でも番号でも受け付けるようにしておくと体験が良くなります。

## 手順 1: sub issues の解決

```bash
gh issue view <ISSUE_NUMBER> --repo {OWNER}/{REPO}
```

```bash
gh api graphql -f query='
query($number: Int!) {
  repository(owner: "{OWNER}", name: "{REPO}") {
    issue(number: $number) {
      title
      subIssues(first: 50) {
        nodes {
          number
          title
          state
        }
      }
    }
  }
}' -F number=<ISSUE_NUMBER> --jq '.data.repository.issue.subIssues.nodes'
```

sub issues がある場合は、親 Issue ではなく子 Issue に対して点数を付ける方が実情に合うことが多いです。なので、

- sub issues があれば「子に付けますか？」と確認
- 承認されたら対象リストを子に差し替える（親は除外）
- なければ Issue 本体を対象とする

という分岐を入れておきます。

ついでに、対象 Issue が Projects に所属していない場合はスキップしてユーザーに報告する、というガードも入れておくと安全です。

## 手順 2: Story Point の判定基準

スキル内に判定基準そのものを書いておくと、Claude がそれに従って点数を出してくれます。

```markdown
ポイントは `規模感 × 複雑さ` で決める。実装者のドメイン知識の有無は考慮しない。

| Point | 規模感 | 複雑さ |
|-------|--------|--------|
| 1 | 数行の変更、単一ファイル | 自明な変更 |
| 2 | 小規模、1-2ファイル | 低い複雑さ |
| 3 | 中規模、複数ファイル | 標準的な複雑さ |
| 5 | 大規模、複数コンポーネント | 高い複雑さ、設計判断が必要 |
| 8 | 非常に大規模 | 非常に高い複雑さ、不確実性が高い |

チームで定義したStory Point基準のissueがあれば、そちらを参照して判定する。
```

ここでのポイントは2つあります。

1. **「ドメイン知識の有無は考慮しない」と明示する**: AI が「人によって難易度が変わる」みたいな評価を始めるとブレるため、評価軸を絞ります
2. **チーム内のリファレンス Issue があれば参照させる**: 「過去にこのくらいの規模で 3 を付けた」という生きた基準があると、判定が安定します

## 手順 3: ユーザーに一括確認

```markdown
| Issue  | タイトル | Story Point | 根拠（要約） |
| ------ | -------- | ----------- | ------------ |
| #1234 | xxx      | 3           | ...          |
| #1235 | yyy      | 5           | ...          |
```

ここを「自動で付けちゃう」にしないのがコツで、必ず承認ステップを挟みます。実運用だと「これは 5 じゃなくて 3 にして」みたいな微調整がほぼ必ず入るので、その差し戻しに対応できる設計にしておきます。

> ユーザーが個別に修正を指示した場合は、該当 Issue のみ修正して再確認する。

と書いておくと、Claude も「他は触らず、その Issue だけ修正してもう一度確認」という振る舞いをしてくれます。

## 手順 4: Story Point の付与

まず Projects のアイテム ID を取得します。

```bash
gh api graphql -f query='
query($number: Int!) {
  repository(owner: "{OWNER}", name: "{REPO}") {
    issue(number: $number) {
      projectItems(first: 10) {
        nodes {
          id
          project { number }
        }
      }
    }
  }
}' -F number=<ISSUE_NUMBER> --jq '.data.repository.issue.projectItems.nodes[] | select(.project.number == {PROJECT_NUMBER}) | .id'
```

そして Story Point を設定します。

```bash
gh project item-edit \
  --project-id {PROJECT_ID} \
  --id <ITEM_ID> \
  --field-id {STORY_POINT_FIELD_ID} \
  --number <STORY_POINT>
```

`{PROJECT_ID}`、`{STORY_POINT_FIELD_ID}` などは Projects ごとに固有の ID です。一度調べてスキル本文に書いておけば、以降は変更不要です（フィールド構成を変えたら更新する必要あり）。

> ID の調べ方は `gh project field-list <PROJECT_NUMBER> --owner <OWNER>` などで取得できます。

## 手順 5: Issue にコメント追加

判定の根拠は Issue コメントとして残します。後からチームメンバーが「なぜ 3 なのか」を追えるようにするためです。

````markdown
```bash
gh issue comment <ISSUE_NUMBER> --repo {OWNER}/{REPO} --body "$(cat <<'EOF'
## Story Point: <POINT>

### 根拠

- <根拠1>
- <根拠2>
- ...
EOF
)"
```
````

`cat <<'EOF'` のヒアドキュメントを使うことで、Markdown のフォーマットをそのまま渡せます。

## 手順 6: Status を Estimated（見積もり済み）に遷移

```bash
gh project item-edit \
  --project-id {PROJECT_ID} \
  --id <ITEM_ID> \
  --field-id {STATUS_FIELD_ID} \
  --single-select-option-id {ESTIMATED_OPTION_ID}
```

Status フィールドはセレクトボックスなので、`--single-select-option-id` で遷移先のオプション ID を渡します。

## 手順 7: 完了報告

```markdown
| Issue  | Story Point | コメントURL | Status           |
| ------ | ----------- | ----------- | ---------------- |
| #1234 | 3           | (URL)       | Estimated |
| #1235 | 5           | (URL)       | Estimated |
```

最後に結果一覧を表で報告させます。コメント URL を入れておくと、各 Issue の判定根拠にすぐ飛べて便利です。

# 設計上の工夫

実際に運用してみて効果があった工夫を3つ紹介します。

## 1. 「承認ステップ」を必ず挟む

一気通貫で実行すると、誤判定があった時に Issue コメントを消す・Status を戻すなどの後始末が発生します。判定結果の一覧を出して承認を求めるステップを挟むだけで、事故率がぐっと下がります。

## 2. 判定基準をスキル内に書く

外部ドキュメントに分けず、SKILL.md の中に判定表を直接書く方が、Claude が安定して参照してくれます。基準が変わったらこのファイルを直すだけ、というシンプルさも維持できます。

## 3. ID 系はハードコードする

Projects の ID や Field ID は変わらないので、調べてスキルに書いておきます。毎回動的に取得する処理を書くこともできますが、ステップが増えて時間がかかるだけなのでオススメしません。

# おわりに

Claude Code Skill は、「自然言語で手順書を書いておくだけ」で再現可能な作業フローを作れる、非常にコスパの良い仕組みです。

Story Point 付与のような「判断は人っぽいけど、手順自体は決まっている」タスクは特に相性が良いと感じています。同じようなパターンとして、

- 障害対応の初動チェックリスト実行
- リリースノート草案の生成
- Issue テンプレートに沿った内容整形

なども Skill 化できそうです。「定型だけどちょっと判断が要る」作業を見つけたら、まず SKILL.md を1つ書いてみるところから始めるのがおすすめです。
