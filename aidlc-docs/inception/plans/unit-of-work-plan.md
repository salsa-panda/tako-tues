# Unit of Work Plan: タコ中

**Project**: タコ中 (Tako-chū / Tako-tues)
**Document Version**: 1.0
**Created**: 2026-04-29
**Status**: ✅ Approved (2026-04-29 / 2026-05-08 v1.1 で Documentation セクション削除を反映)
**Updated**: 2026-04-29 v1.0 — 5 問 + 1 follow-up 全解決、3 Unit 成果物生成、ユーザー承認済み
**Phase**: INCEPTION - Units Generation (Part 1: Planning, Closed)
**Inputs**: requirements.md v1.7 / stories.md v1.3 / story-board.md v1.3 / execution-plan.md v1.3 / application-design v1.0（5 成果物）（後続更新で requirements v2.0 / stories v2.0 / story-board v2.0 / execution-plan v2.0 / application-design v1.1 に進化）

> **2026-05-08 追記**: 本ファイルは Units Generation Planning 段階の **意思決定の記録（履歴）**。要件 v1.8 で **FR-6.3 ChatGPT カスタム GPT スコープ削除**に伴い、本ファイル内の以下の言及は最新仕様には適用されない:
> - Q3=C「ChatGPT GPT を Documentation セクションに分離」
> - Q3-α=A「Documentation セクションは Iteration 4+ で作業」
> - Q4 / Q5 内の `assets/prompts/` 構造への参照
> 最新の確定 Unit 構成は [unit-of-work.md v1.1](../application-design/unit-of-work.md)（8 Unit、Documentation セクション削除済み）を参照。

---

## 0. このドキュメントの目的

execution-plan v1.3 と Application Design v1.0 で「9 Unit = 9 Component 1:1」が確定済み（**v1.4 で C8 削除により 8 Unit に縮減**。本行は立案当時の記録）。
Units Generation ステージでは:

1. 8 Unit を Construction フェーズの **per-unit ループの正式単位**として確定（v1.4 確定値）
2. 各 Unit の責務 / 含まれるストーリー / 依存関係 / 並行立ち上げ可否を明記
3. Greenfield プロジェクトのコード組織戦略を確定（リポジトリディレクトリ構造）

**注意**: Application Design でアーキテクチャ判断はほぼ全て確定済みのため、本ステージは**確定事項を Unit 視点で再構成する作業**が中心。新規の大きな設計判断は基本的に発生しない。

---

## 1. Plan Outline（作業ステップ）

### Step A: 既存成果物の参照
- [x] application-design.md v1.0 の §2 9 コンポーネント早見表
- [x] execution-plan.md v1.3 §4 Suggested Units の Iteration 0 列
- [x] component-dependency.md v1.0 の §1 Dependency Matrix
- [x] services.md v1.0 の §1 Lambda 関数構成

### Step B: Unit 定義の確定
- [x] 9 Unit を 1:1 で Component に対応（U1〜U9）（v1.1 で C8 廃止に伴い 8 Unit に縮減）
- [x] 各 Unit の責務・含まれるストーリー・主要 FR を明記
- [x] 各 Unit の Iteration 0 範囲を再掲

### Step C: 依存マトリクス
- [x] Application Design の Dependency Matrix を Unit 視点に変換
- [x] Walking Skeleton 立ち上げ時の必須依存パスを抽出
- [x] 並行作業可能な Unit 群を識別

### Step D: ストーリーマッピング
- [x] 17 ストーリー（タロウ 13 + ミナミ 5、UI/UX 関連で重複あり）を Unit にマップ（v1.4 で 15 stories に更新）
- [x] 各ストーリーの主担当 Unit と関与する Unit を区別

### Step E: コード組織戦略（Greenfield）
- [x] リポジトリディレクトリ構造を決定（モノレポ vs マルチリポジトリ等）
- [x] Unit 毎のディレクトリ構成
- [x] 共有コード（型定義、契約、ユーティリティ）の配置場所

### Step F: 成果物生成
- [x] unit-of-work.md 生成
- [x] unit-of-work-dependency.md 生成
- [x] unit-of-work-story-map.md 生成

### Step G: 整合性レビュー
- [x] 全 17 ストーリーがいずれかの Unit にマップされている（v1.4 で 15 stories に更新後も整合）
- [x] 全 9 Unit に責務・依存・コード位置が明記されている（v1.1 で 8 Unit に縮減後も整合）
- [x] Walking Skeleton 立ち上げに必要な Unit セットが識別されている

---

## 2. Questions for User（合計 5 問）

> **回答方法**: 各質問の `[Answer]:` 行に直接記入してください。「お任せ」と書いていただければ AI-DLC 推奨案で進めます。
>
> 質問数を 5 問に抑えました（Application Design で大半の判断が済んでいるため）。

---

### Story Grouping（ストーリーグルーピング）

#### Q1. ストーリー → Unit のマッピング方針

各ユーザーストーリー（例: US-T07 サルサ通知ループ）は複数の Component に関与する場合がある。マッピング時に「主担当 Unit」をどう決めるか?

