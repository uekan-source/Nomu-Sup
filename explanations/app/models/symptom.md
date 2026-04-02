# symptom.rb の解説

## このファイルの役割

`Symptom`（症状）モデルを定義するファイルです。アプリ内で扱う「症状」データの構造と、他のモデルとのつながり（アソシエーション）を設定しています。症状は「診断ログ」と「薬」の両方と多対多の関係にあり、このファイルがその橋渡しを担います。

## コード全体

```ruby
class Symptom < ApplicationRecord
  has_many :diagnosis_log_symptoms, dependent: :destroy
  has_many :diagnosis_logs, through: :diagnosis_log_symptoms

  has_many :drug_symptoms, dependent: :destroy
  has_many :drugs, through: :drug_symptoms
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `class Symptom < ApplicationRecord` | `Symptom`（症状）というクラスを定義しています。`ApplicationRecord`を継承（`<`）することで、データベースへの保存・取得などの機能が自動で使えるようになります。 |
| 2 | `has_many :diagnosis_log_symptoms, dependent: :destroy` | `Symptom`は複数の`DiagnosisLogSymptom`（診断ログと症状の中間テーブル）を持つことを示します。`has_many`とは「1対多の関係」を設定するRailsのメソッドです。`dependent: :destroy`は「この症状が削除されたとき、関連する中間テーブルのレコードも一緒に削除する」という設定です。 |
| 3 | `has_many :diagnosis_logs, through: :diagnosis_log_symptoms` | 中間テーブル`diagnosis_log_symptoms`を経由して、複数の`DiagnosisLog`（診断ログ）と関連を持つことを宣言しています。`through:`オプションは「〜を通じて」という意味で、多対多の関係を実現するためのRailsの仕組みです。 |
| 4 | （空行） | コードの可読性のための区切り行です。 |
| 5 | `has_many :drug_symptoms, dependent: :destroy` | `Symptom`は複数の`DrugSymptom`（薬と症状の中間テーブル）を持つことを示します。症状が削除されると、関連する`DrugSymptom`レコードも連動して削除されます。 |
| 6 | `has_many :drugs, through: :drug_symptoms` | 中間テーブル`drug_symptoms`を経由して、複数の`Drug`（薬）と関連を持つことを宣言しています。「この症状に対応する薬はどれか」を簡単に調べられるようになります。 |
| 7 | `end` | `class Symptom`ブロックの終了です。 |

## まとめ

- **`has_many`とは**：1対多の関係を設定するRailsのメソッドです。「1つの症状は複数のレコードを持てる」ことを表現します。
- **中間テーブルとは**：多対多の関係（症状↔診断ログ、症状↔薬）を実現するために使う特別なテーブルです。例えば「1つの症状が複数の診断ログに登場し、1つの診断ログも複数の症状を持てる」という関係を管理します。
- **`through:`オプション**：中間テーブルを使って関連先に「橋渡し」でアクセスする方法です。`symptom.drugs`のように書くだけで、この症状に関連する薬の一覧が取得できます。
- **`dependent: :destroy`**：親レコードが削除された際に、子レコードも自動的に削除するオプションです。データベースに不要なデータが残らないようにするための設計です。
- このファイルはとてもシンプルですが、アプリ全体のデータ構造を理解する上で重要な「関係性の設計図」になっています。
