# AI-DLC State Tracking

## Project Information
- **Project Name**: タコ中 (たこちゅう)
- **Project Type**: Greenfield
- **Start Date**: 2026-04-29T00:00:00Z
- **Current Stage**: INCEPTION - Units Generation v1.0 ✅ **2026-04-29 ユーザー承認済み**（3 成果物 + 8 Unit）。**2026-05-08 v1.8 反映**: FR-6.3 ChatGPT GPT スコープ削除に伴い、Documentation (C8) セクションを廃止。**2026-05-08 v1.9 反映**: タコスの本質的魅力＝最終組み立てが必須な構造として定義、設計原則7「最終組み立てを奪わない」追加、FR-2.2.1 調理深度仕様（肉=調理済み / トルティーヤ=未加熱 / サルサ=別容器 / 完成形禁止）を新設、stories v1.5 / application-design v1.2 / components v1.2 へ波及反映。INCEPTION フェーズ完了状態は維持／次フェーズ: **CONSTRUCTION**
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

## Architectural Decisions (v1.9)
- **レシピ配信**: 生成 AI（Bedrock）動的生成を**廃止**し、リポジトリ同梱の静的 JSON（`assets/recipes/*.json`、手書き・キット種別 1:1）に変更（食中毒リスク回避）
- **Bedrock 用途**: FR-6.1 ダッシュボード煽り文 / FR-6.2 誘惑 Push のみ動的生成。フォールバック静的テンプレートあり
- ~~**ChatGPT カスタム GPT (FR-6.3)**~~ → **v1.8 で削除**: AWS 外サービス（OpenAI ChatGPT）依存を排除し、AWS サーバーレス内で完結させる方針に転換
- **タコスの本質的魅力定義（v1.9 追加）**: タコスは「完成品で届けられない唯一の料理」。ユーザーが手で組み立てる最終工程が物理的に省略できないという構造的特性を、本サービスの「作らせる」コンセプトの根拠として位置づける（設計原則7「最終組み立てを奪わない」）
- **配送内容の調理深度仕様（FR-2.2.1, v1.9 追加）**: 主肉=**調理済み真空パック**（湯煎/レンジで復温のみ） / トルティーヤ=**未加熱**（温め前） / サルサ・トッピング=**別容器分離** / **完成形のタコスは届けない**。Type A 想定所要時間 10〜15分。Type B 強制バリエーション週は「肉から仕込む」を上乗せ

## Stage Progress

### INCEPTION Phase
- [x] Workspace Detection (完了 - Greenfield と判定)
- [x] Reverse Engineering (スキップ - Greenfield のため)
- [x] Requirements Analysis (完了 - **2026-04-29 ユーザー承認済み**)
- [x] User Stories (Part 1 + Part 2 完了 - **2026-04-29 ユーザー承認済み**)
- [x] Workflow Planning (execution-plan.md v1.3 完了 - **2026-04-29 ユーザー承認済み**)
- [x] Application Design (5 成果物 v1.0 完了 - **2026-04-29 ユーザー承認済み**: components / component-methods / services / component-dependency / application-design)
- [x] Units Generation (3 成果物 v1.0 完了 - **2026-04-29 ユーザー承認済み**: unit-of-work / unit-of-work-dependency / unit-of-work-story-map / **8 Unit**。2026-05-08 v1.1 で Documentation セクション廃止)

### CONSTRUCTION Phase
- [ ] Per-Unit Loop
- [ ] Build and Test

### OPERATIONS Phase
- [ ] Operations (placeholder)
