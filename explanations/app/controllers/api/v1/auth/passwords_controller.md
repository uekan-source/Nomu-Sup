# passwords_controller.rb の解説

## このファイルの役割

このファイルは、ユーザーがパスワードを忘れたときの「パスワード再設定」機能を担うコントローラーです。Deviseという認証ライブラリの `PasswordsController` を継承し、JSON形式のAPIとして動作するようカスタマイズしています。具体的には、「パスワード再設定メールの送信」と「新しいパスワードへの変更」という2つの機能を提供します。

## コード全体

```ruby
module Api
  module V1
    module Auth
      class PasswordsController < Devise::PasswordsController
        respond_to :json

        # POST /api/v1/auth/password
        def create
          reset_params = params.require(:user).permit(:email)
          self.resource = resource_class.send_reset_password_instructions(reset_params)

          yield resource if block_given?

          if successfully_sent?(resource)
            render json: { message: 'パスワード再設定メールを送信しました。' }, status: :ok
          else
            render json: { error: resource.errors.full_messages }, status: :unprocessable_content
          end
        end

        # PUT /api/v1/auth/password (新パスワード設定)
        def update
          update_params = params.require(:user).permit(:reset_password_token, :password, :password_confirmation)
          self.resource = resource_class.reset_password_by_token(update_params)

          yield resource if block_given?

          if resource.errors.empty?
            resource.unlock_access! if unlockable?(resource)
            render json: { message: 'パスワードが正しく変更されました。' }, status: :ok
          else
            render json: { error: resource.errors.full_messages }, status: :unprocessable_content
          end
        end
        # rubocop:enable all
      end
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `module Api` | `Api` という名前空間（モジュール）を開始します。モジュールとは、クラスや定数をグループ化するための仕組みで、名前の衝突を防ぎます。 |
| 2 | `module V1` | `V1` という名前空間を開始します。APIのバージョン1を表しています。将来バージョン2が必要になった場合に `V2` を作れるよう、最初からバージョン管理しています。 |
| 3 | `module Auth` | `Auth`（認証）という名前空間を開始します。認証関連のコントローラーをまとめています。 |
| 4 | `class PasswordsController < Devise::PasswordsController` | `PasswordsController` クラスを定義し、Deviseの `PasswordsController` を継承しています。`<` は継承を意味し、Deviseが提供するパスワード管理の機能を引き継ぎます。 |
| 5 | `respond_to :json` | このコントローラーはJSONのみでレスポンスを返すことを宣言しています。`respond_to` とは、対応するフォーマットを指定するメソッドです。 |
| 6 | （空行） | 読みやすさのための空行です。 |
| 7 | `# POST /api/v1/auth/password` | コメントです。このアクションが `POST` メソッドで `/api/v1/auth/password` というURLで呼ばれることを示しています。 |
| 8 | `def create` | `create` アクションの定義開始です。パスワード再設定メールを送信するアクションです。 |
| 9 | `reset_params = params.require(:user).permit(:email)` | リクエストから `:user` キーのパラメーターを必須として取り出し、`:email` のみを許可します。`params` はリクエストで送られてきたデータ、`require` は必須チェック、`permit` はセキュリティのためホワイトリスト（許可リスト）で絞り込む処理です。 |
| 10 | `self.resource = resource_class.send_reset_password_instructions(reset_params)` | Deviseのメソッド `send_reset_password_instructions` を呼び出し、パスワード再設定用のトークンを生成してメールを送信します。`self.resource` に処理結果のユーザーオブジェクトを格納します。 |
| 11 | （空行） | 読みやすさのための空行です。 |
| 12 | `yield resource if block_given?` | ブロック（追加処理）が渡されている場合に実行します。`yield` は呼び出し元に処理を渡すキーワードで、`block_given?` はブロックが存在するかを確認します。通常の使用では省略できるコードです。 |
| 13 | （空行） | 読みやすさのための空行です。 |
| 14 | `if successfully_sent?(resource)` | メールが正常に送信されたか確認します。`successfully_sent?` はDeviseが提供するメソッドで、送信成功かどうかを真偽値で返します。 |
| 15 | `render json: { message: 'パスワード再設定メールを送信しました。' }, status: :ok` | 成功した場合、JSONでメッセージを返します。`status: :ok` はHTTPステータスコード200（成功）を意味します。 |
| 16 | `else` | 送信に失敗した場合の分岐です。 |
| 17 | `render json: { error: resource.errors.full_messages }, status: :unprocessable_content` | エラーが発生した場合、エラーメッセージをJSON形式で返します。`resource.errors.full_messages` はエラーメッセージの配列です。`status: :unprocessable_content` はHTTPステータスコード422（処理不能）です。 |
| 18 | `end` | `if` ブロックの終了です。 |
| 19 | `end` | `create` アクションの終了です。 |
| 20 | （空行） | 読みやすさのための空行です。 |
| 21 | `# PUT /api/v1/auth/password (新パスワード設定)` | コメントです。このアクションが `PUT` メソッドで呼ばれることを示しています。 |
| 22 | `def update` | `update` アクションの定義開始です。実際にパスワードを新しいものに変更するアクションです。 |
| 23 | `update_params = params.require(:user).permit(:reset_password_token, :password, :password_confirmation)` | リクエストから `:reset_password_token`（再設定用トークン）、`:password`（新パスワード）、`:password_confirmation`（確認用パスワード）のみを許可して取得します。 |
| 24 | `self.resource = resource_class.reset_password_by_token(update_params)` | Deviseの `reset_password_by_token` メソッドを使い、トークンを検証してパスワードを変更します。結果のユーザーオブジェクトを `self.resource` に格納します。 |
| 25 | （空行） | 読みやすさのための空行です。 |
| 26 | `yield resource if block_given?` | 12行目と同様、ブロックが渡されていれば実行します。 |
| 27 | （空行） | 読みやすさのための空行です。 |
| 28 | `if resource.errors.empty?` | パスワード変更にエラーがないか確認します。`errors.empty?` はエラーが空（なし）の場合に `true` を返します。 |
| 29 | `resource.unlock_access! if unlockable?(resource)` | アカウントがロックされている場合（ログイン試行回数超過など）、パスワード変更に成功したのでロックを解除します。`unlockable?` はロック機能が有効かを確認するメソッドです。 |
| 30 | `render json: { message: 'パスワードが正しく変更されました。' }, status: :ok` | パスワード変更成功のメッセージをJSONで返します。 |
| 31 | `else` | エラーがある場合の分岐です。 |
| 32 | `render json: { error: resource.errors.full_messages }, status: :unprocessable_content` | エラーメッセージをJSON形式で返します。 |
| 33 | `end` | `if` ブロックの終了です。 |
| 34 | `end` | `update` アクションの終了です。 |
| 35 | `# rubocop:enable all` | RuboCopという静的解析ツール（コードの品質チェックツール）の全ルールを有効にするコメントです。このファイル内で一時的に無効にしたルールを再有効化しています。 |
| 36 | `end` | `PasswordsController` クラスの終了です。 |
| 37 | `end` | `Auth` モジュールの終了です。 |
| 38 | `end` | `V1` モジュールの終了です。 |
| 39 | `end` | `Api` モジュールの終了です。 |

## まとめ

- **Deviseの継承**: `Devise::PasswordsController` を継承することで、パスワード管理の複雑な処理（トークン生成、メール送信など）を自分で書かずに利用できます。
- **Strong Parameters（強力なパラメーター）**: `params.require(...).permit(...)` でリクエストデータを安全に取り扱います。許可していないパラメーターは自動的に除外されるため、不正なデータの混入を防ぎます。
- **2段階のパスワードリセットフロー**: まず `create` でメールを送り（トークン発行）、次に `update` でトークンを検証してパスワードを変更するという2段階の安全な設計になっています。
- **HTTPステータスコードの使い分け**: 成功時は `:ok`（200）、エラー時は `:unprocessable_content`（422）を返し、APIクライアントが処理結果を判別しやすくしています。
- **アカウントロック解除の連携**: パスワード変更成功時にアカウントロックを解除する処理を組み込んでいます。これにより、ロックされたアカウントでもパスワード再設定で復旧できます。
