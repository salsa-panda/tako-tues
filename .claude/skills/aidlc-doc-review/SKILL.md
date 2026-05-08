---
name: aidlc-doc-review
description: AI-DLC プロジェクト（タコ中）の aidlc-docs/ 配下のドキュメント不備を網羅的に洗い出し、修正提案つきの markdown レポートを生成する。「ドキュメントの不備を洗い出して」「ドキュメントをレビューして」「aidlc-docs を点検」「書類審査前にチェック」「doc review」「不整合チェック」などに反応する。承認済みなのに残っているチェックボックス、バージョン参照の不一致、廃止ルールの残置、Unit 数・ストーリー数の食い違い、FR 番号の取り違え、メタ情報欠落などを観点1〜14で網羅検出する。レポート生成のみで、修正は実行しない（修正はユーザー指示後に別途）。
---

# aidlc-doc-review

AI-DLC ワークフローで生成された `aidlc-docs/` 配下のドキュメントについて、ステージ承認後に残ってしまった不備を一括検出し、修正提案つきの markdown レポートを生成するスキル。

書類審査・予選・決勝などのマイルストーン前のセルフチェックを想定。修正そのものは行わず、レポート出力のみ。

## 発火条件

ユーザーが以下のような要求をした時に実行：

- 「ドキュメントの不備を洗い出して」「不備をリスト化」
- 「aidlc-docs をレビュー」「ドキュメントを点検」
- 「書類審査前にチェック」「審査前のセルフチェック」
- 「doc review」「unit 数の整合性チェック」「FR 参照の不整合」
- スラッシュコマンド `/aidlc-doc-review` 経由

修正実行までは要求されていない場合のデフォルト動作。「修正もして」と明示された場合は、レポート生成完了後に別途確認してから着手すること（このスキルの範囲外）。

---

## Step 1: 前提ファイル読込

以下を順番に読む（スキップ禁止）。読まないと不備の判定基準（INCEPTION 完了済みか等）が立てられない：

1. `aidlc-docs/aidlc-state.md` — フェーズ進捗・承認日・公式バージョン番号
2. `aidlc-docs/audit.md` — 承認履歴・各ステージの最終承認バージョン

特に注目するメタ情報：
- 各ステージの承認状況（INCEPTION 完了済 / CONSTRUCTION 着手済 など）
- 公式バージョン番号: requirements / stories / story-board / personas / application-design / unit-of-work
- Unit 数・ストーリー数の「公式値」（aidlc-state.md の記述が正）

---

## Step 2: 出力先の確認

レポートをどこに出力するかをユーザーに確認する。AskUserQuestion を 1 問だけ送る：

- **質問**: レポートの出力先はどこにしますか？
- **選択肢**:
  - `~/Desktop/<プロジェクト名>-doc-review-<YYYY-MM-DD>.md`（永続）
  - `/tmp/<プロジェクト名>-doc-review-<YYYY-MM-DD>.md`（一時）
  - `aidlc-docs/doc-review-<YYYY-MM-DD>.md`（リポジトリ内、共有用）

ユーザーが Other で独自パスを指定した場合はそれに従う。worktree 内に書くのは原則避ける（worktree 削除で失われるため）。

---

## Step 3: 対象ファイルの列挙

`find aidlc-docs -type f -name "*.md" | sort` で全 markdown ファイルを列挙。`wc -l` で行数を把握。

通常は以下の 22 ファイルが対象（プロジェクト進行に応じて増減）：

- aidlc-state.md, audit.md
- inception/plans/ 配下（execution-plan / application-design-plan / story-generation-plan / story-generation-clarification-questions / unit-of-work-plan / user-stories-assessment）
- inception/requirements/ 配下（requirements / requirement-verification-questions）
- inception/user-stories/ 配下（personas / stories / story-board）
- inception/application-design/ 配下（application-design / components / component-methods / component-dependency / services / unit-of-work / unit-of-work-dependency / unit-of-work-story-map）
- construction/, operations/ 配下（あれば）

