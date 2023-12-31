# 説明
Qiita CLIを使って、Qiitaに公開する記事を管理するリポジトリ
masterにマージされることで差分をQiitaに自動公開(更新)する。

# コマンド説明
ブラウザでプレビューする
```shell:
npx qiita preview
```

新規記事を作成する
```shell:
npx qiita new 記事のファイルのベース名
```

`git push`で、github actionsによりQiita記事が投稿、更新される。

その後、
`git pull`
でローカルリポジトリを最新化し下記コマンドで同期する必要がある。

記事ファイルとQiitaを同期する
```shell:
npx qiita pull
```
