# Unit of Work Story Map: タコ中

**Project**: タコ中 (Tako-chū / Tako-tues)
**Document Version**: 1.1
**Created**: 2026-04-29
**Updated**:
- 2026-05-08 v1.1: 要件 v1.8 / stories v1.4 / unit-of-work v1.1 反映。**FR-6.3 廃止に伴い US-T11 / US-M03 行を削除**、Documentation (C8) セクションへの参照を全削除、ストーリー総数 17 → 15 に更新、Hackathon Theme Alignment 表から Documentation 行削除、Iteration 4+ 配置から US-T11 / US-M03 を削除
**Phase**: INCEPTION - Units Generation (Part 2: Generation)

---

## 0. このドキュメントの位置付け

stories.md v1.4 の 15 ストーリー（タロウ 11 + ミナミ 4。v1.1 で旧 ChatGPT GPT 関連 2 ストーリー US-T11 / US-M03 を削除済み）を、**主担当 Unit と関与 Unit に分けてマッピング**する。

**マッピング方針（Q1=D ハイブリッド）**:
1. **状態を保持する Unit を主担当**（DynamoDB の書き込み主が最終的な責任を持つ）
2. **FR Coverage Matrix で確認**（components.md §4 と整合）
3. ~~Documentation (C8) は Unit 外~~ → **v1.1 で削除**（FR-6.3 廃止）

---

## 1. ストーリー → Unit マッピング表

| Story ID | タイトル | 主担当 Unit | 関与 Unit | 主要 FR |
|----------|---------|------------|----------|---------|
| **タロウのジャーニー（13 件）** ||||| 
| US-T01 | 無料トライアル週でサービスを試す | **U1 AuthAndTrial** | U7（UI） | FR-5 |
| US-T02 | タコス摂取を記録する | **U2 IntakeLogging** | U7（UI） | FR-1.3 |
| US-T03 | 金曜夜の強制発注通告を受け取る | **U3 OrderEngine** | U7（UI）, U5（通知） | FR-2.1, FR-2.2 |
| US-T04 | 「キャンセル不可」を試そうとして失敗する | **U3 OrderEngine** | U7（UI） | FR-2.4 |
| US-T13 | 月曜夜に材料受け取り + 静的レシピ配信 | **U4 RecipeLibrary** | U3（Delivered イベント発行）, U7（UI）, U5（Push 配信） | FR-2.3, FR-2.4 |
| US-T14 | 24h カウントダウンタイマーで台所に追い込まれる | **U7 PwaFrontend** | U3（配送ステータス）, U6（罰時刻データ） | FR-1.2, FR-4.1 |
| US-T06 | 火曜夜に Tシャツ発注通知を受け取る（24h ルール） | **U6 Punishment** | U7（UI）, U2（intake チェック）, U3（Order 状態） | FR-1.2 |
| US-T07 | 24h ウィンドウ超過時、Tシャツ罰と同時にサルサ通知が鳴り続ける | **U6 Punishment** | U7（Push） | FR-1.1 |
| US-T08 | 平日昼夕にメキシコ風誘惑 Push を受け取る | **U5 StimulusEngine** | U7（Push） | FR-6.2 |
| US-T09 | 24h カウントダウンウィンドウ内で Push 無視回数に応じてトーンがエスカレートする | **U5 StimulusEngine** | U7（Push） | FR-6.2, FR-6.4 |
| US-T10 | ダッシュボードで煽り文句を見る | **U5 StimulusEngine** | U7（UI） | FR-6.1, FR-4 |
| ~~US-T11~~ | ~~ChatGPT 「タコ中 GPT」と会話してタコス文脈に引き戻される~~ | — | — | **v1.1 で削除**（FR-6.3 廃止） |
| US-T12 | 強制バリエーションウィーク（全部マンゴー味）を通告される | **U3 OrderEngine** | U7（UI）, U4（Type B レシピ）, U5（テーマインジェクト） | FR-2.2 |
| **ミナミのジャーニー（5 件）** ||||| 
| US-M01 | 強制発注通告のスクショを SNS に投稿する | **U3 OrderEngine** | U7（UI / Share） | FR-2.1, FR-4 |
| US-M02 | キャンセル試行→発注増のシュール反応を投稿する | **U3 OrderEngine** | U7（Before/After UI / Share） | FR-2.3 |
| ~~US-M03~~ | ~~ChatGPT GPT の説教モード会話を共有する~~ | — | — | **v1.1 で削除**（FR-6.3 廃止） |
| US-M04 | 届いた材料 + レシピで 30 秒料理動画を投稿する | **U4 RecipeLibrary** | U7（Share for video / 字幕画像 PNG） | FR-2.3 |
| US-M05 | 作らずに Tシャツ罰を受け、"罰受けました" 投稿する | **U6 Punishment** | U7（"罰受けました" UI / Share） | FR-1.2 |

