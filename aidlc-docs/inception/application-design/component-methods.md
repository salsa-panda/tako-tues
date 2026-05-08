# Component Methods: タコ中

**Project**: タコ中 (Tako-chū / Tako-tues)
**Document Version**: 1.1
**Created**: 2026-04-29
**Updated**:
- 2026-05-08 v1.1: 要件 v1.8 / components v1.1 反映。**FR-6.3 廃止に伴い §9 C8. ChatGPTGptAsset セクションを削除**。Iteration 0 未着手リストから C8 ChatGPT GPT アセット行を削除
**Phase**: INCEPTION - Application Design

---

## 0. 設計方針

- **レイヤード構造**: 各コンポーネントを `Handler` / `UseCase` / `Repository` / `External` の 4 層に分ける（Q9=A）
- **メソッドシグネチャ**: Python type hint で記述。詳細なビジネスルール（閾値、分岐の細部）は **Construction Functional Design (per-unit) で扱う**
- **Bedrock 抽象化**: `StimulusGenerator` プロトコルを定義し、`BedrockStimulusGenerator` / `StaticFallbackStimulusGenerator` の 2 実装を切替可能（Q4=B）
- **API 契約**: OpenAPI 3.x YAML をリポジトリ同梱（Q3=A、`assets/openapi/api.yaml` で管理）

---

## 1. 共通型定義（Shared Types）

```python
from datetime import datetime
from enum import Enum
from typing import Literal
from pydantic import BaseModel

UserId = str  # Cognito sub
KitTypeKey = Literal[
    "type-a-pastor",
    "type-a-fish",
    "type-a-veg",
    "type-b-mango-week",
    "type-b-habanero-max",
    "type-b-carnitas-scratch",
    "_default",
]
DeliveryStatus = Literal["decided", "delivered", "consumed"]
EscalationLevel = Literal["A", "B", "C"]
ToneKey = Literal["amigo", "scolding"]

class IntakeRecord(BaseModel):
    user_id: UserId
    recorded_at: datetime
    portions: int = 1
    note: str | None = None

class Order(BaseModel):
    user_id: UserId
    week_id: str  # ISO 8601 week, e.g. "2026-W19"
    kit_type: KitTypeKey
    portions: int
    status: DeliveryStatus
    decided_at: datetime
    delivered_at: datetime | None = None
    is_variation_week: bool = False

class Recipe(BaseModel):
    kit_type: KitTypeKey
    title: str
    servings: int
    steps: list[str]
    tone: ToneKey = "amigo"

class StimulusText(BaseModel):
    text: str
    level: EscalationLevel
    source: Literal["bedrock", "static"]
```

---

## 2. C1. AuthAndTrial

### 2.1 Handler（API Gateway 経由）

```python
def handle_get_me(event: APIGatewayEvent, ctx: LambdaContext) -> APIGatewayResponse: ...
def handle_signup_trial(event: APIGatewayEvent, ctx: LambdaContext) -> APIGatewayResponse: ...
```

### 2.2 UseCase

```python
class AuthAndTrialUseCase:
    def get_user_profile(self, user_id: UserId) -> UserProfile: ...
    def start_trial(self, user_id: UserId) -> TrialState: ...
    def is_trial_active(self, user_id: UserId, at: datetime) -> bool: ...
```

### 2.3 Repository

```python
class UserRepository:
    def get(self, user_id: UserId) -> UserProfile | None: ...
    def upsert(self, profile: UserProfile) -> None: ...
```

### 2.4 External

- **AWS Cognito**: Hosted UI / JWT 検証は API Gateway Authorizer に委譲。Lambda は decoded claim を読むのみ

---

## 3. C2. IntakeLogging

### 3.1 Handler（API Gateway 経由）

```python
def handle_post_intake(event: APIGatewayEvent, ctx: LambdaContext) -> APIGatewayResponse: ...
def handle_get_intakes_weekly(event: APIGatewayEvent, ctx: LambdaContext) -> APIGatewayResponse: ...
```

### 3.2 UseCase

```python
class IntakeLoggingUseCase:
    def record_intake(
        self, user_id: UserId, recorded_at: datetime, portions: int = 1, note: str | None = None
    ) -> IntakeRecord: ...
    def query_weekly(self, user_id: UserId, week_id: str) -> list[IntakeRecord]: ...
    def is_duplicate_within_minute(self, user_id: UserId, at: datetime) -> bool: ...
```

### 3.3 Repository

