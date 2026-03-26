# omniauth_callbacks_controller.rb の解説

## このファイルの役割

このファイルは、Google・GitHub・TwitterなどのSNSアカウントを使ったログイン（OAuth認証）のコールバック処理を担うコントローラーです。Deviseの `OmniauthCallbacksController` を継承し、外部サービスからの認証結果を受け取り、ユーザー情報をデータベースと照合した後、フロントエンド（Reactアプリ）にJWTトークン付きでリダイレクトします。

## コード全体

```ruby
module Api
  module V1
    module Auth
      class OmniauthCallbacksController < Devise::OmniauthCallbacksController
        # Googleからのコールバックを受け取るアクション
        def google_oauth2
          handle_auth
        end

        def github
          handle_auth
        end

        def twitter
          handle_auth
        end

        private

        def handle_auth
          # Googleから送られてきたユーザー情報
          auth = request.env['omniauth.auth']

          user = User.from_omniauth(auth)

          frontend_url = ENV['FRONTEND_URL'].presence || 'http://localhost:5173'

          if user.persisted?
            token, _payload = Warden::JWTAuth::UserEncoder.new.call(user, :user, nil)
            # React側の特定のURLへ、トークンをくっつけてリダイレクト
            redirect_to "#{frontend_url}/oauth/callback?token=#{token}", allow_other_host: true
          else
            # 失敗した場合：Reactのログイン画面にエラーパラメータをつけてリダイレクト
            redirect_to "#{frontend_url}/login?error=oauth_failed", allow_other_host: true
          end
        end
      end
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `module Api` | `Api` という名前空間（モジュール）の開始です。 |
| 2 | `module V1` | APIバージョン1を表す名前空間の開始です。 |
| 3 | `module Auth` | 認証関連のコードをまとめる名前空間の開始です。 |
| 4 | `class OmniauthCallbacksController < Devise::OmniauthCallbacksController` | `OmniauthCallbacksController` クラスを定義し、DeviseのOAuth用コントローラーを継承しています。OAuthとは「外部サービスのアカウントを使って別のサービスにログインする仕組み」です。 |
| 5 | `# Googleからのコールバックを受け取るアクション` | コメントです。コールバックとは「外部サービスが処理完了後に呼び出すURL」のことです。 |
| 6 | `def google_oauth2` | Google認証のコールバックを受け取るアクションです。Deviseが `/auth/google_oauth2/callback` へのリクエストを自動でこのメソッドに転送します。 |
| 7 | `handle_auth` | 共通の認証処理メソッド `handle_auth` を呼び出します。Google・GitHub・Twitterで同じ処理を使いまわしています。 |
| 8 | `end` | `google_oauth2` アクションの終了です。 |
| 9 | （空行） | 読みやすさのための空行です。 |
| 10 | `def github` | GitHub認証のコールバックを受け取るアクションです。 |
| 11 | `handle_auth` | Google認証と同じ `handle_auth` を呼び出します。 |
| 12 | `end` | `github` アクションの終了です。 |
| 13 | （空行） | 読みやすさのための空行です。 |
| 14 | `def twitter` | Twitter認証のコールバックを受け取るアクションです。 |
| 15 | `handle_auth` | 同じく `handle_auth` を呼び出します。 |
| 16 | `end` | `twitter` アクションの終了です。 |
| 17 | （空行） | 読みやすさのための空行です。 |
| 18 | `private` | これ以降に定義するメソッドはクラスの外から呼び出せない「プライベートメソッド」になります。内部実装の詳細を隠すための仕組みです。 |
| 19 | （空行） | 読みやすさのための空行です。 |
| 20 | `def handle_auth` | 各SNS認証で共通の処理をまとめたプライベートメソッドです。 |
| 21 | `# Googleから送られてきたユーザー情報` | コメントです。実際はGoogle・GitHub・Twitterなど、どのサービスからの情報でも対応します。 |
| 22 | `auth = request.env['omniauth.auth']` | OmniAuthミドルウェアが自動的に `request.env['omniauth.auth']` にセットした認証情報（ユーザーID・メール・名前など）を取得します。`request.env` はリクエストの環境変数を格納したハッシュです。 |
| 23 | （空行） | 読みやすさのための空行です。 |
| 24 | `user = User.from_omniauth(auth)` | `auth` の情報を使って、既存ユーザーを探すか新しいユーザーを作成します。`User.from_omniauth` はUserモデルに定義されたクラスメソッドです。 |
| 25 | （空行） | 読みやすさのための空行です。 |
| 26 | `frontend_url = ENV['FRONTEND_URL'].presence \|\| 'http://localhost:5173'` | フロントエンドのURLを環境変数から取得します。`ENV['FRONTEND_URL']` が設定されていない場合は開発用のURL `http://localhost:5173` を使います。`presence` とは「空文字でなければ値を返し、空なら `nil` を返す」メソッドです。 |
| 27 | （空行） | 読みやすさのための空行です。 |
| 28 | `if user.persisted?` | ユーザーがデータベースに保存（登録）されているか確認します。`persisted?` とは「このオブジェクトがデータベースに存在する場合に `true`」を返すActiveRecordのメソッドです。 |
| 29 | `token, _payload = Warden::JWTAuth::UserEncoder.new.call(user, :user, nil)` | JWTトークンを生成します。`Warden::JWTAuth::UserEncoder` はDevise-JWTが提供するクラスで、ユーザー情報を元に署名付きトークンを作成します。`_payload` の `_` はこの値を使わないことを示す慣習です。 |
| 30 | `# React側の特定のURLへ、トークンをくっつけてリダイレクト` | コメントです。 |
| 31 | `redirect_to "#{frontend_url}/oauth/callback?token=#{token}", allow_other_host: true` | フロントエンドのコールバックURLに、JWTトークンをクエリパラメーターとして付けてリダイレクトします。`allow_other_host: true` は外部ドメインへのリダイレクトを明示的に許可するオプションです。 |
| 32 | `else` | ユーザーの保存に失敗した場合の処理です。 |
| 33 | `# 失敗した場合：Reactのログイン画面にエラーパラメータをつけてリダイレクト` | コメントです。 |
| 34 | `redirect_to "#{frontend_url}/login?error=oauth_failed", allow_other_host: true` | OAuth認証に失敗した場合、ログイン画面に `error=oauth_failed` というパラメーターを付けてリダイレクトします。フロントエンドはこのパラメーターを見てエラーメッセージを表示できます。 |
| 35 | `end` | `if` ブロックの終了です。 |
| 36 | `end` | `handle_auth` メソッドの終了です。 |
| 37〜40 | `end` × 4 | それぞれ `OmniauthCallbacksController` クラス、`Auth` モジュール、`V1` モジュール、`Api` モジュールの終了です。 |

## まとめ

- **OAuthとコールバック**: 外部サービス（Google/GitHub/Twitter）でのログイン後、そのサービスが指定のURLを呼び出します。このコントローラーがそのURL（コールバック）を受け取って処理します。
- **DRY原則（繰り返しを避ける）**: `google_oauth2`・`github`・`twitter` の各アクションが全て `handle_auth` を呼ぶだけにすることで、共通処理をひとつのメソッドにまとめています。
- **JWTトークンの発行**: ログイン成功後、`Warden::JWTAuth::UserEncoder` でJWTトークンを生成し、フロントエンドに渡します。フロントエンドはこのトークンを保存して以降のAPIリクエストに使用します。
- **環境変数の活用**: `ENV['FRONTEND_URL']` で本番環境と開発環境のURLを切り替えています。環境変数はコードに直接URLを書かないための安全な手法です。
- **`private` キーワード**: `handle_auth` はコントローラーの外部（ルーティングなど）から直接呼ばれることを意図していないため、`private` で隠蔽しています。設計の意図を明確にする良い習慣です。
