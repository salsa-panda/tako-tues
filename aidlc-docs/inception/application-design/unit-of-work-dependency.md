# Unit of Work Dependency: タコ中

**Project**: タコ中 (Tako-chū / Tako-tues)
**Document Version**: 1.1
**Created**: 2026-04-29
**Updated**:
- 2026-05-08 v1.1: 要件 v1.8 / unit-of-work v1.1 反映。**FR-6.3 廃止に伴い Doc(C8) 列・行を Unit Dependency Matrix から削除**、§§ Documentation (C8) / Doc(C8) に関する全記述を削除（テスト戦略・並行実装スコア・依存テーブル）
**Status**: ✅ Approved (v1.0 2026-04-29) / Updated v1.1 2026-05-08 (Doc(C8) 列・行削除)
**Phase**: INCEPTION - Completed (Units Generation Approved 2026-04-29 / v1.1 反映 2026-05-08)

---

## 0. このドキュメントの位置付け

`component-dependency.md` の §1 Dependency Matrix を **Unit 視点に変換**したもの。各 Unit がいつ・どの順序で実装可能か（Walking Skeleton 並行戦略）を明確にする。

---

## 1. Unit Dependency Matrix

行 = 呼ぶ側（依存元）、列 = 呼ばれる側（依存先）。
凡例: **sync**（同期 API）/ **async**（EventBridge 非同期）/ **shared**（共有リソース）/ **static**（静的アセット）/ **—**（依存なし）

|      | U1 Auth | U2 Intake | U3 Order | U4 Recipe | U5 Stimulus | U6 Punish | U7 PWA | U9 Infra |
|------|---------|-----------|----------|-----------|-------------|-----------|--------|----------|
| **U1**     | —       | —         | —        | —         | —           | —         | —      | shared(Cognito 設定) |
| **U2**     | shared(JWT) | — | —        | —         | async(IntakeRecorded) | async(IntakeRecorded) | — | shared(DDB) |
| **U3**     | shared(JWT) | shared(DDB read) | — | async(Delivered) | async(Delivered) | shared(DDB read) | async(OrderDecided) | shared(DDB, EventBus) |
| **U4**     | shared(JWT) | — | static(KitType) | — | — | — | async(via Delivered) | shared(Lambda bundle) |
| **U5**     | shared(JWT) | async(IntakeRecorded → counter reset) | shared(DDB read) | — | — | — | sync(GET /dashboard/stimulus) | shared(EventBus) |
| **U6**     | shared(JWT) | shared(DDB read) / async(IntakeRecorded → stop salsa) | shared(DDB read) | — | — | — | sync(GET /punishments/history) / async(PunishmentTriggered) | shared(EventBus) |
| **U7**     | sync(GET /me, POST /signup-trial) | sync(POST /intakes) | sync(GET /orders/current) | sync(GET /recipes/{kit_type}) | sync(GET /dashboard/stimulus) | sync(GET /punishments/history) | — | shared(CloudFront) |
| **U9**     | —       | —         | —        | —         | —           | —         | —      | —        |

> **v1.1 注**: 旧「Doc(C8)」列・行は FR-6.3 廃止に伴い削除。

---

## 2. 依存の種類別整理

### 2.1 共有リソース依存（shared via U9）

すべての Unit は U9 Infrastructure に依存する（前提リソース）:

| Unit | U9 から提供される共有リソース |
|------|----------------------------|
| U1 | Cognito User Pool / Hosted UI 設定 |
| U2 | DynamoDB Table / EventBridge Bus |
| U3 | DynamoDB Table / EventBridge Bus / EventBridge Scheduler |
| U4 | Lambda Layer or Lambda Bundle 機構 |
| U5 | DynamoDB Table / EventBridge Bus / EventBridge Scheduler / Web Push 鍵（Parameter Store） |
| U6 | DynamoDB Table / EventBridge Bus / EventBridge Scheduler / Web Push 鍵 |
| U7 | S3 + CloudFront / API Gateway URL |

→ **U9 完成が全 Unit 並行実装の前提**（Q2=A）。

### 2.2 同期 API 依存（PWA → Backend）

U7 PwaFrontend は**全バックエンド Unit を API 経由で呼ぶ唯一の Unit**:

```
U7 ──┬─→ GET /me, POST /signup-trial      → U1
     ├─→ POST /intakes, GET /intakes/weekly → U2
     ├─→ GET /orders/current, POST /cancel-attempt → U3
     ├─→ GET /recipes/{kit_type}            → U4
     ├─→ GET /dashboard/stimulus            → U5
     └─→ GET /punishments/history           → U6
```

