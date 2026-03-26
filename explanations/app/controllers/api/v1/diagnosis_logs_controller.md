# diagnosis_logs_controller.rb の解説

## このファイルの役割

このファイルは、ユーザーの「診断ログ（服薬履歴・症状診断の記録）」を管理するコントローラーです。ログの一覧取得・詳細表示・新規作成・削除に加え、症状をもとに薬を提案する `calculate` アクションも持っています。`before_action` でログインチェックを行い、未ログインのユーザーはアクセスできないよう保護されています。

## コード全体

```ruby
module Api
  module V1
    class DiagnosisLogsController < ApplicationController
      before_action :ensure_logged_in, only: %i[index show create destroy]

      def index
        @logs = current_user.diagnosis_logs
          .includes(:symptoms, :drugs)
          .order(created_at: :desc)

        render json: @logs.as_json(
          include: {
            symptoms: { only: %i[id name] },
            drugs: { only: %i[id name description pharmacist_advice] }
          }
        ), status: :ok
      end

      def calculate
        symptom_ids = params[:symptom_ids] || []
        timing = params[:timing]

        service = DiagnosisService.new(symptom_ids, timing)
        result = service.execute # { drugs: [...], summary: "..." } が返ってくる

        render json: {
          status: 'success',
          suggested_drugs: result[:drugs],
          result_summary: result[:summary],
          symptom_ids: symptom_ids,
          timing: timing
        }, status: :ok
      end

      def show
        @log = current_user.diagnosis_logs
          .includes(:symptoms, :drugs)
          .find(params[:id])

        render json: @log.as_json(
          include: {
            symptoms: { only: %i[id name category] },
            drugs: { only: %i[id name description pharmacist_advice] } # 薬にcategoryがない場合は消しておきます
          }
        ), status: :ok
      rescue ActiveRecord::RecordNotFound
        render json: { error: '履歴が見つかりませんでした' }, status: :not_found
      end

      def create
        symptom_ids = params[:symptom_ids]
        timing = params[:timing]
        drug_ids = params[:drug_ids]
        result_summary = params[:result_summary]

        diagnosis_log = current_user.diagnosis_logs.build(
          timing: timing,
          result_summary: result_summary
        )

        if diagnosis_log.save
          diagnosis_log.symptom_ids = symptom_ids
          diagnosis_log.drug_ids = drug_ids

          render json: {
            status: 'success',
            diagnosis_log_id: diagnosis_log.id
          }, status: :created
        else
          render json: { status: 'error', message: diagnosis_log.errors.full_messages }, status: :unprocessable_content
        end
      end
      # rubocop:enable all

      def destroy
        @log = current_user.diagnosis_logs.find(params[:id])
        if @log.destroy
          render json: { message: '履歴を削除しました' }, status: :ok
        else
          render json: { error: '削除に失敗しました' }, status: :unprocessable_content
        end
      rescue ActiveRecord::RecordNotFound
        render json: { error: '履歴が見つかりませんでした' }, status: :not_found
      end

      private

      # ログインしていない場合に401を返すメソッド
      def ensure_logged_in
        return unless current_user.nil?

        render json: { error: 'ログインが必要です' }, status: :unauthorized
      end
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `module Api` | `Api` という名前空間の開始です。 |
| 2 | `module V1` | APIバージョン1の名前空間の開始です。 |
| 3 | `class DiagnosisLogsController < ApplicationController` | `DiagnosisLogsController` クラスを定義し、`ApplicationController` を継承します。`ApplicationController` を継承することで、認証ヘルパーなどアプリ共通の機能が使えます。 |
| 4 | `before_action :ensure_logged_in, only: %i[index show create destroy]` | `index`・`show`・`create`・`destroy` の各アクションが実行される前に、自動的に `ensure_logged_in` を呼び出します。`before_action` とは「アクション実行前に割り込む処理」のことです。`%i[...]` はシンボルの配列を作る省略記法です。 |
| 6 | `def index` | ログイン中ユーザーの診断ログ一覧を返すアクションです。 |
| 7 | `@logs = current_user.diagnosis_logs` | ログイン中のユーザー（`current_user`）が持つ診断ログを取得します。`@logs` はインスタンス変数で、このアクション内のどこからでも参照できます。 |
| 8 | `.includes(:symptoms, :drugs)` | 関連する `symptoms`（症状）と `drugs`（薬）のデータを一緒に取得します。`includes` は「N+1問題」（ループのたびにSQLが発行される問題）を防ぐ便利なメソッドです。 |
| 9 | `.order(created_at: :desc)` | 作成日時の新しい順（降順）で並び替えます。`:desc` は「降順（大きい順）」を意味します。 |
| 11〜16 | `render json: @logs.as_json(include: {...}), status: :ok` | ログ一覧をJSON形式で返します。`as_json(include: {...})` で関連データ（症状・薬）も含めて出力します。`only:` で必要なカラムのみに絞り込みます。 |
| 19 | `def calculate` | 症状IDと服薬タイミングを受け取り、推奨薬を返すアクションです。ログインなしでも呼び出せます（`before_action` の対象外）。 |
| 20 | `symptom_ids = params[:symptom_ids] \|\| []` | リクエストから症状IDの配列を取得します。値がない場合は空配列 `[]` をデフォルトにします。 |
| 21 | `timing = params[:timing]` | 服薬タイミング（例：「食後」「食前」）をリクエストから取得します。 |
| 23 | `service = DiagnosisService.new(symptom_ids, timing)` | `DiagnosisService` クラスのインスタンスを作成します。サービスクラスとは、コントローラーから複雑なビジネスロジックを切り出した専用クラスです。 |
| 24 | `result = service.execute` | サービスの `execute` メソッドを呼び出して診断を実行します。コメントにあるように `{ drugs: [...], summary: "..." }` という形式のハッシュが返ります。 |
| 26〜32 | `render json: { status: 'success', ... }, status: :ok` | 診断結果をJSON形式で返します。推奨薬リスト・結果サマリー・症状ID・タイミングを含みます。 |
| 35 | `def show` | 特定の診断ログ1件を詳細表示するアクションです。 |
| 36〜38 | `@log = current_user.diagnosis_logs.includes(...).find(params[:id])` | URLパラメーター `:id` で指定されたログを取得します。`find` はIDで1件検索し、見つからない場合は `ActiveRecord::RecordNotFound` 例外を発生させます。 |
| 40〜45 | `render json: @log.as_json(include: {...}), status: :ok` | ログ詳細をJSON形式で返します。`show` では症状に `category` も含まれています。 |
| 46 | `rescue ActiveRecord::RecordNotFound` | `find` でIDが見つからなかった場合の例外処理です。`rescue` とは「例外（エラー）が発生したときに代わりに実行する処理」を記述するキーワードです。 |
| 47 | `render json: { error: '履歴が見つかりませんでした' }, status: :not_found` | 404エラー（見つからない）をJSONで返します。`:not_found` はHTTPステータスコード404です。 |
| 50 | `def create` | 診断ログを新規作成するアクションです。 |
| 51〜54 | `symptom_ids = ...` 〜 `result_summary = ...` | リクエストから症状ID配列・タイミング・薬ID配列・結果サマリーをそれぞれ取得します。 |
| 56〜59 | `diagnosis_log = current_user.diagnosis_logs.build(...)` | 現在のユーザーに紐づいた診断ログのオブジェクトを作成します。`build` はデータベースへの保存はせず、オブジェクトだけ作成します（`new` と同じ意味）。 |
| 61 | `if diagnosis_log.save` | 診断ログをデータベースに保存します。`save` は成功すれば `true`、バリデーションエラーなどで失敗すれば `false` を返します。 |
| 62 | `diagnosis_log.symptom_ids = symptom_ids` | 保存後に症状の関連付けを設定します。先にログを保存してIDを確定させてから中間テーブルに紐付けします。 |
| 63 | `diagnosis_log.drug_ids = drug_ids` | 同様に薬の関連付けを設定します。 |
| 65〜68 | `render json: { status: 'success', diagnosis_log_id: ... }, status: :created` | 作成成功時に、新しいログのIDをJSONで返します。`:created` はHTTPステータスコード201（作成完了）です。 |
| 70 | `render json: { status: 'error', message: ... }, status: :unprocessable_content` | バリデーションエラーのメッセージをJSONで返します。 |
| 75 | `def destroy` | 診断ログを削除するアクションです。 |
| 76 | `@log = current_user.diagnosis_logs.find(params[:id])` | 削除対象のログを取得します。他のユーザーのログは `current_user.diagnosis_logs` から検索するため取得できず、セキュリティを確保しています。 |
| 77 | `if @log.destroy` | ログを削除します。`destroy` はデータベースからレコードを削除し、成功すると `true` を返します。 |
| 78 | `render json: { message: '履歴を削除しました' }, status: :ok` | 削除成功時のメッセージをJSONで返します。 |
| 80 | `render json: { error: '削除に失敗しました' }, status: :unprocessable_content` | 削除失敗時のエラーをJSONで返します。 |
| 82 | `rescue ActiveRecord::RecordNotFound` | ログが見つからない場合の例外処理です。 |
| 83 | `render json: { error: '履歴が見つかりませんでした' }, status: :not_found` | 404エラーをJSONで返します。 |
| 86 | `private` | 以降のメソッドをプライベート（外部から呼び出し不可）にします。 |
| 89 | `def ensure_logged_in` | ログイン状態を確認するプライベートメソッドです。`before_action` から呼ばれます。 |
| 90 | `return unless current_user.nil?` | `current_user` が `nil`（未ログイン）でなければ、何もせずに処理を戻します。`unless` は「〜でなければ」という条件を表します。 |
| 92 | `render json: { error: 'ログインが必要です' }, status: :unauthorized` | 未ログインの場合、401（認証が必要）エラーをJSON形式で返します。`:unauthorized` はHTTPステータスコード401です。 |
| 94〜96 | `end` × 3 | それぞれ `DiagnosisLogsController` クラス、`V1` モジュール、`Api` モジュールの終了です。 |

## まとめ

- **`before_action` によるアクセス制御**: `ensure_logged_in` を `before_action` で設定することで、コントローラー全体の認証ロジックを1か所にまとめています。各アクションに同じチェックを書く必要がありません。
- **`includes` でN+1問題を防止**: 一覧取得時に `.includes(:symptoms, :drugs)` を使うことで、関連データを効率よく取得します。これを使わないと、ループのたびにSQLが発行されてパフォーマンスが低下します。
- **サービスクラスの活用**: 薬の提案ロジックは `DiagnosisService` に切り出すことで、コントローラーをシンプルに保っています。複雑なビジネスロジックをサービスクラスに分離するのは設計の良い習慣です。
- **`rescue` による例外処理**: IDが存在しない場合などの例外を `rescue ActiveRecord::RecordNotFound` でキャッチし、適切な404エラーを返します。Railsでは例外をうまく利用してエラーレスポンスを分岐させることができます。
- **関連付けの順序**: `create` では先にログ本体を `save` してIDを取得し、その後で `symptom_ids=` と `drug_ids=` を設定しています。中間テーブルへの保存にはIDが必要なため、この順序が重要です。
