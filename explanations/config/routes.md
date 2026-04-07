# routes.rb の解説

## このファイルの役割

`config/routes.rb` は、Railsアプリへのリクエスト（URLとHTTPメソッドの組み合わせ）をどのコントローラーのどのアクションに渡すかを定義する「交通整理」のファイルです。このアプリではすべてのAPIエンドポイントが `/api/v1/` というパスの下に整理されており、認証関連のルートはDevise（認証ライブラリ）を使って設定されています。

## コード全体

```ruby
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      get 'health_check', to: 'health_check#index'
      resources :symptoms, only: [:index]
      resources :diagnosis_logs, only: [:index, :show, :create, :destroy] do
        collection do
          post :calculate # 診断のみ（DB保存しない）
        end
      end
      # アルコール分解シミュレーション用
      resources :simulations, only: [] do
        collection do
          post :calculate
        end
      end
      get 'me', to: 'users#show'
      patch 'me', to: 'users#update'
      # DeviseのルートをAPIのパス配下に設定
      devise_for :users, skip: [:sessions, :registrations, :passwords],
        controllers: {
          sessions: 'api/v1/auth/sessions',
          registrations: 'api/v1/auth/registrations',
          omniauth_callbacks: 'api/v1/auth/omniauth_callbacks'
        }
      # パスを /api/v1/auth/signup などに明示的にマッピング
      devise_scope :api_v1_user do
        post 'auth/signup', to: 'auth/registrations#create'
        post 'auth/login', to: 'auth/sessions#create'
        delete 'auth/logout', to: 'auth/sessions#destroy'
        post 'auth/password', to: 'auth/passwords#create'
        put 'auth/password', to: 'auth/passwords#update'
      end
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `Rails.application.routes.draw do` | Railsのルーティング設定をここから書き始めます。このブロック内にすべてのURL定義を記述します。 |
| 2 | `namespace :api do` | URLに `/api` というプレフィックス（前置き）を付け、コントローラーも `Api::` というモジュール（フォルダ）配下のものを使うように設定します。 |
| 3 | `namespace :v1 do` | さらに `/v1` を追加します。これにより全エンドポイントのURLが `/api/v1/...` という形になります。APIのバージョン管理（将来 v2 を追加しやすくする）のためのよくある設計パターンです。 |
| 4 | `get 'health_check', to: 'health_check#index'` | `GET /api/v1/health_check` というリクエストを `HealthCheckController` の `index` アクションに転送します。サーバーが正常に動作しているか確認するためのエンドポイントです。 |
| 5 | `resources :symptoms, only: [:index]` | 症状リスト取得のルートを生成します。`resources` は `index`, `show`, `create`, `update`, `destroy` などのRESTfulなルートを一括生成しますが、`only:` で必要なものだけに絞っています。ここでは `GET /api/v1/symptoms` のみ有効です。 |
| 6 | `resources :diagnosis_logs, only: [:index, :show, :create, :destroy] do` | 診断ログについて4つのRESTfulルートを生成します。`do...end` ブロックでネストしたルートを追加できます。 |
| 7 | `collection do` | `collection` は「特定のIDを持たない、リソースのコレクション全体に対するアクション」を追加するための構文です。URLは `/api/v1/diagnosis_logs/calculate` のようになります。 |
| 8 | `post :calculate` | `POST /api/v1/diagnosis_logs/calculate` というルートを追加します。このエンドポイントは診断結果を計算するだけで、データベースには保存しません。 |
| 12〜16 | `resources :simulations, only: [] do ... end` | `only: []` は「デフォルトのRESTfulルートを1つも生成しない」という指定です。`collection` ブロックで `POST /api/v1/simulations/calculate` だけを追加しています。アルコール分解シミュレーション専用のエンドポイントです。 |
| 19 | `get 'me', to: 'users#show'` | `GET /api/v1/me` を `UsersController#show` に転送します。ログイン中のユーザー自身の情報を取得するためのエンドポイントです。 |
| 20 | `patch 'me', to: 'users#update'` | `PATCH /api/v1/me` を `UsersController#update` に転送します。`PATCH` はリソースの部分的な更新に使うHTTPメソッドです。 |
| 22〜26 | `devise_for :users, skip: [...], controllers: {...}` | 認証ライブラリDeviseのルートを設定します。`skip:` でデフォルトのセッション・登録・パスワードルートを無効化し、代わりに独自のコントローラーを使うように指定しています。 |
| 28 | `devise_scope :api_v1_user do` | Deviseのカスタムルートを定義するブロックです。`devise_scope` によって、`devise_for` の設定と紐付けながら独自URLを明示的に定義できます。 |
| 29 | `post 'auth/signup', to: 'auth/registrations#create'` | `POST /api/v1/auth/signup` でユーザー登録を処理します。 |
| 30 | `post 'auth/login', to: 'auth/sessions#create'` | `POST /api/v1/auth/login` でログインを処理します。 |
| 31 | `delete 'auth/logout', to: 'auth/sessions#destroy'` | `DELETE /api/v1/auth/logout` でログアウトを処理します。`DELETE` はリソースの削除に使うHTTPメソッドで、ここではセッションを削除（＝ログアウト）します。 |
| 32 | `post 'auth/password', to: 'auth/passwords#create'` | `POST /api/v1/auth/password` でパスワードリセットのメール送信を依頼します。 |
| 33 | `put 'auth/password', to: 'auth/passwords#update'` | `PUT /api/v1/auth/password` で新しいパスワードを設定します。`PUT` はリソース全体の置き換えに使うHTTPメソッドです。 |

## まとめ

- **`namespace`** を使うことで `/api/v1/` というURL構造とコントローラーのフォルダ構造を一致させることができます。
- **`resources`** は `only:` オプションで必要なRESTfulルートだけを生成でき、余計なエンドポイントを公開しないセキュリティ上の良い習慣です。
- **`collection`** は特定のIDを持たないリソース操作（一覧や計算処理など）に対してカスタムルートを追加するときに使います。
- **Devise** の `devise_for` と `devise_scope` を組み合わせることで、認証関連のルートをAPIのURL設計に合わせて柔軟にカスタマイズできます。
- **HTTPメソッド（GET/POST/PATCH/PUT/DELETE）** の使い分けはRESTの設計思想に基づいており、取得・作成・更新・削除の操作をURLだけでなくメソッドで区別することで、直感的なAPI設計が実現できます。
