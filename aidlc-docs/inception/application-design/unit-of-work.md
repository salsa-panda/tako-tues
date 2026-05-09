# Unit of Work: タコ中

**Project**: タコ中 (Tako-chū / Tako-tues)
**Document Version**: 1.1
**Created**: 2026-04-29
**Updated**:
- 2026-05-08 v1.1: 要件 v1.8 / components v1.1 反映。**FR-6.3 廃止に伴い §2 Documentation セクション（C8 ChatGPT GPT）を削除**。Total Units を「8 + Documentation 1」→「8（純）」に変更。ディレクトリ構造から `assets/prompts/` を削除、Hackathon Theme Alignment 表から Documentation 行削除、Story-Map 集計表から Documentation 行削除
- 2026-05-08 v1.1（rework v2.0 復元）: ビジネス意図の深掘り rework に伴いバックアップから復元。8 Unit 構成・各 Unit の責務は変化なし。
**Phase**: INCEPTION - Completed (2026-04-29 承認 / 2026-05-08 v1.1 / 2026-05-09 rework 復元)
**Total Units**: 8（U1〜U7, U9）— v1.1 で旧 Documentation セクションを削除
**Decisions**: Q1=D / Q2=A / Q3=C / Q3-α=A / Q4=D / Q5=D（v1.1 注: Q3=C は当時の判断記録。FR-6.3 自体が v1.8 で廃止されたため Documentation セクションは現存しない）

---

## 0. 設計方針サマリー

- **Unit 数**: 8 個（C1〜C7, C9 と 1:1）。**v1.1 で旧 Documentation セクション（C8 ChatGPT GPT）を削除**（FR-6.3 廃止）
- **マッピング方針**: 状態を保持する Unit を主担当 + FR Coverage Matrix で確認（Q1=D）
- **Walking Skeleton 並行戦略**: C9 Infrastructure を最初に固める → 全 Unit 並行で薄く実装（Q2=A）
- **コード組織**: `backend/src/` 配下に Component サブディレクトリ + `frontend/` + `infra/` + `assets/`（Q4=D）
- **共有コード**: 言語依存型は `backend/src/shared/`、宣言的契約は `assets/contracts/`（Q5=D）

---

## 1. Unit Catalog（8 Unit）

### U1. AuthAndTrial（認証・トライアル）

| 項目 | 内容 |
|------|------|
| **Component** | C1 AuthAndTrial |
| **責務** | Cognito Hosted UI 認証 / トライアル期間管理 / JWT 発行支援 |
| **主要 FR** | FR-5 |
| **主要 Story** | US-T01（無料トライアル） |
| **状態** | UserProfile / TrialState（DynamoDB Single-table） |
| **Iteration 0 範囲** | Cognito 認証経路のみ。トライアル日数チェックは固定値 |
| **コード位置** | `backend/src/c1_auth_and_trial/` |
| **Lambda 関与** | api-monolith |
| **デプロイ独立性** | Stack 分離は Iteration 1+（Iteration 0 は単一 Stack） |

### U2. IntakeLogging（摂取記録）

| 項目 | 内容 |
|------|------|
| **Component** | C2 IntakeLogging |
| **責務** | 「TACO!」記録の永続化 / 重複検知 / 週次サマリ提供 / IntakeRecorded イベント発行 |
| **主要 FR** | FR-1.3 |
| **主要 Story** | US-T02（タコス摂取記録） |
| **状態** | IntakeRecord（DynamoDB） |
| **Iteration 0 範囲** | TACO ボタン → DynamoDB 1 行追加。重複検知は Iteration 1+ |
| **コード位置** | `backend/src/c2_intake_logging/` |
| **Lambda 関与** | api-monolith |
| **デプロイ独立性** | Stack 分離は Iteration 1+ |

### U3. OrderEngineAndDeliveryMock（強制発注 + 配送モック）

