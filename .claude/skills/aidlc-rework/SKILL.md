---
name: aidlc-rework
description: AI-DLC ワークフローで完了済みの stage に戻り、関連する下流 stage を含めて再実行したい時に使用。「やり直し」「手戻り」「前の stage に戻りたい」「rework」「rollback」「〇〇 stage からやり直し」など、明示的な手戻り指示があった時に必ずこのスキルを使う。影響する下流 stage を自動で特定し、成果物バックアップ・aidlc-state.md 書き戻し・audit.md 追記を一括実行する。
---

# aidlc-rework

AI-DLC ワークフローで途中まで進んだあと、特定 stage からやり直す必要が生じた時に手戻り操作を自動化するスキル。

## 発火条件

- ユーザーが **明示的に** rework / 手戻り / やり直し / rollback を要求した時のみ実行
- キーワード例: 「Requirements Analysis からやり直したい」「AppDesign に戻って」「Units Generation をもう一度」「前のフェーズを修正したい」
- 曖昧な場合は rework target の stage 名を確認してから実行する

---

## Step 1: 前提ファイル読込

以下を順番に読む（スキップ禁止）:

1. `.aidlc-rule-details/common/workflow-changes.md` — 手戻りの公式手順（全体参照）
2. `.aidlc-rule-details/common/process-overview.md` — stage 依存マップ（Mermaid 図: `:60-85`）
3. `aidlc-docs/aidlc-state.md` — 現在の stage 完了状態

---

## Step 2: rework target の特定

ユーザー入力から「どの stage に戻るか」を特定する。

**stage 一覧（正式名称）:**

| フェーズ | Stage 名 | aidlc-state.md 上の表示 |
|---------|---------|----------------------|
| INCEPTION | Workspace Detection | `Workspace Detection` |
| INCEPTION | Reverse Engineering | `Reverse Engineering` |
| INCEPTION | Requirements Analysis | `Requirements Analysis` |
| INCEPTION | User Stories | `User Stories` |
| INCEPTION | Workflow Planning | `Workflow Planning` |
| INCEPTION | Application Design | `Application Design` |
| INCEPTION | Units Generation | `Units Generation` |
| CONSTRUCTION | Functional Design | `Per-Unit Loop > {unit名} > Functional Design` |
| CONSTRUCTION | NFR Requirements | `Per-Unit Loop > {unit名} > NFR Requirements` |
| CONSTRUCTION | NFR Design | `Per-Unit Loop > {unit名} > NFR Design` |
| CONSTRUCTION | Infrastructure Design | `Per-Unit Loop > {unit名} > Infrastructure Design` |
| CONSTRUCTION | Code Generation | `Per-Unit Loop > {unit名} > Code Generation` |
| CONSTRUCTION | Build and Test | `Build and Test` |

CONSTRUCTION の stage は unit 名も特定すること（全 unit か、特定 unit か）。

**特定できない場合**: 完了済み stage の一覧を `aidlc-state.md` から読み取り、候補として提示してユーザーに確認する。

---

## Step 3: 影響範囲の算出（依存マップ）

rework target から下流に向かって影響する全 stage を列挙する。**「既に完了している（`[x]`）」 stage のみ対象**（未着手は無視）。

### INCEPTION 依存チェーン

rework target が以下の場合、対応する下流すべてが影響を受ける:

```
Requirements Analysis
  └→ User Stories（実行済みなら）
  └→ Workflow Planning
       └→ Application Design（実行済みなら）
            └→ Units Generation（実行済みなら）
                 └→ [全 unit] Functional Design → NFR Requirements → NFR Design
                    → Infrastructure Design → Code Generation
                 └→ Build and Test

User Stories
  └→ Workflow Planning → （以降上と同じ）

Workflow Planning
  └→ Application Design → Units Generation → [全 unit] per-unit設計 → CG → BT

Application Design
  └→ Units Generation → [全 unit] per-unit設計 → CG → BT

Units Generation
  └→ [全 unit] Functional Design → NFR Requirements → NFR Design
     → Infrastructure Design → Code Generation → Build and Test
```

### CONSTRUCTION 依存チェーン（per-unit）

```
Functional Design（unit X）
  └→ NFR Requirements（unit X）→ NFR Design（unit X）
     → Infrastructure Design（unit X）→ Code Generation（unit X）→ Build and Test

NFR Requirements（unit X）
  └→ NFR Design（unit X）→ Infrastructure Design（unit X）
     → Code Generation（unit X）→ Build and Test

NFR Design（unit X）
  └→ Infrastructure Design（unit X）→ Code Generation（unit X）→ Build and Test

Infrastructure Design（unit X）
  └→ Code Generation（unit X）→ Build and Test

Code Generation（unit X）
  └→ Build and Test
```

**全 unit が影響を受ける場合**: `aidlc-state.md` の unit 一覧を読み、全 unit の per-unit stage を対象に含める。

---

## Step 4: 成果物パスの特定

影響 stage ごとに、バックアップすべきファイルを `aidlc-docs/` を `ls` して実際に存在するものを確認する。

### 参照パス（典型例）

