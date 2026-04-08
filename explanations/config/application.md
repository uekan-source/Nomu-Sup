# application.rb の解説

## このファイルの役割

`config/application.rb` は、Railsアプリケーション全体の設定を定義するファイルです。アプリが起動するときに読み込まれ、どのライブラリを使うか、どんな動作をするかなどの基本設定が書かれています。このアプリはAPI専用モードで動いており、セッション（ログイン状態の管理）やコード自動生成（ジェネレーター）の設定もここで行われています。

## コード全体

```ruby
require_relative "boot"

require "rails/all"

# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(*Rails.groups)

module App
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 7.1

    # Please, add to the `ignore` list any other `lib` subdirectories that do
    # not contain `.rb` files, or that should not be reloaded or eager loaded.
    # Common ones are `templates`, `generators`, or `middleware`, for example.
    config.autoload_lib(ignore: %w(assets tasks))

    # Configuration for the application, engines, and railties goes here.
    #
    # These settings can be overridden in specific environments using the files
    # in config/environments, which are processed later.
    #
    # config.time_zone = "Central Time (US & Canada)"
    # config.eager_load_paths << Rails.root.join("extras")

    # Only loads a smaller set of middleware suitable for API only apps.
    # Middleware like session, flash, cookies can be added back manually.
    # Skip views, helpers and assets when generating a new resource.
    config.api_only = true

    config.session_store :cookie_store, key: '_nomu_sup_session'
    config.middleware.use ActionDispatch::Cookies
    config.middleware.use ActionDispatch::Session::CookieStore, config.session_options
    config.generators do |g|
      g.orm :active_record, primary_key_type: :uuid
      g.test_framework :rspec,
          fixtures: false,
          view_specs: false,
          helper_specs: false,
          routing_specs: false
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `require_relative "boot"` | 同じフォルダにある `boot.rb` を読み込みます。`require_relative` は相対パスでファイルを指定する命令です。`boot.rb` にはBundlerの初期設定などが書かれています。 |
| 2 | （空行） | 読みやすくするための空白行です。 |
| 3 | `require "rails/all"` | Railsのすべての機能（ActiveRecord、ActionControllerなど）を一括で読み込みます。 |
| 4 | （空行） | 読みやすくするための空白行です。 |
| 5〜6 | `# Require the gems listed in Gemfile...` | コメント行です。次の行の説明をしています。 |
| 7 | `Bundler.require(*Rails.groups)` | `Gemfile` に書かれた gem（外部ライブラリ）を、現在の環境（development / test / productionなど）に合わせて自動で読み込みます。`*Rails.groups` で現在の環境グループが展開されます。 |
| 8 | （空行） | 読みやすくするための空白行です。 |
| 9 | `module App` | `App` というモジュール（名前空間）を定義します。モジュールは関連するクラスやメソッドをグループ化する仕組みです。 |
| 10 | `class Application < Rails::Application` | `App::Application` クラスを定義します。`<` は継承を意味し、`Rails::Application` の機能をすべて引き継いだ上でカスタマイズします。 |
| 11 | `# Initialize configuration defaults...` | コメント行です。次の設定の説明をしています。 |
| 12 | `config.load_defaults 7.1` | Rails 7.1 で導入されたデフォルト設定を有効にします。新しいバージョンの推奨設定をまとめて適用できます。 |
| 13 | （空行） | 読みやすくするための空白行です。 |
| 14〜16 | `# Please, add to the ignore list...` | コメント行です。次の設定の説明をしています。 |
| 17 | `config.autoload_lib(ignore: %w(assets tasks))` | `lib/` フォルダ内のRubyファイルを自動読み込みします。`assets` と `tasks` フォルダは除外します。`%w(assets tasks)` は `["assets", "tasks"]` という配列を作る省略記法です。 |
| 18 | （空行） | 読みやすくするための空白行です。 |
| 19〜25 | `# Configuration for the application...` | コメント行です。設定の書き方の例や補足が書かれています。 |
| 26 | （空行） | 読みやすくするための空白行です。 |
| 27〜29 | `# Only loads a smaller set of middleware...` | コメント行です。APIモードについての説明です。 |
| 30 | `config.api_only = true` | このアプリをAPI専用モードに設定します。HTMLビューやブラウザ向けの機能が省略され、軽量なAPIサーバーとして動作します。 |
| 31 | （空行） | 読みやすくするための空白行です。 |
| 32 | `config.session_store :cookie_store, key: '_nomu_sup_session'` | セッション（ログイン状態などの一時保存）の保存方法を「Cookieストア」に設定します。`key:` でCookieの名前を指定しています。 |
| 33 | `config.middleware.use ActionDispatch::Cookies` | Cookieを扱うためのミドルウェア（HTTPリクエストとレスポンスの間に入る処理）を追加します。API専用モードではデフォルトでCookieが無効なため、手動で追加します。 |
| 34 | `config.middleware.use ActionDispatch::Session::CookieStore, config.session_options` | セッションをCookieに保存するミドルウェアを追加します。`config.session_options` で32行目の設定を渡しています。 |
| 35 | `config.generators do |g|` | `rails generate` コマンドのカスタマイズを開始するブロックです。ジェネレーターとは、コマンド一つでファイルを自動生成する機能のことです。 |
| 36 | `g.orm :active_record, primary_key_type: :uuid` | データベースのORMとしてActiveRecordを使い、主キー（レコードを一意に識別するID）の型をUUID（ランダムな文字列ID）に設定します。 |
| 37 | `g.test_framework :rspec,` | テストフレームワークとしてRSpecを使うよう設定します。続く行でオプションを指定しています。 |
| 38 | `fixtures: false,` | フィクスチャ（テスト用の初期データファイル）を自動生成しないよう設定します。 |
| 39 | `view_specs: false,` | ビュー（画面）のテストファイルを自動生成しないよう設定します（APIモードにはビューがないため不要）。 |
| 40 | `helper_specs: false,` | ヘルパー（補助メソッド）のテストファイルを自動生成しないよう設定します。 |
| 41 | `routing_specs: false` | ルーティングのテストファイルを自動生成しないよう設定します。 |
| 42 | `end` | `config.generators` ブロックの終了です。 |
| 43 | `end` | `Application` クラス定義の終了です。 |
| 44 | `end` | `App` モジュール定義の終了です。 |

## まとめ

- `config/application.rb` はRailsアプリ全体の「設定書」で、起動時に必ず読み込まれる重要なファイルです。
- `config.api_only = true` により、このアプリはAPIサーバーとして動作し、ビュー関連の不要な機能を省いています。
- API専用モードでもCookieとセッションを使えるよう、ミドルウェアを手動で追加しています。
- 主キーにUUIDを使うことで、推測されにくい安全なIDを自動生成できます。
- `config.generators` の設定により、ファイル自動生成時に不要なテストファイルが作られないようにしています。
