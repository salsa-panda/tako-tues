# Components: タコ中

**Project**: タコ中 (Tako-chū / Tako-tues)
**Document Version**: 1.1
**Created**: 2026-04-29
**Updated**:
- 2026-05-08 v1.1: 要件 v1.8 反映。**FR-6.3 廃止に伴い C8 ChatGPTGptAsset コンポーネントを削除**（9 → 8 コンポーネント）。静的アセット表から `assets/prompts/tako-gpt-*.md` を削除、FR Coverage Matrix から FR-6.3 行削除
- 2026-05-08 v1.1（rework v2.0 復元）: ビジネス意図の深掘り rework に伴いバックアップから復元。コンポーネント構成は変化なし。
**Phase**: INCEPTION - Application Design

---

## 0. 設計方針サマリー

- **コンポーネント粒度**: execution-plan v1.3 の 9 Unit を 1:1 でコンポーネント化（Q1=A）
- **レイヤード構造**: 各コンポーネント内で Handler / UseCase / Repository / External の 4 層（Q9=A）
- **横断責務**: AWS Lambda Powertools（Logger / Tracer / Metrics）を全コンポーネントで標準利用（Q2=B）

---

## 1. Component Catalog（9 コンポーネント）

各コンポーネントは「責務 / 主要 FR / 主要 Story / Iteration 0 範囲 / 公開インターフェース」を持つ。

### C1. AuthAndTrial（認証・トライアル）

| 項目 | 内容 |
|------|------|
| **Unit 対応** | U1 |
| **責務** | Cognito Hosted UI による認証、トライアル期間管理、ユーザー ID 払い出し |
| **主要 FR** | FR-5 |
| **主要 Story** | US-T01 |
| **Iteration 0 範囲** | Cognito 認証経路のみ（トライアル日数チェックは固定値で OK） |
| **公開インターフェース** | <ul><li>REST: `GET /me`, `POST /signup-trial`（ApiMonolith 経由）</li><li>JWT 検証ミドルウェア（他コンポーネントが利用）</li></ul> |

### C2. IntakeLogging（摂取記録）

| 項目 | 内容 |
|------|------|
| **Unit 対応** | U2 |
| **責務** | 「TACO!（=作った）」記録の永続化、重複検知、週次サマリ提供 |
| **主要 FR** | FR-1.3 |
| **主要 Story** | US-T02 |
| **Iteration 0 範囲** | TACO ボタン → DynamoDB 1 行追加（重複検知は Iteration 1+） |
| **公開インターフェース** | <ul><li>REST: `POST /intakes`, `GET /intakes/weekly`</li><li>EventBridge 発行: `IntakeRecorded`</li></ul> |

### C3. OrderEngineAndDeliveryMock（強制発注 + 配送モック）

| 項目 | 内容 |
|------|------|
| **Unit 対応** | U3 |
| **責務** | 金曜 21:00 発注決定（逆比例ロジック + メニュープール選定 + Type A/B 切替）、月曜 21:00 配送遷移、キャンセル試行 → 発注増ロジック |
| **主要 FR** | FR-2.1, FR-2.2, FR-2.4 |
| **主要 Story** | US-T03, US-T04, US-T12, US-M01, US-M02 |
| **Iteration 0 範囲** | 逆比例ロジック最小実装 + メニュー 2〜3 種固定 + キャンセル UI なし。Type B バリエーションウィークは Iteration 後半 |
| **公開インターフェース** | <ul><li>EventBridge 受信: スケジューラトリガー（金 21:00 / 月 21:00）</li><li>EventBridge 発行: `OrderDecided`, `Delivered`</li><li>REST: `GET /orders/current`, `POST /orders/cancel-attempt`</li></ul> |

### C4. RecipeLibrary（静的レシピライブラリ）

| 項目 | 内容 |
|------|------|
| **Unit 対応** | U4 |
| **責務** | リポジトリ同梱の静的 JSON（`assets/recipes/*.json`）からキット種別 1:1 でレシピを返す。未登録キーは `_default.json` フォールバック |
| **主要 FR** | FR-2.3 |
| **主要 Story** | US-T13, US-M04 |
| **Iteration 0 範囲** | レシピ JSON 2〜3 種 + `_default.json`。Bedrock 動的生成は不採用（v1.7） |
| **公開インターフェース** | <ul><li>REST: `GET /recipes/{kit_type}`</li><li>EventBridge 受信: `Delivered`（PWA への Push 通知トリガー）</li></ul> |