```python
class IntakeRepository:
    def insert(self, record: IntakeRecord) -> None: ...
    def list_by_week(self, user_id: UserId, week_id: str) -> list[IntakeRecord]: ...
    def list_recent(self, user_id: UserId, since: datetime) -> list[IntakeRecord]: ...
```

### 3.4 Event Publisher

```python
class IntakeEventPublisher:
    def publish_intake_recorded(self, record: IntakeRecord) -> None:
        # EventBridge custom bus に IntakeRecorded を発行
        ...
```

---

## 4. C3. OrderEngineAndDeliveryMock

### 4.1 Handler

```python
# API Gateway 経由
def handle_get_current_order(event: APIGatewayEvent, ctx: LambdaContext) -> APIGatewayResponse: ...
def handle_cancel_attempt(event: APIGatewayEvent, ctx: LambdaContext) -> APIGatewayResponse: ...

# EventBridge スケジューラ経由（5 関数の中の 1 つに集約予定、後述 services.md）
def handle_friday_decide_order(event: ScheduledEvent, ctx: LambdaContext) -> None: ...
def handle_monday_deliver(event: ScheduledEvent, ctx: LambdaContext) -> None: ...
```

### 4.2 UseCase

```python
class OrderEngineUseCase:
    def decide_next_week_order(self, user_id: UserId, now: datetime) -> Order: ...
    def transition_to_delivered(self, user_id: UserId, week_id: str, at: datetime) -> Order: ...
    def attempt_cancel(self, user_id: UserId, week_id: str) -> Order: ...
    def calculate_portions(self, recent_intake_count: int) -> int:
        # BASE + max(0, THRESHOLD - N) * MULTIPLIER
        # 詳細は Functional Design で確定
        ...
    def select_kit(self, is_variation_week: bool, rng_seed: int) -> KitTypeKey: ...
```

### 4.3 Repository

```python
class OrderRepository:
    def get_current(self, user_id: UserId) -> Order | None: ...
    def upsert(self, order: Order) -> None: ...
    def list_history(self, user_id: UserId, limit: int) -> list[Order]: ...
```

### 4.4 Event Publisher

```python
class OrderEventPublisher:
    def publish_order_decided(self, order: Order) -> None: ...
    def publish_delivered(self, order: Order) -> None: ...
```

---

## 5. C4. RecipeLibrary

### 5.1 Handler（API Gateway 経由）

```python
def handle_get_recipe(event: APIGatewayEvent, ctx: LambdaContext) -> APIGatewayResponse: ...
```

### 5.2 UseCase

```python
class RecipeLibraryUseCase:
    def get_recipe(self, kit_type: KitTypeKey) -> Recipe:
        # キット種別 1:1 で対応するレシピを返す。未登録キーは _default.json を返す
        ...
```

### 5.3 Repository（静的 JSON 読み取り）

```python
class StaticRecipeRepository:
    def __init__(self, recipes_dir: Path): ...
    def get(self, kit_type: KitTypeKey) -> Recipe: ...
    def get_default(self) -> Recipe: ...
    def list_all_kit_types(self) -> list[KitTypeKey]: ...
```

> **注**: Lambda パッケージに `assets/recipes/*.json` をバンドル。Lambda 起動時に全レシピをメモリにロード（Iteration 0 では JSON 数件のためコスト無視できる）。

### 5.4 Event Subscriber

```python
class RecipeNotificationSubscriber:
    def on_delivered(self, event: DeliveredEvent) -> None:
        # Recipe 取得 → Web Push 通知のためのトリガー
        ...
```

---

## 6. C5. StimulusEngine

### 6.1 Handler

```python
# EventBridge スケジューラ経由
def handle_weekday_lunch_push(event: ScheduledEvent, ctx: LambdaContext) -> None: ...
def handle_weekday_evening_push(event: ScheduledEvent, ctx: LambdaContext) -> None: ...

# DynamoDB Streams 経由（Iteration 1+）
def handle_intake_stream(event: DynamoDBStreamEvent, ctx: LambdaContext) -> None: ...
def handle_push_ignored_timer(event: ScheduledEvent, ctx: LambdaContext) -> None: ...

# API Gateway 経由
def handle_get_dashboard_stimulus(event: APIGatewayEvent, ctx: LambdaContext) -> APIGatewayResponse: ...
```

### 6.2 UseCase