→ U7 は API モックで先行実装可能（OpenAPI YAML が `assets/contracts/openapi/api.yaml` にあるため）。

### 2.3 非同期イベント依存（EventBridge `tako-bus`）

5 イベントによる Unit 間連携:

```
[scheduler-friday in U3] ──publish OrderDecided──→ (U7 通知用、Iteration 1+)
[scheduler-monday in U3] ──publish Delivered─────→ U4, U5, U7
[U2 POST /intakes]       ──publish IntakeRecorded→ U5（counter reset）, U6（stop salsa）
[U5 internal]            ──publish EscalationLevelChanged→ U7（通知用）
[scheduler-tuesday in U6]──publish PunishmentTriggered→ U7（通知用）
```

→ EventBridge カスタムバスは**全イベント受信先を api-monolith に集約**するため（services.md §4.3）、Unit 間の Lambda 直接 invoke はゼロ。Unit Coupling Principle P1 を遵守。

### 2.4 共有 DynamoDB（read アクセス）

複数 Unit が同じテーブルを読む:

| 読む側 | 読むデータ | 元の書き込み Unit |
|-------|-----------|-----------------|
| U3 | Intake count（過去 7 日） | U2 |
| U5 | Intake records（24h 内） | U2 |
| U5 | Order（kit_type 等） | U3 |
| U6 | Intake records（24h 内） | U2 |
| U6 | Order（status） | U3 |

→ Single-table 設計（PK = USER#user_id, SK = INTAKE#timestamp など）で読み取りパスは決定論的。Unit Coupling Principle P2「DB スキーマ = API 契約」を遵守。

### 2.5 静的アセット依存

| 依存元 | 依存先 |
|-------|-------|
| U4 | `assets/recipes/*.json`（Lambda パッケージにバンドル） |
| U7 | `assets/openapi/api.yaml`（クライアント自動生成） |

---

## 3. Walking Skeleton 並行戦略（Q2=A）

### 3.1 Phase 1: U9 Infrastructure（最初・単独）

```
Day 0          Day 1          Day 2          Day 3
[U9 立ち上げ───────────────────────────────]
   - DynamoDB Single-table（PK/SK + GSI1/GSI2）
   - EventBridge カスタムバス tako-bus
   - EventBridge Scheduler 5 cron ルール
   - API Gateway HTTP API + JWT Authorizer
   - Cognito User Pool + Client
   - S3 + CloudFront（空バケットでも可）
   - CloudWatch ロググループ
   - AWS Budgets ¥3,000
   - 単一 Stack: tako-iteration-0-stack
```

### 3.2 Phase 2: 全 Unit 並行（AI が同時実装）

```
Day 3〜Day 5: U9 完成後

Track 1（バックエンド core）  Track 2（体験）       Track 3（フロント）
─────────────────────       ───────────         ──────────────────
U1 AuthAndTrial              U4 RecipeLibrary    U7 PwaFrontend
U2 IntakeLogging             U5 StimulusEngine   （Vite + React）
U3 OrderEngine               
U6 Punishment                                    
```

実装は AI が並行で書くため、**Track 区分は便宜上のもの**で、人間が分担する必要はない。Iteration 0 完了基準は**全 Track が薄く立ち上がり結合できること**。

### 3.3 Phase 3: 結合確認（Walking Skeleton 達成）

```
Day 5〜Day 6: E2E シナリオ通る

E2E #1（罰回避）: U7 でログイン → U2 で TACO 記録 → U6 で罰なし確認
E2E #2（罰発火）: U7 でログイン → 何もしない → 火 21:00 cron → U6 で Tシャツ罰 + サルサ 1 回
```

達成基準（Construction フェーズで検証 — Phase 3 結合確認時に各項目を [x] にチェック）:
- [ ] AWS にデプロイされ、PWA URL でアクセス可
- [ ] Cognito ログインが通る
- [ ] TACO ボタンで DynamoDB に書ける
- [ ] 月曜 21:00 cron で配送ステータスが遷移する（手動で時刻を変えてテスト可）
- [ ] 火曜 21:00 cron で 24h 内 intake 無ければ Tシャツフラグが立つ + サルサ Push 1 回
- [ ] 24h カウントダウン UI が表示される

---

## 4. Iteration 1 以降の依存変化

### 4.1 Iteration 1+ で追加される依存

