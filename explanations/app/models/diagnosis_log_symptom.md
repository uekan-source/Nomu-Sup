# diagnosis_log_symptom.rb の解説

  ## このファイルの役割

  `DiagnosisLogSymptom` は、「診断ログ（DiagnosisLog）」と「症状（Symptom）」を結びつける**中間テーブル**のモデルです。ユーザーが特定の診断ログに対してどのような症状を記録したかを管理するために使われます。アプリ全体では、診断ログと症状の多対多の関係を実現するために機能しています。

  ## コード全体

  ```ruby
  class DiagnosisLogSymptom < ApplicationRecord
    belongs_to :diagnosis_log
    belongs_to :symptom
      end
      ```

      ## 行ごとの解説

      | 行番号 | コード | 解説 |
      |--------|--------|------|
      | 1 | `class DiagnosisLogSymptom < ApplicationRecord` | `DiagnosisLogSymptom` というクラスを定義します。`< ApplicationRecord` で継承し、Railsのデータベース機能が利用できます。 |
      | 2 | `belongs_to :diagnosis_log` | このモデルが `DiagnosisLog`（診断ログ）に属することを表します。`belongs_to` は「〜に属する」という関連付けで、このレコードは必ず1つの診断ログと対応します。 |
      | 3 | `belongs_to :symptom` | このモデルが `Symptom`（症状）にも属することを表します。1つのレコードは必ず1つの症状と対応します。 |
      | 4 | `end` | クラス定義の終わりを示します。 |

      ## まとめ

      - `DiagnosisLogSymptom` は「診断ログ」と「症状」の多対多の関係を実現する中間テーブルのモデルです
      - `belongs_to` を2つ宣言することで、2つのモデルへの所属関係を定義しています
      - `DiagnosisLogDrug` と非常に似た構造で、対象が「薬」か「症状」かの違いだけです
      - 中間テーブルのおかげで、1つの診断ログに複数の症状を、また1つの症状を複数の診断ログに関連付けることが可能になります
      - このシンプルなモデルが、「ユーザーの症状記録」という重要な機能を支えています