```python
class StimulusUseCase:
    def __init__(self, generator: StimulusGenerator, repo: EscalationRepository): ...
    def generate_dashboard_stimulus(self, user_id: UserId, level: EscalationLevel) -> StimulusText: ...
    def generate_push_stimulus(self, user_id: UserId, level: EscalationLevel) -> StimulusText: ...
    def increment_ignore_counter(self, user_id: UserId, push_id: str) -> EscalationLevel: ...
    def reset_counter(self, user_id: UserId, reason: Literal["intake", "tap", "next_window"]) -> None: ...
    def current_level(self, ignore_count: int) -> EscalationLevel: ...
```

### 6.3 Generator Abstraction（Q4=B）

```python
from typing import Protocol

class StimulusGenerator(Protocol):
    def generate(self, kind: Literal["dashboard", "push"], level: EscalationLevel, context: dict) -> str: ...

class BedrockStimulusGenerator:
    def __init__(self, model_id: str, fallback: StimulusGenerator): ...
    def generate(self, kind, level, context) -> str:
        # Bedrock を呼ぶ。失敗時は self.fallback.generate(...) を呼ぶ
        ...

class StaticFallbackStimulusGenerator:
    def __init__(self, templates_path: Path): ...
    def generate(self, kind, level, context) -> str:
        # 静的テンプレートからランダム選定
        ...
```

> **Iteration 0 では `StaticFallbackStimulusGenerator` のみ使用**（Q11=A）。Iteration 1+ で `BedrockStimulusGenerator` に切替。

### 6.4 Repository

```python
class EscalationRepository:
    def get_counter(self, user_id: UserId, window_id: str) -> int: ...
    def increment_counter(self, user_id: UserId, window_id: str) -> int: ...
    def reset_counter(self, user_id: UserId, window_id: str) -> None: ...

class PushSubscriptionRepository:
    def list_by_user(self, user_id: UserId) -> list[PushSubscription]: ...
    def upsert(self, sub: PushSubscription) -> None: ...
```

### 6.5 Event Publisher / Push Sender

```python
class EscalationEventPublisher:
    def publish_level_changed(self, user_id: UserId, new_level: EscalationLevel) -> None: ...

class WebPushSender:
    def send(self, sub: PushSubscription, payload: WebPushPayload) -> None: ...
```

---

## 7. C6. Punishment

### 7.1 Handler

```python
# EventBridge スケジューラ経由
def handle_tuesday_checkpoint(event: ScheduledEvent, ctx: LambdaContext) -> None: ...
def handle_salsa_loop_tick(event: ScheduledEvent, ctx: LambdaContext) -> None:
    # Iteration 1+ で 30 分間隔ループ用に追加
    ...

# DynamoDB Streams 経由（Iteration 1+）
def handle_intake_stream_for_punishment(event: DynamoDBStreamEvent, ctx: LambdaContext) -> None: ...

# API Gateway 経由
def handle_get_punishments_history(event: APIGatewayEvent, ctx: LambdaContext) -> APIGatewayResponse: ...
```

### 7.2 UseCase

```python
class PunishmentUseCase:
    def evaluate_24h_window(self, user_id: UserId, now: datetime) -> bool:
        # delivered 後 24h 経過 + intake なし → True
        ...
    def fire_tshirt_order(self, user_id: UserId) -> PunishmentRecord: ...
    def start_salsa_loop(self, user_id: UserId) -> None: ...
    def stop_salsa_loop(self, user_id: UserId, reason: Literal["intake_recorded"]) -> None: ...
    def list_history(self, user_id: UserId) -> list[PunishmentRecord]: ...
```

### 7.3 Repository

```python
class PunishmentRepository:
    def insert(self, record: PunishmentRecord) -> None: ...
    def list_by_user(self, user_id: UserId, limit: int) -> list[PunishmentRecord]: ...

class SalsaLoopStateRepository:
    def is_active(self, user_id: UserId) -> bool: ...
    def set_active(self, user_id: UserId, active: bool) -> None: ...
```

### 7.4 Event Publisher

```python
class PunishmentEventPublisher:
    def publish_punishment_triggered(self, record: PunishmentRecord) -> None: ...
```

---

## 8. C7. PwaFrontend

PWA は Vite + React（Q10=A-3）で構築。コンポーネントというより**画面群と Service Worker**。

### 8.1 主要画面（React Component）

```typescript
// src/pages/
- HomePage.tsx          // TACO ボタン + 24h カウントダウン + 配送ステータス
- DashboardPage.tsx     // 煽り文 + 摂取履歴グラフ（Iteration 1+）
- RecipePage.tsx        // 今週の調理ガイド
- HistoryPage.tsx       // 罰履歴 / 強制トリガー履歴
- SharePage.tsx         // SNS 投稿用画像生成（Iteration 1+）

// src/components/
- TacoButton.tsx
- CountdownTimer.tsx
- StimulusBanner.tsx
- DeliveryStatusCard.tsx
- ShareButton.tsx
- ServiceWorkerRegistrar.tsx
```