| 項目 | 内容 |
|------|------|
| **Component** | C3 OrderEngineAndDeliveryMock |
| **責務** | 金 21:00 強制発注決定（逆比例 + メニュー選定 + Type A/B 切替） / 月 21:00 配送遷移 / キャンセル試行 → 発注増 |
| **主要 FR** | FR-2.1, FR-2.2, FR-2.4 |
| **主要 Story** | US-T03（強制発注通告） / US-T04（キャンセル試行） / US-T12（強制バリエーションウィーク） / US-M01（発注スクショ投稿） / US-M02（キャンセル試行投稿） |
| **状態** | Order / DeliveryStatus（DynamoDB） |
| **Iteration 0 範囲** | 逆比例ロジック最小実装 + メニュー 2-3 種固定 + キャンセル UI なし。Type B バリエーションは Iteration 1+ |
| **コード位置** | `backend/src/c3_order_engine/` |
| **Lambda 関与** | api-monolith + scheduler-friday-decide-order + scheduler-monday-deliver |
| **デプロイ独立性** | スケジューラ Lambda はそれぞれ独立 |

### U4. RecipeLibrary（静的レシピライブラリ）

| 項目 | 内容 |
|------|------|
| **Component** | C4 RecipeLibrary |
| **責務** | キット種別 1:1 でレシピ取得 / `_default.json` フォールバック / Bedrock 動的生成は不採用 |
| **主要 FR** | FR-2.3 |
| **主要 Story** | US-T13（材料受け取り + レシピ配信） / US-M04（料理動画投稿） |
| **状態** | なし（静的 JSON は読み取り専用） |
| **Iteration 0 範囲** | レシピ JSON 2-3 種 + `_default.json`。全キット種別カバレッジは Iteration 1+ |
| **コード位置** | `backend/src/c4_recipe_library/` + `assets/recipes/*.json` |
| **Lambda 関与** | api-monolith（Lambda パッケージに JSON バンドル） |
| **デプロイ独立性** | レシピ JSON 単独更新でも Lambda 再デプロイが必要（Iteration 0）。Iteration 1+ で S3 配信に移行を検討 |

### U5. StimulusEngine（刺激エンジン）

| 項目 | 内容 |
|------|------|
| **Component** | C5 StimulusEngine |
| **責務** | 平日昼夕の誘惑 Push / 24h ウィンドウ内 Push 無視カウンタ / Level A/B/C エスカレーション / 煽り文生成 / `StimulusGenerator` Protocol（Bedrock / Static の 2 実装） |
| **主要 FR** | FR-6.1, FR-6.2, FR-6.4, FR-6.5 |
| **主要 Story** | US-T08（誘惑 Push） / US-T09（エスカレーション） / US-T10（煽り文表示） |
| **状態** | EscalationCounter / PushSubscription（DynamoDB） |
| **Iteration 0 範囲** | 1 日 1 回 Push のみ + Static フォールバック実装。エスカレーション・Bedrock 連携は Iteration 1+ |
| **コード位置** | `backend/src/c5_stimulus_engine/` |
| **Lambda 関与** | api-monolith + scheduler-weekday-stimulus-lunch + scheduler-weekday-stimulus-evening |
| **デプロイ独立性** | スケジューラ Lambda 2 つは独立 |

### U6. Punishment（罰: サルサループ + Tシャツ罰モック）

| 項目 | 内容 |
|------|------|
| **Component** | C6 Punishment |
| **責務** | 火 21:00 チェックポイント / 24h カウントダウン超過判定 / Tシャツ発注フラグ / サルサ通知 30 分間隔ループ |
| **主要 FR** | FR-1.1, FR-1.2 |
| **主要 Story** | US-T06（Tシャツ発注通知） / US-T07（サルサ通知ループ） / US-M05（罰受けました投稿） |
| **状態** | PunishmentRecord / SalsaLoopState（DynamoDB） |
| **Iteration 0 範囲** | 火 21:00 チェック + Tシャツフラグ + サルサ 1 回。30 分間隔ループは Iteration 1+ |
| **コード位置** | `backend/src/c6_punishment/` |
| **Lambda 関与** | api-monolith + scheduler-tuesday-checkpoint |
| **デプロイ独立性** | スケジューラ Lambda は独立 |

### U7. PwaFrontend（PWA フロントエンド）