### C5. StimulusEngine（刺激エンジン）

| 項目 | 内容 |
|------|------|
| **Unit 対応** | U5 |
| **責務** | 平日昼夕の誘惑 Push 配信、24h ウィンドウ内 Push 無視カウンタ集計 + Level A/B/C エスカレーション、ダッシュボード煽り文生成。`StimulusGenerator` 抽象（Bedrock / Static の 2 実装切替） |
| **主要 FR** | FR-6.1, FR-6.2, FR-6.4, FR-6.5 |
| **主要 Story** | US-T08, US-T09, US-T10 |
| **Iteration 0 範囲** | 1 日 1 回 Push のみ + Static フォールバック実装。エスカレーション・Bedrock 連携は Iteration 1+ |
| **公開インターフェース** | <ul><li>EventBridge 受信: スケジューラトリガー（平日 12:00, 18:00）, `Delivered`, `IntakeRecorded`</li><li>EventBridge 発行: `EscalationLevelChanged`</li><li>REST: `GET /dashboard/stimulus`（煽り文取得）</li></ul> |

### C6. Punishment（罰: サルサループ + Tシャツ罰モック）

| 項目 | 内容 |
|------|------|
| **Unit 対応** | U6 |
| **責務** | 火曜 21:00 チェックポイント Lambda、24h カウントダウン超過判定、Tシャツ発注フラグ立て、サルサ通知 30 分間隔ループ |
| **主要 FR** | FR-1.1, FR-1.2 |
| **主要 Story** | US-T06, US-T07, US-M05 |
| **Iteration 0 範囲** | 火曜 21:00 チェック + Tシャツフラグ + サルサ 1 回。30 分間隔ループは Iteration 1+ |
| **公開インターフェース** | <ul><li>EventBridge 受信: スケジューラトリガー（火 21:00）, `IntakeRecorded`</li><li>EventBridge 発行: `PunishmentTriggered`</li><li>REST: `GET /punishments/history`</li></ul> |

### C7. PwaFrontend（PWA フロントエンド）

| 項目 | 内容 |
|------|------|
| **Unit 対応** | U7 |
| **責務** | ダッシュボード UI（24h カウントダウン、TACO ボタン、煽り文表示、調理ガイド、罰履歴、Share）、Web Push 購読管理、Service Worker |
| **主要 FR** | FR-4.1, FR-4.2, FR-4.3, FR-4.4 |
| **主要 Story** | US-T14, US-T10, US-M01〜M05 |
| **Iteration 0 範囲** | Vite + React で 3 画面（TACO ボタン / 24h カウントダウン / 配送ステータス）。Share 機能・履歴グラフは Iteration 1+ |
| **公開インターフェース** | <ul><li>S3 + CloudFront で静的配信</li><li>API Gateway HTTP API クライアント</li><li>Web Push（VAPID）</li></ul> |

### ~~C8. ChatGPTGptAsset（ChatGPT カスタム GPT）~~ → **v1.1 で削除**

> **削除理由**: 要件 v1.8 で FR-6.3（ChatGPT カスタム GPT による会話誘導）をスコープから削除したため、対応コンポーネントも廃止。
> ID は再利用しない（C9 Infrastructure はそのまま C9 のまま）。

### C9. Infrastructure（CDK 横断インフラ）

| 項目 | 内容 |
|------|------|
| **Unit 対応** | U9 |
| **責務** | AWS CDK (Python) による IaC。全 Stack の定義、CI/CD、CloudWatch、AWS Budgets、Cognito、API Gateway、DynamoDB、EventBridge カスタムバス |
| **主要 FR** | NFR 全般 |
| **主要 Story** | — |
| **Iteration 0 範囲** | 単一 Stack で全 Unit 動作可能な最小 CDK。Stack 分離・CI/CD パイプラインは Iteration 1+ |
| **公開インターフェース** | <ul><li>CDK Synth / CDK Deploy</li><li>GitHub Actions（Iteration 1+）</li></ul> |