---

## Step 4: 観点別の事前 grep スキャン

明示的なマーカーは grep で先に拾い、Step 5 のエージェント精読の前提にする：

```bash
# 観点1: 未チェックのチェックボックス
grep -rn "^\s*-\s*\[\s\]\|^\s*\*\s*\[\s\]\|^\s*[0-9]\+\.\s*\[\s\]" aidlc-docs/

# 観点1: TODO/FIXME/TBD/XXX/HACK（"Hackathon" 偽陽性は除外して目視）
grep -rni "TODO\|FIXME\|TBD\|XXX\|HACK" aidlc-docs/

# 観点3: プレースホルダ
grep -rn "<placeholder>\|<TBD>\|<TODO>\|\[ここに記述\]\|要記入\|未定\|後で書く\|???" aidlc-docs/
```

CONSTRUCTION 配下の `[ ]` は未着手を示すので「不備ではない」（INCEPTION 完了済みの場合）。これを判定軸として明確に持つこと。

---

## Step 5: 並列エージェントによる精読

ファイルを 2 グループに分け、`general-purpose` エージェントを **並列**（同一メッセージ内で 2 回 Agent ツールコール）で起動する：

### Group A: ステート / プラン / 要件
- aidlc-state.md, audit.md
- inception/plans/ 全ファイル
- inception/requirements/ 全ファイル

### Group B: ユーザーストーリー / 設計成果物
- inception/user-stories/ 全ファイル
- inception/application-design/ 全ファイル

各エージェントへの指示には **必ず以下を含める**：

1. **コンテキスト**:
   - aidlc-state.md の現在ステータス（INCEPTION 完了済か等）
   - 公式バージョン番号
   - 「8 Unit + Documentation 1」のような確定済の重要事実
2. **検出する 14 観点（後述）**
3. **返却形式**（JSON 風で file/line/severity/category/finding/evidence/suggestion）
4. **クロスファイル整合性の重点項目**: 8 vs 9 Unit、15 vs 17 ストーリー、FR-2.3/2.4、ペルソナ ID、Unit ID、バージョン番号

エージェントが Top 5 重大不備の要約も返すように指示すると、後段のサマリ生成が楽になる。

---

## Step 6: 検出すべき不備の観点（14 観点）

| # | 観点 | 検出例 |
|---|------|--------|
| 1 | 未チェックのチェックボックス（承認済なのに [ ]） | プランファイルの Step A〜G が全て [ ] |
| 2 | TODO/FIXME/TBD/XXX/HACK マーカー | "Hackathon" のような偽陽性は除外 |
| 3 | プレースホルダ（`<TBD>`, `???`, 要記入, 未定, 後で書く） | Q15 の Answer が「よくわからないので後で教えてください」 |
| 4 | 書きかけ（見出しだけ・箇条書きマーカーだけ・空コードブロック・空テーブル行） | — |
| 5 | 「...」「省略」「以下同様」での打ち切り | — |
| 6 | 参照リンク・バージョン番号の不一致 | execution-plan が requirements v1.6 を参照（実体 v1.7） |
| 7 | 必須セクション欠落（Status/Updated/Approved） | application-design 配下のメタ情報欠落 |
| 8 | 見出し階層の崩れ | `## 7. Success Criteria` 配下が `### 6.1` |
| 9 | Mermaid / ASCII 図のシンタックスエラー | — |
| 10 | メタ情報の不備（古い Phase 表記、未承認文言の残置） | `Phase: INCEPTION - Workflow Planning` のまま |
| 11 | 状態管理の不整合（aidlc-state.md と実ファイルの食い違い） | エントリ順が時系列でない |
| 12 | audit.md と他ファイルのバージョン番号・承認記録の不一致 | — |
| 13 | 数値・固有名詞の食い違い（クロスファイル） | 8 vs 9 Unit、15 vs 17 ストーリー、FR-2.3 vs FR-2.4 |
| 14 | チェックボックスが [x] なのに本文・成果物が未記入 | — |

