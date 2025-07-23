---
title: 【Rails】activerecord-import で柔軟に upsert を実現する方法
tags:
  - Ruby
  - Rails
  - UPSERT
  - activerecord-import
private: false
updated_at: '2025-07-23T15:54:32+09:00'
id: 4393698aeb8a9dcd521a
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

[activerecord-import](https://github.com/zdennis/activerecord-import) は Rails で一括インポートを実現する強力な gem です。`on_duplicate_key_update` や `on_duplicate_key_ignore` といったオプションを使うことで upsert（既存レコードの更新または新規作成）を実現できます。

しかし、これらのオプションはデータベースに対象カラムの一意制約（ユニークインデックス）が存在することが前提となっています。一意制約がない場合、これらのオプションは期待通りに動作せず、重複レコードが作られてしまいます。

そのため、import に渡すデータを事前に既存レコードかどうかで区別し、適切に処理する必要があります。本記事では、そのような場合の実装パターンを紹介します。

## 実装

### 前提

以下のような User テーブルがあるとします：

- `store_id`：外部キー（Store テーブルへの参照）
- `company_id`：外部キー（Company テーブルへの参照）
- `name`：ユーザー名

User テーブルに対して `store_id`, `company_id`, `name` の 3 つのカラムを bulk import する処理があり、`store_id` と `company_id` の組み合わせが既に存在する場合は upsert する要件があるものとします。

### 一意制約がある場合の実装

まず、`store_id`と`company_id`の組み合わせに一意制約が設定されている場合の実装例を示します。この場合は activerecord-import のオプションを使用して簡潔に upsert を実現できます。

#### 既存レコードを無視する場合

```ruby
# インポート対象のデータ
user_attributes = [
  { store_id: 1, company_id: 1, name: "田中太郎" },
  { store_id: 1, company_id: 2, name: "鈴木花子" },
  { store_id: 2, company_id: 1, name: "佐藤次郎" }
]

# 重複する場合は既存レコードを保持（新しいデータを無視）
User.import!(
  user_attributes,
  on_duplicate_key_ignore: true
)
```

#### 既存レコードを更新する場合

```ruby
# インポート対象のデータ
user_attributes = [
  { store_id: 1, company_id: 1, name: "更新後の名前" },
  { store_id: 3, company_id: 3, name: "新規ユーザー" }
]

# 重複する場合は指定したカラムを更新
User.import!(
  user_attributes,
  on_duplicate_key_update: [:name, :updated_at]
)
```

**注意点**: これらのオプションを使用するには、対象となるカラム（この例では `store_id` と `company_id`）に一意制約（ユニークインデックス）が設定されている必要があります。

```ruby
# マイグレーションファイルでの一意制約の例
add_index :users, [:store_id, :company_id], unique: true
```

なお、Active Record の validation（例：`validates :store, uniqueness: { scope: [:company] }`）が設定されていたとしても`activerecord-import`は 無視するようです。

### 一意制約がない場合の実装

一意制約が設定されていない場合は、以下のような手動での処理分岐が必要になります。

#### 既存レコードがある場合に無視する

既存レコードと重複するデータをスキップして、新規レコードのみをインポートする場合の実装例です：

```ruby
# インポート対象のデータ
# user_attributes = [
#   { store_id: 1, company_id: 1, name: "田中太郎" },
#   { store_id: 1, company_id: 2, name: "鈴木花子" },
#   { store_id: 2, company_id: 1, name: "佐藤次郎" }
# ]

# 既存レコードの store_id と company_id の集合を取得
# 重複のないSetオブジェクト=Hashでのアクセスが可能
existing_users =
  User
    .where(
      store_id: user_attributes.pluck(:store_id),
      company_id: user_attributes.pluck(:company_id)
    )
    .pluck(:store_id, :company_id)
    .to_set

# 既存レコードを除外
# Set オブジェクトなのでハッシュで include チェックが可能 (O(1))
new_user_attributes =
  user_attributes.reject do |attr|
    existing_users.include?([attr[:store_id], attr[:company_id]])
  end

return if new_user_attributes.empty?

# バッチサイズごとに分割してインポート
BATCH_SIZE = 1000
new_user_attributes.each_slice(BATCH_SIZE) do |chunked_attributes|
  User.import!(chunked_attributes)
end
```

このアプローチのポイント：

- `to_set` を使用することで、既存レコードの存在チェックを O(1) で実行可能
- 既存レコードと重複するデータは事前に除外されるため、重複エラーが発生しない
- バッチサイズで分割することで、大量データでもメモリ効率的に処理

#### 既存レコードがある場合に更新する

既存レコードは更新し、新規レコードは作成する場合の実装例です：

```ruby
# インポート対象のデータ
# import_attributes = [
#   { store_id: 1, company_id: 1, name: "更新後の名前" },
#   { store_id: 3, company_id: 3, name: "新規ユーザー" }
# ]

# 既存の User レコードを一度に取得
# store_id と company_id の組み合わせをキーとし、対応する User を値とするハッシュ
existing_users_map =
  User
    .where(
      store_id: import_attributes.map { |attr| attr[:store_id] },
      company_id: import_attributes.map { |attr| attr[:company_id] }
    )
    .index_by do |user|
      [user.store_id, user.company_id]
    end

# 新規作成と更新を分割してそれぞれ一括実行
existing_records = []
new_records = []

import_attributes.each do |import_attribute|
  user = existing_users_map[
    [import_attribute[:store_id], import_attribute[:company_id]]
  ]

  if user
    # 既存レコードの場合は属性を更新
    user.assign_attributes(
      name: import_attribute[:name]
    )
    existing_records << user
  else
    # 新規レコードの場合はハッシュとして追加
    new_records << {
      store_id: import_attribute[:store_id],
      company_id: import_attribute[:company_id],
      name: import_attribute[:name]
    }
  end
end

# 既存レコードの更新（on_duplicate_key_update を使用）
if existing_records.present?
  User.import!(
    existing_records,
    on_duplicate_key_update: [:name, :updated_at]
  )
end

# 新規レコードの作成
if new_records.present?
  User.import!(new_records)
end
```

このアプローチのポイント：

- `index_by` を使用してキーでの高速アクセスを可能にしている
- 既存レコードと新規レコードを分離して処理することで、それぞれに最適化された処理を実行
- 既存レコードには `on_duplicate_key_update` を使用して確実に更新
- メモリ上でレコードを分類してから一括処理するため、データベースへのアクセス回数が最小限

## まとめ

activerecord-import の `on_duplicate_key_update` や `on_duplicate_key_ignore` オプションは便利ですが、DB レベルの一意制約が必要という制約があります。本記事で紹介したように、事前に既存レコードをチェックして処理を分岐させることで、一意制約がない場合でも柔軟な upsert を実現できます。

重要なポイント：

- パフォーマンスを考慮し、Set や Hash を活用
- 既存レコードのチェックは一括で行い、N+1 問題を回避
- バッチ処理により大量データでも安定した処理を実現

このアプローチにより、activerecord-import を使った柔軟で高速な一括データ処理が可能になります。