**合計**: 15 ストーリー（v1.4 時点、US-T05 は v1.1 で US-T13 に統合済み、US-T11 / US-M03 は v1.4 で削除）

---

## 2. Unit 別ストーリー集計

### U1. AuthAndTrial
- **主担当 1 件**: US-T01

### U2. IntakeLogging
- **主担当 1 件**: US-T02

### U3. OrderEngineAndDeliveryMock
- **主担当 5 件**: US-T03, US-T04, US-T12, US-M01, US-M02

### U4. RecipeLibrary
- **主担当 2 件**: US-T13, US-M04

### U5. StimulusEngine
- **主担当 3 件**: US-T08, US-T09, US-T10

### U6. Punishment
- **主担当 3 件**: US-T06, US-T07, US-M05

### U7. PwaFrontend
- **主担当 1 件**: US-T14
- **関与（UI レイヤー）**: 全 15 ストーリー（PWA は全機能の入り口。v1.1 で US-T11 / US-M03 削除を反映）

### U9. Infrastructure
- **主担当 0 件**（横断責務、ストーリー直接担当なし）

### ~~Documentation（C8 ChatGPT GPT）~~ → **v1.1 で削除**
- 旧 US-T11 / US-M03 は要件 v1.8 で FR-6.3 廃止に伴い削除済み

---

## 3. ストーリー Iteration 配置

execution-plan v1.3 の Iteration 0（Walking Skeleton）+ 反復改善モデルに従い、各ストーリーを Iteration に配置:

### Iteration 0（Walking Skeleton）— 5/8 目標

最低限通すべき E2E シナリオ #1, #2 に必要なストーリーのみ:

| Iteration 0 で受入基準を満たすストーリー | Unit |
|-----------------------------------|------|
| US-T01 無料トライアル（簡易版） | U1 |
| US-T02 タコス摂取記録 | U2 |
| US-T03 強制発注通告（基本版） | U3 |
| US-T13 材料受け取り + 静的レシピ配信（レシピ 2-3 種） | U4 |
| US-T14 24h カウントダウン UI（最低限） | U7 |
| US-T06 Tシャツ発注通知（罰フラグのみ） | U6 |
| US-T07 サルサ通知 1 回（ループなし） | U6 |

### Iteration 1（5/8〜5/15 目標）

- US-T08 平日昼夕誘惑 Push（毎日 2 回フル）
- US-T09 エスカレーション Level A/B/C 完全実装
- US-T10 ダッシュボード煽り文（Static のまま）
- US-T07 サルサ 30 分間隔ループ完全実装
- US-T04 キャンセル試行 → 発注増

### Iteration 2（5/15〜5/22 目標）

- US-T12 強制バリエーションウィーク（Type B）発火
- US-T13 全キット種別レシピ JSON 拡充
- US-M01, US-M02 SNS 投稿用 UI（スクショ映え）

### Iteration 3（5/22〜5/27 目標）

- US-T09 Bedrock 動的生成 ON（StimulusGenerator 実装切替）
- US-M04, US-M05 Share for video / "罰受けました" 投稿 UI
- E2E 通し確認 + 予選プレゼン磨き

### Iteration 4+（決勝向け 6 月）

> v1.1 で旧「US-T11 / US-M03（Documentation セクション）」項目を削除（FR-6.3 廃止）。
- 全シナリオ磨き込み
- バイラル素材作成

---

## 4. Hackathon Theme Alignment Coverage

stories.md v1.3 §「Hackathon Theme Alignment Check」で全ストーリーがテーマ「人をダメにするサービス」に適合確認済み。Unit 視点で再確認:

