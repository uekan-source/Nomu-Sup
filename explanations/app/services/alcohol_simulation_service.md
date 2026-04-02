# alcohol_simulation_service.rb の解説

## このファイルの役割

`AlcoholSimulationService`（アルコールシミュレーションサービス）クラスを定義するファイルです。ユーザーが入力した体重・性別・体質・飲んだお酒の情報をもとに、「体内の純アルコール量（g）」と「アルコールが抜けるまでの時間」を計算し、飲酒に関するアドバイスを生成します。このようなビジネスロジック（計算処理）をコントローラやモデルから切り出して専用クラスにまとめたのがServiceオブジェクトパターンです。

## コード全体

```ruby
class AlcoholSimulationService
  def initialize(weight:, gender:, constitution:, drinks:)
    # 体重が未入力または0の場合は、計算エラー（ゼロ除算）を防ぐため平均的な60kgをフォールバックに設定
    @weight = weight.to_f.positive? ? weight.to_f : 60.0
    @gender = gender
    @constitution = constitution
    @drinks = drinks.is_a?(Array) ? drinks : []
  end

  def execute
    pure_alcohol = calculate_pure_alcohol
    metabolism_rate = calculate_metabolism_rate

    # 分解時間を計算（ゼロ除算回避）
    time_in_hours = metabolism_rate.positive? ? pure_alcohol / metabolism_rate : 0

    {
      pure_alcohol_g: pure_alcohol.round(1),
      time_in_hours: time_in_hours.round(2),
      formatted_time: format_time(time_in_hours),
      advice: generate_advice(pure_alcohol, time_in_hours)
    }
  end

  private

  def calculate_pure_alcohol
    @drinks.sum do |drink|
      volume = drink[:volume].to_f
      abv = drink[:abv].to_f
      # 純アルコール量(g) = 容量(ml) × (アルコール度数(%) / 100) × 0.8(比重)
      volume * (abv / 100.0) * 0.8
    end
  end

  def calculate_metabolism_rate
    base_rate = @gender == 'female' ? 0.08 : 0.1
    rate = @weight * base_rate
    rate *= 0.8 if @constitution == 'weak'
    rate
  end

  def format_time(hours)
    return '0分' if hours <= 0

    h = hours.floor
    m = ((hours - h) * 60).round

    if h.positive? && m.positive?
      "約#{h}時間#{m}分"
    elsif h.positive?
      "約#{h}時間"
    else
      "約#{m}分"
    end
  end

  def generate_advice(alcohol_g, hours)
    [
      quantity_advice(alcohol_g),
      time_advice(hours)
    ].compact.join("\n\n")
  end

  def quantity_advice(alcohol_g)
    if alcohol_g >= 40
      [
        "⚠️ 【多量飲酒のサイン】\n",
        '純アルコール量が40gを超えています。厚生労働省が推奨する「節度ある適度な飲酒」の基準を',
        '大きく上回っており、肝機能への負担が懸念されます。脱水を防ぐため、同量以上の水分（和らぎ水）を必ず摂取してください。'
      ].join
    elsif alcohol_g >= 20
      [
        "💡 【適量オーバーの目安】\n",
        '純アルコール量が20gを超えています。これ以上のペースで飲むと、翌朝にアルコールが残りやすくなります。',
        'チェイサーを挟みながらゆっくり楽しみましょう。'
      ].join
    end
  end

  def time_advice(hours)
    if hours.positive?
      [
        "🚗 【重要：運転について】\n",
        "アルコールが完全に抜けるまで【#{format_time(hours)}】ほどかかる見込みです。",
        '※この時間はあくまで計算上の目安であり、当日の体調や空腹状態によってさらに長引く可能性があります。',
        '完全に抜けるまでは、車の運転や危険な作業は絶対にお控えください。'
      ].join
    else
      '✅ 選択された量からはアルコールの影響は少ないと推測されますが、体調に異変を感じた場合は無理をしないでください。'
    end
  end
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `class AlcoholSimulationService` | `AlcoholSimulationService`というクラスを定義しています。`ApplicationRecord`を継承していないため、データベースとは直接関係しない「計算専用」のクラスです。 |
| 2 | `def initialize(weight:, gender:, constitution:, drinks:)` | `initialize`（イニシャライズ）はインスタンス作成時に自動で呼ばれる特別なメソッドです。`weight:`のようにコロンをつけた引数は「キーワード引数」といい、`AlcoholSimulationService.new(weight: 60, gender: 'male', ...)`のように名前を指定して渡せます。 |
| 3 | `# 体重が未入力または0の場合は...` | コメント行です。次の行の処理の意図を説明しています。 |
| 4 | `@weight = weight.to_f.positive? ? weight.to_f : 60.0` | `to_f`は数値を浮動小数点数（小数）に変換するメソッドです。`positive?`は値が0より大きければ`true`を返します。`?`から始まる三項演算子（`条件 ? 真の値 : 偽の値`）で、体重が正の数なら入力値を、そうでなければ`60.0`（デフォルト値）を`@weight`に設定します。`@weight`のように`@`で始まる変数は「インスタンス変数」といい、クラス内のどのメソッドからでも使えます。 |
| 5 | `@gender = gender` | 引数`gender`（性別）をインスタンス変数`@gender`に保存します。 |
| 6 | `@constitution = constitution` | 引数`constitution`（体質）をインスタンス変数`@constitution`に保存します。 |
| 7 | `@drinks = drinks.is_a?(Array) ? drinks : []` | `is_a?(Array)`は「Arrayクラスのインスタンスかどうか」を確認するメソッドです。`drinks`が配列なら`drinks`をそのまま、配列でなければ空の配列`[]`を設定します。予期しない入力への防御処理です。 |
| 8 | `end` | `initialize`メソッドの終了です。 |
| 10 | `def execute` | このサービスの「メインの実行メソッド」です。外部からはこのメソッドを呼び出して計算結果を受け取ります。 |
| 11 | `pure_alcohol = calculate_pure_alcohol` | 後で定義する`calculate_pure_alcohol`メソッドを呼び出して、純アルコール量（g）を計算します。 |
| 12 | `metabolism_rate = calculate_metabolism_rate` | 後で定義する`calculate_metabolism_rate`メソッドを呼び出して、1時間あたりのアルコール代謝量を計算します。 |
| 14 | `# 分解時間を計算（ゼロ除算回避）` | コメント行です。次の行でゼロ除算（0で割る）エラーを防いでいることを説明しています。 |
| 15 | `time_in_hours = metabolism_rate.positive? ? pure_alcohol / metabolism_rate : 0` | 代謝量が0より大きければ「純アルコール量 ÷ 代謝量 = 分解時間（時間）」を計算します。代謝量が0の場合はゼロ除算エラーになるため、0を返します。 |
| 17〜22 | `{ pure_alcohol_g: ..., ... }` | ハッシュ（Rubyで`{キー: 値}`と書くデータ構造）を返しています。計算結果を4つの項目にまとめて返却します：純アルコール量・分解時間（小数）・フォーマット済み時間・アドバイス文章。 |
| 19 | `pure_alcohol.round(1)` | `round(1)`は小数点第1位で四捨五入するメソッドです。 |
| 20 | `time_in_hours.round(2)` | 小数点第2位で四捨五入します。 |
| 24 | `private` | この行以降に定義するメソッドはクラス外から直接呼び出せない「プライベートメソッド」になります。内部の実装詳細を隠すための設計で、外からは`execute`だけを使ってもらう意図です。 |
| 26〜33 | `def calculate_pure_alcohol ... end` | 純アルコール量を計算するメソッドです。`@drinks`配列の各飲み物について「容量(ml) × (アルコール度数/100) × 0.8」を計算し、`sum`で合計します。0.8はアルコールの比重（密度）です。 |
| 35〜40 | `def calculate_metabolism_rate ... end` | 1時間あたりのアルコール代謝量（g/時）を計算するメソッドです。女性は`0.08`、男性は`0.1`を基本レートとして体重を掛けます。体質が`'weak'`（お酒が弱い）の場合は`0.8`倍に減らします。 |
| 42〜55 | `def format_time(hours) ... end` | 時間（小数）を「約2時間30分」のような読みやすい文字列に変換するメソッドです。`floor`は小数点以下を切り捨てるメソッドです。時間部分・分部分それぞれが0かどうかで表示形式を切り替えています。 |
| 57〜62 | `def generate_advice(alcohol_g, hours) ... end` | 飲酒量アドバイスと運転アドバイスの2つを生成してまとめるメソッドです。`compact`は配列から`nil`を取り除くメソッドで、アドバイスが生成されなかった（`nil`が返った）場合を除外します。`join("\n\n")`は配列の要素を改行2つでつなぎます。 |
| 64〜78 | `def quantity_advice(alcohol_g) ... end` | 飲酒量に応じたアドバイスを生成するメソッドです。40g以上なら「多量飲酒の警告」、20g以上なら「適量オーバーの注意」、20g未満なら`nil`（アドバイスなし）を返します。 |
| 80〜91 | `def time_advice(hours) ... end` | アルコール分解時間に基づく運転に関するアドバイスを生成するメソッドです。時間が0より大きければ分解完了までの時間を含む警告を、0以下ならアルコールの影響が少ない旨のメッセージを返します。 |
| 92 | `end` | `class AlcoholSimulationService`ブロックの終了です。 |

## まとめ

- **Serviceオブジェクトパターンとは**：コントローラやモデルが複雑になりすぎないよう、ビジネスロジック（計算・変換処理）を専用のサービスクラスに切り出す設計パターンです。このクラスがその典型例です。
- **`initialize`と`@`インスタンス変数**：`initialize`で受け取った値を`@変数名`に保存しておくことで、クラス内のすべてのメソッドからその値を使えるようになります。
- **`private`の使い方**：外部から使ってほしいメソッド（`execute`）だけを公開し、内部の計算メソッドは`private`で隠すことで、クラスの「使い方」をシンプルに保てます。
- **三項演算子（`? :`）**：`条件 ? 真の場合の値 : 偽の場合の値`という書き方です。if文を1行でコンパクトに書ける便利な記法です。
- **ゼロ除算の防止**：`metabolism_rate.positive?`で0での割り算を防いでいます。このようなエッジケース（例外的な入力値）への対処は、堅牢なプログラムを作る上でとても重要な考え方です。
