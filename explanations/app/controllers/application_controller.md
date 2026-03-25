# application_controller.rb の解説

## このファイルの役割

`ApplicationController` は、Railsアプリケーション内のすべてのコントローラが継承する「親コントローラ」です。ここに書いた処理は、全コントローラで共通して使えるようになります。このファイルでは、ログイン中のユーザーを特定する `current_user` メソッドと、Deviseで使うパラメータの許可設定を定義しています。

## コード全体

```ruby
class ApplicationController < ActionController::API
  include ActionController::MimeResponds

  before_action :configure_permitted_parameters, if: :devise_controller?

  def current_user
    auth_header = request.headers['Authorization']
    token = auth_header.split.last if auth_header.present?

    # トークンがない（未ログイン・ゲスト）場合は nil を返す
    return nil if token.blank? || token == 'null'

    begin
      # トークンがあれば解読してユーザーを探す
      payload = Warden::JWTAuth::TokenDecoder.new.call(token)
      User.find_by(id: payload['sub'])
    rescue StandardError
      # トークンの期限切れや不正な場合はゲスト（nil）として扱う
      nil
    end
  end

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
    devise_parameter_sanitizer.permit(:account_update, keys: [:name])
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `class ApplicationController < ActionController::API` | `ApplicationController` というクラスを定義します。`< ActionController::API` は「ActionController::APIを継承する」という意味で、Rails APIモード用の基本機能をすべて引き継ぎます。 |
| 2 | `include ActionController::MimeResponds` | `include` はモジュールの機能を取り込む命令です。`MimeResponds` を取り込むことで、`respond_to` などのレスポンス形式切り替えメソッドが使えるようになります。 |
| 3 | （空行） | 読みやすさのための空行です。 |
| 4 | `before_action :configure_permitted_parameters, if: :devise_controller?` | `before_action` とは、コントローラのアクションが実行される前に自動で呼び出されるフック（仕掛け）です。`if: :devise_controller?` という条件付きで、Devise（認証ライブラリ）のコントローラが動くときだけ `configure_permitted_parameters` メソッドを実行します。 |
| 5 | （空行） | 読みやすさのための空行です。 |
| 6 | `def current_user` | `current_user` メソッドの定義を開始します。このメソッドは「今リクエストしているユーザーは誰か？」を返します。 |
| 7 | `auth_header = request.headers['Authorization']` | HTTPリクエストのヘッダーから `Authorization` の値を取り出し、`auth_header` 変数に代入します。JWTトークンはここに `Bearer eyJ...` の形式で入っています。 |
| 8 | `token = auth_header.split.last if auth_header.present?` | `auth_header` が存在する場合のみ、文字列をスペースで分割（`split`）して最後の要素（`last`）を取り出します。`"Bearer eyJxxx"` → `"eyJxxx"` というイメージです。 |
| 9 | （空行） | 読みやすさのための空行です。 |
| 10 | `# トークンがない（未ログイン・ゲスト）場合は nil を返す` | コメントです。コードの実行には影響しません。 |
| 11 | `return nil if token.blank? \|\| token == 'null'` | `token` が空（`blank?`）または文字列 `'null'` の場合は、`nil`（何もない）を返してメソッドを終了します。未ログイン状態を表します。 |
| 12 | （空行） | 読みやすさのための空行です。 |
| 13 | `begin` | 例外処理（エラーが起きたときの対処）のブロックを開始します。`begin`～`rescue`～`end` の構文です。 |
| 14 | `# トークンがあれば解読してユーザーを探す` | コメントです。コードの実行には影響しません。 |
| 15 | `payload = Warden::JWTAuth::TokenDecoder.new.call(token)` | `Warden::JWTAuth::TokenDecoder` というクラスを使ってJWTトークンを解読し、その中身（ペイロード）を `payload` 変数に格納します。ペイロードにはユーザーIDなどの情報が含まれています。 |
| 16 | `User.find_by(id: payload['sub'])` | ペイロードの `'sub'`（subject）キーに入っているユーザーIDを使って、データベースからユーザーを検索します。見つかればUserオブジェクトを、見つからなければ `nil` を返します。 |
| 17 | `rescue StandardError` | `begin` ブロック内で何らかのエラー（`StandardError`）が発生した場合の処理です。トークンの期限切れや改ざんなどのエラーをここで捕まえます。 |
| 18 | `# トークンの期限切れや不正な場合はゲスト（nil）として扱う` | コメントです。コードの実行には影響しません。 |
| 19 | `nil` | エラーが発生した場合は `nil` を返します。つまり「ログインしていないゲスト」として扱います。 |
| 20 | `end` | `begin`～`rescue` ブロックの終了です。 |
| 21 | `end` | `current_user` メソッドの終了です。 |
| 22 | （空行） | 読みやすさのための空行です。 |
| 23 | `protected` | この行より下に定義するメソッドのアクセス修飾子を `protected` にします。`protected` メソッドは、クラス内部やサブクラスからは呼び出せますが、外部からは直接呼び出せません。 |
| 24 | （空行） | 読みやすさのための空行です。 |
| 25 | `def configure_permitted_parameters` | `configure_permitted_parameters` メソッドの定義を開始します。Deviseのパラメータ許可設定を行うメソッドです。 |
| 26 | `devise_parameter_sanitizer.permit(:sign_up, keys: [:name])` | ユーザー新規登録（`sign_up`）時に、`:name` パラメータの受け取りを許可します。Deviseはデフォルトでメールとパスワードしか受け付けないので、追加パラメータはここで明示的に許可が必要です。 |
| 27 | `devise_parameter_sanitizer.permit(:account_update, keys: [:name])` | アカウント更新（`account_update`）時にも `:name` パラメータを許可します。 |
| 28 | `end` | `configure_permitted_parameters` メソッドの終了です。 |
| 29 | `end` | `ApplicationController` クラスの終了です。 |

## まとめ

- **全コントローラの共通処理**を `ApplicationController` にまとめることで、コードの重複を防げます。
- **`before_action`** を使うと、特定の条件のときだけ前処理を自動実行できます。
- **JWTトークン認証**では、HTTPリクエストのヘッダーからトークンを取り出し、デコードしてユーザーを特定する流れを学べます。
- **`begin`～`rescue`** による例外処理で、トークンが不正・期限切れの場合も安全に `nil` を返しアプリがクラッシュしないようにしています。
- **Devise のパラメータ許可**（`devise_parameter_sanitizer.permit`）で、デフォルト以外のカラム（ここでは `name`）を安全に受け取る方法を学べます。

