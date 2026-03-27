# symptoms_controller.rb の解説

## このファイルの役割

このファイルは症状データを返すコントローラーです。診断ログを記録したり、シミュレーションを行う際に、ユーザーが選択できる「症状・体質・気分/予定」の選択肢一覧をデータベースから取得してJSON形式で返します。`timing`（タイミング：飲酒前・飲酒中・飲酒後など）パラメータで絞り込み、カテゴリ別に分けて返すのが特徴です。

## コード全体

```ruby
module Api
  module V1
      class SymptomsController < ApplicationController
            def index
                    base_query = Symptom.where(timing: params[:timing])

                            render json: {
                                      moods: base_query.where(category: 2), # 気分・予定
                                                symptoms: base_query.where(category: 0), # 症状
                                                          constitutions: base_query.where(category: 1) # 体質
                                                                  }, status: :ok
                                                                        end
                                                                            end
                                                                              end
                                                                              end
                                                                              ```

                                                                              ## 行ごとの解説

                                                                              | 行番号 | コード | 解説 |
                                                                              |--------|--------|------|
                                                                              | 1 | `module Api` | `Api` モジュールを定義します。APIに関連するコードをまとめる名前空間です。 |
                                                                              | 2 | `module V1` | バージョン1を表す `V1` モジュールを定義します。 |
                                                                              | 3 | `class SymptomsController < ApplicationController` | `SymptomsController` クラスを定義し、`ApplicationController` を継承します。症状データのHTTPリクエストをここで処理します。 |
                                                                              | 4 | `def index` | `index` アクションを定義します。`GET /api/v1/symptoms` のようなリクエストが来たときに呼ばれます。 |
                                                                              | 5 | `base_query = Symptom.where(timing: params[:timing])` | `Symptom`（症状）モデルのレコードを `timing`（タイミング）条件で絞り込んだクエリを `base_query` 変数に代入します。`.where` はSQLの `WHERE` 句に相当するActiveRecordメソッドで、条件に合うレコードだけを取得します。この時点ではまだSQLは実行されていません（遅延実行）。 |
                                                                              | 6 | （空行） | 読みやすさのための空行です。 |
                                                                              | 7 | `render json: {` | JSONレスポンスを返し始めます。複数のキーを持つハッシュ（辞書）を返します。 |
                                                                              | 8 | `moods: base_query.where(category: 2), # 気分・予定` | `base_query` にさらに `category: 2` の条件を追加して絞り込んだ結果を `moods` キーにセットします。`#` 以降はコメントで「気分・予定」カテゴリであることを説明しています。 |
                                                                              | 9 | `symptoms: base_query.where(category: 0), # 症状` | `category: 0` の条件で絞り込んだ結果を `symptoms` キーにセットします。身体的な症状（頭痛・吐き気など）がこれにあたります。 |
                                                                              | 10 | `constitutions: base_query.where(category: 1) # 体質` | `category: 1` の条件で絞り込んだ結果を `constitutions` キーにセットします。体質（お酒に強い・弱いなど）がこれにあたります。 |
                                                                              | 11 | `}, status: :ok` | JSONオブジェクトを閉じ、HTTPステータスコード200（`:ok`）を指定してレスポンスを返します。 |
                                                                              | 12 | `end` | `index` メソッドの終わりです。 |
                                                                              | 13 | `end` | `SymptomsController` クラスの終わりです。 |
                                                                              | 14 | `end` | `V1` モジュールの終わりです。 |
                                                                              | 15 | `end` | `Api` モジュールの終わりです。 |

                                                                              ## まとめ

                                                                              - **ActiveRecordの `.where` メソッド**: `Model.where(条件)` でデータベースからレコードを絞り込めます。SQLを直接書かなくてもRubyのコードで条件指定できるのがRailsの強みです。
                                                                              - **クエリの重ね掛け（チェーン）**: `base_query.where(category: 2)` のように、既に絞り込んだクエリにさらに条件を追加できます。これを「クエリのチェーン」といい、コードの重複を避けられます。
                                                                              - **遅延実行（Lazy Evaluation）**: Railsの `.where` はすぐにSQLを実行せず、実際にデータが必要になった時点（`render json:` でシリアライズする時）に実行されます。これを「遅延実行」と言います。
                                                                              - **カテゴリを数値で管理**: `category: 0`、`category: 1`、`category: 2` のように数値でカテゴリを管理しています。コメントがないと意味がわからなくなるため、`# 症状` のようなコメントが重要です。
                                                                              - **一度の取得・カテゴリ別に返却**: `timing` での絞り込みを `base_query` として共通化し、カテゴリ別に3つのキーで返すことで、フロントエンドが1回のAPIコールで必要な全データを受け取れます。