- **A. 状態を保持する Unit を主担当**（例: US-T07 → C6 Punishment、状態は罰履歴 / サルサループ ON-OFF）
- **B. ユーザー視点で最も触れる Unit を主担当**（例: US-T07 → C7 PWA、ユーザーは通知を見るのみ）
- **C. 主要 FR が割り当てられている Unit を主担当**（FR Coverage Matrix と一致、機械的）
- **D. AI-DLC 推奨に任せる**（→ A + C のハイブリッドを推奨。状態を持つ Unit を主担当 + FR で確認）

[Answer]: D

---

### Dependencies（依存関係）

#### Q2. Walking Skeleton 立ち上げ時の Unit 並行戦略

execution-plan v1.3 では Iteration 0 で全 Unit を「薄く同時に立ち上げる」方針。実際の作業順序は?

- **A. C9 Infrastructure 完成 → 全 Unit 並行**（CDK Stack 立ち上げ後に全 Unit 並行で薄く実装）
- **B. C9 Infrastructure と並行で C1 Auth / C2 Intake / C7 PWA Frontend を並行**（インフラ固まる前にローカルでユースケース実装）
- **C. Application Design の Lambda 6 関数を「単一 Lambda パッケージ」として一括実装**（Unit 境界はコード内で守るが、デプロイ単位は 1 つ）
- **D. AI-DLC 推奨に任せる**（→ A 推奨。実装は AI が並行するので C9 を最初に固めるのが安全）

[Answer]: A

---

### Technical Considerations（技術的考慮）

#### Q3. ChatGPT GPT Asset (C8) の Unit としての扱い

C8 は他 Unit と比べて極めて異質（コード無し、OpenAI 側で動く、リポジトリ同梱は System Prompt のみ）。Unit として独立扱いするか?

- **A. 独立した Unit (U8) として扱う**（成果物 = `assets/prompts/`、READMEへのリンク、テストはマニュアル確認）
- **B. C9 Infrastructure (U9) の一部に統合**（リポジトリアセット管理として）
- **C. Unit から外し、ドキュメント扱い**（Unit 一覧では参考言及のみ）
- **D. AI-DLC 推奨に任せる**（→ A 推奨。Iteration 4+ で着手するスケジュール上の独立性が必要）

[Answer]: C

---

### Code Organization（Greenfield コード組織）

#### Q4. リポジトリ構造

タコ中はモノリシックなリポジトリ（`tako-tues`）で開発する前提。コード組織は?

- **A. Component ごとにトップレベルディレクトリ**（`src/c1_auth/`, `src/c2_intake/`, ..., `src/c9_infra/`）
- **B. 機能タイプで分離**（`backend/`, `frontend/`, `infra/`, `assets/`）+ Component 単位はバックエンド配下のサブディレクトリ
- **C. 単一 Python パッケージ + フロントエンド分離**（`tako_chu/`, `frontend/`, `infra/`, `assets/`）+ Component 単位は `tako_chu/` 配下のモジュール
- **D. AI-DLC 推奨に任せる**（→ B または C のハイブリッド推奨。`backend/src/` 配下にレイヤード 4 層 × Component サブディレクトリ）

[Answer]: D

#### Q5. 共有コード（型定義 / 契約 / ユーティリティ）の配置

複数 Unit から参照される共有コード（DynamoDB スキーマ型、EventBridge イベントスキーマ、Pydantic モデル等）はどこに配置するか?

- **A. `backend/src/shared/`**（Python 共有モジュール、各 Component から import）
- **B. `assets/contracts/`**（JSON Schema / OpenAPI YAML / Pydantic 型定義を **言語非依存の契約**として集約）
- **C. A と B の両方**（型実装は A、宣言的契約は B）
- **D. AI-DLC 推奨に任せる**（→ C 推奨。実装言語に依存する型は backend/src/shared、言語横断の契約は assets/contracts）

[Answer]: D

---

## 3. 質問への回答後の流れ

1. ユーザーが上記 5 問の `[Answer]:` 行を埋めて返信
2. AI が曖昧性チェック
3. 曖昧性が解消されたら 3 つの成果物（unit-of-work.md / unit-of-work-dependency.md / unit-of-work-story-map.md）を一括生成
4. ユーザー承認 → CONSTRUCTION フェーズ（per-unit ループ）へ

---

## 4. 進行状況チェックリスト

- [x] Unit of Work Plan を作成（本ファイル）
- [x] ユーザーが 5 問に回答
- [x] 曖昧性レビュー＋必要に応じて follow-up 質問（Q3-α=A 解消）
- [x] 3 つの Unit 成果物を生成（unit-of-work / unit-of-work-dependency / unit-of-work-story-map）
- [x] aidlc-state.md / audit.md 更新
- [x] ユーザー承認（2026-04-29 / 2026-05-08 v1.1 で C8 削除反映）

---

## 5. 推奨案サマリー（迷ったらこれで OK）

- Q1=D（A+C ハイブリッド: 状態保持 Unit を主担当 + FR で確認）
- Q2=D（A: C9 完成 → 全 Unit 並行）
- Q3=D（A: U8 独立 Unit）
- Q4=D（B または C のハイブリッド: backend/src/ 配下に Component サブディレクトリ）
- Q5=D（C: 言語依存型は backend/src/shared、宣言的契約は assets/contracts）

「全部 D」または「推奨でお任せ」と書いていただければそのまま進めます。
