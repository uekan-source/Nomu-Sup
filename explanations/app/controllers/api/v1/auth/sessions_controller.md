# sessions_controller.rb の解説

## このファイルの役割

`SessionsController` は、ユーザーの「ログイン（サインイン）」と「ログアウト（サインアウト）」を担当するコントローラです。Deviseの `SessionsController` を継承し、JWT（JSONウェブトークン）を使ったAPI向けのログイン処理にカスタマイズしています。ログイン成功時はJWTトークンをJSON形式で返し、フロントエンドがそのトークンを使って認証を行います。

## コード全体

```ruby
module Api
  module V1
    module Auth
      class SessionsController < Devise::SessionsController
        # APIモードなのでJSON形式でレスポンスを返す
        respond_to :json

        # ログイン（サインイン）
        def create
          user = User.find_by(email: params[:user][:email])

          if user&.valid_password?(params[:user][:password])
            token, _payload = Warden::JWTAuth::UserEncoder.new.call(user, :user, nil)

            Rails.logger.info "Generated Token: #{token}"

            render json: {
              status: 'success',
              token: token,
              data: user
            }, status: :ok
          else
            render json: {
              status: 'error',
              message: 'メールアドレスまたはパスワードが間違っています。'
            }, status: :unauthorized
          end
        end

        # ログアウト（サインアウト）
        def destroy
          (Devise.sign_out_all_scopes ? sign_out : sign_out(resource_name))
          render json: {
            status: 'success',
            message: 'ログアウトしました'
          }, status: :ok
        end

        private

        # Deviseに渡すパラメーターの形式を定義
        def sign_in_params
          params.require(:user).permit(:email, :password)
        end

        # 認証失敗時の挙動をカスタマイズ（401 Unauthorizedを返す）
        def respond_to_on_destroy
          head :no_content
        end
      end
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `module Api` | `Api` というモジュールを定義します。モジュールは名前空間（グループ名）として使われ、コントローラをAPIのグループにまとめます。 |
| 2 | `module V1` | `V1`（バージョン1）モジュールを定義します。APIのバージョン管理のためにネストしています。`/api/v1/...` というURLパスに対応します。 |
| 3 | `module Auth` | `Auth`（認証）モジュールを定義します。認証関連のコントローラをまとめるグループです。 |
| 4 | `class SessionsController < Devise::SessionsController` | `SessionsController` クラスを定義し、`Devise::SessionsController` を継承します。Deviseのログイン処理を引き継ぎつつ、API向けにカスタマイズします。 |
| 5 | `# APIモードなのでJSON形式でレスポンスを返す` | コメントです。コードの実行には影響しません。 |
| 6 | `respond_to :json` | このコントローラはJSON形式のレスポンスのみを返すように設定します。ブラウザ向けのHTML形式は返しません。 |
| 7 | （空行） | 読みやすさのための空行です。 |
| 8 | `# ログイン（サインイン）` | コメントです。コードの実行には影響しません。 |
| 9 | `def create` | `create` メソッドの定義を開始します。HTTPの `POST` リクエストに対応するアクションで、ログイン処理を行います。 |
| 10 | `user = User.find_by(email: params[:user][:email])` | リクエストパラメータから `:user` キーの `:email` を取り出し、そのメールアドレスでユーザーをデータベースから検索します。見つからない場合は `nil` が返ります。 |
| 11 | （空行） | 読みやすさのための空行です。 |
| 12 | `if user&.valid_password?(params[:user][:password])` | `user` が存在し（`&.` はnilセーフ演算子で、`user` が `nil` の場合はエラーを出さず `nil` を返す）、かつパスワードが正しい場合の条件分岐です。`valid_password?` はDeviseが提供するパスワード検証メソッドです。 |
| 13 | `token, _payload = Warden::JWTAuth::UserEncoder.new.call(user, :user, nil)` | JWTトークンを生成します。`UserEncoder` にユーザー情報を渡してトークンを作成します。`_payload` の先頭の `_` は「使わない変数」を示す慣習です。 |
| 14 | （空行） | 読みやすさのための空行です。 |
| 15 | `Rails.logger.info "Generated Token: #{token}"` | Railsのログに「Generated Token: （トークン文字列）」と出力します。デバッグや動作確認のために使います。本番環境ではセキュリティ上削除することが推奨されます。 |
| 16 | （空行） | 読みやすさのための空行です。 |
| 17〜21 | `render json: { status: 'success', token: token, data: user }, status: :ok` | ログイン成功時のレスポンスをJSON形式で返します。`status: 'success'`、生成した `token`、ユーザー情報 `data: user` を含み、HTTPステータスコード `200 OK` を返します。 |
| 22 | `else` | パスワードが間違っている、またはユーザーが見つからない場合の処理です。 |
| 23〜26 | `render json: { status: 'error', message: 'メールアドレスまたはパスワードが間違っています。' }, status: :unauthorized` | ログイン失敗時のレスポンスをJSON形式で返します。HTTPステータスコード `401 Unauthorized`（認証失敗）を返します。 |
| 27 | `end` | `if`～`else` ブロックの終了です。 |
| 28 | `end` | `create` メソッドの終了です。 |
| 29 | （空行） | 読みやすさのための空行です。 |
| 30 | `# ログアウト（サインアウト）` | コメントです。コードの実行には影響しません。 |
| 31 | `def destroy` | `destroy` メソッドの定義を開始します。HTTPの `DELETE` リクエストに対応するアクションで、ログアウト処理を行います。 |
| 32 | `(Devise.sign_out_all_scopes ? sign_out : sign_out(resource_name))` | Deviseの設定 `sign_out_all_scopes` が `true` ならすべてのスコープからサインアウト、`false` なら現在のリソース（ユーザー）のみサインアウトします。 |
| 33〜36 | `render json: { status: 'success', message: 'ログアウトしました' }, status: :ok` | ログアウト成功のレスポンスをJSON形式で返します。HTTPステータスコード `200 OK` を返します。 |
| 37 | `end` | `destroy` メソッドの終了です。 |
| 38 | （空行） | 読みやすさのための空行です。 |
| 39 | `private` | この行より下のメソッドを `private`（外部から直接呼び出せない）にします。内部処理用のメソッドに使います。 |
| 40 | （空行） | 読みやすさのための空行です。 |
| 41 | `# Deviseに渡すパラメーターの形式を定義` | コメントです。コードの実行には影響しません。 |
| 42 | `def sign_in_params` | `sign_in_params` メソッドの定義を開始します。Deviseに渡すパラメータを安全にフィルタリングします。 |
| 43 | `params.require(:user).permit(:email, :password)` | `params` から `:user` キーの存在を必須とし（`require`）、その中の `:email` と `:password` のみを許可します（`permit`）。これは「ストロングパラメータ」と呼ばれるセキュリティ機能で、意図しないパラメータが渡されるのを防ぎます。 |
| 44 | `end` | `sign_in_params` メソッドの終了です。 |
| 45 | （空行） | 読みやすさのための空行です。 |
| 46 | `# 認証失敗時の挙動をカスタマイズ（401 Unauthorizedを返す）` | コメントです。コードの実行には影響しません。 |
| 47 | `def respond_to_on_destroy` | `respond_to_on_destroy` メソッドの定義を開始します。Deviseのデフォルト挙動をオーバーライド（上書き）するメソッドです。 |
| 48 | `head :no_content` | レスポンスボディなしで、HTTPステータスコード `204 No Content` を返します。JWTはサーバー側にセッションを保持しないため、ログアウト時はこのようにシンプルなレスポンスを返します。 |
| 49〜53 | `end` × 5 | 各ブロック（`respond_to_on_destroy`, `SessionsController`, `Auth`, `V1`, `Api`）の終了です。 |

## まとめ

- **モジュールのネスト**（`module Api > module V1 > module Auth`）でURLの階層を表現し、コントローラを整理できます。
- **`&.`（nilセーフ演算子）**を使うと、`nil` に対してメソッドを呼び出してもエラーにならず安全に処理できます。
- **JWTトークン**はログイン成功時にサーバーが生成し、フロントエンドに渡します。以降のリクエストはこのトークンで認証します。
- **ストロングパラメータ**（`params.require.permit`）により、受け取るパラメータを明示的に制限してセキュリティを高めます。
- **HTTPステータスコード**（`:ok`=200, `:unauthorized`=401, `:no_content`=204）を適切に使い分けることで、APIクライアントが結果を正しく判断できます。
