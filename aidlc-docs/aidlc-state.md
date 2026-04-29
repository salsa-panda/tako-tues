# AI-DLC State Tracking

## Project Information
- **Project Name**: タコ中 (たこちゅう)
- **Project Type**: Greenfield
- **Start Date**: 2026-04-29T00:00:00Z
- **Current Stage**: INCEPTION - Workflow Planning v1.3 ✅ **2026-04-29 ユーザー承認済み**（要件 v1.7 + Stories v1.3 + Story-Board v1.3 + execution-plan.md v1.3 / Walking Skeleton ファースト + 反復モデル）／次ステージ: **Application Design**
- **Hackathon**: AWS Summit Japan 2026 AI-DLC ハッカソン応募作品
- **Critical Deadlines**:
  - 書類審査エントリー: 2026-05-10 23:59 JST
  - 予選（対面 麻布台ヒルズ）: 2026-05-30
  - 決勝（対面 幕張メッセ AWS Summit Japan）: 2026-06-26

## Workspace State
- **Existing Code**: No
- **Programming Languages**: N/A
- **Build System**: N/A
- **Project Structure**: Empty
- **Reverse Engineering Needed**: No
- **Workspace Root**: /Users/tsubasa/git/github.com/tsubasaxZZZ/tako-tues

## Code Location Rules
- **Application Code**: Workspace root (NEVER in aidlc-docs/)
- **Documentation**: aidlc-docs/ only
- **Structure patterns**: See code-generation.md Critical Rules

## Extension Configuration
| Extension | Enabled | Mode | Decided At |
|-----------|---------|------|------------|
| security-baseline | No | — | Requirements Analysis (Q14=B) |
| property-based-testing | Yes (Tentative) | Partial | Requirements Analysis (Q15 unanswered, AI-DLC 推奨で仮置き) |

## Architectural Decisions (v1.7)
- **レシピ配信**: 生成 AI（Bedrock）動的生成を**廃止**し、リポジトリ同梱の静的 JSON（`assets/recipes/*.json`、手書き・キット種別 1:1）に変更（食中毒リスク回避）
- **Bedrock 用途**: FR-6.1 ダッシュボード煽り文 / FR-6.2 誘惑 Push のみ動的生成。フォールバック静的テンプレートあり
- **ChatGPT カスタム GPT (FR-6.3)**: 動的 AI 体験のメインタッチポイント、System Prompt のみで状態管理

## Stage Progress

### INCEPTION Phase
- [x] Workspace Detection (完了 - Greenfield と判定)
- [x] Reverse Engineering (スキップ - Greenfield のため)
- [x] Requirements Analysis (完了 - **2026-04-29 ユーザー承認済み**)
- [x] User Stories (Part 1 + Part 2 完了 - **2026-04-29 ユーザー承認済み**)
- [x] Workflow Planning (execution-plan.md v1.3 完了 - **2026-04-29 ユーザー承認済み**)
- [ ] Application Design (EXECUTE 予定 - 共有契約確定が中心：DynamoDB スキーマ / EventBridge イベント / REST API / Web Push ペイロード / レシピ JSON Schema)
- [ ] Units Generation (EXECUTE 予定 - 審査基準上必須化、9 Unit 暫定)

### CONSTRUCTION Phase
- [ ] Per-Unit Loop
- [ ] Build and Test

### OPERATIONS Phase
- [ ] Operations (placeholder)