| 追加要素 | 影響 Unit |
|---------|----------|
| DynamoDB Streams ハンドラ | U5（counter reset）, U6（stop salsa） |
| Bedrock 呼び出し | U5（StimulusGenerator 実装切替） |
| サルサ 30 分間隔ループ | U6（scheduler-salsa-loop-tick 追加） |
| 強制バリエーションウィーク（Type B） | U3, U4（kit_type 増加） |
| Push エスカレーション完全実装 | U5（Level A/B/C 切替） |
| Share / 動画字幕画像 | U7（html-to-image 等） |

### 4.2 Iteration 4+ で追加

> v1.1 で旧「Documentation (C8) の作業開始（ChatGPT GPT 公開）」記述を削除（FR-6.3 廃止）。
> Iteration 4+ は Share 機能 / 動画字幕画像 / バイラル素材作成へ前倒し（execution-plan v1.4 参照）。

---

## 5. デプロイ単位（CDK Stack 分離）

### 5.1 Iteration 0: 単一 Stack

```
tako-iteration-0-stack
└── 全リソースを 1 Stack に集約
```

### 5.2 Iteration 1+: Stack 分離（独立デプロイ）

```
tako-core-stack            ← U9 が責任
├── DynamoDB / EventBus / Cognito

tako-api-stack             ← U9（API GW）+ U1〜U6（Lambda コード）
├── API Gateway HTTP API
└── api-monolith Lambda

tako-scheduler-stack       ← U9（Scheduler）+ U3, U5, U6（Lambda コード）
├── EventBridge Scheduler
└── 5 個の scheduler-* Lambda

tako-frontend-stack        ← U9（S3+CloudFront）+ U7（PWA bundle）
├── S3 Bucket
└── CloudFront Distribution

tako-monitoring-stack      ← U9 が責任
├── CloudWatch Alarms
└── AWS Budgets
```

### 5.3 Stack 分離後の依存

```
tako-core-stack（前提・全 Stack の基盤）
   ↑
   ├── tako-api-stack（依存: core 出力 ARN）
   ├── tako-scheduler-stack（依存: core 出力 ARN）
   ├── tako-frontend-stack（依存: core の API URL）
   └── tako-monitoring-stack（依存: 全 Stack のリソース）
```

---

## 6. テスト境界

各 Unit の単体テストでカバーすべき外部依存（component-dependency.md §6 を Unit 視点に再掲）:

| Unit | Mock すべき依存 |
|------|-----------------|
| U1 | Cognito（moto） |
| U2 | DynamoDB（moto）, EventBridge（boto3 stub）, freezegun |
| U3 | DynamoDB（moto）, EventBridge（stub）, freezegun |
| U4 | ファイルシステム（pyfakefs もしくは実 JSON） |
| U5 | DynamoDB（moto）, EventBridge（stub）, StimulusGenerator Protocol（FakeGenerator）, Web Push（stub） |
| U6 | DynamoDB（moto）, EventBridge（stub）, freezegun, Web Push（stub） |
| U7 | API クライアント（MSW で mock）, 各 React Component を Storybook 風に独立確認 |
| U9 | CDK Snapshot Test（CDK assertions） |

---

## 7. 並行実装可能性スコア

各 Unit の「他 Unit を待たずに着手できる度合い」:

| Unit | スコア | 補足 |
|------|------|------|
| U9 | ★★★★★ | 何にも依存しない（最初に着手） |
| U1 | ★★★★☆ | U9 完成のみ依存 |
| U2 | ★★★★☆ | U9 完成のみ依存 |
| U3 | ★★★★☆ | U9 完成のみ依存（U2 のスキーマは契約として参照） |
| U4 | ★★★★★ | U9 完成のみ依存（静的 JSON のみ、最も独立） |
| U5 | ★★★☆☆ | U9 + U2/U3 の DDB 契約に依存（読み取り） |
| U6 | ★★★☆☆ | U9 + U2/U3 の DDB 契約に依存（読み取り） |
| U7 | ★★★★☆ | U9 + 各 API の OpenAPI 契約があれば並行可能（モック先行） |

→ **U9 が完成すれば即座に全 Unit を並行実装できる**ため、Walking Skeleton の所要日数は U9 立ち上げ + 全 Unit 並行実装の MAX で決まる。

---

## 8. 参照

- [unit-of-work.md](./unit-of-work.md) - 8 Unit の責務（v1.1 で Documentation セクション削除）
- [unit-of-work-story-map.md](./unit-of-work-story-map.md) - ストーリー → Unit マップ
- [component-dependency.md](./component-dependency.md) - Component レベルの詳細依存
- [services.md](./services.md) - Lambda 構成 + EventBridge ルーティング
- [../plans/execution-plan.md](../plans/execution-plan.md) - Walking Skeleton 戦略全体
