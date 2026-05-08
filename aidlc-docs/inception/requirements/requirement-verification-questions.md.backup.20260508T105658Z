# 要件確認質問: タコ中（たこちゅう）

**Project**: タコ中 (Tako-chū / Tako-tues)
**Document Version**: 1.0
**Created**: 2026-04-29
**Status**: ✅ Closed (2026-04-29) — 全 17 問解決（Q15 のみ AI-DLC 推奨で仮置き、OI-1 として Construction NFR Requirements に引き継ぎ）
**Phase**: INCEPTION - Requirements Analysis (Q&A, Closed)

このドキュメントは、タコ中プロジェクトの要件を明確化するための質問集です。
各質問の `[Answer]:` タグの後に **A〜X の選択肢の文字** を記入してください。
「Other」を選んだ場合は、その後に詳細を記述してください。

すべて回答できたら「done」「完了」などとお知らせください。

---

## カテゴリ 1: プロジェクトのスコープと真剣度

### Question 1
このプロジェクトのスコープはどれが最も近いですか？

A) フルスケール本番サービス — 実際にユーザーに提供する商用サービスとして開発する（決済・配送・法務含む）
B) MVP（最小実用プロダクト） — コア体験だけ動く最小プロダクト。Shopify/Uber Eats などは将来の連携想定でモック可
C) PoC（概念実証） — 「依存ループ」のメイン体験を実演できるプロトタイプ。データはダミーOK
D) パロディ／アート作品 — 動作デモは作るが、ユーモア／風刺としての完成度を重視（実際の発注は行わない）
X) Other (please describe after [Answer]: tag below)

[Answer]: 一旦Cで進めます。

---

### Question 2
ターゲットユーザーは誰ですか？（複数該当する場合は最も主要なものを選んでください）

A) 一般消費者（タコス愛好家） — 個人向けの toC サービス
B) 自分自身一人だけ — 自分専用のセルフ依存ツール
X) Other (please describe after [Answer]: tag below)

[Answer]: A （ただし最初は自分向けに作って、将来的に一般公開も視野に入れる形が理想）

---

### Question 3
このプロジェクトのデプロイターゲットは？

A) クラウド（AWS / GCP / Azure 等のマネージドサービス）
B) ローカル／自宅サーバー（Raspberry Pi、自宅 Mac など）
C) ローカル PC で動くデスクトップアプリ
D) スマートフォンアプリ（iOS / Android）
X) Other (please describe after [Answer]: tag below)

[Answer]: A （AWSをフル活用する)

---

## カテゴリ 2: コア機能のスコープ

### Question 4
最初のリリースに **絶対に含める** コア機能はどれですか？（最も優先度が高いもの 1 つを選んでください）

A) AI によるタコス欲予測＋自動発注ループ（金曜予測 → 月曜配送）
B) 「作らされる」強制トリガー（Tシャツ送付・サルサ通知）
C) Claude/MCP 経由の「タコス話しかしない」会話エージェント
D) Apple HealthKit 連携で「タコス摂取量 × 幸福度」可視化ダッシュボード
E) すべてを最初から統合した完全ループ
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 5
外部 API 連携について、どのレベルで実装しますか？

A) 本番統合 — Shopify/Uber Eats/HealthKit すべての実 API キーを使い実発注
B) サンドボックス統合 — 各 API のサンドボックス／テスト環境を使う（実発注なし）
C) モック統合 — 実 API は呼ばず、ダミーデータと擬似応答で挙動を再現
D) 段階的 — まずモック、後でサンドボックス、最後に本番（フェーズ分けしたい）
X) Other (please describe after [Answer]: tag below)

[Answer]: 最初はC、次にB

---

### Question 6
「火曜朝に Tシャツが家に届く」機能は実装しますか？

A) 実装する — 実際の Print-on-Demand サービス（例: Printful）と連携してリアルに発送する
B) 実装する — ただし「発注済み」の通知だけ出すモック（実物は送らない）
C) 実装しない — 概念は残すが MVP からは外す
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 7
「Claude が MCP 経由でタコスの話しかしない」体験は、どう実現しますか？

A) 専用の MCP サーバーを作り、ユーザーのカレンダー／健康／在庫情報を Claude Desktop / Claude Code に提供する
B) Claude API を直接呼び出す独自チャット UI を作る（System Prompt でタコス縛り）
C) 両方（MCP サーバー＋独自 UI）
D) この機能は MVP からは外す
X) Other (please describe after [Answer]: tag below)

