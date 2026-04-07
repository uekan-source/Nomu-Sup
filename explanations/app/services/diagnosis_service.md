# diagnosis_service.rb の解説

## このファイルの役割

`DiagnosisService` は、ユーザーが選択した症状と飲酒タイミングをもとに、最適な薬・商品を最大3つ選んで返すサービスクラスです。症状ごとにスコアを計算し、禁忌チェックやバランス調整を経て、おすすめ商品と健康アドバイス文を生成します。Railsアプリにおけるサービスオブジェクト（ビジネスロジックを専門に担うクラス）の典型的な例です。

## コード全体

```ruby
# rubocop:disable Metrics/ClassLength
class DiagnosisService
  def initialize(symptom_ids, timing)
    @symptom_ids = symptom_ids || []
    @timing = timing.to_i
  end

  # rubocop:disable Metrics/AbcSize, Metrics/MethodLength, Metrics/CyclomaticComplexity, Metrics/PerceivedComplexity
  def execute
    selected_symptoms = Symptom.where(id: @symptom_ids).index_by(&:id)

    # 選択されたタイミング、または「3(いつでも)」の薬を候補にする
    valid_drugs = Drug.where(timing: [@timing, 3]).includes(:drug_symptoms)

    has_stomach_trouble = selected_symptoms.values.any? do |s|
      s.name.include?('胃') || s.name.include?('吐き気') || s.name.include?('食欲不振')
    end
    has_diarrhea = selected_symptoms.values.any? { |s| s.name.include?('下痢') || s.name.include?('ゆるい') }
    # 頭痛フラグ
    has_headache = selected_symptoms.values.any? { |s| s.name.include?('頭痛') }

    scored_drugs = valid_drugs.map do |drug|
      score = 0
      matched_count = 0

      drug.drug_symptoms.each do |ds|
        symptom = selected_symptoms[ds.symptom_id]
        if symptom
          score += symptom.category == 1 ? 30 : 20
          matched_count += 1
        end
      end

      is_contraindicated = false
      is_contraindicated = true if has_stomach_trouble && drug.name.include?('バファリン')
      is_contraindicated = true if has_diarrhea && drug.name.include?('牛乳')
      is_contraindicated = true if has_stomach_trouble && drug.name.include?('ウィルキンソン')
      is_contraindicated = true if @timing.zero? && drug.name.include?('OS-1')
      is_contraindicated = true if !has_headache && (drug.name.include?('タイレノール') || drug.name.include?('バファリン'))

      score = -9999 if is_contraindicated

      if score.zero? && !is_contraindicated
        score += 5 if drug.timing == @timing
        universal_keywords = %w[ポカリスエット カゴメ]
        score += 3 if universal_keywords.any? { |kw| drug.name.include?(kw) }
      end

      match_ratio = 0.0
      match_ratio = matched_count.to_f / drug.drug_symptoms.size if drug.drug_symptoms.size.positive? && score.positive?
      final_score = score + match_ratio

      { drug: drug, score: final_score, random: rand, symptom_count: drug.drug_symptoms.size }
    end

    safe_drugs = scored_drugs.reject { |item| item[:score].negative? }

    sorted_drugs = safe_drugs.sort_by do |item|
      [
        -item[:score],
        -item[:symptom_count],
        item[:random]
      ]
    end

    suggested_drugs = select_realistic_drugs(sorted_drugs.pluck(:drug), @timing)
    summary = generate_summary(selected_symptoms.values)

    { drugs: suggested_drugs, summary: summary }
  end
  # rubocop:enable all

  private

  def select_realistic_drugs(sorted_drug_objects, _timing)
    return [] if sorted_drug_objects.empty?

    liver_support_group = ['ヘパリーゼGX', 'ヘパリーゼW', 'ウコンの力', 'ウコンの力 超MAX', 'ヘパリーゼHi']
    already_has_liver_support = false
    filtered_drugs = []

    sorted_drug_objects.each do |drug|
      if liver_support_group.include?(drug.name)
        next if already_has_liver_support
        already_has_liver_support = true
      end
      filtered_drugs << drug
    end

    selected = filtered_drugs.take(3)

    if selected.size == 3 && selected.map(&:category).uniq.size == 1
      dominant_category = selected.first.category
      alternative_drug = filtered_drugs.drop(3).find { |d| d.category != dominant_category }
      selected[2] = alternative_drug if alternative_drug
    end

    non_best_match_keywords = %w[牛乳 ラムネ バナナ 鉄分 キレートレモン]
    if selected.size > 1 && non_best_match_keywords.any? { |kw| selected.first.name.include?(kw) }
      swap_index = selected.find_index { |d| non_best_match_keywords.none? { |kw| d.name.include?(kw) } }
      selected[0], selected[swap_index] = selected[swap_index], selected[0] if swap_index&.positive?
    end

    selected
  end
  # rubocop:enable all

  def generate_summary(symptoms)
    names = symptoms.map(&:name)
    advices = []

    if names.any? { |n| n.include?('頭痛') || n.include?('渇く') || n.include?('水分') }
      advices << if @timing.zero?
        'すでに脱水気味のようです。この状態でお酒を飲むと血中アルコール濃度が急上昇しやすくなります。チェイサー(水)を普段より多めに飲むよう心がけてください。'
      else
        'アルコールによる脱水が起きているサインです。お酒と同じかそれ以上の水分（水や経口補水液）をこまめに摂ることを強くおすすめします。'
      end
    end

    advices << '肝臓の代謝を助ける成分を摂りつつ、こまめな水分補給と十分な休息を心がけてください。' if advices.empty?

    advices.take(3).join("\n\n")
  end
  # rubocop:enable all
end
```