### 8.2 API クライアント（OpenAPI 自動生成想定）

```typescript
// src/api/client.ts （openapi-typescript-codegen で生成）
export class TakoApiClient {
  getMe(): Promise<UserProfile>;
  postIntake(req: IntakeRequest): Promise<IntakeRecord>;
  getCurrentOrder(): Promise<Order>;
  getRecipe(kitType: KitTypeKey): Promise<Recipe>;
  getDashboardStimulus(): Promise<StimulusText>;
  getPunishmentsHistory(): Promise<PunishmentRecord[]>;
  postCancelAttempt(weekId: string): Promise<Order>;
}
```

### 8.3 Service Worker

```typescript
// public/sw.js
self.addEventListener('push', (event) => { ... });
self.addEventListener('notificationclick', (event) => { ... });
```

### 8.4 Web Push 購読管理

```typescript
// src/services/pushSubscription.ts
async function subscribeToWebPush(vapidPublicKey: string): Promise<PushSubscription>;
async function sendSubscriptionToBackend(sub: PushSubscription): Promise<void>;
```

---

## ~~9. C8. ChatGPTGptAsset~~ → **v1.1 で削除**

> 要件 v1.8 / components v1.1 で FR-6.3 廃止に伴いコンポーネントごと削除。後続セクション番号は維持（§10 = C9 Infrastructure はそのまま）。

---

## 10. C9. Infrastructure (CDK)

### 10.1 Stack 構成（Iteration 0 では単一、Iteration 1+ で分離）

```python
# infra/
- app.py
- stacks/
  - core_stack.py        # DynamoDB / EventBridge カスタムバス / Cognito
  - api_stack.py         # API Gateway HTTP API + ApiMonolith Lambda
  - scheduler_stack.py   # EventBridge Scheduler + 5 個のスケジューラ Lambda
  - frontend_stack.py    # S3 + CloudFront
  - monitoring_stack.py  # CloudWatch / Budgets
```

### 10.2 公開コンストラクト（他 Stack から参照されるもの）

```python
class TakoCoreStack(Stack):
    table: dynamodb.Table          # Single-table
    event_bus: events.EventBus     # tako-bus
    user_pool: cognito.UserPool
    user_pool_client: cognito.UserPoolClient
```

---

## 11. メソッド総覧表（API Endpoint → Component）

| Method | Path | Component | UseCase メソッド |
|--------|------|-----------|---------------|
| GET | /me | C1 | `get_user_profile` |
| POST | /signup-trial | C1 | `start_trial` |
| POST | /intakes | C2 | `record_intake` |
| GET | /intakes/weekly | C2 | `query_weekly` |
| GET | /orders/current | C3 | （Repository 直接） |
| POST | /orders/cancel-attempt | C3 | `attempt_cancel` |
| GET | /recipes/{kit_type} | C4 | `get_recipe` |
| GET | /dashboard/stimulus | C5 | `generate_dashboard_stimulus` |
| GET | /punishments/history | C6 | `list_history` |

---

## 12. Iteration 0 で実装するメソッドの絞り込み

✅ Iteration 0 で必須:

- `AuthAndTrialUseCase.get_user_profile` / `start_trial`（簡易版）
- `IntakeLoggingUseCase.record_intake` / `query_weekly`
- `OrderEngineUseCase.decide_next_week_order` / `transition_to_delivered`
- `RecipeLibraryUseCase.get_recipe`
- `StimulusUseCase.generate_dashboard_stimulus` / `generate_push_stimulus`（**Static Fallback のみ**）
- `PunishmentUseCase.evaluate_24h_window` / `fire_tshirt_order` / `start_salsa_loop`（1 回だけ）

⚠️ Iteration 1+ に延期:

- `OrderEngineUseCase.attempt_cancel`（キャンセル UI が無いため）
- `OrderEngineUseCase.select_kit` の Type B バリエーション
- `StimulusUseCase.increment_ignore_counter` / `reset_counter`
- `BedrockStimulusGenerator`
- `PunishmentUseCase.start_salsa_loop` の 30 分間隔ループ

❌ Iteration 0 で未着手:

- なし（v1.1 で C8 ChatGPT GPT アセットを削除済み）
