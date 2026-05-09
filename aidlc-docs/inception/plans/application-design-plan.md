# Application Design Plan: タコ中

**Project**: タコ中 (Tako-chū / Tako-tues)
**Document Version**: 1.0
**Created**: 2026-04-29
**Status**: ✅ Approved (2026-04-29)
**Updated**: 2026-04-29 v1.0 — 12 問 + 2 follow-up 全解決、5 設計成果物生成、ユーザー承認済み
**Phase**: INCEPTION - Application Design (Plan, Closed)
**Inputs**: requirements.md v1.7 / stories.md v1.3 / story-board.md v1.3 / execution-plan.md v1.3（後続更新で requirements v1.8 / stories v1.4 / story-board v1.4 / execution-plan v1.4 に進化）

---

## 0. このドキュメントの目的

Workflow Planning v1.3 で「Walking Skeleton ファースト + 反復モデル」が決まった。
Application Design では **Iteration 0 を最速で立ち上げる前に必須となる "共有契約" と "コンポーネント境界"** を確定する。
具体的には:

1. コンポーネント識別と責務（components.md）
2. メソッドシグネチャ（component-methods.md）
3. サービス層・オーケストレーション（services.md）
4. コンポーネント依存・通信パターン（component-dependency.md）
5. 上記を統合した application-design.md

> **重要**: 詳細なビジネスロジック（逆比例の閾値、エスカレーション分岐の細部）は **Construction の Functional Design (per-unit) で扱う**。本ステージでは「コンポーネントの境界・責務・インターフェース・通信方式」を決める。

---

## 1. Plan Outline（作業ステップ）

### Step A: 既存成果物の再読込
- [x] requirements.md v1.7 の FR/NFR/Tech Stack 全項（現在は v2.0 が最新）
- [x] stories.md v1.3 の Acceptance Criteria（受入基準）（現在は v2.0 が最新）
- [x] story-board.md v1.3 のタイムラインと UI コピー
- [x] execution-plan.md v1.3 の 9 Unit 暫定マッピング（v1.4 で 8 Unit に縮減済み）

### Step B: コンポーネント識別
- [x] 9 Unit を「コンポーネント」に対応付け（1:1 とは限らない）（v1.4 で 8 Unit に縮減済み）
- [x] 横断コンポーネント（Auth Context、Push Subscription Manager 等）の有無を判定
- [x] 静的アセット（レシピ JSON、フォールバックテンプレート）の格納コンポーネント決定

### Step C: メソッドシグネチャ（高レベル）
- [x] 各コンポーネントの主要メソッドの I/O 型を列挙
- [x] DynamoDB アクセスパターンに直結するメソッドを優先抽出
- [x] Bedrock 呼び出しメソッドの抽象化（フォールバック含む）

### Step D: サービス層 / オーケストレーション
- [x] EventBridge Scheduler でトリガーされる **Scheduled Service**（金 21:00 / 月 21:00 / 火 21:00 / 平日 12:00 / 18:00）
- [x] Lambda Handler の責務分離（API Gateway 経由 / EventBridge 経由）
- [x] Single-table DynamoDB のキー戦略(PK/SK パターン)

### Step E: 依存・通信パターン
- [x] Unit 間連携を Unit Coupling Principles P1（イベント駆動）に従い設計
- [x] EventBridge カスタムバス上のイベント名を確定
- [x] 同期通信が必要な箇所（PWA → API）と非同期通信（バックエンド間）を区別

### Step F: 成果物生成
- [x] components.md 生成
- [x] component-methods.md 生成
- [x] services.md 生成
- [x] component-dependency.md 生成
- [x] application-design.md（統合）生成

### Step G: 整合性レビュー
- [x] 全 FR がいずれかのコンポーネントにマッピングされている
- [x] 全 User Story の受入基準を満たすメソッドが定義されている
- [x] Walking Skeleton で必要な最小契約が揃っている
- [x] OI-14（レシピ JSON Schema）の方針が明文化されている

---

## 2. Questions for User（合計 12 問）

> **回答方法**: 各質問の `[Answer]:` 行に直接記入してください。複数選択（A, B 同時）も可、補足コメント歓迎。
> 不明な点は「不明」「お任せ」と書いていただければ、AI-DLC 推奨案で進めます。

---

### Component Identification（コンポーネント境界）

#### Q1. コンポーネント粒度

execution-plan v1.3 で 9 Unit を暫定定義済み（U1〜U9）。Application Design でこの 9 Unit を**そのままコンポーネント境界として採用するか**、さらに細分化／統合するか?

