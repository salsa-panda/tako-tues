# Story Generation Plan — タコ中 v2.0 rework

**Project**: タコ中（たこちゅう）
**Plan Version**: 2.0（rework）
**Based On**: requirements.md v2.0
**Approach**: Persona × Journey ハイブリッド（既存 15 ストーリーの framing 更新 + 必要に応じて新規追加）
**Created**: 2026-05-08

---

## rework 方針

requirements.md v2.0 で確定した 5 つの核心:
1. **文化移植ミッション**: Taco Tuesday を日本に根付かせる
2. **認知の占有**: タコスが「頭の中に住み着く」状態がエンドステート
3. **強制が主役**: 外圧でコントロールを奪う（コントロール喪失の笑い）
4. **幸せなダメになり方**: 依存は苦痛でなく快感（「最高じゃないですか」）
5. **タコスの必然性を明示**: Taco Tuesday 文化 + 組み立て儀式が根拠

これらをペルソナ・ストーリー・ストーリーボードに一貫して反映する。

---

## Clarification Questions

以下の質問に [Answer]: タグで回答してください。回答後、PART 2（生成）を実行します。

---

### Question 1
**タロウのペルソナ更新の深度**

タロウ（コア生活侵食体験者）のペルソナ記述をどこまで更新しますか？

A) **キャッチコピー + Motivations のみ更新**: 「料理から逃げてきたのに…」を「火曜日になるとタコスのことを考えてしまう男」に変え、Motivations に「認知占有体験をメタ視点で楽しむ」を追加。Goals / Pain Points / AC は維持。
B) **Goals / Pain Points も含めて更新**: Goals に「Taco Tuesday が自分の火曜日のデフォルトになる（文化的受容）」を追加。Pain Points に「火曜 21:00 が近づくと不安になる自分に気づいている」を追加。
C) **全面改訂（ゼロから書き直し）**: v2.0 の Concept Statement を中心に、キャッチコピー・Profile・Goals・Pain Points・Motivations・How タコ中 Affects をすべて書き直す。
X) Other (please describe after [Answer]: tag below)

[Answer]: B  

---

### Question 2
**「Taco Tuesday の日本化」を明示するストーリー**

文化移植ミッション（Q1=B の回答）を具体的なユーザーストーリーとして表現しますか？

A) **既存ストーリーの framing 更新のみ**: 新規 US は追加しない。US-T01（トライアル）や US-T13（材料受け取り）の Story テキスト・Motivation 欄に「Taco Tuesday 体験の一歩」という表現を織り込む。
B) **新規ストーリー 1 本追加（US-T15）**: "As a タロウ, I want to タコ中を 3 ヶ月続けることで、火曜日が自然と「タコスを作る日」になっていてほしい。so that Taco Tuesday が自分の生活リズムの一部になる" という cultural adoption ストーリーを追加する。
C) **ミナミのストーリーに文化伝道師角度を追加**: US-M01（強制発注スクショ投稿）などを「日本の友人に Taco Tuesday を布教する」という framing で書き直す新規 US を追加。
D) **B + C 両方**: US-T15 + ミナミの文化伝道師ストーリーを追加。
X) Other (please describe after [Answer]: tag below)

[Answer]: B 

---

### Question 3
**Story Board（story-board.md）の再生成スコープ**

story-board.md（体験ストーリーボード: ガントチャート・シーケンス図・タッチポイントマップ）の更新範囲は？

A) **軽量更新**: § の見出しや説明文に「Taco Tuesday 日本化」「認知の占有」「幸せなダメになり方」の文言を追記する。ガント・シーケンス図の構造は維持。
B) **「認知占有の体験弧」を追加**: Week 0（トライアル）→ Week 4（習慣化）→ Week 12（"最高じゃないですか" 到達）という長期依存アーク図を新セクションとして追加。既存図は維持。
C) **全面改訂**: v2.0 の Concept Statement と設計原則を中心に story-board.md を一から書き直す。
X) Other (please describe after [Answer]: tag below)

[Answer]: B 

---

## 実行チェックリスト（PART 2 で使用）

### 準備
- [x] Q1〜Q3 の回答受領・分析完了（Q1=B, Q2=B, Q3=B）
- [x] 曖昧さゼロ確認

### personas.md 更新
- [x] タロウ: キャッチコピー更新（認知占有 framing）
- [x] タロウ: Goals 更新（Taco Tuesday デフォルト化 / 幸せな降伏を追加）
- [x] タロウ: Pain Points 更新（火曜 21:00 近づく不安を追加）
- [x] タロウ: Motivations 更新（認知占有メタ体験 / 幸せな降伏を追加）
- [x] タロウ: How タコ中 Affects 更新（3 ヶ月後の状態を追記）
- [x] ミナミ: キャッチコピー更新（Taco Tuesday 伝道師 framing）
- [x] ミナミ: Motivations 更新（日本初 Taco Tuesday インフルエンサー角度）
- [x] ミナミ: How タコ中 Affects 更新（文化伝播経路を明記）

### stories.md 更新
- [x] Story Index のタイトル・説明文を v2.0 整合に更新（16 ストーリーに）
- [x] US-T01: "so that" に「Taco Tuesday が生活に侵食していく体験の第一歩」framing を追加
- [x] US-T13: "so I can" に「材料が届いた瞬間 = Taco Tuesday の儀式が始まる感覚」framing を追加
- [x] US-T14: "so I can" に「時間支配が頭を占有していくのを実感」framing を追加
- [x] US-T07: "so I can" に「コントロール喪失の幸せ」framing を追加
- [x] US-M01: "so I can" に「Taco Tuesday 文化を布教するコンテンツ」framing を追加
- [x] US-M04: "so I can" に「Taco Tuesday コンテンツシリーズで文化移植」framing を追加
- [x] US-T15 新規追加（文化的受容ストーリー、Q2=B）
- [x] Persona × Story Coverage Matrix 更新（12 + 4 件）
- [x] FR Coverage Map 更新（US-T15 / 文化移植 KPI 追加）
- [x] Theme Alignment 更新（v2.0 観点 / US-T15 ◎◎ 追加）

### story-board.md 更新
- [x] Q3=B: §11「認知占有の体験弧（Week 0→4→12）」を新セクションとして追加（既存図 §2〜§10 は維持）
- [x] 「文化移植ミッション」「認知の占有」「幸せなダメになり方」のキーワードを §11 全体に反映
- [x] Document Version 1.4 → 2.0 に更新

---

## 成果物一覧

| ファイル | 場所 | 内容 |
|---------|------|------|
| `personas.md` (v2.0) | `aidlc-docs/inception/user-stories/` | タロウ + ミナミ ペルソナ（v2.0 深掘り版） |
| `stories.md` (v2.0) | `aidlc-docs/inception/user-stories/` | 15〜17 ストーリー（v2.0 整合版） |
| `story-board.md` (v2.0) | `aidlc-docs/inception/user-stories/` | 体験ストーリーボード（Q3 の範囲で更新） |

---

**回答方法**: 各 `[Answer]:` の後に選択肢記号（A / B / C / D / X）を記入してください。
