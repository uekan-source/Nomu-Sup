# simulations_controller.rb の解説

## このファイルの役割

このファイルはアルコールシミュレーション機能のコントローラーです。フロントエンドから送られてきた「体重・性別・体質・飲んだお酒の情報」をもとに、`AlcoholSimulationService`（計算ロジック）を呼び出して血中アルコール濃度のシミュレーション結果を返します。コントローラー自体は計算を行わず、パラメータを受け取ってサービスに渡し、結果をJSONで返すだけというシンプルな役割を担っています。

## コード全体

```ruby
module Api
  module V1
      class SimulationsController < ApplicationController
            def calculate
                    weight = params[:weight].to_f
                            gender = params[:gender].presence || 'male'
                                    constitution = params[:constitution].presence || 'normal'
                                            drinks = params[:drinks] || []

                                                    service = AlcoholSimulationService.new(
                                                              weight: weight,
                                                                        gender: gender,
                                                                                  constitution: constitution,
                                                                                            drinks: drinks
                                                                                                    )
                                                                                                    
                                                                                                            result = service.execute
                                                                                                            
                                                                                                                    render json: {
                                                                                                                              status: 'success',
                                                                                                                                        data: result
                                                                                                                                                }, status: :ok
                                                                                                                                                      end
                                                                                                                                                          end
                                                                                                                                                            end
                                                                                                                                                            end
                                                                                                                                                            ```
                                                                                                                                                            
                                                                                                                                                            ## 行ごとの解説
                                                                                                                                                            
                                                                                                                                                            | 行番号 | コード | 解説 |
                                                                                                                                                            |--------|--------|------|
                                                                                                                                                            | 1 | `module Api` | `Api` モジュールを定義します。APIに関連するコードをまとめる名前空間（グループ）です。 |
                                                                                                                                                            | 2 | `module V1` | バージョン1を表す `V1` モジュールを `Api` の中に定義します。将来バージョンアップしたときに `V2` を別に作れます。 |
                                                                                                                                                            | 3 | `class SimulationsController < ApplicationController` | `SimulationsController` クラスを定義し、`ApplicationController` を継承します。シミュレーション機能に関するHTTPリクエストをここで処理します。 |
                                                                                                                                                            | 4 | `def calculate` | `calculate` アクションを定義します。`POST /api/v1/simulations/calculate` のようなリクエストが来たときに実行されます。 |
                                                                                                                                                            | 5 | `weight = params[:weight].to_f` | リクエストパラメータから体重（`weight`）を取得し、`.to_f` で浮動小数点数（小数を扱える数値型）に変換します。例：`"65"` → `65.0` |
                                                                                                                                                            | 6 | `gender = params[:gender].presence \|\| 'male'` | 性別パラメータを取得します。`.presence` は「値が存在して、かつ空文字でない場合にその値を返し、そうでなければ `nil` を返す」メソッドです。値がない場合は `'male'`（デフォルト）を使います。 |
                                                                                                                                                            | 7 | `constitution = params[:constitution].presence \|\| 'normal'` | 体質パラメータを取得します。値がなければデフォルトで `'normal'`（普通体質）を使います。 |
                                                                                                                                                            | 8 | `drinks = params[:drinks] \|\| []` | 飲んだお酒のリストを取得します。パラメータがなければ空の配列 `[]` をデフォルトにします。 |
                                                                                                                                                            | 9 | （空行） | 読みやすさのために空行を入れています。 |
                                                                                                                                                            | 10 | `service = AlcoholSimulationService.new(` | `AlcoholSimulationService` クラスのインスタンス（実体）を作成します。`.new()` はクラスからオブジェクトを生成するRubyの書き方です。 |
                                                                                                                                                            | 11 | `weight: weight,` | サービスに体重を渡します。`weight: weight` はキーワード引数で、「`weight` というキーに変数 `weight` の値を渡す」という意味です。 |
                                                                                                                                                            | 12 | `gender: gender,` | サービスに性別を渡します。 |
                                                                                                                                                            | 13 | `constitution: constitution,` | サービスに体質を渡します。 |
                                                                                                                                                            | 14 | `drinks: drinks` | サービスに飲み物リストを渡します。 |
                                                                                                                                                            | 15 | `)` | `AlcoholSimulationService.new(...)` の閉じ括弧です。 |
                                                                                                                                                            | 16 | （空行） | 読みやすさのために空行を入れています。 |
                                                                                                                                                            | 17 | `result = service.execute` | サービスの `execute` メソッドを呼び出し、シミュレーション結果を `result` 変数に格納します。実際の計算はサービス側が行います。 |
                                                                                                                                                            | 18 | （空行） | 読みやすさのために空行を入れています。 |
                                                                                                                                                            | 19 | `render json: {` | JSONレスポンスを返し始めます。 |
                                                                                                                                                            | 20 | `status: 'success',` | レスポンスの `status` フィールドに `'success'` という文字列をセットします。 |
                                                                                                                                                            | 21 | `data: result` | レスポンスの `data` フィールドに計算結果を入れます。 |
                                                                                                                                                            | 22 | `}, status: :ok` | JSONオブジェクトを閉じ、HTTPステータスコード200（`:ok`）を指定してレスポンスを返します。 |
                                                                                                                                                            | 23〜26 | `end` × 4 | `calculate` メソッド、`SimulationsController` クラス、`V1` モジュール、`Api` モジュールをそれぞれ閉じます。 |
                                                                                                                                                            
                                                                                                                                                            ## まとめ
                                                                                                                                                            
                                                                                                                                                            - **コントローラーは薄く保つ**: このコントローラーはパラメータを受け取って渡すだけで、計算ロジックは `AlcoholSimulationService` に委譲しています。これが「Fat Model, Skinny Controller（コントローラーは薄く）」という設計原則です。
                                                                                                                                                            - **`.to_f` で型変換**: HTTPリクエストのパラメータは全て文字列で来るため、数値として使うには `.to_f`（小数）や `.to_i`（整数）で変換する必要があります。
                                                                                                                                                            - **`.presence` でデフォルト値**: `nil` や空文字の場合に `||` でデフォルト値を設定するパターンはRailsでよく使われます。`.presence` は `nil` と空文字を同時に処理できて便利です。
                                                                                                                                                            - **サービスオブジェクトパターン**: 複雑なビジネスロジックをコントローラーに直接書かず、専用のサービスクラスに切り出すのはRailsの一般的なパターンです。
                                                                                                                                                            - **キーワード引数**: `method(key: value)` という形で引数を渡すと、何の値を渡しているかがコードを読む人に伝わりやすくなります。
