# application_record.rb の解説

## このファイルの役割

このファイルは、アプリケーション内のすべてのモデル（データベースのテーブルに対応するクラス）が継承する「基底クラス（親クラス）」です。Railsのすべてのモデルは最終的にこのクラスを通じて `ActiveRecord::Base` の機能を利用します。`primary_abstract_class` の指定により、このクラス自体はデータベースのテーブルと直接紐づかない「抽象クラス」として機能します。

## コード全体

```ruby
class ApplicationRecord < ActiveRecord::Base
  primary_abstract_class
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `class ApplicationRecord < ActiveRecord::Base` | `ApplicationRecord` というクラスを定義し、`ActiveRecord::Base` を継承します。`ActiveRecord::Base` はRailsが提供するクラスで、データベースとのやり取り（検索・保存・削除など）を行うための機能が詰まっています。 |
| 2 | `primary_abstract_class` | このクラスを「主要な抽象クラス」として宣言します。「抽象クラス」とは、それ自体はインスタンス（実体）を作らず、他のクラスに継承されることを目的としたクラスです。これにより、`ApplicationRecord` に対応するデータベーステーブルは存在しないことを明示します。 |
| 3 | `end` | クラス定義の終了です。 |

## まとめ

- **すべてのモデルの親クラス**：このアプリケーションの `User`、`Drug`、`Symptom` などすべてのモデルは `ApplicationRecord` を継承しています。つまり、すべてのモデルは間接的に `ActiveRecord::Base` の機能を使えます。
- **抽象クラスとは**：`primary_abstract_class` を指定することで、`ApplicationRecord` 自体はデータベースのテーブルと紐づかない「抽象的な基盤」として機能します。
- **共通処理を書く場所**：将来的にすべてのモデルに共通のメソッドや設定を追加したい場合、このファイルに書けば全モデルに反映されます。例えば、全モデル共通のバリデーションやコールバックをここに定義できます。
- **Railsの慣習（Convention over Configuration）**：このファイルはRailsが自動生成するファイルで、Railsの「設定より規約」という思想に基づいています。開発者は特別な設定なしに、このクラスを継承するだけでActiveRecordの全機能を使えます。
- **シンプルさの重要性**：たった3行のファイルですが、アプリ全体のモデルの基盤として非常に重要な役割を果たしています。