[Answer]: X(MCPではなくてプロンプトかskillで縛る形が理想)

---

## カテゴリ 3: 技術スタック

### Question 8
バックエンドの主要言語／ランタイムの希望は？

A) TypeScript / Node.js（MCP SDK、Anthropic SDK の親和性が高い）
B) Python（AI/データ分析に強い、HealthKit 連携は工夫が必要）
C) Go / Rust など、性能重視
D) おまかせ — AI-DLC が最適なものを推奨してほしい
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 9
データ永続化（ユーザーのタコス履歴、欲予測モデル等）はどうしますか？

A) クラウド DB（PostgreSQL on RDS、Supabase、Firestore など）
B) ローカル SQLite ファイル
C) JSON ファイル等のシンプルなファイル保存
D) おまかせ
X) Other (please describe after [Answer]: tag below)

[Answer]: A(基本無料枠内で収まるものが理想)

---

### Question 10
HealthKit 連携が必要となると iOS アプリが必要ですが、どう扱いますか？

A) ネイティブ iOS アプリ（Swift / SwiftUI）を作る
B) HealthKit は将来の拡張として今回はスキップ（Web で擬似的な「幸福度」入力）
C) ヘルスデータは Apple Health から手動エクスポート → ファイル取り込み
X) Other (please describe after [Answer]: tag below)

[Answer]: X(モック)

---

## カテゴリ 4: 非機能要件

### Question 11
パフォーマンス／スケール要件はどのレベルですか？

A) 大規模（10,000+ ユーザーを想定、水平スケール前提）
B) 中規模（〜1,000 ユーザー、シングルリージョンで十分）
C) 小規模（〜100 ユーザー、個人〜小サークル）
D) シングルユーザー（自分または家族のみ）
X) Other (please describe after [Answer]: tag below)

[Answer]: C

---

### Question 12
プライバシー／コンプライアンスについて、特に意識すべき制約はありますか？

A) 個人医療データ扱いに準じる配慮（HealthKit 連携前提、暗号化必須）
B) 一般的な PII 保護（パスワードハッシュ、HTTPS など標準的なベストプラクティス）
C) 個人プロジェクトのため最低限でよい
X) Other (please describe after [Answer]: tag below)

[Answer]: C

---

### Question 13
予算とコスト制約は？

A) 月額数百円〜数千円以内（個人持ち出し前提）
B) 月額数千〜数万円OK（実 API、クラウド料金含めても問題ない）
C) コスト制約なし（投資前提）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## カテゴリ 5: 拡張機能 (AI-DLC Extensions)

### Question 14: Security Extensions
セキュリティ拡張ルールをこのプロジェクトで強制適用しますか？

A) Yes — すべての SECURITY ルールをブロッキング制約として強制（本番運用するアプリに推奨）
B) No — SECURITY ルールをスキップ（PoC、プロトタイプ、実験的プロジェクトに適切）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 15: Property-Based Testing Extension
プロパティベーステスト（PBT）ルールをこのプロジェクトで強制適用しますか？

A) Yes — すべての PBT ルールをブロッキング制約として強制（ビジネスロジック、データ変換、シリアライゼーション、ステートフルなコンポーネントを持つプロジェクトに推奨）
B) Partial — 純粋関数とシリアライゼーション往復のみ PBT 適用（限定的なアルゴリズム複雑性のプロジェクトに適切）
C) No — PBT ルールをスキップ（シンプルな CRUD、UI のみ、薄い統合層など、ビジネスロジックがほぼないプロジェクトに適切）
X) Other (please describe after [Answer]: tag below)

[Answer]: B（Partial）※ AI-DLC 推奨で仮置き、Construction NFR Requirements で確定予定（OI-1）

---

## カテゴリ 6: ビジネスコンテキスト

### Question 16
このプロジェクトの成功基準（What does done look like?）は何ですか？

A) 自分が実際にこのアプリの「依存ループ」を体験して笑える／満足できる
B) 友人・SNS でデモして話題になる（バイラル／話題性）
C) 商用ローンチして実ユーザーがつき、収益化できる
D) コード自体の学習体験（Claude/MCP/各種 API 連携の習熟）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 17
リリース時期の目標はありますか？

A) 1〜2 週間以内（最短デモ）
B) 1 か月以内（コア機能完成）
C) 3 か月以内（拡張機能含めしっかり作る）
D) 期限なし（完成度重視で時間をかける）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

回答が終わったら「done」または「完了」とお知らせください。
