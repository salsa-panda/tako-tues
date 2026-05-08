---
name: aidlc-hackathon-review
description: Use when reviewing the tako-tues hackathon project against AWS Summit Japan 2026 AI-DLC hackathon judging criteria (document screening, preliminary, final) — produces no-flattery scores per axis with file:line evidence and a prioritized improvement list aimed at winning 1st place.
---

# aidlc-hackathon-review

AWS Summit Japan 2026 AI-DLC ハッカソン（テーマ「人をダメにするサービス」）の審査基準に従い、タコ中（tako-tues）プロジェクトの成果物を忖度なしで採点し、1 位獲得に向けた具体的な改善アクションを出力するスキル。

## When to Use

- INCEPTION フェーズ完了後、書類審査提出前
- 書類審査通過後、予選のデモ・プレゼン準備前
- 予選後、決勝の改善版提出前
- 大きな設計変更・実装完了のマイルストーン時

## Before You Start

<HARD-GATE>
このスキルは「1 位を狙う」ために存在する。褒めることが目的ではない。
根拠なしの高得点を付けてはいけない。
採点者（審査員）の視点から見て「弱い」と映る部分は弱いと言う。
ユーザーへの忖度は 0。
</HARD-GATE>

## Process

### Step 1: ステージ判定

`aidlc-docs/aidlc-state.md` を読み、当日日付と提出期限から直近の審査ステージを判定する。

| 期限 | ステージ | 評価軸 |
|---|---|---|
| 2026-05-10 23:59 JST 以前 | 書類審査 | Intent / Unit分解 / 創造性 / ドキュメント品質 |
| 2026-05-30 以前 | 予選 | MVPデモ完成度 / AI-DLCプロセス / 創造性 / プレゼン |
| 2026-06-26 以前 | 決勝 | デモ完成度 / AI-DLCプロセス / 創造性×ビジネス価値 / プレゼン |

### Step 2: 必須提出物チェック（失格リスクを先に排除）

`submission-checklist.md`（このスキルと同フォルダ）を読み込み、現在のステージで
規約上「必須」とされている提出物が揃っているかをチェックする。
**未充足があれば採点より先に報告する。**

### Step 3: 採点（0-100 点 × 4 軸）

`rubric.md`（このスキルと同フォルダ）から採点基準を読み込み、各軸を採点する。

**各軸の出力形式（必須）:**
```
### [軸名]: XX点 / 100点  [バンド]

**根拠（強み）**
- [aidlc-docs/xxx.md:NN] 引用テキスト — なぜ評価されるか
- [aidlc-docs/xxx.md:NN] ...
- [aidlc-docs/xxx.md:NN] ...

**減点要因**
- 具体的な不足点（「〜がない」「〜が弱い」と明示）

**審査員の目には:**
（審査員から1文でどう見えるか）
```

`file:line` 形式の根拠引用は各軸 **最低 3 個**。引用できなければ「根拠不十分」として
採点保留にし、ユーザーにファイルの確認を促す。

### Step 4: 弱点の分類

採点で見つかった弱点を以下の 4 分類に整理する:

| 分類 | 定義 | 対応方針 |
|---|---|---|
| **致命的** | 未対処なら失格・書類落ちリスク | 今日中に対処 |
| **Big Rock** | 対処すれば +5〜10 点の大きな改善 | 2日以内 |
| **Quick Win** | 1時間以内に対処でき +2〜3 点 | 今日のうちに |
| **軽微** | あれば良いが必須ではない | 予選前に検討 |

### Step 5: アクション Top 5（残り日数から逆算）

残り日数を算出し、現実的に対処できる改善リストを優先順位・所要時間目安・種別付きで出す。

```
| 優先 | アクション | 分類 | 所要時間目安 | 期待効果 |
|---|---|---|---|---|
| 1 | ... | ... | ...h | ... |
```

## Output Format

実行後は以下のフォーマットで出力する（リポジトリ内 `aidlc-docs/reviews/YYYY-MM-DD-<stage>.md` にも保存）:

```
## ステージ: [書類審査 / 予選 / 決勝]（提出期限: YYYY-MM-DD）

---

### [GATE CHECK] 必須提出物チェック
- [ ] 提出物1
- [ ] 提出物2
（未充足があれば ❌ を付けて採点前に報告）

---

## 総合スコア

| 軸 | スコア | バンド |
|---|---|---|
| 軸1 | XX/100 | Strong |
| 軸2 | XX/100 | Adequate |
| 軸3 | XX/100 | Strong |
| 軸4 | XX/100 | Weak |
| **平均** | **XX/100** | - |

---

## 軸別評価
（各軸の詳細）

---

## 致命的弱点・Big Rocks
（優先対処が必要な項目）

---

## Quick Wins
（今日中に対処できる項目）

---

## アクション Top 5（残りN日）
| 優先 | アクション | 分類 | 所要時間 | 期待効果 |
|---|---|---|---|---|

---

## 1位狙い差別化提案
（他チームと差別化するために伸ばすべき軸の提言）
```

## Anti-Patterns（これをやると無効）

- 「全体的には良いドキュメント」など曖昧な総評で濁す
- 根拠引用（file:line）のない採点（= 感想にすぎない）
- 1 位狙いという文脈を忘れて「書類通過できれば十分」な採点をする
- 必須提出物（公開 GitHub URL 等）のチェック漏れ
- Open Items（未解決の課題）を「あとで良い」と軽視する

## 保存先

レビュー結果は会話出力と同時に以下のパスで保存する:
```
aidlc-docs/reviews/YYYY-MM-DD-<stage>.md
```

例:
- `aidlc-docs/reviews/2026-05-08-document-screening.md`
- `aidlc-docs/reviews/2026-05-28-preliminary.md`
- `aidlc-docs/reviews/2026-06-24-final.md`