- **A. 9 Unit = 9 コンポーネントに 1:1 対応**（execution-plan の構成を維持）（v1.4 で C8 削除、現 **8 コンポーネント**）
- **B. 1:1 を基本にしつつ、横断的責務を持つコンポーネント（Auth Context / Push Subscription Manager / Notification Composer）を追加**
- **C. もっと粗く統合**（例: U1+U2 を「User Domain」、U3+U4 を「Order Domain」、U5+U6 を「Notification & Punishment Domain」）
- **D. AI-DLC 推奨に任せる**

[Answer]: D

#### Q2. 横断的責務の扱い

ログ・メトリクス・エラーハンドリング・認可チェック・PII マスキング等の横断的責務をどう実装するか?

- **A. 各 Lambda Handler に直接書く**（最小・最速）
- **B. AWS Lambda Powertools（Logger / Tracer / Metrics）を全 Lambda で標準利用**（推奨）
- **C. カスタム共通ライブラリを作成**（過剰になる可能性）
- **D. AI-DLC 推奨に任せる**（→ B 推奨）

[Answer]: B

---

### Component Methods（メソッドシグネチャ）

#### Q3. API インターフェース定義方針

PWA ↔ Backend の REST API 契約をどの方式で記述するか?

- **A. OpenAPI 3.x YAML をリポジトリ同梱**（標準・PWA 側で型生成可能）
- **B. Markdown のテーブル形式で簡易定義**（軽量、Walking Skeleton 向き）
- **C. TypeScript / Pydantic 型定義から自動生成**（コード優先）
- **D. AI-DLC 推奨に任せる**（→ Walking Skeleton では B、Iteration 1+ で A に格上げを推奨）

[Answer]: A

#### Q4. メソッドの抽象化深度

Bedrock 呼び出し（FR-6.1 煽り文 / FR-6.2 誘惑 Push）をどう抽象化するか?

- **A. 直接 boto3 を Lambda Handler から呼ぶ**（薄い・最速）
- **B. 内部に `StimulusGenerator` 抽象クラスを置き、`BedrockStimulusGenerator` / `StaticFallbackStimulusGenerator` の 2 実装を切替可能にする**（テスタビリティ高、フォールバックが綺麗）
- **C. LangChain や類似の抽象化レイヤーを導入**（PoC では過剰）
- **D. AI-DLC 推奨に任せる**（→ B 推奨。フォールバック静的テンプレ運用が要件にあるため）

[Answer]: B

---

### Service Layer Design（サービス層）

#### Q5. Lambda 関数の粒度

サービス境界 ↔ Lambda 関数の対応をどうするか?

- **A. 1 Unit = 1 Lambda**（最大 9 関数、シンプル）
- **B. ユースケース単位で細分**（例: `IntakeRecord` / `IntakeQueryWeekly` を別 Lambda に分ける、12〜15 関数想定）
- **C. 機能で集約**（API 経由は Monolith Lambda、スケジューラ系は別途、計 3〜4 関数）
- **D. AI-DLC 推奨に任せる**（→ B 推奨。デプロイ単位を小さく保てて反復が速い）

[Answer]: C

#### Q6. EventBridge イベントスキーマ

Unit 間の非同期連携で扱うイベント名と payload を確定する。Walking Skeleton で**最低限必要**なイベントは以下を想定:

1. `OrderDecided`: 金曜 21:00 の発注決定（U3 → U4, U7）
2. `Delivered`: 月曜 21:00 の配送（U3 → U4, U5, U7）
3. `IntakeRecorded`: 「TACO!」記録（U2 → U5, U6）
4. `EscalationLevelChanged`: Push 無視カウンタ昇格（U5 内部）
5. `PunishmentTriggered`: 火曜 21:00 罰発火（U6 → U7）

これでよいか?

- **A. 上記 5 件で OK**
- **B. 上記 5 件 + 追加（コメントで指定）**
- **C. もっと少なくしたい（コメントで指定）**
- **D. AI-DLC 推奨に任せる**（→ A）

[Answer]: A

---

### Component Dependencies（依存・通信）

#### Q7. DynamoDB データモデル方針

DynamoDB のテーブル戦略は?

- **A. Single-table 設計**（PK/SK + GSI、AWS 推奨、Single-user 想定で十分）
- **B. Multi-table 設計**（User / Order / Intake / Recipe / Notification 等で分離、可読性高）
- **C. Single-table を基本にしつつ、レシピライブラリだけは別配置（リポジトリ同梱 JSON）**
- **D. AI-DLC 推奨に任せる**（→ C を推奨。レシピは静的化決定済みのため）

