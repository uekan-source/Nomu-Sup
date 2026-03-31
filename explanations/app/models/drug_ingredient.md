# drug_ingredient.rb の解説

  ## このファイルの役割

  `DrugIngredient` は、薬（Drug）と成分（Ingredient）を結びつける「中間テーブル」のモデルです。1つの薬は複数の成分を持ち、1つの成分も複数の薬に含まれるという「多対多（many-to-many）」の関係を表現するために使われます。このモデル自体はシンプルで、`belongs_to` による2つの関連の宣言だけで構成されています。

  ## コード全体

  ```ruby
  class DrugIngredient < ApplicationRecord
    belongs_to :drug
    belongs_to :ingredient
      end
      ```

      ## 行ごとの解説

      | 行番号 | コード | 解説 |
      |--------|--------|------|
      | 1 | `class DrugIngredient < ApplicationRecord` | `DrugIngredient` というクラス（設計図）を定義しています。`ApplicationRecord` を継承（`<`）することで、データベースとのやり取りに必要な機能を自動的に引き継ぎます。クラス名 `DrugIngredient` はスネークケースのテーブル名 `drug_ingredients` に対応します。 |
      | 2 | `belongs_to :drug` | `belongs_to`（〜に属する）とは、この中間テーブルのレコードが必ず1つの `Drug`（薬）と紐づいていることを宣言するRailsのメソッドです。これにより `drug_ingredient.drug` のように関連する薬のデータにアクセスできます。 |
      | 3 | `belongs_to :ingredient` | 同様に、この中間テーブルのレコードが必ず1つの `Ingredient`（成分）と紐づいていることを宣言しています。`drug_ingredient.ingredient` で関連する成分のデータにアクセスできます。 |
      | 4 | `end` | クラス定義の終了を示します。 |

      ## まとめ

      - `DrugIngredient` は薬（Drug）と成分（Ingredient）の多対多の関係を表す「中間モデル」です。
      - `belongs_to` は「このレコードは必ず相手側のレコードに属する」という関連を定義するメソッドで、外部キー（`drug_id`、`ingredient_id`）によって結びつきが管理されます。
      - このモデルがあることで、`Drug` 側では `has_many :ingredients, through: :drug_ingredients` と書くことができ、薬に含まれる全成分を取得できます。
      - `ApplicationRecord` を継承することで、Active Record の機能（データの取得・保存・削除など）が使えるようになります。
      - 中間テーブルのモデルはシンプルに見えますが、データの整合性（両方の外部キーが必ず存在すること）を保つ重要な役割を担っています。
