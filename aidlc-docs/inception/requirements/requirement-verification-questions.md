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

---

## ↓ rework v2.0 追加（2026-05-09）— ビジネス意図深掘り

> **元の要件確認質問（Q1〜Q17）は上記。以下は rework v2.0 でのビジネス意図深掘り質問と回答。**

# ビジネス意図深掘り — 要件確認質問

**目的**: Concept Statement と設計原則を「なぜタコスか」「人をダメにするとはどういう状態か」という観点で深化させるための確認。

---

## Question 1
「なぜタコスなのか」——他の食材（ラーメン・ピザ・カレー）では実現できない、タコスにしかない必然性は何ですか？

A) **手作業の儀式性**: タコスは「具材を自分で巻く」という参加型の行為が不可欠で、完成品受取ではなく"作る"体験がコアになるため（ピザや弁当は受け取って終わり）
B) **Taco Tuesday 文化**: 火曜日 = タコスという週次リズムが英語圏文化に根付いており、曜日と食のペアリングがすでに存在する（プロジェクト名 `tako-tues` の由来でもある）
C) **材料が「部品」として届く構造**: トルティーヤ・肉・野菜・ソースが分離した状態で届き、「組み立て」を強制できる食材形態は他に少ない（1 食 = 1 セッションの強制力が高い）
D) **シュールさ・ネタ性**: タコスは日本では「エキゾチックだが知られている」微妙な立ち位置であり、ハッカソンでの話題性（Taco Tuesday のパロディ）としてネタになりやすい
E) **上記複数の組み合わせ** (どれとどれか [Answer] 後に記述)
X) Other (please describe after [Answer]: tag below)

[Answer]:B を日本に根づかせたい 

---

## Question 2
「人をダメにする」のエンドステートを定義してください。このサービスを 3 ヶ月使い続けたユーザーは、具体的にどういう状態になっていますか？

A) **習慣・リズムの書き換え**: 火曜日が「タコスを作る日」として生活カレンダーに定着し、タコスなしでは火曜日が落ち着かない感覚が生まれている（行動依存）
B) **思考の占有**: 平日の Push 通知・煽り文によってタコスのことを考える時間が増え、意図せずタコスが「頭の中に住み着く」状態（認知占有）
C) **主体的選択の喪失**: 「何を食べるか自分で決める」という意思決定をこのサービスに委ねてしまい、タコス以外の選択肢を選ぶのが面倒になる（意思決定の外部委託）
D) **自虐的な自己同一性**: 「タコス依存者」としての自覚を持ちながら抜けられない状態を面白がる、コミカルな自己パロディ（笑える依存）
E) **上記複数の組み合わせ** (どれとどれか [Answer] 後に記述)
X) Other (please describe after [Answer]: tag below)

[Answer]: B 

---

## Question 3
「侵食」「強制」「逃げ場喪失」のうち、このサービスが最も重視すべき依存メカニズムはどれですか？（Concept Statement のトーンを決めるために確認）

A) **侵食が主役**: 「知らないうちに生活の中にタコスが増えていく」という自然浸透感を中心に。ユーザーは気づいたら囲まれている（ゆっくり侵される恐怖感・コミカル）
B) **強制が主役**: 発注システム・Push・罰（Tシャツ）など「外圧で行動させられる」構造を前面に。ユーザーは能動的に選んでいない（コントロール喪失の笑い）
C) **逃げ場喪失が主役**: 無視するとエスカレートする罰・食材は届いてしまっている・材料を捨てる罪悪感——「退路を断つ」設計を中心に。逃げようとするほど深みにはまる
D) **三者は等価**: バランスよく三層で実現する（侵食=通知・生活浸透、強制=発注トリガー、逃げ場喪失=罰・エスカレーション）
X) Other (please describe after [Answer]: tag below)

[Answer]: B 

---

## Question 4
「人をダメにする」は、このサービスの文脈でポジティブ・ネガティブ・どちらで描写しますか？

A) **ポジティブ（幸せなダメになり方）**: タコスが好きなユーザーにとって「ダメになる」は快適な依存。「最高じゃないですか」のトーン。Concept Statement は明るく・楽しく書く
B) **ネガティブ（反面教師・パロディ）**: ソーシャルメディア中毒・ガチャ課金などの依存設計を食で再現するブラックコメディ。「こんなサービスがあったら怖い」という批評性を持つ
C) **中立（技術デモとして淡々と）**: 依存ループの「仕組み」を見せることを目的とし、ユーザーへの善悪評価は持たない。エンジニアリング観点から面白さを語る
D) **意図的な曖昧さ**: 「幸せなのかダメなのかわからない」という宙吊り感がコンセプトの核。Concept Statement もその曖昧さを活かして書く
X) Other (please describe after [Answer]: tag below)

[Answer]: A 

---

## Question 5
Concept Statement に「なぜタコスか」の説明を明示的に入れますか？

A) **入れる（明示的に説明）**: 「タコスは〇〇だから選んだ」と一文で根拠を述べる。審査員が読んだとき「なるほど」と思える論拠を埋め込む
B) **入れない（余白として残す）**: 「タコスだから」という自明性の演出を意図的に選ぶ。理由を語らず「当たり前のようにタコス」という狂気感でインパクトを出す
C) **設計原則の中に散りばめる**: Concept Statement には書かず、設計原則（1〜6）の中にタコス固有の理由を自然に組み込む
X) Other (please describe after [Answer]: tag below)

[Answer]:A  

---

**回答方法**: 各 `[Answer]:` の後に、選択肢の記号（A / B / C / D / X）を記入してください。複数選択や補足説明も歓迎します。
