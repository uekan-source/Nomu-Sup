# registrations_controller.rb の解説

## このファイルの役割

`RegistrationsController` は、新規ユーザーの「アカウント登録（サインアップ）」を担当するコントローラです。Deviseの `RegistrationsController` を継承し、API向けにカスタマイズしています。登録成功時はJWTトークンをJSON形式で即座に返し、登録と同時にログイン済み状態にできます。登録失敗時はバリデーションエラーをJSON形式で返します。

## コード全体

```ruby
module Api
  module V1
    module Auth
      class RegistrationsController < Devise::RegistrationsController
        respond_to :json

        before_action :configure_sign_up_params, only: [:create]

        def create
          build_resource(sign_up_params)

          resource.save
          if resource.persisted?
            token, _payload = Warden::JWTAuth::UserEncoder.new.call(resource, :user, nil)

            render json: {
              status: 'success',
              token: token,
              data: resource
            }, status: :ok
          else
            render json: {
              status: 'error',
              errors: resource.errors.full_messages
            }, status: :unprocessable_content
          end
        end

        protected

        def configure_sign_up_params
          devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
        end

        def sign_up_params
          params.require(:user).permit(:name, :email, :password, :password_confirmation)
        end
      end
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `module Api` | `Api` モジュールを定義します。APIコントローラをまとめる名前空間です。 |
| 2 | `module V1` | バージョン1のAPIをまとめるモジュールです。将来APIのバージョンを上げる際に `V2` などを追加できます。 |
| 3 | `module Auth` | 認証関連のコントローラをまとめるモジュールです。 |
| 4 | `class RegistrationsController < Devise::RegistrationsController` | `RegistrationsController` クラスを定義し、`Devise::RegistrationsController` を継承します。Deviseのユーザー登録機能をベースにしてAPI向けにカスタマイズします。 |
| 5 | `respond_to :json` | このコントローラはJSON形式のレスポンスのみを返します。HTML形式のレスポンスは返しません。 |
| 6 | （空行） | 読みやすさのための空行です。 |
| 7 | `before_action :configure_sign_up_params, only: [:create]` | `create` アクションが実行される前だけ `configure_sign_up_params` メソッドを呼び出します。`only: [:create]` で特定のアクションに限定しています。 |
| 8 | （空行） | 読みやすさのための空行です。 |
| 9 | `def create` | `create` メソッドの定義を開始します。HTTPの `POST` リクエストに対応し、新規ユーザー登録処理を行います。 |
| 10 | `build_resource(sign_up_params)` | Deviseが提供する `build_resource` メソッドです。`sign_up_params`（登録パラメータ）を使って、まだ保存していないUserオブジェクト（`resource`）を作成します。 |
| 11 | （空行） | 読みやすさのための空行です。 |
| 12 | `resource.save` | 作成したUserオブジェクト（`resource`）をデータベースに保存します。バリデーションエラーがある場合は保存に失敗し、`persisted?` が `false` になります。 |
| 13 | `if resource.persisted?` | `persisted?` は「データベースに正常に保存されたか？」を確認するメソッドです。保存成功なら `true`、失敗なら `false` を返します。 |
| 14 | `token, _payload = Warden::JWTAuth::UserEncoder.new.call(resource, :user, nil)` | 登録したユーザー（`resource`）のJWTトークンを生成します。登録と同時にログイン済み状態のトークンを発行します。 |
| 15 | （空行） | 読みやすさのための空行です。 |
| 16〜20 | `render json: { status: 'success', token: token, data: resource }, status: :ok` | 登録成功時のレスポンスをJSON形式で返します。`status: 'success'`、JWTトークン、登録されたユーザー情報を含み、HTTPステータスコード `200 OK` を返します。 |
| 21 | `else` | ユーザー登録に失敗した場合（バリデーションエラーなど）の処理です。 |
| 22〜25 | `render json: { status: 'error', errors: resource.errors.full_messages }, status: :unprocessable_content` | 登録失敗時のレスポンスをJSON形式で返します。`resource.errors.full_messages` にはバリデーションエラーのメッセージ一覧が配列で入っています。HTTPステータスコード `422 Unprocessable Content` を返します。 |
| 26 | `end` | `if`～`else` ブロックの終了です。 |
| 27 | `end` | `create` メソッドの終了です。 |
| 28 | （空行） | 読みやすさのための空行です。 |
| 29 | `protected` | この行より下のメソッドを `protected` にします。サブクラスからは呼び出せますが、外部からは直接呼び出せません。 |
| 30 | （空行） | 読みやすさのための空行です。 |
| 31 | `def configure_sign_up_params` | `configure_sign_up_params` メソッドの定義を開始します。Deviseのパラメータ許可設定を行うメソッドです。 |
| 32 | `devise_parameter_sanitizer.permit(:sign_up, keys: [:name])` | 新規登録（`sign_up`）時に `:name` パラメータの受け取りを許可します。Deviseのデフォルトではメールとパスワードしか受け付けないため、`name` を追加しています。 |
| 33 | `end` | `configure_sign_up_params` メソッドの終了です。 |
| 34 | （空行） | 読みやすさのための空行です。 |
| 35 | `def sign_up_params` | `sign_up_params` メソッドの定義を開始します。登録時に受け取るパラメータをストロングパラメータで制限します。 |
| 36 | `params.require(:user).permit(:name, :email, :password, :password_confirmation)` | `:user` キーを必須とし、その中の `:name`、`:email`、`:password`、`:password_confirmation` のみを許可します。`:password_confirmation` はパスワード確認入力（「パスワードをもう一度入力」）に使います。 |
| 37 | `end` | `sign_up_params` メソッドの終了です。 |
| 38〜41 | `end` × 4 | 各ブロック（`RegistrationsController`, `Auth`, `V1`, `Api`）の終了です。 |

## まとめ

- **`build_resource`** はDeviseが提供するメソッドで、Userオブジェクトを作成しつつまだ保存しない（後で `save` を呼ぶ）流れが学べます。
- **`resource.persisted?`** で保存成功・失敗を判定し、成功なら登録完了トークン、失敗ならエラーメッセージを返す分岐構造が理解できます。
- **`resource.errors.full_messages`** でバリデーションエラーメッセージ一覧を取得し、API経由でフロントエンドにエラー内容を伝える方法が学べます。
- **`before_action`の `only:` オプション**で特定のアクションにだけ前処理を適用できます。
- **登録と同時にJWTトークンを発行**することで、ユーザーが登録後すぐにログイン状態になれるUX（ユーザー体験）を実現しています。
