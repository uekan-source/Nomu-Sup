# drug_symptom.rb の解説

  ## このファイルの役割

  `DrugSymptom` は、薬（Drug）と症状（Symptom）を結びつける「中間テーブル」のモデルです。1つの薬は複数の症状に効果があり、1つの症状も複数の薬で対応できるという「多対多（many-to-many）」の関係を表現するために使われます。このモデルは `belongs_to` による2つの関連の宣言で構成されており、薬と症状の橋渡し役を担います。

  ## コード全体

  ```ruby
  class DrugSymptom < ApplicationRecord
    belongs_to :drug
    belongs_to :symptom
      end
      ```

      ## 行ごとの解説

      | 行番号 | コード | 解説 |
      |--------|--------|------|
      | 1 | `class DrugSymptom < ApplicationRecord` | `DrugSymptom` というクラス（設計図）を定義しています。`ApplicationRecord` を継承（`<`）することで、データベースとのやり取りに必要な機能を自動的に引き継ぎます。クラス名 `DrugSymptom` はRailsの規則により、テーブル名 `drug_symptoms` に対応します。 |
      | 2 | `belongs_to :drug` | `belongs_to`（〜に属する）は、この中間テーブルのレコードが必ず1つの `Drug`（薬）と紐づいていることを宣言するRailsのメソッドです。データベース上では `drug_id` という外部キーで管理され、`drug_symptom.drug` のように関連する薬のデータにアクセスできます。 |
      | 3 | `belongs_to :symptom` | 同様に、この中間テーブルのレコードが必ず1つの `Symptom`（症状）と紐づいていることを宣言しています。データベース上では `symptom_id` という外部キーで管理され、`drug_symptom.symptom` で関連する症状のデータにアクセスできます。 |
      | 4 | `end` | クラス定義の終了を示します。 |

      ## まとめ

      - `DrugSymptom` は薬（Drug）と症状（Symptom）の多対多の関係を実現する「中間モデル」です。
      - `belongs_to` は「このレコードは必ず相手側のレコードに属する」という関連を定義するメソッドで、外部キー（`drug_id`、`symptom_id`）によって結びつきが管理されます。
      - このモデルがあることで、`Drug` 側では `has_many :symptoms, through: :drug_symptoms` と書くことができ、ある薬が対処できる症状の一覧を取得できます。
      - 逆に `Symptom` 側では `has_many :drugs, through: :drug_symptoms` と書くことで、ある症状に効く薬の一覧を取得できます。
      - `DrugIngredient` モデルと同じ構造をしており、Railsにおける多対多の関係の典型的な実装パターンです。
