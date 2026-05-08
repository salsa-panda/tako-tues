# タコ中（Tako-chū / tako-tues）

> **タコスは偉大である。本サービスは、ユーザーのタコス欲を _予測しない_。_生成し、煽り、強制する_。**

**AWS Summit Japan 2026 AI-DLC Hackathon 応募作品**
テーマ「人をダメにするサービス」 — 世界初の **積極的依存支援プラットフォーム** PoC

---

## 🌮 何が「ダメ」になるサービスなのか

多くの AI サービスは「ユーザーが何を欲しているかを **予測** し、最適化する」方向に進化してきた。
**タコ中は、その逆を行く。**

- AI は **欲を予測する側でなく、欲を生成する側** に立つ
- ユーザーが食べないほど来週の発注量は **逆比例で増える**（地獄）
- 完成品は届かない。届くのは **材料**。台所に立たされるのはユーザー本人
- 24h 以内に作らないと **Tシャツ送付 + サルサ通知ループ**（30 分間隔・鳴り続ける）
- 平日昼夕に **誘惑 Push** がスマホを揺らし、無視するほどトーンがエスカレート

食事だけでなく **台所・冷蔵庫・通知音・SNS の話題** まで、タコスが日常に侵食する。

> （免責: 医療機器ではない／パロディ的要素を含む）

---

## ⚙️ 設計原則

| # | 原則 | 概要 |
|---|------|------|
| 1 | 生活侵食 ＞ 単発体験 | 1 回の摂取で終わらず、生活のあらゆる場面にタコスが現れる |
| 2 | 作らせる ＞ 食べさせる | 完成品ではなく材料を届け、調理時間と労力をユーザーに投じさせる |
| 3 | 強制 ＞ 提案 | ユーザーの欲を「当てに行く」のではなく「外圧で作り出す」 |
| 4 | 逃げ場をなくす | 食べない／作らない選択肢を取りづらい状況をデザインする |
| 5 | シュール ＞ 真面目 | テーマ「人をダメにする」に沿った "おかしみ" を優先 |
| 6 | AI は欲を生成する側 | AI は予測器ではなく **刺激生成器・調理サポーター** として使う |

---

## 🏗️ アーキテクチャ

```mermaid
flowchart LR
    subgraph User["End User"]
        U((Tarō / Minami))
    end

    subgraph Client["Client Side"]
        PWA[C7 PWA<br/>Vite+React]
        Cognito[(Cognito<br/>Hosted UI)]
    end

    subgraph AWS["AWS Cloud (ap-northeast-1)"]
        APIGW[API Gateway<br/>HTTP API + JWT Auth]
        ApiMono[Lambda<br/>api-monolith]
        Scheds[Lambda × 5<br/>schedulers]
        DDB[(DynamoDB<br/>Single-table)]
        EBBus[EventBridge<br/>tako-bus]
        EBSched[EventBridge<br/>Scheduler]
        S3[(S3 + CloudFront<br/>PWA static)]
        Bedrock[Amazon Bedrock<br/>Iteration 1+]
        WP[Web Push VAPID]
        CW[CloudWatch /<br/>Budgets]
    end

    U --> PWA
    PWA -->|HTTPS| APIGW --> ApiMono
    PWA --> Cognito

    EBSched --> Scheds
    Scheds --> DDB
    Scheds --> EBBus
    Scheds --> Bedrock
    Scheds --> WP

    ApiMono --> DDB
    ApiMono --> EBBus
    EBBus -->|route| ApiMono

    PWA --> S3
    ApiMono -.->|logs/metrics| CW
    Scheds -.->|logs/metrics| CW

    style PWA fill:#FFA726,stroke:#E65100,color:#000
    style ApiMono fill:#4CAF50,stroke:#1B5E20,color:#fff
    style Scheds fill:#4CAF50,stroke:#1B5E20,color:#fff
    style EBBus fill:#BBDEFB,stroke:#0D47A1,color:#000
    style EBSched fill:#BBDEFB,stroke:#0D47A1,color:#000
    style DDB fill:#C8E6C9,stroke:#1B5E20,color:#000
    style Bedrock fill:#FFEBEE,stroke:#B71C1C,color:#000
```

### 主要 AWS サービス

- **API Gateway HTTP API + JWT Authorizer** — Cognito Hosted UI の JWT を検証
- **AWS Lambda × 6** — `api-monolith` × 1 ＋ EventBridge Scheduler 駆動の `scheduler-*` × 5
- **Amazon DynamoDB** — Single-table 設計（`PK = USER#{user_id}`、GSI1: 週次集計、GSI2: アクティブユーザー）
- **Amazon EventBridge カスタムバス `tako-bus`** — 5 イベント（`OrderDecided` / `Delivered` / `IntakeRecorded` / `EscalationLevelChanged` / `PunishmentTriggered`）で Unit 間を疎結合
- **EventBridge Scheduler** — 金 21:00 発注決定 / 月 21:00 配送モック / 火 21:00 罰チェックポイント / 平日 12:00・18:00 誘惑 Push
- **Amazon Bedrock** — 煽り文・誘惑 Push 文の動的生成（Iteration 1+）。Iteration 0 は静的フォールバック
- **Amazon S3 + CloudFront** — PWA 静的配信
- **Amazon Cognito Hosted UI** — 認証
- **AWS Lambda Powertools (Python)** — Logger / Tracer / Metrics 標準利用
- **AWS CDK (Python)** — IaC

### Unit 構成（疎結合 8 Unit）

