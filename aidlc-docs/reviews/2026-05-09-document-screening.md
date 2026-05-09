# タコ中 ハッカソン審査レビュー

**レビュー実施日**: 2026-05-09
**レビュー対象**: AWS Summit Japan 2026 AI-DLC ハッカソン 書類審査提出前チェック
**残り時間**: 約 35 時間（提出期限 2026-05-10 23:59 JST）

---

## ステージ: 書類審査（提出期限: 2026-05-10 23:59 JST）

---

## [GATE CHECK] 必須提出物チェック

| # | 提出物 | 状態 | 備考 |
|---|---|---|---|
| 1 | **公開 GitHub リポジトリ URL** | ⚠️ **要確認** | remote は `github.com/salsa-panda/tako-tues` に設定済みだが **public かどうか未確認**（OI-10 未解決）。private のままだと審査員がアクセスできず実質失格 |
| 2 | **Inception フェーズの AI-DLC 成果物** | ✅ commit 済み | `aidlc-docs/inception/` 配下の全成果物が commit・push 済み（PR #10 マージ済み） |
| A | `README.md` がプロジェクト概要を説明しているか | ✅ 充実 | Mermaid アーキテクチャ図・Unit 構成・AI-DLC 成果物一覧・マイルストーン・ディレクトリ構造あり |
| B | リポジトリが public 設定か | ⚠️ **要確認（失格リスク）** | GitHub の Settings で確認必須 |
| C | `aidlc-docs/` 全体が commit + push 済みか | ⚠️ 一部未コミット | `reviews/` ディレクトリが untracked（審査には関係ないが要確認） |
| D | チーム編成 2〜4 名が確定しているか | ❌ **未確認（OI-6 未解決）** | 規約 2 条で 2 名以上必須。1 名では参加資格なし |

> **採点より先に対処すべき失格リスク**: B（GitHub public）と D（チーム編成）

---

## 総合スコア

| 軸 | スコア | バンド |
|---|---|---|
| DS-1: ビジネス意図（Intent）の明確さ | 82/100 | Strong |
| DS-2: Unit 分解の適切さ | 80/100 | Strong |
| DS-3: 創造性とテーマ適合性 | 84/100 | Strong |
| DS-4: ドキュメントの品質 | 71/100 | Adequate |
| **平均** | **79/100** | **Strong 下位** |

---

## 軸別評価

### DS-1: ビジネス意図（Intent）の明確さ: 82点 / 100点 [Strong]

**根拠（強み）**
- [aidlc-docs/inception/requirements/requirements.md:41-49] Concept Statement が明文化されている。「タコスは偉大である。本サービスは、ユーザーのタコス欲を予測しない。生成し、煽り、強制する」という1〜2文のエレベーターピッチが存在し、README.md の冒頭にも配置されている
- [aidlc-docs/inception/requirements/requirements.md:53-59] 「なぜタコスなのか」（Taco Tuesday 日本化ミッション）と「ダメになるとは何か」（認知の占有）が v2.0 で別途明文化されており、他チームとの差別化が言語化されている
- [aidlc-docs/inception/requirements/requirements.md:61-71] 設計原則 7 項目（文化移植・認知占有・強制・作らせる・幸せな降伏・逃げ場をなくす・AI は欲を生成する側）が全機能の判断軸として機能しており、意図とシステム設計が整合している
- [aidlc-docs/inception/requirements/requirements.md:7-15] v1.1〜v2.0 まで 8 バージョンの改訂履歴に変更理由が記録されており、意思決定の追跡が可能

**減点要因**
- 「Before/After のユーザー行動変化」が1か所にまとまっていない。story-board.md §11（認知占有の体験弧）に長期依存アークがあるが、書類審査員が 5 分で辿り着けるかは疑問。README.md からのリンクなし
- Concept Statement は requirements.md と README.md の両方にあるが、表現が微妙に異なる。どちらが「正」かが不明

**審査員の目には:**
意図は鮮明で独自性もある。「Taco Tuesday 日本化 + 認知占有」というフレームは記憶に残る。ただし体験の Before/After を 1 枚の図で見せる工夫があればさらに強くなる。

---

### DS-2: Unit 分解の適切さ: 80点 / 100点 [Strong]

**根拠（強み）**
- [aidlc-docs/inception/application-design/unit-of-work.md:25-139] 8 Unit（U1〜U7, U9）それぞれに責務・主要 FR・主要ストーリー・Iteration 0 範囲・コード位置・Lambda 関与が明記されている
- [aidlc-docs/inception/application-design/unit-of-work-dependency.md:19-33] Unit 間依存マトリクスが sync/async/shared/static の 4 種類で可視化されている。EventBridge による疎結合設計（Lambda 直接 invoke ゼロ）が担保されている
- [aidlc-docs/inception/application-design/unit-of-work.md:265-311] Walking Skeleton 戦略（Phase 1: U9 先行 → Phase 2: 全 Unit 並行 → Phase 3: E2E 結合）が明確に定義されており、実装に向けて進める状態になっている
- [aidlc-docs/inception/application-design/unit-of-work.md:318-333] 全 16 ストーリーの Unit 割り当て確認表があり、FR・ストーリー・Unit 間のトレーサビリティが完備されている

