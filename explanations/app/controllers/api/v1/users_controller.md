# users_controller.rb の解説

## このファイルの役割

このファイルは、ログイン中のユーザー情報を取得・更新するためのAPIコントローラーです。`show` アクションでユーザーの情報をJSON形式で返し、`update` アクションでユーザーのプロフィールを変更します。認証済みのユーザーのみが自分の情報にアクセスできる仕組みになっています。

## コード全体

```ruby
module Api
  module V1
    class UsersController < ApplicationController
      def show
        if current_user
          render json: {
            id: current_user.id,
            name: current_user.name,
            email: current_user.email,
            weight: current_user.weight,
            gender: current_user.gender,
            constitution: current_user.constitution
          }, status: :ok
        else
          render json: { error: 'ユーザーが見つかりません' }, status: :unauthorized
        end
      end

      def update
        if current_user&.update(user_params)
          render json: { message: 'プロフィールを更新しました', user: current_user }, status: :ok
        else
          render json: { errors: current_user.errors.full_messages }, status: :unprocessable_content
        end
      end

      private

      def user_params
        params.require(:user).permit(:name, :email, :weight, :gender, :constitution)
      end
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `module Api` | `Api` という名前空間（モジュール）を定義します。名前空間とは、クラス名の衝突を防ぐための「フォルダ」のようなものです。 |
| 2 | `module V1` | `Api` の中に `V1`（バージョン1）という名前空間をさらに定義します。APIのバージョン管理に使われます。 |
| 3 | `class UsersController < ApplicationController` | `UsersController` というクラスを定義し、`ApplicationController` を継承します。継承することで、認証などの共通機能を引き継ぎます。 |
| 4 | `def show` | `show` アクションを定義します。GETリクエストでユーザー情報を返すために使われます。 |
| 5 | `if current_user` | `current_user` はDevise（認証ライブラリ）が提供するメソッドで、現在ログイン中のユーザーを返します。ログインしていない場合は `nil` を返します。 |
| 6〜13 | `render json: { id: ..., ... }, status: :ok` | ログイン中のユーザーの情報をJSON形式でレスポンスとして返します。`status: :ok` はHTTPステータスコード200（成功）を意味します。 |
| 7 | `id: current_user.id,` | ユーザーのIDをJSONに含めます。 |
| 8 | `name: current_user.name,` | ユーザーの名前をJSONに含めます。 |
| 9 | `email: current_user.email,` | ユーザーのメールアドレスをJSONに含めます。 |
| 10 | `weight: current_user.weight,` | ユーザーの体重をJSONに含めます。 |
| 11 | `gender: current_user.gender,` | ユーザーの性別をJSONに含めます。 |
| 12 | `constitution: current_user.constitution` | ユーザーの体質情報をJSONに含めます。 |
| 14 | `else` | `current_user` が存在しない（ログインしていない）場合の処理です。 |
| 15 | `render json: { error: 'ユーザーが見つかりません' }, status: :unauthorized` | エラーメッセージをJSONで返します。`status: :unauthorized` はHTTPステータスコード401（認証エラー）を意味します。 |
| 16 | `end` | `if` ブロックの終了です。 |
| 17 | `end` | `show` メソッドの終了です。 |
| 19 | `def update` | `update` アクションを定義します。PATCHリクエストでユーザー情報を更新するために使われます。 |
| 20 | `if current_user&.update(user_params)` | `&.` は「ぼっち演算子（safe navigation operator）」と呼ばれます。`current_user` が `nil` でない場合のみ `update` メソッドを呼び出します。`user_params` は更新を許可するパラメータです。 |
| 21 | `render json: { message: 'プロフィールを更新しました', user: current_user }, status: :ok` | 更新成功時に成功メッセージと更新後のユーザー情報をJSONで返します。 |
| 22 | `else` | 更新に失敗した場合（バリデーションエラーなど）の処理です。 |
| 23 | `render json: { errors: current_user.errors.full_messages }, status: :unprocessable_content` | バリデーションエラーメッセージの一覧をJSONで返します。`status: :unprocessable_content` はHTTPステータスコード422（処理不能）を意味します。 |
| 24 | `end` | `if` ブロックの終了です。 |
| 25 | `end` | `update` メソッドの終了です。 |
| 27 | `private` | この行以降のメソッドは「プライベートメソッド」になります。外部から直接呼び出せないようにします。 |
| 29 | `def user_params` | 更新を許可するパラメータを定義するプライベートメソッドです。Strong Parametersと呼ばれるセキュリティの仕組みです。 |
| 30 | `params.require(:user).permit(:name, :email, :weight, :gender, :constitution)` | リクエストパラメータの `user` キーの中から、許可された項目（name, email, weight, gender, constitution）だけを取り出します。これにより、不正なパラメータが送られてきても無視されます。 |
| 31 | `end` | `user_params` メソッドの終了です。 |
| 32 | `end` | `UsersController` クラスの終了です。 |
| 33 | `end` | `V1` モジュールの終了です。 |
| 34 | `end` | `Api` モジュールの終了です。 |

## まとめ

- **`show` アクション**：ログイン中のユーザー情報（ID、名前、メール、体重、性別、体質）をJSON形式で返します。未ログインの場合は401エラーを返します。
- **`update` アクション**：ログイン中のユーザーのプロフィールを更新します。`&.`（ぼっち演算子）を使ってnilチェックを安全に行います。
- **`user_params` メソッド**：Strong Parametersを使って、更新を許可するパラメータのみを受け付けます。これはセキュリティ上の重要な仕組みです。
- **モジュール構造**：`module Api > module V1 > class UsersController` という入れ子構造で、APIのバージョン管理を実現しています。
- **HTTPステータスコード**：成功時は `:ok`（200）、認証エラーは `:unauthorized`（401）、バリデーションエラーは `:unprocessable_content`（422）を返しています。