| 項目 | 内容 |
|------|------|
| **Component** | C7 PwaFrontend |
| **責務** | ダッシュボード UI（24h カウントダウン / TACO ボタン / 煽り文表示 / 調理ガイド / 罰履歴 / Share）/ Web Push 購読管理 / Service Worker |
| **主要 FR** | FR-4.1, FR-4.2, FR-4.3, FR-4.4 |
| **主要 Story** | US-T14（24h カウントダウン UI） |
| **関与 Story（UI レイヤー）** | US-T01〜T14 全て（PWA からの操作起点）, US-M01〜M05（Share UI 経路） |
| **状態** | クライアント側のみ（PushSubscription はサーバ管理） |
| **Iteration 0 範囲** | Vite + React で 3 画面（TACO ボタン / 24h カウントダウン / 配送ステータス）。Share・履歴グラフは Iteration 1+ |
| **コード位置** | `frontend/` |
| **Lambda 関与** | なし（S3 + CloudFront で静的配信） |
| **デプロイ独立性** | フロント単独で再デプロイ可能（CloudFront キャッシュ無効化を伴う） |

### U9. Infrastructure（CDK 横断インフラ）

| 項目 | 内容 |
|------|------|
| **Component** | C9 Infrastructure |
| **責務** | AWS CDK (Python) IaC / Stack 群 / IAM / CloudWatch / AWS Budgets / Cognito / API Gateway / DynamoDB / EventBridge カスタムバス + Scheduler / S3 + CloudFront |
| **主要 FR** | NFR 全般 |
| **主要 Story** | — |
| **状態** | CDK Stack（コード）+ デプロイ済み AWS リソース |
| **Iteration 0 範囲** | 単一 Stack で全 Unit 動作可能な最小 CDK。Stack 分離・CI/CD パイプラインは Iteration 1+ |
| **コード位置** | `infra/` |
| **Lambda 関与** | 全 Lambda の定義主体 |
| **デプロイ独立性** | 全 Stack のオーナー（他 Unit はこの Stack 上にデプロイされる） |

---

## ~~2. Documentation セクション（C8 ChatGPT GPT）~~ → **v1.1 で削除**

> 要件 v1.8 / components v1.1 で FR-6.3 廃止に伴い、本セクションは廃止。後続セクション番号は維持（§3 = コード組織戦略）。

#### Unit 外配置の根拠

C8 を per-unit ループから外した判断（Q3=C）の根拠:

- **技術的独立性**: ChatGPT カスタム GPT は OpenAI Platform 上の作業であり、AWS リソースへの依存がない。CDK スタック定義も不要なため、per-unit ループの依存チェーン（U9 先行 → 各 Unit 並行）に乗せられない
- **進行管理の合理性**: AWS 側 E2E（Iteration 0〜3）が安定してから着手することでシステム全体との整合を確認できる（Q-α=C 決定、`execution-plan.md v1.3 §5.3` 参照）
- **コア機能の保証**: Unit 外 = 実装計画から除外ではなく、**進行管理の単位を分けている**。FR-6.3（3 層エージェントの最終層）は `aidlc-state.md` の記録どおり Iteration 4+ で正式着手し、PoC 完成基準（E2E シナリオ全通り）の一部として扱われる

---

## 3. コード組織戦略（Greenfield）

### 3.1 リポジトリ構造