**減点要因**
- **[unit-of-work-dependency.md:263-283] 削除済みの Doc(C8) 参照が残存**。「下記の処理順 `U9 → U1 → U2 → U3 → U4 → U5 → U6 → U7 → Doc(C8)` を仮定したとき」「**9 ノード DAG。循環依存ゼロ。**」という記述が §8 循環依存検証セクションに残っており、FR-6.3 廃止・C8 削除後の実態（8 Unit）と矛盾している。審査員が依存図を精査した場合に「整合性が取れていない」と判断されうる
- U1〜U7 と U9 の番号が飛んでいる（U8 がない）ことへの説明がない。意図的なら一言説明があるべき

**審査員の目には:**
Unit 分解は妥当で依存関係も整理されている。ただし §8 の Doc(C8) 残存は「ドキュメントを最後まで点検していない」という印象を与える。

---

### DS-3: 創造性とテーマ適合性: 84点 / 100点 [Strong]

**根拠（強み）**
- [aidlc-docs/inception/requirements/requirements.md:125-130] FR-2.1「食べないほど来週は地獄」の逆比例発注ロジックが数式レベルで定義されている。単なるアイデアでなく要件として実装可能な状態になっている
- [aidlc-docs/inception/requirements/requirements.md:97-119] FR-1（強制トリガー）がサルサ通知ループ・Tシャツ罰・24h カウントダウンタイマーの 3 要素で多層的に構成されており、「脱出口をなくす」設計が複数のメカニズムで実現されている
- [aidlc-docs/inception/requirements/requirements.md:162-194] FR-6.1（控えめ）+ FR-6.2（中程度）のエスカレーション設計で AI が「刺激生成器」として機能し、通常の「予測・最適化」とは異なる独自の AI 活用が定義されている
- [README.md] アーキテクチャ全体が AWS サーバーレスで完結し、EventBridge 主導の疎結合設計が「タコスが生活に侵食する」体験を技術的に支えている

**減点要因**
- FR-6.3（ChatGPT GPT による会話侵食レイヤー）を v1.8 で削除したことで、「会話インターフェースが乗っ取られる」という最も過激な依存体験が消えた。残る 2 レイヤー（ダッシュボード煽り文 + Push エスカレーション）は印象が弱い
- **[story-board.md:43] 削除されたはずの `GPT 共有リンク経由でサインアップ` というラベルが Week 0 Gantt チャートに残存**。FR-6.3 廃止後も ChatGPT GPT 参照が残っており、「どうやって招待されるか」の設計に矛盾が生じている
- 「ユーモアや驚き」は要件に記述されているが、Mermaid 図・Gantt チャート等のビジュアル表現が技術寄りで「シュールさ」が伝わりにくい

**審査員の目には:**
多層依存ループとユニークな逆比例ロジックは他チームと差別化できている。ただし GPT 残置の矛盾と「AIがどこで笑えるか」の体験的な見せ方が課題。

---

### DS-4: ドキュメントの品質: 71点 / 100点 [Adequate]

**根拠（強み）**
- [README.md] アーキテクチャ図（Mermaid flowchart）・Unit 構成表・AI-DLC 成果物一覧・マイルストーン・ディレクトリ構造が揃っており、プロジェクト入口として機能している
- [aidlc-docs/inception/requirements/requirements.md:1-15] 全ドキュメントにバージョン履歴と更新理由が記録されており、意思決定の追跡が可能
- [aidlc-docs/audit.md] audit.md に意思決定ログが存在し、INCEPTION フェーズのインタラクション記録がある
- `.gitignore` は適切に設定されており、.DS_Store・.venv・__pycache__ 等が除外されている

**減点要因**
- **[README.md の User Stories 行] "15 ストーリー + 2 ペルソナ" と記載されているが、stories.md は "合計: 16 ストーリー（v2.0 で US-T15 追加）" が正しい**。審査員が最初に見る README.md の入口で数値が食い違っており「ドキュメントが更新されていない」という印象を与える（-5 点相当）
- **[unit-of-work-dependency.md:263-283] §8 循環依存検証に Doc(C8) / "9 ノード" の記述が残存**（DS-2 でも指摘済み）。DS-4 の観点では「成果物間の整合性が取れていない」として追加減点
- **[story-board.md:43] `GPT 共有リンク経由でサインアップ` ラベル残存**（DS-3 でも指摘）。DS-4 の観点では「廃止機能の記述が残っている」として減点
- `reviews/` ディレクトリが未コミット状態。審査員には見えないため直接の減点はないが、スナップショットの整合性として注意