## 行ごとの解説

| 行番号 | コード | 解説 |
|--------|--------|------|
| 1 | `# rubocop:disable Metrics/ClassLength` | コード品質チェックツール「RuboCop」に対して、このクラスの長さ制限のチェックを一時的にオフにする指示です。処理が多いため例外扱いにしています。 |
| 2 | `class DiagnosisService` | `DiagnosisService` というクラスの定義開始。サービスオブジェクトとは、モデルやコントローラーに書くには複雑すぎるビジネスロジックをまとめた専用クラスのことです。 |
| 3 | `def initialize(symptom_ids, timing)` | インスタンスを作るときに呼ばれる初期化メソッド。症状IDの配列 `symptom_ids` と飲酒タイミング `timing` を受け取ります。 |
| 4 | `@symptom_ids = symptom_ids || []` | 受け取った症状IDを `@symptom_ids` というインスタンス変数に保存します。`nil` が渡された場合は空の配列 `[]` を使います。`||` は「左が nil や false なら右を使う」という演算子です。 |
| 5 | `@timing = timing.to_i` | タイミングを整数（Integer）に変換して保存します。`.to_i` は文字列を整数に変換するメソッドです（例：`"1"` → `1`）。 |
| 6 | `end` | `initialize` メソッドの終了。 |
| 9 | `def execute` | このサービスの主要メソッド。「診断を実行して結果を返す」という役割を担います。 |
| 10 | `selected_symptoms = Symptom.where(id: @symptom_ids).index_by(&:id)` | データベースから対象症状を取得し、`{ 症状ID => 症状オブジェクト }` というハッシュ形式に変換します。`index_by` は配列をキーで引けるハッシュに変換するメソッドです。 |
| 13 | `valid_drugs = Drug.where(timing: [@timing, 3]).includes(:drug_symptoms)` | 指定タイミング、または「3（いつでも）」の薬を候補として取得します。`includes` は関連するデータ（drug_symptoms）をまとめて取得するためのN+1問題対策です。 |
| 15〜17 | `has_stomach_trouble = ...` | 選択された症状の中に「胃」「吐き気」「食欲不振」が含まれているかをフラグとして保存します。`any?` は「ひとつでも条件に合うものがあれば true を返す」メソッドです。 |
| 18 | `has_diarrhea = ...` | 「下痢」「ゆるい」という症状があるかのフラグ。 |
| 20 | `has_headache = ...` | 「頭痛」という症状があるかのフラグ。これは禁忌チェックで使います。 |
| 22〜68 | `scored_drugs = valid_drugs.map do |drug| ... end` | 候補薬ごとにスコアを計算し、ハッシュの配列として返します。`map` は配列の各要素を加工して新しい配列を作るメソッドです。 |
| 29〜35 | `drug.drug_symptoms.each do |ds| ...` | その薬に紐づく症状ごとにループ。ユーザーが選んだ症状と一致すれば、症状カテゴリに応じてスコアを加算します（カテゴリ1なら30点、それ以外なら20点）。 |
| 32 | `score += symptom.category == 1 ? 30 : 20` | 三項演算子（`条件 ? 真の値 : 偽の値`）を使っています。症状カテゴリが1なら30点、それ以外は20点を加算します。 |
| 40〜46 | 禁忌チェック | 胃の不調があるのにバファリンを推薦しない、頭痛がないのに頭痛薬を出さないなど、危険な組み合わせを弾くロジックです。 |
| 49 | `score = -9999 if is_contraindicated` | 禁忌に該当する薬はスコアを極端にマイナスにして確実に除外します。 |
| 54〜58 | `if score.zero? && !is_contraindicated` | スコアが0点（症状が1つも一致しなかった）かつ禁忌でない薬に対して、最低限のスコアを付与して「おすすめ0件」を防ぐロジックです。 |
| 63〜65 | `match_ratio` / `final_score` | 症状カバー率（マッチした症状数 ÷ その薬の全症状数）を計算し、スコアに少し足します。同点時の微調整として機能します。 |
| 67 | `{ drug: drug, score: final_score, random: rand, ... }` | 薬オブジェクト、スコア、ランダム値、症状数をハッシュにまとめて返します。`rand` は0〜1のランダムな小数を返す組み込みメソッドで、同点時のシャッフルに使います。 |
| 71 | `safe_drugs = scored_drugs.reject { ... }` | マイナススコア（禁忌薬）を完全に除外します。`reject` は条件に合う要素を取り除いたリストを返すメソッドです。 |
| 74〜80 | `sorted_drugs = safe_drugs.sort_by do |item| ...` | スコアの高い順→症状カバー数の多い順→ランダムの順に並び替えます。マイナス符号（`-`）は降順（大きい順）にするためのテクニックです。 |
| 83 | `suggested_drugs = select_realistic_drugs(...)` | スコア順リストをさらにバランス調整するプライベートメソッドを呼び出します。 |
| 84 | `summary = generate_summary(...)` | 症状に応じたアドバイス文を生成するメソッドを呼び出します。 |
| 86 | `{ drugs: suggested_drugs, summary: summary }` | 最終結果をハッシュとして返します。呼び出し元のコントローラーはこのデータをJSONとして返却します。 |
| 90 | `private` | これ以降に定義するメソッドはクラスの外から呼び出せない「内部専用メソッド」になります。 |
| 93 | `def select_realistic_drugs(sorted_drug_objects, _timing)` | 引数 `_timing` の先頭にアンダースコアがついているのは「使わない引数」を明示するRubyの慣習です。 |
| 94 | `return [] if sorted_drug_objects.empty?` | 候補薬が空の場合は空配列を即座に返して処理を終了します。ガード節（Guard Clause）と呼ばれるパターンです。 |
| 99 | `liver_support_group = [...]` | ヘパリーゼ・ウコン系など肝臓サポート商品をグループ化したリストです。 |
| 103〜111 | ループによるフィルタリング | 同じ系統（肝臓サポート系）の商品が重複しないよう、フラグを使って1商品だけ通過させるフィルタリングを行います。 |
| 114 | `selected = filtered_drugs.take(3)` | フィルタリング後のリストから先頭3件を取り出します。`take(n)` は配列の最初のn個を返すメソッドです。 |
| 119〜123 | バランス調整ロジック | 選ばれた3件が全て同じカテゴリ（全部薬、または全部コンビニ商品など）の場合、4位以降から別カテゴリを探して3番目と入れ替えます。 |
| 128〜132 | 1位の見栄え調整 | 牛乳・ラムネなどは1位（BEST MATCH）表示にインパクトがないため、より薬らしい商品があれば1位に入れ替えるロジックです。 |
| 131 | `selected[0], selected[swap_index] = selected[swap_index], selected[0]` | Rubyの多重代入を使った値の交換（スワップ）です。一時変数を使わずに2つの値を入れ替えられます。 |
| 140〜170 | `def generate_summary(symptoms)` | 選択された症状の名前をもとに、最大3件のアドバイス文を生成して返します。症状ごとに飲酒前（`@timing == 0`）と飲酒後で異なるメッセージを出し分けています。 |

## まとめ

- **サービスオブジェクトパターン**：コントローラーやモデルに書くには複雑すぎるロジックを専用クラスに切り出すことで、コードの見通しが良くなります。
- **スコアリングロジック**：症状とのマッチ度に応じてポイントを加算し、最終的に高スコア順に並べることで「最もおすすめな薬」を決定しています。
- **禁忌チェック**：危険な組み合わせ（胃が弱いのにバファリンを出すなど）をスコアに `-9999` を設定することで確実に除外しています。
- **ガード節**：メソッドの先頭で異常系（空の配列など）を早めに `return` することで、ネストが深くなるのを防いでいます。
- **`any?` / `reject` / `sort_by` などの Enumerable メソッド**：Rubyの配列操作メソッドを活用することで、繰り返し処理を簡潔に記述しています。
