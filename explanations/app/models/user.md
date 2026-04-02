# user.rb の解説

## このファイルの役割

`User`（ユーザー）モデルを定義するファイルです。ユーザー認証（ログイン・登録・パスワードリセット）に関する機能を`Devise`というgemを使って設定し、JWT（JSON Web Token）によるAPIトークン認証やGoogle・GitHub・TwitterなどのSNSログイン（OmniAuth）にも対応しています。アプリの「誰がログインしているか」を管理する中心的なモデルです。

## コード全体

```ruby
class User < ApplicationRecord
  include Devise::JWT::RevocationStrategies::JTIMatcher

  devise :database_authenticatable, :registerable,
         :recoverable, :validatable,
         :jwt_authenticatable, :omniauthable,
         jwt_revocation_strategy: self,
         omniauth_providers: %i[google_oauth2 github twitter]

  has_many :diagnosis_logs, dependent: :destroy

  def self.jwt_revocation_strategy
    self
  end

  # OmniAuthのデータからユーザーを探す、または作るメソッド
  def self.from_omniauth(auth)
    provider_email = auth.info.email || "#{auth.uid}-#{auth.provider}@example.com"
    where(email: provider_email).first_or_create do |user|
      user.provider = auth.provider
      user.uid = auth.uid
      user.password = Devise.friendly_token[0, 20]
      user.name = auth.info.name || 'ユーザー'
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `class User < ApplicationRecord` | `User`クラスを定義しています。`ApplicationRecord`を継承することで、データベース操作の機能が使えます。 |
| 2 | `include Devise::JWT::RevocationStrategies::JTIMatcher` | `include`とは、モジュール（機能のまとまり）をクラスに取り込む命令です。ここでは`JTIMatcher`という戦略を取り込んでいます。これはJWTトークンを「無効化（失効）」できる仕組みで、ログアウト時にトークンを使えなくするために使います。 |
| 3 | （空行） | 区切り行です。 |
| 4〜8 | `devise :database_authenticatable, ...` | `devise`メソッドで認証機能の設定をしています。各シンボル（`:`で始まる値）の意味は以下の通りです。`:database_authenticatable`はメールとパスワードでの認証、`:registerable`はユーザー登録機能、`:recoverable`はパスワードリセット機能、`:validatable`はメール・パスワードの入力チェック、`:jwt_authenticatable`はJWTトークンでのAPI認証、`:omniauthable`はSNSログイン対応です。 |
| 7 | `jwt_revocation_strategy: self` | JWTトークンの失効戦略として「このクラス自身（`self`）」を使うことを指定しています。2行目で取り込んだ`JTIMatcher`の機能がここで使われます。 |
| 8 | `omniauth_providers: %i[google_oauth2 github twitter]` | SNSログインのプロバイダー（提供元）を指定しています。`%i[...]`はシンボルの配列を作るRubyの記法で、Google・GitHub・Twitterの3つに対応しています。 |
| 9 | （空行） | 区切り行です。 |
| 10 | `has_many :diagnosis_logs, dependent: :destroy` | 1人のユーザーが複数の`DiagnosisLog`（診断ログ）を持てることを示します。ユーザーが削除されると、そのユーザーの診断ログもすべて削除されます。 |
| 11 | （空行） | 区切り行です。 |
| 12〜14 | `def self.jwt_revocation_strategy ... end` | JWTの失効戦略として「クラス自身（`self`）」を返すクラスメソッドです。`def self.メソッド名`と書くと「クラスメソッド」（インスタンスではなくクラスに対して呼び出すメソッド）になります。`JTIMatcher`が内部でこのメソッドを呼び出して使います。 |
| 15 | （空行） | 区切り行です。 |
| 16 | `# OmniAuthのデータからユーザーを探す、または作るメソッド` | コメント行です。次のメソッドが何をするか説明しています。 |
| 17 | `def self.from_omniauth(auth)` | SNSログイン時に呼ばれるクラスメソッドです。`auth`引数にはSNSから取得したユーザー情報（メールアドレス、名前など）が入っています。 |
| 18 | `provider_email = auth.info.email || "#{auth.uid}-#{auth.provider}@example.com"` | SNSから取得したメールアドレスを使います。メールアドレスが取得できない場合（`nil`の場合）は、`||`（または）の右側のダミーメールアドレスを使います。`#{}`はRubyの文字列への変数埋め込み記法です。 |
| 19 | `where(email: provider_email).first_or_create do |user|` | `where`はデータベースを検索するメソッドです。指定したメールアドレスのユーザーを探し、見つかればそのユーザーを返します。見つからなければ`first_or_create`がブロック（`do...end`）内の処理を実行して新しいユーザーを作ります。 |
| 20 | `user.provider = auth.provider` | SNSの種類（`"google_oauth2"`や`"github"`など）をユーザーに保存します。 |
| 21 | `user.uid = auth.uid` | SNS側でのユーザーID（一意な識別子）を保存します。 |
| 22 | `user.password = Devise.friendly_token[0, 20]` | SNSログインではパスワード入力がないため、Deviseの`friendly_token`メソッドでランダムな20文字のパスワードを自動生成して設定します。 |
| 23 | `user.name = auth.info.name || 'ユーザー'` | SNSから取得した名前を保存します。名前が取得できない場合はデフォルトで`'ユーザー'`という文字列を設定します。 |
| 24 | `end` | `first_or_create`のブロックの終了です。 |
| 25 | `end` | `from_omniauth`メソッドの終了です。 |
| 26 | `end` | `class User`ブロックの終了です。 |

## まとめ

- **Deviseとは**：Railsでユーザー認証を簡単に実装できるgemです。`devise`メソッドに複数のシンボルを渡すだけで、登録・ログイン・パスワードリセットなどの機能が使えるようになります。
- **JWTとは**：「JSON Web Token」の略で、ユーザー認証に使うトークン（証明書のようなもの）です。APIとフロントエンドが分離した構成でよく使われ、毎回データベースにアクセスせずに認証できます。
- **OmniAuthとは**：Google・GitHub・TwitterなどのSNSアカウントでログインできる仕組みです。ユーザーが自分でパスワードを管理しなくてよいため、UX（使いやすさ）が向上します。
- **`first_or_create`とは**：「DBに存在すれば取得し、なければ作成する」という便利なメソッドです。SNSログインで何度アクセスしても重複ユーザーが作られないようにするための重要な処理です。
- **`self.`メソッドとは**：クラスメソッドと呼ばれ、`User.from_omniauth(auth)`のようにクラスに対して直接呼び出せるメソッドです。インスタンス（個別のユーザーオブジェクト）に依存しない処理に使います。