---

## Step 7: レポート生成

検出結果を 1 つの markdown ファイルにまとめる。**構成は厳守**：

```
# <プロジェクト名> ドキュメント不備レビューレポート

- 対象: aidlc-docs/ 配下 全 N ファイル
- 生成日: <YYYY-MM-DD>
- 基準: aidlc-state.md による現在ステータス記述

## エグゼクティブサマリ
（重大な系統 3〜5 個を表で）

### 重大不備 Top N（着手順）
（ファイル・行番号・重要度つきの表）

## 不備分類（観点別の件数集計）
（14 観点別の件数）

## ファイル別 詳細不備リスト
### 🗂 N. <ファイルパス>
（行 / 重要度 / 観点 / 不備 / 修正提案 のテーブル）

## 修正の進め方（推奨手順）
（横断的影響が大きい順）

## 補足：不備に該当しない（ただし要注意）の項目

## 集計サマリ
- 🔴 High: N 件
- 🟡 Medium: N 件
- 🟢 Low: N 件
- 合計: N 件
（マイルストーン日付があれば残日数も併記）
```

### 重要度の基準

- 🔴 **High**: Construction 着手前に必ず修正すべき。FR 番号取り違え・Unit 数の食い違い・廃止ルールの UI 残置・承認後の Phase ヘッダ未更新など、後続フェーズや審査資料に直接悪影響するもの
- 🟡 **Medium**: 書類審査前に修正推奨。バージョン参照の古さ・メタ情報欠落・15 vs 17 ストーリー数表記など
- 🟢 **Low**: 時間があれば修正。タイポ、注記の補足、未着手チェックリストへの用途明示など

---

## Step 8: ユーザーへの通知

レポート生成完了後、以下を 1 メッセージでまとめて伝える：

- 出力先のフルパス
- 検出件数（High / Medium / Low / 合計）
- 重大な系統 3 つの要約
- マイルストーン日付があれば残日数（aidlc-state.md の Critical Deadlines を参照）
- **「修正を進める場合は声かけてください」**と明示し、勝手に修正を始めない

---

## やってはいけないこと

- ❌ レポート生成と同時に aidlc-docs/ 配下のファイルを編集する（ユーザーが明示要求しない限り修正は別タスク）
- ❌ worktree 内（`.claude/worktrees/.../`）にレポートを書く（worktree 削除で消える）
- ❌ CONSTRUCTION 配下の `[ ]` を「不備」として報告する（INCEPTION 完了直後の状態では未着手で正常）
- ❌ aidlc-state.md と実ファイルのどちらが「正」かをユーザーに確認せず判断する → 判断に迷う場合はユーザーに確認
- ❌ Step 1 をスキップする（読まずに「INCEPTION 完了済」を仮定すると判定基準が壊れる）

---

## 過去の実行例（参考: 2026-05-08）

- 対象 22 ファイル
- 検出 44 件（High 17 / Medium 18 / Low 9）
- 重大系統:
  1. **Unit 数の二重管理**（8 Unit + Doc 1 vs 9 Unit）
  2. **FR-2.3 / 2.4 取り違え**（stories.md US-T04・US-M02、unit-of-work-story-map.md）
  3. **廃止 72h ルール残置**（story-board.md の UI コピー 3 箇所）
- 出力先: `~/Desktop/tako-tues-doc-review-report.md`

このスキルが今後発見すべき類似パターン:
- ステージ承認後の Phase ヘッダ未更新
- プランファイルのチェックボックス全 [ ] 残置
- バージョン番号の伝播漏れ（v1.6 → v1.7 化など）
- 廃止された仕様（72h ルールなど）の UI コピー・ASCII 図への残置
- Unit / Component / Story の ID 参照不整合
