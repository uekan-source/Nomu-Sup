# Nomu-Sup（ノムサプ）｜ 薬剤師監修・二日酔い対策ソムリエ

![Ruby](https://img.shields.io/badge/Ruby-3.3-CC342D?style=flat&logo=ruby&logoColor=white)
![Rails](https://img.shields.io/badge/Rails-7.1-CC0000?style=flat&logo=rubyonrails&logoColor=white)
![React](https://img.shields.io/badge/React-18-61DAFB?style=flat&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?style=flat&logo=typescript&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat&logo=postgresql&logoColor=white)

**サービスURL**: https://www.nomu-sup.com/

> 「酔っ払っていても、辛い朝でも、**数タップで専門家の正解に辿り着けるツール**」

![Nomu-Sup トップ画面](./docs/images/top.png)

---

## 目次

- [サービス概要](#サービス概要)
- [このサービスへの思い](#このサービスへの思い)
- [ターゲットユーザー](#ターゲットユーザー)
- [主な機能](#主な機能)
- [使用技術](#使用技術)
- [技術選定理由](#技術選定理由)
- [インフラ構成図](#インフラ構成図)
- [ER図](#er図)
- [画面遷移図](#画面遷移図)
- [こだわった実装](#こだわった実装)
- [ゲストログイン情報](#ゲストログイン情報)

---

## サービス概要

**「飲み会前」「飲酒中」「翌朝」** の3つのタイミングに合わせて、薬学的根拠に基づいた「今、コンビニや薬局で買うべき対策セット」を提案するWebアプリです。

ユーザーの状態を選択するだけで、**薬剤師の思考プロセスを再現したアルゴリズム**が、最適なサプリメント・市販薬・食品の組み合わせをレコメンドします。

> ※本アプリは「診断・治療」を目的としたものではなく、あくまで「セルフメディケーションを支援する情報提供」という位置付けです。

---

## このサービスへの思い

薬剤師として働く中で、二日酔いに苦しむ人々が「脱水状態なのにカフェインを摂る」といった、**医学的に逆効果な対処法を選んでしまっている**場面を多く見てきました。

「科学的に正しいタイミングで、正しい成分を摂取すれば、翌日のパフォーマンスは守れる」という確信があります。

しかし、飲み会の最中や二日酔いの時に専門情報を調べるのは困難です。そこで、**「酔っ払っていても、辛い朝でも、数タップで専門家の正解に辿り着けるツール」** を提供し、人々の健康と翌日の生産性を守りたいと考え開発しました。

---

## ターゲットユーザー

**20〜30代の働く世代・学生**

付き合いでの飲酒機会が多く、翌日の仕事や学業への影響を最小限にしたいというニーズがあります。健康管理のために「コンビニで数百円の対策」を講じるハードルが低く、スマホでの情報検索が習慣化しているため、このターゲット層に刺さるサービスと考えました。

---

## 主な機能

### 1. 薬剤師ロジック搭載・3ステップ診断

![3ステップ診断フロー](./docs/images/diagnosis_flow.gif)

「タイミング（前・中・翌朝）」「症状」「体質」を選択するだけで、最適な対策を診断します。二日酔いで文字を読むのが辛い状態でも直感的に操作できるよう、全選択肢をオリジナルのイラストで視覚化しています。

| 飲み会直前（予防） | 飲み会後（代謝促進） | 翌朝（対症療法） |
|:---:|:---:|:---:|
| 胃粘膜保護・肝機能補助のセットを提案 | アルコール分解を助ける食品・水分を提案 | 症状に合わせた鎮痛剤・漢方薬を提案 |

---

### 2. あなたへの処方箋（対策レコメンド）

![処方箋レコメンド画面](./docs/images/prescription.gif)

- **具体的な商品名の提示**: 「ヘパリーゼW」「五苓散」など、コンビニや薬局ですぐ買える商品名を提案
- **薬剤師のワンポイントアドバイス**: 「なぜこの成分が必要か」「飲むときの注意点」をわかりやすく解説
- **購買導線の確保**: 「Amazonで探す」ボタンや、Google Maps API連携による「近くのコンビニ/薬局を探す」ボタンを実装

---

### 3. アルコール分解シミュレーター

![アルコール分解シミュレーター](./docs/images/simulator.gif)

体重・性別・体質を入力し、飲んだお酒をタップしてカートに追加するだけで、**「体からアルコールが完全に抜けるまでの目安時間」** を自動計算します。飲酒運転の防止や、翌日の予定に向けたセルフコントロールを支援します。

---

### 4. マイ薬箱（診断履歴の保存機能）

![マイ薬箱画面](./docs/images/medicine_box.gif)

過去の診断結果とお薬の提案履歴をマイページに保存。ユーザーごとの体重や体質データも保存され、次回以降のシミュレーション入力が省力化されます。

---

### 5. Google / メールアドレス / GitHub / X 認証

手軽にログインできるよう、OAuthを利用したGoogle・GitHub・Xログインと、通常のメールアドレスログインを実装しています。

---

## 使用技術

| カテゴリ | 技術 |
|:---|:---|
| フロントエンド | React 18 / TypeScript 5 / Tailwind CSS / Vite |
| バックエンド | Ruby 3.3 / Ruby on Rails 7.1（APIモード） |
| データベース | PostgreSQL 16 |
| インフラ | Vercel（フロントエンド）/ Render（バックエンド） |
| 認証 | Devise + devise-jwt（Warden::JWTAuth） |
| 外部API | Google Maps API |
| その他 | axios / ESLint / RuboCop |
| バージョン管理 | Git / GitHub |

---

## 技術選定理由

### フロントエンド（React × TypeScript × Tailwind CSS）

本サービスは「酔っ払っていても使える」という要件から、**インタラクティブなUIの応答速度が最重要**でした。ページ遷移のたびに全体をリロードするMPAではなく、コンポーネント単位で再レンダリングが走るSPA構成を選択することで、快適な操作感を実現しています。

- **React**: 診断フローのステップ管理や、選択状態の即時フィードバックなど、複雑なUI状態を`useState`/`useReducer`でシンプルに管理できるため採用しました。再利用可能なコンポーネント設計により、診断画面・レコメンド画面・シミュレーター画面で共通UIを効率的に流用できました。
- **TypeScript**: バックエンドから受け取るレコメンドデータや診断ロジックの入力値に型定義を設けることで、「症状の型」「対策アイテムの型」などを明確に管理し、実装ミスを未然に防ぎました。
- **Tailwind CSS**: 「酔っ払っていても使える」というコンセプトを体現するため、ボタンサイズやコントラストを細かく調整する必要がありました。ユーティリティファーストのTailwindは、デザインの微調整をHTMLから直接行えるため、スピーディなUI改善サイクルに適していると判断しました。

### バックエンド（Ruby on Rails 7.1 APIモード）

薬剤師ロジックのアルゴリズムは複雑な条件分岐を含みます。RailsはActive Recordによるモデルへのロジック集約が得意であり、**「薬剤師の判断基準をコードに落とし込む」作業との相性が良い**と判断しました。「設定より規約」の原則により定型作業を減らし、ドメインロジックの実装に集中できた点も採用理由のひとつです。フロントエンドとの明確な責務分離のため、APIモードを選択しました。

### データベース（PostgreSQL 16）

診断ロジックに必要な「タイミング」「症状」「体質」「対策アイテム」の関係性は複数テーブルにまたがる複雑なリレーションを持ちます。複雑なJOINや集計クエリへの堅牢さ、JSONBカラムによる柔軟なデータ格納の観点からPostgreSQLを採用しました。

### インフラ（Vercel × Render）

- **Vercel**: React/Viteとの親和性が高く、GitHubへのpushで自動デプロイが完結します。フロントエンドのホスティングとして、開発から本番まで最小の運用コストで管理できると判断しました。
- **Render**: 無料枠でPostgreSQLが使用でき、RailsアプリのデプロイドキュメントもVercelと同様に充実しているため採用しました。

---

## インフラ構成図

![インフラ構成図](./docs/images/infrastructure.png)

---

## ER図

![ER図](./docs/images/er_diagram.png)

---

## 画面遷移図

[Figmaで画面遷移図を確認する](https://www.figma.com/board/jgqi4ETQw4dOLl1JuJnck2/Nomu-Sup?node-id=0-1&t=XH75YybKbkBBjj3x-1)

---

## こだわった実装

### 1. JWT認証の設計（Rails APIモード × devise-jwt）

本サービスはフロントエンド（React）とバックエンド（Rails APIモード）を完全に分離したSPA構成です。セッションベースのCookie認証ではなく、**JWTによるステートレスな認証**を採用することで、Vercel（フロント）とRender（バックエンド）という異なるオリジン間での認証を実現しています。

#### バックエンド：トークン発行（SessionsController）

```ruby
# app/controllers/api/v1/auth/sessions_controller.rb
module Api
  module V1
    module Auth
      class SessionsController < Devise::SessionsController
        respond_to :json

        def create
          user = User.find_by(email: params[:user][:email])

          if user&.valid_password?(params[:user][:password])
            token, _payload = Warden::JWTAuth::UserEncoder.new.call(user, :user, nil)

            render json: {
              status: 'success',
              token: token,
              data: user
            }, status: :ok
          else
            render json: {
              status: 'error',
              message: 'メールアドレスまたはパスワードが間違っています。'
            }, status: :unauthorized
          end
        end

        def respond_to_on_destroy
          head :no_content
        end
      end
    end
  end
end
```

ログイン成功時に `Warden::JWTAuth::UserEncoder` でJWTを生成し、フロントエンドに返します。**Deviseの `valid_password?` メソッド**を使うことで、bcryptによるパスワードハッシュの照合を安全に行っています。

#### バックエンド：トークン検証（ApplicationController）

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  include ActionController::MimeResponds

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
end
```

`current_user` はトークンの有無に応じて `User` オブジェクトまたは `nil` を返します。**ログイン必須のエンドポイントとゲスト利用可能なエンドポイントを柔軟に制御**できる設計にしました。トークンの期限切れや改ざんは `rescue StandardError` でキャッチし、ゲストとして扱うことでアプリのクラッシュを防いでいます。

#### フロントエンド：Axiosインターセプターによる自動付与（client.ts）

```typescript
// frontend/src/api/client.ts
import axios from 'axios';

const client = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api/v1',
  headers: { 'Content-Type': 'application/json' },
});

// リクエストインターセプター：送信前に毎回実行される処理
client.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  config.headers.Accept = 'application/json';
  return config;
});

export default client;
```

**Axiosのインターセプター**を使うことで、すべてのAPIリクエストに対して `localStorage` のトークンを自動的に `Authorization: Bearer {token}` ヘッダーとして付与します。個々のAPI呼び出しでトークンを手動設定する必要がなくなり、**認証ロジックを一元管理**できました。

---

### 2. 薬剤師の「消去法ロジック」のアルゴリズム設計

本サービスの核心は、「胃が痛い人にはNSAIDsを勧めない」「お酒に弱い体質の場合は代謝を助ける成分を優先する」といった、**薬剤師が実際に行う消去法の判断プロセス**をアルゴリズムに落とし込んだ点です。

```ruby
# app/services/diagnosis_service.rb
class DiagnosisService
  def initialize(timing:, symptoms:, constitution:)
    @timing       = timing
    @symptoms     = symptoms
    @constitution = constitution
  end

  def call
    drugs = Drug.for_timing(@timing)
    drugs = filter_by_symptoms(drugs)
    drugs = filter_by_constitution(drugs)
    drugs.order(:priority)
  end

  private

  def filter_by_symptoms(drugs)
    @symptoms.reduce(drugs) do |filtered, symptom|
      DrugSymptom.apply_filter(filtered, symptom)
    end
  end

  def filter_by_constitution(drugs)
    return drugs unless @constitution == 'weak'
    drugs.where(suitable_for_weak: true)
  end
end
```

`DiagnosisService` をサービスオブジェクトとして独立させ、コントローラーからロジックを切り離しました。これにより**ファットコントローラーを防ぎ**、単体テストも書きやすい構成を実現しています。

---

### 3. 「酔っ払っていても使える」UIの設計

**① 選択肢の全イラスト化**：文字を読まなくてもイラストをタップするだけで診断が進むよう、全選択肢にオリジナルイラストを設定しています。

**② ステップ表示によるコンテキストの明確化**：1度に全情報を表示せず、ステップバイステップで1質問ずつ表示する設計を採用。プログレスバーで「今どこにいるか」を明示し、認知負荷を下げました。

**③ 最重要情報を最上部に配置**：レコメンド結果は「BEST MATCH」商品を最上部に配置し、薬剤師のアドバイスは補足情報として構成することで、迷わず最重要情報に辿り着ける導線を作りました。

---

## ゲストログイン情報

採用担当者の方がスムーズにサービスを体験いただけるよう、ゲストアカウントをご用意しています。

```
【ゲストユーザー情報】
Email    : guest@nomu-sup.com
Password : password
```

> ※ ゲストアカウントのデータは定期的にリセットされます。

---

## 競合との差別化

| サービス | 弱点 | Nomu-Supの優位性 |
|:---|:---|:---|
| 飲酒記録アプリ | 「記録」が主目的で、その場の「対策」機能が弱い | 今すぐ取るべき行動を具体的に提示 |
| 医療相談サイト | 回答までに時間がかかる | 数タップで即座に結果を表示 |
| WEB記事 | 個人の状況に合わせた情報提供ができない | 体質・症状による消去法ロジックで個別最適化 |

→ **薬剤師の判断基準をアルゴリズム化**することで、「今、自分が何をすべきか」を数秒で導き出す点に独自性があります。

---

## 今後の展望

- [ ] プッシュ通知機能（飲み会前のリマインダー）
- [ ] お気に入り商品の登録・管理
- [ ] アルコール分解シミュレーターの精度向上（食事状況の入力追加）
- [ ] LINE連携による手軽な診断起動