```
INCEPTION:
  Requirements Analysis:
    aidlc-docs/inception/requirements/
  User Stories:
    aidlc-docs/inception/user-stories/
  Workflow Planning:
    aidlc-docs/inception/plans/execution-plan.md
  Application Design:
    aidlc-docs/inception/application-design/
    aidlc-docs/inception/plans/application-design-plan.md
  Units Generation:
    aidlc-docs/inception/application-design/unit-of-work*.md
    aidlc-docs/inception/plans/unit-of-work-plan.md

CONSTRUCTION ({unit-name}):
  Functional Design:
    aidlc-docs/construction/{unit-name}/functional-design/
  NFR Requirements:
    aidlc-docs/construction/{unit-name}/nfr-requirements/
  NFR Design:
    aidlc-docs/construction/{unit-name}/nfr-design/
  Infrastructure Design:
    aidlc-docs/construction/{unit-name}/infrastructure-design/
  Code Generation:
    aidlc-docs/construction/{unit-name}/code/
  Build and Test:
    aidlc-docs/construction/build-and-test/
```

実際のパスは `ls` で確認し、存在しないパスはリストから除外する。

---

## Step 5: 実行プラン提示（dry-run）

ユーザーに以下を **表形式** で提示し、最終 go/no-go を求める（これだけが確認ステップ）:

```
## AI-DLC Rework 実行プラン（dry-run）

### rework target: [stage 名]
### 変更理由: [ユーザーの入力をそのまま引用]

#### 影響 stage（aidlc-state.md 書き戻し対象）
| Stage | 現在の状態 | 変更後 |
|-------|-----------|--------|
| [stage名] | [x] 完了 | [ ] リセット |
| ...   | ...       | ...    |

#### バックアップ対象ファイル
| 元のパス | バックアップ後のパス |
|---------|------------------|
| aidlc-docs/inception/requirements/requirements.md | aidlc-docs/inception/requirements/requirements.md.backup.[TIMESTAMP] |
| ...     | ...              |

#### Current Stage 書き換え
- 変更前: [現在の Current Stage 行]
- 変更後: INCEPTION - [target stage名] に戻り再実行

#### audit.md 追記プレビュー
[Change Request Log Format のプレビューを表示]

---
**上記の操作を実行しますか？（go / no-go）**
```

`no-go` の場合はスキルを終了。ファイルへの変更は一切行わない。

---

## Step 6: 自動実行（go 後）

ユーザーが `go` を選択したら、確認なしで以下を順番に実行する。

### 6-1. ファイルバックアップ

```
タイムスタンプ取得: ISO 8601 UTC 形式（例: 2026-05-08T12:34:56Z → 20260508T123456Z）

各対象ファイルを Bash でリネーム:
  mv {file} {file}.backup.{TIMESTAMP}

対象ディレクトリの場合はディレクトリごとリネーム:
  mv {dir}/ {dir}.backup.{TIMESTAMP}/
```

削除は禁止。バックアップファイルは必ず残す。

### 6-2. aidlc-state.md の書き戻し

1. `aidlc-docs/aidlc-state.md` を **Read** で読む
2. **Edit** で以下を書き換える（Write による上書き禁止）:
   - `### INCEPTION Phase` / `### CONSTRUCTION Phase` の該当 stage: `[x]` → `[ ]`
   - `**Current Stage**:` 行を rework target に書き換え

**書き換え例（Application Design が target の場合）:**
```
変更前:
- [x] Application Design (5 成果物 v1.0 完了...)
- [x] Units Generation (3 成果物 v1.0 完了...)

変更後:
- [ ] Application Design
- [ ] Units Generation

**Current Stage** 行:
変更前: INCEPTION - Units Generation v1.0 ✅ 完了 / 次フェーズ: CONSTRUCTION
変更後: INCEPTION - Application Design に戻り再実行
```

### 6-3. audit.md への追記

`aidlc-docs/audit.md` を **Read** で末尾を確認し、**Edit** で以下を末尾に追加する。

**Write による上書きは絶対禁止。**

追記フォーマット（`workflow-changes.md:262-272` 準拠）:

```markdown
## Change Request - [target stage 名]
**Timestamp**: [ISO 8601 UTC タイムスタンプ]
**Request**: [ユーザーの元のリクエスト原文]
**Current State**: [rework 前の Current Stage]
**Impact Assessment**: [影響する stage の一覧]
**User Confirmation**: go（ユーザーが実行プランに同意）
**Action Taken**: バックアップ作成・aidlc-state.md 書き戻し・audit.md 追記を完了
**Artifacts Affected**: [バックアップしたファイルパスの一覧]

---
```

---

## Step 7: 完了報告と再開ガイド

自動実行が完了したら以下を報告する:

```
## AI-DLC Rework 完了

✅ バックアップ: [N] 件
✅ aidlc-state.md: [M] stage をリセット
✅ audit.md: 追記完了

**次のアクション**: AI-DLC ワークフローを「[rework target stage]」から再開してください。
再開するには、通常の AI-DLC リクエスト（例: 「Application Design を再実行して」）を送ってください。
```

スキルはここで終了。AI-DLC ワークフローの再実行はメインワークフローが担当する。

---

## セーフティ規定

| ルール | 理由 |
|-------|------|
| `audit.md` は **Edit 追記のみ**（Write 上書き禁止） | CLAUDE.md の `### Correct Tool Usage for audit.md` 規約 |
| バックアップファイルは削除しない | rollback の rollback を可能にする |
| `aidlc-state.md` の編集は Read → Edit パターン | Write による全体上書きで他の情報が消えるのを防ぐ |
| dry-run 段階（Step 5）ではファイル変更ゼロ | go/no-go 前は read-only |
| タイムスタンプは ISO 8601 UTC 統一 | `audit.md` 既存エントリとの整合 |
| 1 回の go/no-go 確認（Step 5）だけは必須 | AI-DLC の USER CONTROL 原則 |
