# database.yml の解説

## このファイルの役割

`config/database.yml` は、RailsアプリがデータベースへどのようにRailsアプリが接続するかを定義する設定ファイルです。このアプリではPostgreSQLを使用しており、開発（development）・テスト（test）・本番（production）の3つの環境ごとに接続先やデータベース名などを設定しています。YAMLという設定ファイル形式で書かれています。

## コード全体

```yaml
# PostgreSQL. Versions 9.3 and up are supported.
#
# Install the pg driver:
#   gem install pg
#
# Configure Using Gemfile
#   gem "pg"
#
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: db
  port: 5432
  username: postgres
  password: password

development:
  <<: *default
  database: app_development

test:
  <<: *default
  database: app_test

production:
  <<: *default
  url: <%= ENV['DATABASE_URL'] %>
  database: app_production
  username: app
  password: <%= ENV["APP_DATABASE_PASSWORD"] %>
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1〜13 | `# PostgreSQL...` | `#` から始まる行はコメントです。PostgreSQL用のgemのインストール方法が説明されています。コメントはプログラムの動作に影響しません。 |
| 15 | `default: &default` | `default` というセクションを定義し、`&default` という「アンカー（anchor）」を付けています。アンカーとは、後で同じ内容を再利用するための名前付けです。YAMLの便利な機能です。 |
| 16 | `adapter: postgresql` | 使用するデータベースの種類を指定します。RailsはMySQL・SQLiteなども使えますが、このアプリはPostgreSQLを使います。 |
| 17 | `encoding: unicode` | 文字コードをUnicode（UTF-8）に設定します。日本語を含む多言語テキストを正しく扱うために重要です。 |
| 18 | `pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>` | データベースへの同時接続数（コネクションプール）の上限を設定します。`<%= %>` はERB記法で、Rubyコードを埋め込める構文です。環境変数 `RAILS_MAX_THREADS` が設定されていなければデフォルトで5が使われます。 |
| 19 | `host: db` | 接続先のデータベースサーバーのホスト名です。`db` はDockerを使った開発環境でのサービス名で、Docker Composeで定義されたPostgreSQLコンテナを指しています。 |
| 20 | `port: 5432` | PostgreSQLのデフォルトポート番号です。ポートとは、同一サーバー上の複数のサービスを区別するための番号です。 |
| 21 | `username: postgres` | データベース接続に使うユーザー名です。 |
| 22 | `password: password` | 開発環境用のパスワードです。本番環境では環境変数を使うため、ここに書いても問題ありません。 |
| 24 | `development:` | 開発環境（ローカルで作業するとき）の設定セクションです。 |
| 25 | `<<: *default` | `*default` で先ほど定義したアンカーの内容をすべて引き継ぎます（マージキー）。`<<:` は「この設定をすべてコピーする」という意味です。これにより共通設定を繰り返し書かずに済みます。 |
| 26 | `database: app_development` | 開発環境で使うデータベース名です。`default` の設定に加えてこの項目だけが上書きされます。 |
| 28 | `test:` | テスト実行時（`rspec` などを動かすとき）の設定セクションです。 |
| 29 | `<<: *default` | 同じく共通設定を引き継ぎます。 |
| 30 | `database: app_test` | テスト用のデータベース名です。テスト実行ごとにデータが削除・再作成されるため、開発用とは別のデータベースを使います。 |
| 32 | `production:` | 本番環境（実際にユーザーが使う環境）の設定セクションです。 |
| 33 | `<<: *default` | 共通設定を引き継ぎます。 |
| 34 | `url: <%= ENV['DATABASE_URL'] %>` | 本番環境ではデータベース接続URLを環境変数から読み込みます。セキュリティのため、本番の接続情報はソースコードに直接書かず、サーバーの環境変数として設定するのが鉄則です。 |
| 35 | `database: app_production` | 本番環境のデータベース名です。 |
| 36 | `username: app` | 本番環境のデータベースユーザー名です。 |
| 37 | `password: <%= ENV["APP_DATABASE_PASSWORD"] %>` | 本番環境のパスワードは環境変数から読み込みます。パスワードをソースコードに書くことは重大なセキュリティリスクになるため、必ず環境変数を使います。 |

## まとめ

- **YAML形式**はインデント（字下げ）でデータ構造を表現します。インデントがずれると設定が正しく読まれないため注意が必要です。
- **YAMLのアンカー（`&`）とエイリアス（`*`）**を使うことで、共通設定（`default`）を各環境で使い回すことができ、設定の重複を避けられます。
- **環境変数（`ENV`）**を使うことで、パスワードや接続URLなどの機密情報をソースコードに直接書かずに済みます。これはセキュリティ上の重要な設計原則です。
- Railsには**development・test・production**という3つの標準環境があり、それぞれ独立したデータベースを持つことでテスト中の誤操作が本番データに影響しないように設計されています。
- **コネクションプール**はデータベース接続を使いまわす仕組みで、毎回接続を作り直す処理コストを下げ、高速化に貢献します。
