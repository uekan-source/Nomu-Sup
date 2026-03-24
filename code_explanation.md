# NomuSup コード解説

## 概要

Nomu-Sup（ノムサプ）は、薬剤師監修の二日酔い対策Webアプリです。
「飲み会前」「飲酒中」「翌朝」の3つのタイミングに合わせて、薬学的根拠に基づいた対策セット（サプリ・市販薬・食品）を提案します。

- 本番URL: https://nomu-sup-frontend.vercel.app
- 本体リポジトリ: https://github.com/uekan-source/NomuSup

---

## 技術スタック

| レイヤー | 技術 |
|--------|------|
| フロントエンド | React 18 / TypeScript 5 / Tailwind CSS |
| バックエンド | Ruby 3.3 / Ruby on Rails 7.1 (API Mode) |
| データベース | PostgreSQL 16 |
| フロントエンドホスティング | Vercel |
| バックエンドホスティング | Render |
| 認証 | JWT + Google OAuth |
| CI/CD | GitHub Actions (RSpec / RuboCop) |

---

## ディレクトリ構成

app/controllers/api/v1/ にAPIエンドポイントが定義されています。

- auth/ : 認証（JWT・Google OAuth）
- diagnosis_logs_controller.rb : 診断履歴
- simulations_controller.rb : アルコールシミュレーション
- symptoms_controller.rb : 症状一覧
- users_controller.rb : ユーザー管理

app/services/ にビジネスロジックがあります。

- diagnosis_service.rb : 診断アルゴリズム
- alcohol_simulation_service.rb : アルコール分解計算

---

## モデル設計

User（ユーザー）が DiagnosisLog（診断履歴）を持ちます。
DiagnosisLogは DiagnosisLogSymptom（選択症状）と DiagnosisLogDrug（提案薬）を持ちます。

Drug（薬）は DrugSymptom で Symptom（症状）と多対多の関連を持ちます（スコア・カテゴリ付き）。
Drug は DrugIngredient で Ingredient（成分）とも関連しています。

- Drug: コンビニ・薬局で買える商品（ヘパリーゼ、五苓散、ポカリスエットなど）
- Symptom: 「頭痛」「吐き気」「胃が痛い」などの症状
- DrugSymptom: 薬×症状の効果スコアとカテゴリ（医薬品/コンビニ商品）の中間テーブル
- DiagnosisLog: ユーザーごとの診断履歴

---

## diagnosis_service.rb（診断アルゴリズム）

本アプリのコアロジックです。選択した症状IDとタイミングをもとに最適な対策アイテムを最大3つ提案します。

### 処理の流れ

1. 選択された症状を取得
2. タイミングに合った薬候補を絞り込む
3. 各薬にスコアを付ける（4段階のロジック）
4. 禁忌・リスク薬を除外
5. バランスを考慮して上位3つを選ぶ
6. 症状に合ったアドバイス文を生成

### スコアリングロジック

**① 症状マッチングスコア**
DrugSymptomテーブルを使い、選択症状に合致するほど高スコアを付けます。

**② 禁忌・リスク回避（最重要）**

薬剤師ならではの安全チェックで、条件に該当する薬はスコアを大幅マイナスします。

| 条件 | 除外される薬 |
|------|------------|
| 胃・吐き気・食欲不振がある | バファリン（胃負担）、ウィルキンソン（炭酸刺激） |
| 下痢・軟便がある | 牛乳（腸に負担） |
| 頭痛がない | タイレノール、バファリン |

**③ 汎用おすすめ度の底上げ**
ポカリスエット・カゴメなど定番商品にボーナスを加算します。

**④ 同点時の微調整**
スコアが同じ場合、カバーできる症状数が多い薬を優先します。

### 重複排除ロジック

ヘパリーゼGX・ヘパリーゼW・ウコンの力など肝臓サポート系は1つだけ表示します。

### BEST MATCH調整

ラムネ・バナナなどがBEST MATCH（1位）になると視覚的インパクトが弱いため、2位・3位に薬やドリンク剤があれば自動的に順番を入れ替えます。

### アドバイス文生成（generate_summary）

症状の組み合わせに応じて薬剤師監修のアドバイスを生成します。

- 頭痛・渇き → 「アルコールによる脱水が起きているサインです。こまめな水分補給を強くおすすめします。」
- 胃・吐き気 → 「胃腸が深刻なダメージを受けています。消化の良い温かいものを摂り、油物は避けてください。」

---

## alcohol_simulation_service.rb（アルコール分解シミュレーター）

体重・性別・体質・飲んだお酒の量から、体内アルコール濃度が0になるまでの目安時間を計算します。ウィドマーク公式（Widmark formula）を参考にしており、性別・体質による分布定数の違いも考慮しています。

---

## 認証機能

- メールアドレス＋パスワード認証（パスワードリセット機能あり）
- Google OAuth（ワンタップログイン）

JWTトークンをフロントエンドに返し、以降のリクエストはAuthorizationヘッダーで認証します。

---

## CI/CD

GitHub Actionsで以下を自動化しています。

- RSpec: Pushのたびに自動テスト実行
- RuboCop: Rubyのコードスタイルチェック

---

## インフラ構成

- Vercel（フロントエンド: React）
- Render（バックエンド: Rails API）
- PostgreSQL（データベース）

DockerfileとDocker Composeも整備されており、ローカル開発環境をコンテナで再現できます。

---

## まとめ

| 特徴 | 実装のポイント |
|------|-------------|
| 薬剤師ロジック | DiagnosisServiceの禁忌チェック・スコアリング |
| UI/UXへの配慮 | 酔った状態でも操作できるイラスト選択UI |
| 安全性 | 胃腸・頭痛状態に応じた禁忌薬の自動除外 |
| パーソナライズ | 体重・性別・体質データの保存と活用 |
| 購買導線 | Amazonリンク・Google Maps連携 |
