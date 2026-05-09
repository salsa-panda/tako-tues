# Services: タコ中

**Project**: タコ中 (Tako-chū / Tako-tues)
**Document Version**: 1.1
**Created**: 2026-04-29
**Updated**:
- 2026-05-08 v1.1: 要件 v1.8 / components v1.1 反映。**FR-6.3 廃止に伴い「Iteration 0 では未実装」リストから ChatGPT GPT アセット行を削除**
- 2026-05-08 v1.1（rework v2.0 復元）: ビジネス意図の深掘り rework に伴いバックアップから復元。サービス定義は変化なし。
**Phase**: INCEPTION - Completed (2026-04-29 承認 / 2026-05-08 v1.1 / 2026-05-09 rework 復元)

---

## 0. 設計方針

- **Lambda 関数の粒度**: Q5=C-1（API Monolith Lambda 1 + スケジューラ Lambda 5、計 6 関数）
- **同期通信**: PWA → API Gateway HTTP API → ApiMonolith Lambda → UseCase（C1〜C6 の API Handler を内部で分岐）
- **非同期通信**: EventBridge カスタムバス（5 イベント）+ DynamoDB Streams（補助、Iteration 1+）
- **スケジューラ**: EventBridge Scheduler（cron）→ 5 個のスケジューラ Lambda

---

## 1. Lambda 関数構成（計 6 関数）

| # | Lambda 関数名 | トリガー | 担当範囲 | コンポーネント |
|---|--------------|---------|---------|---------------|
| 1 | **api-monolith** | API Gateway HTTP API | 全 REST エンドポイント（9 個）を内部で path-based に分岐 | C1〜C6 の Handler 層 |
| 2 | **scheduler-friday-decide-order** | EventBridge Scheduler `cron(0 21 ? * FRI *)` | 金 21:00 強制発注決定 | C3 |
| 3 | **scheduler-monday-deliver** | EventBridge Scheduler `cron(0 21 ? * MON *)` | 月 21:00 配送遷移 + Delivered イベント発行 | C3 |
| 4 | **scheduler-tuesday-checkpoint** | EventBridge Scheduler `cron(0 21 ? * TUE *)` | 火 21:00 罰チェックポイント | C6 |
| 5 | **scheduler-weekday-stimulus-lunch** | EventBridge Scheduler `cron(0 12 ? * MON-FRI *)` | 平日 12:00 誘惑 Push | C5 |
| 6 | **scheduler-weekday-stimulus-evening** | EventBridge Scheduler `cron(0 18 ? * MON-FRI *)` | 平日 18:00 誘惑 Push | C5 |

> **Iteration 1+ で追加が想定される Lambda**:
> - `event-stream-intake-handler`（DynamoDB Streams で IntakeRecorded を捌く、エスカレーションリセット用）
> - `scheduler-salsa-loop-tick`（30 分間隔のサルサ通知ループ用）

---

## 2. ApiMonolith Lambda の内部ルーティング

```python
# src/handlers/api_monolith/handler.py
from aws_lambda_powertools.event_handler import APIGatewayHttpResolver

app = APIGatewayHttpResolver()

# 各コンポーネントの API Handler をマウント
from c1_auth_and_trial.handlers import register_routes as register_c1
from c2_intake_logging.handlers import register_routes as register_c2
from c3_order_engine.handlers import register_routes as register_c3
from c4_recipe_library.handlers import register_routes as register_c4
from c5_stimulus_engine.handlers import register_routes as register_c5
from c6_punishment.handlers import register_routes as register_c6

register_c1(app)
register_c2(app)
register_c3(app)
register_c4(app)
register_c5(app)
register_c6(app)

def lambda_handler(event, context):
    return app.resolve(event, context)
```

### 2.1 ルーティング表

| Path | HTTP Method | Component | UseCase |
|------|-------------|-----------|---------|
| /me | GET | C1 | `get_user_profile` |
| /signup-trial | POST | C1 | `start_trial` |
| /intakes | POST | C2 | `record_intake` |
| /intakes/weekly | GET | C2 | `query_weekly` |
| /orders/current | GET | C3 | （Repository 直接） |
| /orders/cancel-attempt | POST | C3 | `attempt_cancel` |
| /recipes/{kit_type} | GET | C4 | `get_recipe` |
| /dashboard/stimulus | GET | C5 | `generate_dashboard_stimulus` |
| /punishments/history | GET | C6 | `list_history` |

---

