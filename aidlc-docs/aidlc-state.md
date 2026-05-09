# AI-DLC State Tracking

## Project Information
- **Project Name**: タコ中 (たこちゅう)
- **Project Type**: Greenfield
- **Start Date**: 2026-04-29T00:00:00Z
- **Current Stage**: INCEPTION フェーズ完了 ✅ **2026-05-08**（rework v2.0 完了: RA v2.0 / US v2.0 / WP v2.0 / AppDesign 復元 / UnitsGen 復元）/ ドキュメント整備 **2026-05-09**（ストーリー数・コンポーネント数・残存参照の修正）／次フェーズ: **CONSTRUCTION**
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
- **Workspace Root**: github.com/salsa-panda/tako-tues（各開発者のローカルパスに依存しない）

## Code Location Rules
- **Application Code**: Workspace root (NEVER in aidlc-docs/)
- **Documentation**: aidlc-docs/ only
- **Structure patterns**: See code-generation.md Critical Rules

## Extension Configuration
| Extension | Enabled | Mode | Decided At |
|-----------|---------|------|------------|
| security-baseline | No | — | Requirements Analysis (Q14=B) |
| property-based-testing | Yes (Tentative) | Partial | Requirements Analysis (Q15 unanswered, AI-DLC 推奨で仮置き) |

## Architectural Decisions (v1.8)
- **レシピ配信**: 生成 AI（Bedrock）動的生成を**廃止**し、リポジトリ同梱の静的 JSON（`assets/recipes/*.json`、手書き・キット種別 1:1）に変更（食中毒リスク回避）
- **Bedrock 用途**: FR-6.1 ダッシュボード煽り文 / FR-6.2 誘惑 Push のみ動的生成。フォールバック静的テンプレートあり
- ~~**ChatGPT カスタム GPT (FR-6.3)**~~ → **v1.8 で削除**: AWS 外サービス（OpenAI ChatGPT）依存を排除し、AWS サーバーレス内で完結させる方針に転換

## Stage Progress

### INCEPTION Phase
- [x] Workspace Detection (完了 - Greenfield と判定)
- [x] Reverse Engineering (スキップ - Greenfield のため)
- [x] Requirements Analysis (完了 - **2026-05-08 v2.0 rework 承認済み** — Taco Tuesday 日本化 / 認知占有 / 強制主役 / 幸せなダメ を Concept Statement に明文化)
- [x] User Stories (Part 1+2 完了 - **2026-05-08 v2.0 rework 完了** — personas v2.0 / stories v2.0（US-T15 追加・16 ストーリー）/ story-board v2.0（§11 認知占有の体験弧追加）)
- [x] Workflow Planning (execution-plan.md v2.0 完了 - **2026-05-08 ユーザー承認済み**: AppDesign・UnitsGen はバックアップ復元、CONSTRUCTION へ直行)
- [x] Application Design (バックアップ v1.1 復元 - **2026-05-08**: 機能変化なし、rework v2.0 復元注記追加)
- [x] Units Generation (バックアップ v1.1 復元 - **2026-05-08**: 8 Unit 構成変化なし、rework v2.0 復元注記追加)

### CONSTRUCTION Phase
- [ ] Per-Unit Loop
- [ ] Build and Test

### OPERATIONS Phase
- [ ] Operations (placeholder)