| Unit | 担当ストーリーのテーマ適合密度 |
|------|-------------------------------|
| U1 | ⭕（罠の入口） |
| U2 | ⭕（行動可視化） |
| U3 | ◎（意思決定の放棄 / キャンセル不可 / 多様性剥奪） |
| U4 | ◎（台所侵食） |
| U5 | ◎（食生活主導権喪失 / エスカレーション） |
| U6 | ◎（罰の物理化 / 平穏が許されない） |
| U7 | ⭕（時間支配 / UI 入口） |
| U9 | — |
| ~~Documentation~~ | **v1.1 で削除**（FR-6.3 廃止） |

✅ 全 Unit が「人をダメにする」コアコンセプトに紐付き済み。

---

## 5. ストーリー受入基準の Unit 内テスト責任

各ストーリーの受入基準（Given/When/Then BDD）の検証責任は**主担当 Unit が持つ**。関与 Unit はモック / スタブで補う:

| ストーリー | 主担当の検証方法 | 関与 Unit のモック例 |
|-----------|---------------|-------------------|
| US-T03 | U3 の単体テスト（freezegun で金 21:00 を再現、moto で DDB） | U7 → API クライアント MSW、U5 → Push 送信 stub |
| US-T07 | U6 の単体テスト（freezegun で火 21:00、moto） | U7 → Push 受信は手動確認、U2 の intake チェックは moto |
| US-T13 | U4 の単体テスト（pyfakefs or 実 JSON、Delivered イベントで起動） | U3 → イベント送信 stub、U7 → 表示は手動 |
| US-T14 | U7 の Storybook + Vitest（タイマー UI のスナップショット） | U6 → 罰時刻データを fixture 化 |

---

## 6. 完整性検証

### 6.1 全ストーリーがマップ済みか

✅ 15 ストーリー全てがマップ済み（v1.4 = 11 タロウ + 4 ミナミ。US-T05 統合済み、US-T11 / US-M03 削除済み）

### 6.2 主担当 Unit が「状態を保持する Unit」になっているか（Q1=D 検証）

| Story | 主担当 Unit | 状態保持確認 |
|-------|-----------|-----------|
| US-T01 | U1 | ✅ TrialState |
| US-T02 | U2 | ✅ IntakeRecord |
| US-T03 | U3 | ✅ Order |
| US-T04 | U3 | ✅ Order の portions 増加 |
| US-T06 | U6 | ✅ PunishmentRecord |
| US-T07 | U6 | ✅ SalsaLoopState |
| US-T08 | U5 | ✅ EscalationCounter |
| US-T09 | U5 | ✅ EscalationCounter |
| US-T10 | U5 | ✅ 煽り文キャッシュ |
| US-T12 | U3 | ✅ Order の is_variation_week |
| US-T13 | U4 | ⚠️ 状態なし（ただしレシピ JSON が "状態" 相当）。FR で確認 → FR-2.3 主担当 |
| US-T14 | U7 | ⚠️ 状態なし（クライアント UI）。FR で確認 → FR-1.2/FR-4.1 連携、UI 主体のため U7 |
| US-M01, M02, M04, M05 | 元ストーリーと同 Unit | ✅ |
| ~~US-T11, US-M03~~ | ~~Documentation~~ | **v1.1 で削除**（FR-6.3 廃止） |

→ Q1=D（A+C ハイブリッド）の方針通り、状態を持つ Unit に集約され、U4 / U7 の "状態無し" ケースも FR で確認できているため健全。

### ~~6.3 ChatGPT GPT 関連ストーリーが Documentation に分離されているか（Q3=C 検証）~~ → **v1.1 で削除**

> 要件 v1.8 で FR-6.3 自体をスコープから削除したため、本検証項目は廃止。

---

## 7. 参照

- [unit-of-work.md](./unit-of-work.md) - 8 Unit（v1.1 で Documentation 削除）
- [unit-of-work-dependency.md](./unit-of-work-dependency.md) - Unit 間依存
- [../user-stories/stories.md](../user-stories/stories.md) - 元ストーリー集（v1.4）
- [../user-stories/personas.md](../user-stories/personas.md) - ペルソナ（タロウ / ミナミ）
- [components.md](./components.md) - FR Coverage Matrix
- [../plans/execution-plan.md](../plans/execution-plan.md) - Iteration 配置の元