## 3. スケジューラ Lambda の責務

### 3.1 scheduler-friday-decide-order

- **トリガー**: EventBridge Scheduler、毎週金 21:00 JST
- **処理フロー**:
  1. アクティブユーザー一覧を C1 Repository から取得
  2. 各ユーザーについて C2 から過去 7 日間の摂取数 N を取得
  3. C3 UseCase `decide_next_week_order(user_id, now)` を呼び発注決定
  4. DynamoDB に Order を upsert
  5. EventBridge `OrderDecided` イベント発行（PWA 通知用、未来の Iteration で）

### 3.2 scheduler-monday-deliver

- **トリガー**: EventBridge Scheduler、毎週月 21:00 JST
- **処理フロー**:
  1. `decided` 状態の Order を全 user 分取得
  2. C3 UseCase `transition_to_delivered(user_id, week_id, now)` で `delivered` に遷移
  3. EventBridge `Delivered` イベント発行
  4. （Iteration 0 では）この時点で C4 RecipeLibrary を呼んで Web Push を送る簡易実装でも可

### 3.3 scheduler-tuesday-checkpoint

- **トリガー**: EventBridge Scheduler、毎週火 21:00 JST
- **処理フロー**:
  1. `delivered` 状態の Order を全 user 分取得
  2. C2 から delivered 後 24h 内の摂取記録があるかチェック
  3. なければ C6 UseCase `fire_tshirt_order(user_id)` + `start_salsa_loop(user_id)` を呼ぶ
  4. EventBridge `PunishmentTriggered` イベント発行
  5. Order 状態を `consumed`（罰により締切）に遷移

### 3.4 scheduler-weekday-stimulus-lunch / evening

- **トリガー**: EventBridge Scheduler、平日 12:00 / 18:00 JST
- **処理フロー**:
  1. アクティブユーザー一覧を取得
  2. 各ユーザーについて現在の `EscalationLevel` を C5 Repository から取得（Iteration 0 では常に Level A）
  3. C5 UseCase `generate_push_stimulus(user_id, level)` で Push 文を生成
  4. C5 WebPushSender で Web Push 送信
  5. （Iteration 1+）60 分後に「Push 無視タイマー」を起動するため EventBridge OneTime Schedule を設定

---

## 4. オーケストレーションパターン

### 4.1 Choreography（イベント駆動の分散調整）

タコ中ではほぼ全ての Unit 間連携を **Choreography（イベント駆動の分散調整）** で行う。
中央集権的な Saga Orchestrator は置かない。

```
[scheduler-friday-decide-order]
    └── EventBridge: OrderDecided
            └── api-monolith (C3) が Order を DynamoDB に書き込み済み
            └── PWA は次回 GET /orders/current で見る

[scheduler-monday-deliver]
    └── EventBridge: Delivered
            ├── (C4) RecipeLibrary がレシピを返す → Web Push
            ├── (C5) StimulusEngine が EscalationCounter を初期化
            └── (C7) PWA がリアルタイム更新（Iteration 1+ でポーリング or WebSocket）

[api-monolith POST /intakes (C2)]
    └── EventBridge: IntakeRecorded
            ├── (C5) StimulusEngine が EscalationCounter を 0 リセット
            └── (C6) Punishment が Salsa Loop を停止

[scheduler-tuesday-checkpoint]
    └── EventBridge: PunishmentTriggered
            └── (C7) PWA が罰履歴を更新（Iteration 1+ でリアルタイム）
```

### 4.2 同期 vs 非同期の使い分け

| パターン | 例 |
|---------|---|
| **同期（API Gateway → Lambda）** | PWA からの読み書き、ユーザー操作起点 |
| **非同期（EventBridge カスタムバス）** | スケジューラ起点、Unit 間連携、状態遷移通知 |
| **非同期（DynamoDB Streams、Iteration 1+）** | テーブル変更を契機にした派生処理（Intake 書き込み → Escalation Counter リセット等、推奨パターン Q8=B） |

### 4.3 EventBridge カスタムバス: `tako-bus`

