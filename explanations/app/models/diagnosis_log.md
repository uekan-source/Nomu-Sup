# diagnosis_log.rb の解説

## このファイルの役割

このファイルは「診断ログ」を表すモデルです。ユーザーがアプリで症状を入力して診断を行ったときの記録（ログ）を管理します。どのユーザーが・いつ・どんな症状で診断したか・どんな薬が関係したかという情報を保持し、症状（Symptom）や薬（Drug）との関連も定義しています。

## コード全体

```ruby
class DiagnosisLog < ApplicationRecord
  belongs_to :user, optional: true # ログインなしでも診断できる場合は optional: true
  has_many :diagnosis_log_symptoms, dependent: :destroy
  has_many :symptoms, through: :diagnosis_log_symptoms

  has_many :diagnosis_log_drugs, dependent: :destroy
  has_many :drugs, through: :diagnosis_log_drugs

  enum :timing, { before_drinking: 0, during_drinking: 1, after_drinking: 2 }
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `class DiagnosisLog < ApplicationRecord` | `DiagnosisLog`（診断ログ）クラスを定義し、`ApplicationRecord` を継承します。これにより `diagnosis_logs` テーブルと自動的に紐づきます。 |
| 2 | `belongs_to :user, optional: true` | 「診断ログは1人のユーザーに属する」という関連を定義します。`optional: true` は「ユーザーが存在しなくてもよい（未ログインでも診断できる）」という意味です。通常の `belongs_to` は必須ですが、`optional: true` でその制約を外しています。 |
| 3 | `has_many :diagnosis_log_symptoms, dependent: :destroy` | 「この診断ログは複数の `DiagnosisLogSymptom`（中間テーブル）を持つ」という関連を定義します。`dependent: :destroy` は「診断ログが削除されたとき、関連する中間テーブルのレコードも一緒に削除する」という意味です。 |
| 4 | `has_many :symptoms, through: :diagnosis_log_symptoms` | 中間テーブル（`diagnosis_log_symptoms`）を経由して、複数の症状（`Symptom`）と関連づける「多対多（many-to-many）」の関係を定義します。`through:` は「〜を経由して」という意味です。 |
| 6 | `has_many :diagnosis_log_drugs, dependent: :destroy` | 「この診断ログは複数の `DiagnosisLogDrug`（中間テーブル）を持つ」という関連を定義します。薬との中間テーブルも、診断ログ削除時に一緒に削除されます。 |
| 7 | `has_many :drugs, through: :diagnosis_log_drugs` | 中間テーブル（`diagnosis_log_drugs`）を経由して、複数の薬（`Drug`）と関連づける「多対多」の関係を定義します。 |
| 9 | `enum :timing, { before_drinking: 0, during_drinking: 1, after_drinking: 2 }` | `timing`（タイミング）という列挙型（enum）を定義します。データベースには数値（0、1、2）で保存されますが、コード上では `before_drinking`（飲酒前）、`during_drinking`（飲酒中）、`after_drinking`（飲酒後）という名前で扱えます。 |
| 10 | `end` | クラス定義の終了です。 |

## まとめ

- **アソシエーション（関連）の設定**：`belongs_to`、`has_many`、`has_many ... through:` を使って、ユーザー・症状・薬との関係を定義しています。Railsでは「アソシエーション」と呼ばれる仕組みです。
- **多対多の関係**：1つの診断ログは複数の症状と複数の薬を持てます。これを実現するために、`diagnosis_log_symptoms` や `diagnosis_log_drugs` という中間テーブルを使っています。
- **`optional: true` によるゲスト対応**：通常 `belongs_to` は紐づくレコードが必須ですが、`optional: true` を指定することで、未ログインのゲストユーザーでも診断ログを作成できるようにしています。
- **`enum` による飲酒タイミングの管理**：`enum` を使うことで、数値（0/1/2）を人間が読みやすい名前（before_drinking/during_drinking/after_drinking）で扱えます。また、`log.before_drinking?` や `log.before_drinking!` といった便利なメソッドが自動生成されます。
- **`dependent: :destroy` によるデータ整合性**：診断ログを削除したとき、関連する中間テーブルのデータも自動削除されるため、孤立したレコードが残るのを防いでいます。