---

## 2. 横断的責務（Cross-Cutting Concerns）

Q2=B に従い、**専用コンポーネントは設けず、全 Lambda で AWS Lambda Powertools を標準利用**する。

| 責務 | 実装手段 | 適用範囲 |
|------|---------|---------|
| 構造化ログ | `aws_lambda_powertools.Logger` | 全 Lambda |
| 分散トレーシング | `aws_lambda_powertools.Tracer`（X-Ray 有効化） | 全 Lambda |
| カスタムメトリクス | `aws_lambda_powertools.Metrics` | 全 Lambda |
| エラーハンドリング | Powertools `@event_source` デコレータ + try/except + DLQ | 全 Lambda |
| 認可チェック | API Gateway JWT Authorizer（C1 が払い出した Cognito JWT を検証） | 全 API エンドポイント |
| シークレット管理 | AWS Systems Manager Parameter Store / Secrets Manager（VAPID 鍵、Bedrock モデル ID） | 全 Lambda |
| PII マスキング | Powertools `Logger.append_keys` + 明示的にメールアドレス等を除外 | 全 Lambda（PoC レベル） |

---

## 3. 公開インターフェース総覧

### 3.1 同期（HTTP / API Gateway）

| エンドポイント | コンポーネント | 概要 |
|---|---|---|
| `GET /me` | C1 | 自ユーザー情報 |
| `POST /signup-trial` | C1 | トライアル開始 |
| `POST /intakes` | C2 | TACO 記録 |
| `GET /intakes/weekly` | C2 | 週次摂取数 |
| `GET /orders/current` | C3 | 今週の発注 |
| `POST /orders/cancel-attempt` | C3 | キャンセル試行（→ 発注増） |
| `GET /recipes/{kit_type}` | C4 | レシピ取得 |
| `GET /dashboard/stimulus` | C5 | 煽り文取得 |
| `GET /punishments/history` | C6 | 罰履歴 |

### 3.2 非同期（EventBridge カスタムバス）

| イベント名 | 発行元 | 受信先 |
|---|---|---|
| `OrderDecided` | C3 | C7 |
| `Delivered` | C3 | C4, C5, C7 |
| `IntakeRecorded` | C2 | C5, C6 |
| `EscalationLevelChanged` | C5 | C7 |
| `PunishmentTriggered` | C6 | C7 |

### 3.3 静的アセット

| パス | コンポーネント | 用途 |
|---|---|---|
| `assets/recipes/*.json` | C4 | レシピライブラリ（Lambda にバンドル） |
| `assets/recipes/_default.json` | C4 | フォールバックレシピ |

---

## 4. FR Coverage Matrix（FR → Component）

| FR | Component | Iteration 0 で必要 |
|---|---|---|
| FR-1.1 サルサ通知 | C6 | ✅（1 回鳴れば OK） |
| FR-1.2 Tシャツ罰 | C6 | ✅ |
| FR-1.3 摂取記録 | C2 | ✅ |
| FR-2.1 逆比例ロジック | C3 | ✅（最小） |
| FR-2.2 メニュー / Type A/B | C3 | ⚠️ Type A のみ |
| FR-2.3 レシピ配信 | C4 | ✅（2〜3 種） |
| FR-2.4 配送モック | C3 | ✅ |
| FR-4 ダッシュボード | C7 | ⚠️ 3 画面のみ |
| FR-5 認証・トライアル | C1 | ✅（認証のみ） |
| FR-6.1 煽り文 | C5 | ⚠️ 静的のみ |
| FR-6.2 誘惑 Push | C5 | ⚠️ 1 日 1 回 |
| ~~FR-6.3 ChatGPT GPT~~ | ~~C8~~ | **v1.1 で削除**（要件 v1.8 で廃止） |
| FR-6.4 エスカレーション | C5 | ❌（Iteration 1+） |
| FR-6.5 生成テキスト | C5 | ⚠️ 静的のみ |

✅ 全 FR がいずれかのコンポーネントに割り当て済み。