```
tako-tues/
├── README.md
├── CLAUDE.md
├── aidlc-docs/                    # AI-DLC 成果物（既存）
│   ├── inception/
│   ├── construction/              # Iteration 0 開始時に作成
│   ├── operations/
│   ├── aidlc-state.md
│   └── audit.md
├── backend/                       # バックエンド（Python）
│   ├── pyproject.toml
│   ├── src/
│   │   ├── shared/                # 共有コード（Q5=D の言語依存部分）
│   │   │   ├── types.py           # 共通 Pydantic モデル
│   │   │   ├── events.py          # EventBridge イベントクラス
│   │   │   ├── powertools.py      # Logger / Tracer / Metrics 設定
│   │   │   └── ddb.py             # DynamoDB クライアント・キー戦略
│   │   ├── c1_auth_and_trial/     # U1
│   │   │   ├── handlers/
│   │   │   ├── usecases/
│   │   │   ├── repositories/
│   │   │   └── external/
│   │   ├── c2_intake_logging/     # U2（同構造）
│   │   ├── c3_order_engine/       # U3（同構造）
│   │   ├── c4_recipe_library/     # U4（同構造）
│   │   ├── c5_stimulus_engine/    # U5（同構造、StimulusGenerator Protocol を含む）
│   │   ├── c6_punishment/         # U6（同構造）
│   │   └── handlers/              # 6 Lambda 関数のエントリーポイント
│   │       ├── api_monolith.py
│   │       ├── scheduler_friday.py
│   │       ├── scheduler_monday.py
│   │       ├── scheduler_tuesday.py
│   │       ├── scheduler_weekday_lunch.py
│   │       └── scheduler_weekday_evening.py
│   └── tests/                     # pytest + Hypothesis (PBT Partial)
│       ├── unit/                  # 各 Unit の単体テスト
│       └── integration/           # E2E テスト
├── frontend/                      # U7 PWA（Vite + React）
│   ├── package.json
│   ├── vite.config.ts
│   ├── public/
│   │   └── sw.js                  # Service Worker
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   ├── api/                   # OpenAPI 自動生成クライアント
│   │   └── services/
│   └── tests/
├── infra/                         # U9 Infrastructure（AWS CDK Python）
│   ├── pyproject.toml
│   ├── cdk.json
│   ├── app.py
│   └── stacks/
│       ├── core_stack.py          # DynamoDB / EventBridge / Cognito
│       ├── api_stack.py           # API Gateway + api-monolith Lambda
│       ├── scheduler_stack.py     # 5 個のスケジューラ Lambda
│       ├── frontend_stack.py      # S3 + CloudFront
│       └── monitoring_stack.py    # CloudWatch / Budgets
├── assets/                        # 言語非依存の契約 + 静的アセット
│   ├── contracts/                 # Q5=D の宣言的契約
│   │   ├── openapi/
│   │   │   └── api.yaml           # OpenAPI 3.x（Q3=A）
│   │   ├── eventbridge/
│   │   │   └── events.schema.json # EventBridge イベント JSON Schema
│   │   ├── recipe-schema.json     # レシピ JSON Schema（OI-14）
│   │   └── webpush-payload.schema.json
│   ├── recipes/                   # U4 静的レシピライブラリ
│   │   ├── _default.json
│   │   ├── type-a-pastor.json
│   │   ├── type-a-fish.json
│   │   ├── type-a-veg.json
│   │   ├── type-b-mango-week.json
│   │   ├── type-b-habanero-max.json
│   │   └── type-b-carnitas-scratch.json
│   └── images/                    # SVG イラスト・通知画像（v1.1 で旧 prompts/ を削除）
└── .github/
    └── workflows/                 # GitHub Actions（Iteration 1+）
```

### 3.2 Unit ↔ ディレクトリの対応

| Unit | ディレクトリ |
|------|-------------|
| U1 AuthAndTrial | `backend/src/c1_auth_and_trial/` |
| U2 IntakeLogging | `backend/src/c2_intake_logging/` |
| U3 OrderEngineAndDeliveryMock | `backend/src/c3_order_engine/` |
| U4 RecipeLibrary | `backend/src/c4_recipe_library/` + `assets/recipes/` |
| U5 StimulusEngine | `backend/src/c5_stimulus_engine/` |
| U6 Punishment | `backend/src/c6_punishment/` |
| U7 PwaFrontend | `frontend/` |
| U9 Infrastructure | `infra/` |

### 3.3 各 Unit のレイヤー構造

`backend/src/c{N}_{name}/` 配下は共通で 4 層構造（Q9=A レイヤード継承）:

```
c{N}_{name}/
├── __init__.py
├── handlers/         # API Gateway / EventBridge エントリーポイント
├── usecases/         # ビジネスロジック
├── repositories/     # DynamoDB アクセス
└── external/         # 外部サービス（Bedrock / Web Push / Cognito）
```

---

## 4. Iteration 0 並行戦略（Q2=A 反映）

### 4.1 立ち上げ順序

```
Phase 1: U9 Infrastructure（最初・単独）
   ↓
Phase 2: 全 Unit 並行（U1, U2, U3, U4, U5, U6, U7 を AI が同時に薄く実装）
   ↓
Phase 3: Walking Skeleton 結合確認（E2E シナリオ #1, #2 が貧弱に通る）
```

### 4.2 Phase 1: U9 単独で固めるべきもの