[Answer]: C

#### Q8. 通信パターンの優先順位

Unit 間通信のデフォルト方式を選ぶ場合の優先順位は?

- **A. EventBridge カスタムバス > DynamoDB Streams > 直接 invoke**（最も疎結合）
- **B. EventBridge を主、DynamoDB Streams を補助**（イベントが二箇所に分散しない設計）
- **C. すべて EventBridge に寄せる**（DynamoDB Streams は使わない）
- **D. AI-DLC 推奨に任せる**（→ B 推奨。Intake → Escalation 等は DynamoDB Streams が直感的）

[Answer]: B

---

### Design Patterns（設計パターン）

#### Q9. 設計スタイル

全体の設計スタイル?

- **A. レイヤードアーキテクチャ**（Handler / UseCase / Repository / External の 4 層、Python では `ports & adapters` 風）
- **B. ファンクショナル / 軽量**（Lambda Handler 内に直接ロジック、必要な時だけ関数分割）
- **C. Clean Architecture**（PoC では過剰）
- **D. AI-DLC 推奨に任せる**（→ A 軽量版を推奨。ユニットテストが書きやすく、反復改善で差し替えやすい）

[Answer]: A

#### Q10. PWA フロントエンドの構成

PWA（U7）の技術選択?

- **A. Vanilla JS / TypeScript + 軽量フレームワーク（Vite + Vue or Svelte）**（PoC スピード重視）
- **B. React + Next.js（CSR）**（エコシステム最大）
- **C. plain HTML + 最小 JS**（Walking Skeleton 向き、Iteration 後半で C → A に格上げ）
- **D. AI-DLC 推奨に任せる**（→ Walking Skeleton では C、Iteration 1+ で A への移行を提案）

[Answer]: A

---

### Walking Skeleton 接続

#### Q11. Walking Skeleton で接続する Bedrock の扱い

Iteration 0（Walking Skeleton）で Bedrock を**実際に呼ぶか**、**完全フォールバックのみ**にするか?

- **A. Iteration 0 では完全フォールバック（静的テンプレートのみ）。Bedrock 連携は Iteration 1+**（最速）
- **B. Iteration 0 で Bedrock 1 経路だけ実装し、フォールバックも併設**（後の差し替えがスムーズ）
- **C. Iteration 0 から全 Bedrock 経路を実装**（リスクあり、推奨せず）
- **D. AI-DLC 推奨に任せる**（→ A 推奨。Q-α=C で GPT を切り離した方針と整合）

[Answer]: A

#### Q12. レシピ JSON Schema（OI-14 解消）

レシピ JSON のスキーマをどこまで厳密に定義するか?

- **A. 最小限の型のみ（`title`, `kit_type`, `servings`, `steps[]`, `tone: "amigo"`）**（Walking Skeleton 向き）
- **B. JSON Schema で厳密定義 + バリデーション（Lambda 起動時にスキーマチェック）**（堅牢）
- **C. Pydantic モデルで定義し Python から型エラーを発生させる**（型安全）
- **D. AI-DLC 推奨に任せる**（→ A から開始し Iteration 1 以降で C に格上げを推奨）

[Answer]: A

---

## 3. 質問への回答後の流れ

1. ユーザーが上記 12 問の `[Answer]:` 行を埋めて返信
2. AI が回答を**曖昧性チェック**（"mix of" "不明" 等を発見したら追加質問）
3. 曖昧性が解消されたら設計成果物を一括生成
4. ユーザー承認 → 次ステージ（Units Generation）

---

## 4. 進行状況チェックリスト

- [x] Application Design Plan を作成（本ファイル）
- [x] ユーザーが 12 問に回答
- [x] 曖昧性レビュー＋必要に応じて follow-up 質問（Q5-α / Q10-α 解消）
- [x] 5 つの設計成果物を生成（components / component-methods / services / component-dependency / application-design）
- [x] aidlc-state.md / audit.md 更新
- [x] ユーザー承認（2026-04-29）

---

## 5. Open Items 引き継ぎ

execution-plan v1.3 §8 から本ステージで扱うもの:

- **OI-14（レシピ JSON Schema）**: Q12 で方針決定 → 成果物に反映
- **OI-1（PBT Partial 対象関数）**: Construction の NFR Requirements ステージで扱う（本ステージでは扱わない）
- **OI-12' / OI-13'（Bedrock 煽り文の品質ガードレール / コスト管理）**: Q4 / Q11 で抽象化方針を決定 → Functional Design / NFR Design で詳細化

人/対外系（OI-6 〜 OI-10）はユーザー側の宿題として継続。