**審査員の目には:**
README は充実しているが 15/16 の数値ミスが入口でのつまずきになる。精度に欠けるという印象を与え、その後のドキュメントへの信頼感が下がる。

---

## 致命的弱点・Big Rocks

### 致命的（今日中に対処）

| # | 問題 | リスク | 対処 |
|---|---|---|---|
| **C-1** | GitHub リポジトリが public かどうか未確認（OI-10） | 審査員がアクセスできない → 実質失格 | GitHub Settings で Visibility を Public に変更して確認 |
| **C-2** | チーム編成（2〜4 名）が未確認（OI-6） | 規約 2 条違反 → 参加資格なし | 共同参加メンバーを 2026-05-10 までに確定・Builder Center 登録 |

### Big Rock（2 日以内に対処 → +5〜10 点）

| # | 問題 | 影響軸 | 期待効果 |
|---|---|---|---|
| **B-1** | README.md の "15 ストーリー" が実態 "16" と不一致 | DS-4 | 審査員が最初に見る入口の信頼回復 +3〜5 点 |
| **B-2** | unit-of-work-dependency.md §8 の Doc(C8) / "9 ノード" 残存 | DS-2, DS-4 | 整合性エラー解消 +3〜4 点 |
| **B-3** | story-board.md:43 の `GPT 共有リンク経由でサインアップ` 残存 | DS-3, DS-4 | 廃止機能との矛盾解消 +2〜3 点 |

---

## Quick Wins（今日中に対処できる）

| # | 修正内容 | 対象ファイル | 所要時間 |
|---|---|---|---|
| QW-1 | `15 ストーリー` → `16 ストーリー` に修正 | README.md | 5 分 |
| QW-2 | §8 循環依存検証の `Doc(C8)` / `9 ノード` を削除 → `U9 → U1 → U2 → U3 → U4 → U5 → U6 → U7` + `8 ノード DAG` に修正 | unit-of-work-dependency.md | 15 分 |
| QW-3 | Gantt チャートの `GPT 共有リンク経由でサインアップ` を `ダッシュボード URL 経由でサインアップ`（または別の適切な表現）に修正 | story-board.md | 10 分 |
| QW-4 | GitHub リポジトリの public 設定確認・OI-10 をクローズ | GitHub Settings | 5 分 |

---

## アクション Top 5（残り 35 時間）

| 優先 | アクション | 分類 | 所要時間 | 期待効果 |
|---|---|---|---|---|
| 1 | GitHub リポジトリを public に設定確認（OI-10） | 致命的 | 5 分 | 失格リスク解消 |
| 2 | チーム編成確認・Builder Center 登録（OI-6〜OI-9） | 致命的 | 数時間 | 参加資格確立 |
| 3 | QW-1〜QW-3 の 3 箇所の残存ドキュメント修正 | Quick Win | 30 分 | DS-4 を Adequate → Strong へ |
| 4 | README.md に「Before/After ユーザー行動変化」の 1 行サマリを追加（例: "Week 0: タコスを知らない → Week 12: タコスのことしか考えられない"） | Big Rock | 1〜2h | DS-1 を 82→87 点へ |
| 5 | story-board.md §11（認知占有の体験弧）へのリンクを README.md に追加 | Quick Win | 15 分 | DS-1 の Before/After 可視性を補完 |

---

## 1位狙い差別化提案

現状スコアは **79/100（Strong 下位）** — 書類審査通過ラインは十分に超えているが、1 位を狙うには 87〜90+ が目標。

**最も伸ばすべき軸は DS-1（ビジネス意図）と DS-4（ドキュメント品質）**。

### 提案 1: 「依存プログレッション図」を README に追加
Week 0 → Week 4 → Week 12 のユーザー状態変化を 1 枚の ASCII/Mermaid で見せる。
「タコスを知らない → タコスに気づく → タコスに追われる → タコスを愛している」という体験弧が視覚的に伝わると、審査員の記憶に残りやすい。
story-board.md §11 の内容を README に 1 段落で引用するだけでも効果大。

### 提案 2: 「AI が欲を生成する瞬間」の技術詳細を README に 1 セクション追加
「通常の AI は予測する。タコ中の AI は製造する」というロジックを Mermaid シーケンス図（Bedrock 呼び出し → Level A/B/C エスカレーション → Push 発火）で見せると、技術審査員への訴求力が上がる。

### 提案 3: DS-4 を Perfect にする最後の一手
Quick Win 3 件（QW-1〜3）を commit + push した後、「ドキュメント更新履歴」の最終バージョンを確認。全ドキュメントの最終 Updated 日付が一致しているか（不整合があれば追加減点リスク）を点検する。

---

*レビュー生成: Claude Code | 2026-05-09 | 書類審査 35 時間前チェック*