| ID | Unit | 責務 |
|----|------|------|
| U1 | AuthAndTrial | Cognito 認証 / 無料トライアル管理 |
| U2 | IntakeLogging | 摂取（=作った）記録 / 週次サマリ |
| U3 | OrderEngineAndDeliveryMock | 強制発注ロジック / 配送モック / キャンセル試行（→ 罰増） |
| U4 | RecipeLibrary | キット種別 1:1 の **静的レシピ JSON 配信**（食中毒リスク回避のため動的生成は廃止） |
| U5 | StimulusEngine | 煽り文 / 誘惑 Push / エスカレーション（Bedrock + 静的フォールバック） |
| U6 | Punishment | 罰チェックポイント / Tシャツ発注モック / サルサ通知ループ |
| U7 | PwaFrontend | Vite + React + Service Worker + Web Push |
| U9 | Infrastructure | CDK Stack / IAM / Monitoring / CI/CD |

Unit 間通信はすべて **EventBridge カスタムバス経由**。Lambda 直接 invoke はゼロ。AWS サーバーレスで完結（外部 ChatGPT GPT 等への依存なし）。

---

## 📋 AI-DLC プロセス成果物

本プロジェクトは [AWS AI-DLC（AI-Driven Development Lifecycle）](https://github.com/aws-samples/sample-ai-driven-development-lifecycle) ワークフローに完全準拠して開発された。
**INCEPTION フェーズ全 7 ステージ完了済み**。

### INCEPTION フェーズ成果物

| ステージ | 主な成果物 | リンク |
|----------|-----------|-------|
| Workspace Detection | Greenfield 判定 | [aidlc-state.md](./aidlc-docs/aidlc-state.md) |
| Requirements Analysis | 要件 v1.8（FR / NFR / 設計原則） | [requirements.md](./aidlc-docs/inception/requirements/requirements.md) |
| User Stories | 15 ストーリー + 2 ペルソナ（タロウ / ミナミ） | [stories.md](./aidlc-docs/inception/user-stories/stories.md) / [personas.md](./aidlc-docs/inception/user-stories/personas.md) / [story-board.md](./aidlc-docs/inception/user-stories/story-board.md) |
| Workflow Planning | Walking Skeleton + 反復モデル v1.4 | [execution-plan.md](./aidlc-docs/inception/plans/execution-plan.md) |
| Application Design | 8 コンポーネント / レイヤード 4 層 / 5 イベント | [application-design.md](./aidlc-docs/inception/application-design/application-design.md) |
| Units Generation | 8 Unit | [unit-of-work.md](./aidlc-docs/inception/application-design/unit-of-work.md) / [unit-of-work-dependency.md](./aidlc-docs/inception/application-design/unit-of-work-dependency.md) / [unit-of-work-story-map.md](./aidlc-docs/inception/application-design/unit-of-work-story-map.md) |
| Audit Trail | 全インタラクションの ISO 8601 タイムスタンプ付きログ | [audit.md](./aidlc-docs/audit.md) |

### 開発スタイル: Walking Skeleton + 反復モデル

書類審査と PoC 完成を切り離し、**最速で E2E が動く骨組み（Iteration 0）を作って動かしながら磨く** 反復スタイル。
予選（5/30）・決勝（6/26）に向けて 12〜17 反復サイクルを想定。

詳細は [execution-plan.md](./aidlc-docs/inception/plans/execution-plan.md) を参照。

---

## 🎯 リリース目標

| マイルストーン | 期日 | 状態 |
|--------------|------|------|
| 書類審査エントリー（Inception 成果物 + 公開 GitHub） | 2026-05-10 23:59 JST | INCEPTION 完了 ✅ |
| 予選 MVP デモ（麻布台ヒルズ・対面） | 2026-05-30 | Iteration 0〜3 で対応 |
| 決勝デモ（幕張メッセ AWS Summit Japan・対面） | 2026-06-26 | Iteration 4+ |

---

## 📁 ディレクトリ構造（INCEPTION 完了時点）

```
tako-tues/
├── README.md                    # ← 本ファイル
├── CLAUDE.md                    # AI-DLC ワークフロー指示書
├── aidlc-docs/                  # AI-DLC 成果物（INCEPTION 全 7 ステージ）
│   ├── aidlc-state.md
│   ├── audit.md
│   └── inception/
│       ├── requirements/
│       ├── user-stories/
│       ├── plans/
│       └── application-design/
└── reference/                   # ハッカソン規約・参考資料
```

CONSTRUCTION フェーズで以下が追加される予定:

```
├── backend/                     # Python Lambda (Powertools)
│   └── src/
│       ├── c1_auth_and_trial/
│       ├── c2_intake_logging/
│       ├── c3_order_engine/
│       ├── c4_recipe_library/
│       ├── c5_stimulus_engine/
│       └── c6_punishment/
├── frontend/                    # Vite + React PWA
├── infra/                       # AWS CDK (Python)
└── assets/
    ├── contracts/openapi/api.yaml
    └── recipes/                 # 静的レシピ JSON（手書き、キット種別 1:1）
```

---

## 🔗 関連リンク

- **応募チーム**: salsa-panda
- **ハッカソン**: AWS Summit Japan 2026 AI-DLC Hackathon
- **テーマ**: 人をダメにするサービス
- **AI-DLC**: [AWS Sample - AI-Driven Development Lifecycle](https://github.com/aws-samples/sample-ai-driven-development-lifecycle)

---

## 📝 ライセンス

本プロジェクトはハッカソン応募作品です。コード・ドキュメントの再利用条件は確定後に追記予定。
