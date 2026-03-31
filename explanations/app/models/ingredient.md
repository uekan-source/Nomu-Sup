# ingredient.rb の解説

## このファイルの役割

`Ingredient` は、薬（Drug）に含まれる「成分」を表すモデルです。1つの成分は複数の薬に含まれる可能性があり、中間テーブル `drug_ingredients` を経由して薬と多対多（many-to-many）の関係を持ちます。このモデルは成分データの「本体」として機能し、`DrugIngredient` 中間モデルを通じて薬との関連を管理します。

## コード全体

```ruby
class Ingredient < ApplicationRecord
  has_many :drug_ingredients, dependent: :destroy
  has_many :drugs, through: :drug_ingredients
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `class Ingredient < ApplicationRecord` | `Ingredient` というクラス（設計図）を定義しています。`ApplicationRecord` を継承（`<`）することで、データベースとのやり取りに必要な機能を自動的に引き継ぎます。クラス名 `Ingredient` はRailsの規則により、テーブル名 `ingredients` に対応します。 |
| 2 | `has_many :drug_ingredients, dependent: :destroy` | `has_many`（多数持つ）は、1つの成分が複数の `DrugIngredient`（薬と成分の中間レコード）を持てることを宣言するRailsのメソッドです。`dependent: :destroy` は「この成分が削除されたとき、関連する `drug_ingredients` のレコードも一緒に削除する」という設定で、データの孤立（orphan）を防ぎます。 |
| 3 | `has_many :drugs, through: :drug_ingredients` | `through:` オプション付きの `has_many` は「中間テーブルを経由した多対多の関連」を定義します。`ingredient.drugs` と書くことで、この成分を含む薬の一覧を直接取得できます。Railsが自動的に `drug_ingredients` テーブルを経由してSQLを組み立ててくれます。 |
| 4 | `end` | クラス定義の終了を示します。 |

## まとめ

- `Ingredient` は薬の「成分」を管理するモデルで、`Drug` との多対多の関係を `DrugIngredient` 中間テーブルを通じて実現しています。
- `has_many :drug_ingredients, dependent: :destroy` の `dependent: :destroy` は重要なオプションで、成分を削除したときに関連する中間レコードも自動削除し、データの整合性を保ちます。
- `has_many :drugs, through: :drug_ingredients` により、`ingredient.drugs` という直感的な書き方で関連する薬の一覧が取得できます。
- `DrugIngredient` モデル側には `belongs_to :ingredient` があり、両側から関連が定義されることで完全な多対多の関係が成立します。
- `Drug` モデル側にも同様に `has_many :ingredients, through: :drug_ingredients` が定義されており、双方向からデータにアクセスできる設計になっています。
