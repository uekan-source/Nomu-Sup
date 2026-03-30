# diagnosis_log_drug.rb の解説

  ## このファイルの役割

  `DiagnosisLogDrug` は、「診断ログ（DiagnosisLog）」と「薬（Drug）」を結びつける**中間テーブル**のモデルです。例えば、ある診断ログに複数の薬が関連付けられる場合、その組み合わせを管理するのがこのモデルの役割です。アプリ全体では、ユーザーが飲んでいる薬と診断ログを紐付けるために使われています。

  ## コード全体

  ```ruby
  class DiagnosisLogDrug < ApplicationRecord
    belongs_to :diagnosis_log
    belongs_to :drug
      end
      ```

      ## 行ごとの解説

      | 行番号 | コード | 解説 |
      |--------|--------|------|
      | 1 | `class DiagnosisLogDrug < ApplicationRecord` | `DiagnosisLogDrug` というクラス（設計図）を定義します。`< ApplicationRecord` は「ApplicationRecordを継承する」という意味で、Railsのデータベース機能が使えるようになります。 |
      | 2 | `belongs_to :diagnosis_log` | このモデルは `DiagnosisLog`（診断ログ）に属することを宣言します。`belongs_to` とは「〜に属する」という関連付けで、この中間テーブルのレコードは必ず1つの診断ログとセットになります。 |
      | 3 | `belongs_to :drug` | このモデルは `Drug`（薬）にも属することを宣言します。1つの診断ログ薬レコードは、必ず1つの薬とセットになります。 |
      | 4 | `end` | クラス定義の終わりを示します。 |

      ## まとめ

      - `DiagnosisLogDrug` は「診断ログ」と「薬」の多対多の関係を実現する中間テーブルのモデルです
      - `belongs_to` を2つ使うことで、両方のモデルへの「所属」関係を定義しています
      - 中間テーブルを使うことで、1つの診断ログに複数の薬を、1つの薬を複数の診断ログに関連付けることができます
      - Railsでは中間テーブルのモデル名を「AとBを結びつける」という命名規則でつけることが多いです（例：DiagnosisLog + Drug = DiagnosisLogDrug）
      - このファイルはシンプルですが、アプリの「どの薬をいつ飲んだか」を記録するために重要な役割を担っています