- 単一 CDK Stack（`tako-iteration-0-stack`）
- DynamoDB Single-table（PK/SK + GSI1/GSI2）
- EventBridge カスタムバス `tako-bus`
- EventBridge Scheduler（5 個の cron ルール、ターゲットは未定義でも可）
- API Gateway HTTP API + JWT Authorizer
- Cognito User Pool + User Pool Client
- S3 + CloudFront（PWA 静的配信、空バケットでも可）
- CloudWatch ロググループ
- AWS Budgets ¥3,000 アラート

これらが完成すれば、各 Unit はリソース ARN / 名前を参照しつつ独立に実装できる。

### 4.3 Phase 2: 並行実装可能な Unit

```
U9 Infrastructure 完成後 ─┬─→ U1 AuthAndTrial（Cognito 経路実装）
                        ├─→ U2 IntakeLogging（POST /intakes 実装）
                        ├─→ U3 OrderEngineAndDeliveryMock（金月スケジューラ実装）
                        ├─→ U4 RecipeLibrary（静的 JSON + GET /recipes 実装）
                        ├─→ U5 StimulusEngine（最小 Push 実装、Static のみ）
                        ├─→ U6 Punishment（火曜チェックポイント実装）
                        └─→ U7 PwaFrontend（Vite + React で 3 画面実装）
```

実装順序の依存はほぼ無し（Unit Coupling Principles が効く）。AI に並行で書かせる。

### 4.4 Phase 3: 結合確認

- E2E シナリオ #1（罰回避パス）: 強制発注 → 配送 → TACO 記録 → 罰なし
- E2E シナリオ #2（罰発火パス）: 強制発注 → 配送 → 24h 経過 → Tシャツ + サルサ 1 回

---

## 5. 整合性確認

### 5.1 全 16 ストーリーの Unit 割り当て確認

詳細は `unit-of-work-story-map.md` 参照。本ファイルでは件数のみ確認:

| Unit | 主担当ストーリー数 |
|------|-------------------|
| U1 | 1（US-T01） |
| U2 | 1（US-T02） |
| U3 | 5（US-T03, T04, T12, M01, M02） |
| U4 | 2（US-T13, M04） |
| U5 | 3（US-T08, T09, T10） |
| U6 | 3（US-T06, T07, M05） |
| U7 | 2（US-T14, US-T15） |
| U9 | 0（横断） |
| **合計** | **16**（v2.0 で US-T15 追加。旧 17 - 削除 2 + 追加 1） |

✅ 全 16 ストーリーがマップ済み（v2.0: US-T15「文化的受容」を U7 に追加）。

### 5.2 全 8 Component / 8 Unit の責務明記確認

| Component | Unit | 責務記載 |
|-----------|------|---------|
| C1 | U1 | ✅ |
| C2 | U2 | ✅ |
| C3 | U3 | ✅ |
| C4 | U4 | ✅ |
| C5 | U5 | ✅ |
| C6 | U6 | ✅ |
| C7 | U7 | ✅ |
| ~~C8~~ | ~~（Documentation）~~ | **v1.1 で削除**（FR-6.3 廃止） |
| C9 | U9 | ✅ |

✅ 全 Component に責務明記済み。

### 5.3 Walking Skeleton 必須セット

Iteration 0 で立ち上げる必須 Unit:

- ✅ U9（最初に固める）
- ✅ U1, U2, U3, U4, U6（バックエンドの強制ループ最低限）
- ⚠️ U5（1 日 1 回 Push のみ、Static 実装）
- ⚠️ U7（3 画面のみ）
- v1.1 で旧「Documentation（C8 GPT）」行を削除（FR-6.3 廃止）

---

## 6. 参照

- [components.md](./components.md) - 9 コンポーネント責務
- [component-methods.md](./component-methods.md) - メソッドシグネチャ
- [services.md](./services.md) - Lambda 構成 + サービス層
- [component-dependency.md](./component-dependency.md) - 依存マトリクス
- [application-design.md](./application-design.md) - 統合参照
- [unit-of-work-dependency.md](./unit-of-work-dependency.md) - Unit 間依存マトリクス
- [unit-of-work-story-map.md](./unit-of-work-story-map.md) - ストーリー → Unit マップ
