# drug.rb の解説

  ## このファイルの役割

  `Drug` モデルは、アプリ内の「薬・食品」データを表すモデルです。薬の種類（医薬品か食品か）や服用タイミング（食前・食中・食後・いつでも）を管理します。また、薬と診断ログ・成分・症状との関係を定義しており、アプリ全体のデータ設計の中心的な役割を担っています。

  ## コード全体

  ```ruby
  class Drug < ApplicationRecord
    has_many :diagnosis_log_drugs, dependent: :destroy
        has_many :diagnosis_logs, through: :diagnosis_log_drugs

            has_many :drug_ingredients, dependent: :destroy
                has_many :ingredients, through: :drug_ingredients

                    has_many :drug_symptoms, dependent: :destroy
                        has_many :symptoms, through: :drug_symptoms

                            enum :category, { medicine: 0, food: 1 }
  enum :timing, { before: 0, during: 1, after: 2, any: 3 }
end
  ```

  ## 行ごとの解説

  | 行番号 | コード | 解説 |
  |--------|--------|------|
  | 1 | `class Drug < ApplicationRecord` | `Drug`（薬）というクラスを定義します。`< ApplicationRecord` で継承し、Railsのデータベース操作機能が使えるようになります。 |
  | 2 | `has_many :diagnosis_log_drugs, dependent: :destroy` | 1つの薬は複数の `DiagnosisLogDrug`（中間テーブル）を持つことを表します。`has_many` は「〜を複数持つ」という関連付けです。`dependent: :destroy` は「薬が削除されたとき、関連する中間テーブルのレコードも一緒に削除する」という設定です。 |
  | 3 | `has_many :diagnosis_logs, through: :diagnosis_log_drugs` | 中間テーブル `:diagnosis_log_drugs` を経由して、複数の `DiagnosisLog`（診断ログ）と関連付けることを表します。`through:` オプションを使うことで**多対多の関係**を実現しています。 |
  | 4 | （空行） | 可読性のための区切りです。 |
  | 5 | `has_many :drug_ingredients, dependent: :destroy` | 1つの薬は複数の `DrugIngredient`（薬と成分の中間テーブル）を持つことを表します。薬が削除されると関連する中間テーブルも一緒に削除されます。 |
  | 6 | `has_many :ingredients, through: :drug_ingredients` | 中間テーブルを経由して、複数の `Ingredient`（成分）に関連付けることを表します。薬に含まれる成分の一覧を取得できます。 |
  | 7 | （空行） | 可読性のための区切りです。 |
  | 8 | `has_many :drug_symptoms, dependent: :destroy` | 1つの薬は複数の `DrugSymptom`（薬と症状の中間テーブル）を持つことを表します。薬が削除されると関連する中間テーブルも削除されます。 |
  | 9 | `has_many :symptoms, through: :drug_symptoms` | 中間テーブルを経由して、複数の `Symptom`（症状）に関連付けることを表します。薬に関連する症状の一覧を取得できます。 |
  | 10 | （空行） | 可読性のための区切りです。 |
  | 11 | `enum :category, { medicine: 0, food: 1 }` | `category`（カテゴリ）フィールドを**列挙型（enum）**として定義します。`enum` とは「あらかじめ決められた選択肢しか入れられない」という型です。`medicine: 0`（医薬品）か `food: 1`（食品）の2種類だけが設定できます。データベースには数値で保存されますが、コード上では名前で扱えます。 |
  | 12 | `enum :timing, { before: 0, during: 1, after: 2, any: 3 }` | `timing`（服用タイミング）フィールドを列挙型で定義します。`before: 0`（食前）、`during: 1`（食中）、`after: 2`（食後）、`any: 3`（いつでも）の4種類から選べます。 |
  | 13 | `end` | クラス定義の終わりを示します。 |

  ## まとめ

  - `Drug` モデルは「薬・食品」を表し、アプリのデータモデルの中核を担っています
  - `has_many ... through:` を使って多対多の関係（診断ログ・成分・症状）を実現しています
  - `dependent: :destroy` によって、薬を削除した際に関連する中間テーブルも自動削除される安全な設計になっています
  - `enum` を使って「カテゴリ」と「服用タイミング」を選択肢として管理しており、不正な値が入らないようにしています
  - `has_many`・`belongs_to`・`through` の使い方を学ぶための典型的な例となっています
