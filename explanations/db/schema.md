# schema.rb の解説

## このファイルの役割

`db/schema.rb` は、データベースのテーブル構造を定義したファイルです。マイグレーション（データベースの変更履歴ファイル）を実行した結果が自動的にここに反映されます。このアプリで使われる全テーブルのカラム（列）、データ型、インデックス（検索高速化のための設定）、外部キー（テーブル間のリレーション）がすべてここに記述されています。

## コード全体

```ruby
# This file is auto-generated from the current state of the database. Instead
# of editing this file, please use the migrations feature of Active Record to
# incrementally modify your database, and then regenerate this schema definition.
#
# This file is the source Rails uses to define your schema when running `bin/rails
# db:schema:load`. When creating a new database, `bin/rails db:schema:load` tends to
# be faster and is potentially less error prone than running all of your
# migrations from scratch. Old migrations may fail to apply correctly if those
# migrations use external dependencies or application code.
#
# It's strongly recommended that you check this file into your version control system.

ActiveRecord::Schema[7.1].define(version: 2026_03_13_144028) do
  # These are extensions that must be enabled in order to support this database
  enable_extension "pgcrypto"
  enable_extension "plpgsql"

  create_table "diagnosis_log_drugs", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.uuid "diagnosis_log_id", null: false
    t.uuid "drug_id", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["diagnosis_log_id"], name: "index_diagnosis_log_drugs_on_diagnosis_log_id"
    t.index ["drug_id"], name: "index_diagnosis_log_drugs_on_drug_id"
  end

  create_table "diagnosis_log_symptoms", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.uuid "diagnosis_log_id", null: false
    t.uuid "symptom_id", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["diagnosis_log_id"], name: "index_diagnosis_log_symptoms_on_diagnosis_log_id"
    t.index ["symptom_id"], name: "index_diagnosis_log_symptoms_on_symptom_id"
  end

  create_table "diagnosis_logs", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.uuid "user_id"
    t.integer "timing"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.text "result_summary"
  end

  create_table "drug_ingredients", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.uuid "drug_id", null: false
    t.uuid "ingredient_id", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["drug_id"], name: "index_drug_ingredients_on_drug_id"
    t.index ["ingredient_id"], name: "index_drug_ingredients_on_ingredient_id"
  end

  create_table "drug_symptoms", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.uuid "drug_id", null: false
    t.uuid "symptom_id", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["drug_id"], name: "index_drug_symptoms_on_drug_id"
    t.index ["symptom_id"], name: "index_drug_symptoms_on_symptom_id"
  end

  create_table "drugs", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.string "name"
    t.integer "category"
    t.integer "timing"
    t.text "description"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.text "pharmacist_advice"
  end

  create_table "ingredients", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.string "name"
    t.text "detail"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  create_table "symptoms", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.string "name"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.integer "timing"
    t.integer "category"
  end

  create_table "users", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|
    t.string "email"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.string "jti"
    t.string "encrypted_password", default: "", null: false
    t.string "reset_password_token"
    t.datetime "reset_password_sent_at"
    t.datetime "remember_created_at"
    t.string "name"
    t.string "provider"
    t.string "uid"
    t.integer "weight"
    t.string "gender"
    t.string "constitution"
    t.index ["email"], name: "index_users_on_email", unique: true
    t.index ["jti"], name: "index_users_on_jti"
    t.index ["provider", "uid"], name: "index_users_on_provider_and_uid", unique: true
    t.index ["reset_password_token"], name: "index_users_on_reset_password_token", unique: true
  end

  add_foreign_key "diagnosis_log_drugs", "diagnosis_logs"
  add_foreign_key "diagnosis_log_drugs", "drugs"
  add_foreign_key "diagnosis_log_symptoms", "diagnosis_logs"
  add_foreign_key "diagnosis_log_symptoms", "symptoms"
  add_foreign_key "drug_ingredients", "drugs"
  add_foreign_key "drug_ingredients", "ingredients"
  add_foreign_key "drug_symptoms", "drugs"
  add_foreign_key "drug_symptoms", "symptoms"
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1〜11 | `# This file is auto-generated...` | コメント行です。このファイルは自動生成されるため直接編集せず、マイグレーションファイルを使って変更するよう案内しています。 |
| 13 | `ActiveRecord::Schema[7.1].define(version: 2026_03_13_144028) do` | Rails 7.1のスキーマ定義を開始します。`version:` にはマイグレーションのバージョン（最後に実行したマイグレーションのタイムスタンプ）が入ります。 |
| 15 | `enable_extension "pgcrypto"` | PostgreSQLの暗号化関連機能を有効にします。UUIDを生成する `gen_random_uuid()` 関数を使うために必要です。 |
| 16 | `enable_extension "plpgsql"` | PostgreSQLのプログラミング言語拡張（PL/pgSQL）を有効にします。ストアドプロシージャなどの機能が使えるようになります。 |
| 18 | `create_table "diagnosis_log_drugs", id: :uuid, default: -> { "gen_random_uuid()" }, force: :cascade do |t|` | `diagnosis_log_drugs`（診断ログと薬の中間テーブル）を作成します。主キーはUUID型で `gen_random_uuid()` で自動生成されます。`force: :cascade` はテーブルが既に存在する場合に再作成する設定です。 |
| 19 | `t.uuid "diagnosis_log_id", null: false` | 診断ログのIDを格納するUUID型カラムです。`null: false` は値が必須（NULLを許可しない）であることを意味します。 |
| 20 | `t.uuid "drug_id", null: false` | 薬のIDを格納するUUID型カラムです。こちらも必須項目です。 |
| 21〜22 | `t.datetime "created_at"`, `t.datetime "updated_at"` | レコードの作成日時と更新日時を自動管理するカラムです。Railsが自動で値を入力してくれます。 |
| 23〜24 | `t.index [...]` | 外部キーカラムにインデックスを作成します。インデックスとは、データベースの検索を高速化する「目次」のようなものです。 |
| 27〜33 | `create_table "diagnosis_log_symptoms" ...` | `diagnosis_log_symptoms`（診断ログと症状の中間テーブル）を作成します。構造は `diagnosis_log_drugs` と同様です。 |
| 35〜41 | `create_table "diagnosis_logs" ...` | `diagnosis_logs`（診断ログテーブル）を作成します。`user_id`（ユーザーID）、`timing`（タイミング、整数型）、`result_summary`（結果要約、テキスト型）を持ちます。`user_id` は `null: false` がないため任意項目です。 |
| 43〜50 | `create_table "drug_ingredients" ...` | `drug_ingredients`（薬と成分の中間テーブル）を作成します。薬IDと成分IDの組み合わせで薬の成分情報を管理します。 |
| 52〜59 | `create_table "drug_symptoms" ...` | `drug_symptoms`（薬と症状の中間テーブル）を作成します。どの薬がどの症状に対応するかを管理します。 |
| 61〜69 | `create_table "drugs" ...` | `drugs`（薬テーブル）を作成します。`name`（名前）、`category`（カテゴリ）、`timing`（服用タイミング）、`description`（説明）、`pharmacist_advice`（薬剤師のアドバイス）を持ちます。 |
| 71〜76 | `create_table "ingredients" ...` | `ingredients`（成分テーブル）を作成します。`name`（名前）と `detail`（詳細）を持ちます。 |
| 78〜84 | `create_table "symptoms" ...` | `symptoms`（症状テーブル）を作成します。`name`（症状名）、`timing`（タイミング）、`category`（カテゴリ）を持ちます。 |
| 86〜101 | `create_table "users" ...` | `users`（ユーザーテーブル）を作成します。メールアドレス、パスワード（暗号化済み）、名前、OAuth認証用の `provider`・`uid`、体重・性別・体質などのプロフィール情報を持ちます。`jti` はJWT（認証トークン）の一意識別子です。 |
| 102〜105 | `t.index [...]` | usersテーブルの各カラムにインデックスを設定します。メールアドレスは `unique: true` で一意制約（重複禁止）を設けています。 |
| 107〜114 | `add_foreign_key ...` | 外部キー制約を追加します。たとえば `add_foreign_key "diagnosis_log_drugs", "diagnosis_logs"` は「diagnosis_log_drugsテーブルのdiagnosis_log_idは必ずdiagnosis_logsテーブルに存在するIDでなければならない」という制約です。 |
| 115 | `end` | スキーマ定義全体の終了です。 |

## まとめ

- `db/schema.rb` はデータベースの「設計図」で、アプリが使う全テーブルの構造が一目でわかります。
- 全テーブルの主キーはUUID型を使用しており、セキュリティと分散システムへの対応に優れています。
- 中間テーブル（`diagnosis_log_drugs`、`diagnosis_log_symptoms`など）を使うことで、多対多のリレーション（一つの診断ログが複数の薬や症状を持てる）を実現しています。
- `add_foreign_key` による外部キー制約で、データの整合性をデータベースレベルで保証しています。
- インデックスを外部キーカラムに設定することで、テーブル結合時の検索パフォーマンスを向上させています。