```yaml
EventBus: tako-bus
Rules:
  - Name: route-order-decided
    EventPattern:
      source: ["tako.order"]
      detail-type: ["OrderDecided"]
    Targets:
      - Lambda: notification-fanout-c7  # Iteration 1+ で追加

  - Name: route-delivered
    EventPattern:
      source: ["tako.order"]
      detail-type: ["Delivered"]
    Targets:
      - Lambda: api-monolith  # 内部で C4/C5/C7 ハンドラに分岐
      
  - Name: route-intake-recorded
    EventPattern:
      source: ["tako.intake"]
      detail-type: ["IntakeRecorded"]
    Targets:
      - Lambda: api-monolith  # C5/C6 ハンドラに分岐

  - Name: route-escalation-changed
    EventPattern:
      source: ["tako.stimulus"]
      detail-type: ["EscalationLevelChanged"]
    Targets:
      - Lambda: api-monolith  # C7 通知用

  - Name: route-punishment
    EventPattern:
      source: ["tako.punishment"]
      detail-type: ["PunishmentTriggered"]
    Targets:
      - Lambda: api-monolith  # C7 通知用
```

> **設計意図**: スケジューラ Lambda 5 個 + ApiMonolith 1 個の構造を維持するため、EventBridge イベントの受信先も `api-monolith` に集約する。Iteration 1+ で trafffic が増えたら専用 Lambda に分離する。

---

## 5. DynamoDB Single-table 設計（Q7=C, Q8=B）

### 5.1 キー戦略

```
PK (Partition Key)        | SK (Sort Key)              | Entity
USER#{user_id}            | PROFILE                    | UserProfile
USER#{user_id}            | TRIAL                      | TrialState
USER#{user_id}            | INTAKE#{ISO8601-timestamp} | IntakeRecord
USER#{user_id}            | ORDER#{week_id}            | Order
USER#{user_id}            | PUNISHMENT#{timestamp}     | PunishmentRecord
USER#{user_id}            | ESCALATION#{window_id}     | EscalationCounter
USER#{user_id}            | PUSH-SUB#{endpoint_hash}   | PushSubscription
USER#{user_id}            | SALSA-LOOP                 | SalsaLoopState
```

### 5.2 GSI

```
GSI1: GSI1PK = WEEK#{week_id}, GSI1SK = USER#{user_id}
  → 週次バッチで全ユーザーの Order を取得（金/月/火スケジューラで使用）

GSI2: GSI2PK = ACTIVE-USERS, GSI2SK = USER#{user_id}
  → アクティブユーザー一覧の取得（スケジューラで使用、ユーザー数 100 以下を想定）
```

### 5.3 DynamoDB Streams 用途（Iteration 1+）

```
INTAKE#{...} の INSERT → C5 EscalationCounter リセット + C6 SalsaLoopState 停止
ORDER の UPDATE (status: delivered) → 不要（既に EventBridge で処理）
```

---

## 6. Iteration 0 で実装するサービスの絞り込み

### Iteration 0 必須

✅ `api-monolith`（最小: GET /me, POST /intakes, GET /orders/current, GET /recipes/{kit_type}, GET /punishments/history）
✅ `scheduler-friday-decide-order`（逆比例ロジック最小、メニュー固定）
✅ `scheduler-monday-deliver`（配送ステータス遷移）
✅ `scheduler-tuesday-checkpoint`（24h チェック + Tシャツフラグ + サルサ 1 回）
✅ EventBridge カスタムバス（イベント定義のみ、受信先は ApiMonolith）

### Iteration 1+ で追加

⚠️ `scheduler-weekday-stimulus-lunch` / `evening`（Iteration 0 では 1 回 Push のみ）
⚠️ DynamoDB Streams ハンドラ
⚠️ `scheduler-salsa-loop-tick`（30 分間隔ループ）
⚠️ `BedrockStimulusGenerator` の有効化

### Iteration 0 では未実装

なし（v1.1 で C8 ChatGPT GPT アセットを削除済み）

---

## 7. NFR 影響

| NFR | このサービス層での扱い |
|-----|---------------------|
| NFR-1 パフォーマンス | api-monolith は Cold Start に注意。Provisioned Concurrency は Iteration 1+ で検討 |
| NFR-2 コスト | api-monolith 1 個に集約しているためデプロイ・観測コスト最小。Bedrock 呼び出しは Iteration 1+ |
| NFR-3 プライバシー | API Gateway JWT Authorizer + Powertools の PII マスキング |
| NFR-4 信頼性 | EventBridge カスタムバスはマネージドで信頼性高。スケジューラ失敗は CloudWatch Alarm |
| NFR-5 開発・保守性 | レイヤード構造 + UseCase 単体テスト + Powertools 統一 |
| NFR-6 スピード | api-monolith の集約により、ステージング・デプロイのリードタイム短縮 |
