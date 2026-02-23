# AI営業ロープレ v2 — 引き継ぎ書

> 作成日: 2026-02-23  
> リポジトリ: https://github.com/kitayama-ai/ai-sales-roleplay  
> デプロイURL: https://kitayama-ai.github.io/ai-sales-roleplay/

---

## 1. プロジェクト概要

Gemini Live API（WebSocket）を使った**音声ベースの営業ロールプレイ練習ツール**。
AIが顧客役を演じ、営業担当者が音声で会話→終了後に10軸で自動採点→弱点をピンポイント練習できる。

### 技術スタック

- **フロントエンド**: 単一HTML（`index.html`、約843行）。CSS/JSインライン。フレームワークなし。
- **音声API**: Gemini Live API（WebSocket `wss://generativelanguage.googleapis.com/ws/...`）
- **マイク入力**: Web Audio API → AudioWorklet → PCM 16kHz → base64 → WS送信
- **音声再生**: PCMチャンクをバッチ→AudioBufferSource で再生（24kHz）
- **採点**: Gemini 2.0 Flash REST API（`temperature: 0`、JSON出力）
- **データ保存**: localStorage（履歴・利用統計・設定）
- **GAS連携**: 失注データ取得用 doGet API（GAS側デプロイ必要）

---

## 2. ファイル構成

```
index.html          ← 全機能が入った単一ファイル（HTML+CSS+JS）
HANDOFF.md          ← この引き継ぎ書
```

### index.html の構造（主要セクション）

| 行範囲  | セクション     | 内容                                                                     |
| ------- | -------------- | ------------------------------------------------------------------------ |
| 1-85    | `<style>`      | CSSカスタムプロパティ、ダークテーマ、レスポンシブ                        |
| 86-201  | `<body>` HTML  | ヘッダー、サイドバー設定、チャットエリア、結果モーダル、利用状況モーダル |
| 228-253 | CONFIG定数     | HINTS, PERSONAS, DIFFICULTY, SCENARIOS, SUB_NAMES                        |
| 255-270 | INIT           | イベントリスナー、initScoreBars、ローカルストレージ読み込み              |
| 324-462 | startSession() | WebSocket接続、setup送信、onmessage（音声再生・転写表示）                |
| 467-536 | マイク・再生   | PCM AudioWorklet、バッチ再生、drainQ                                     |
| 569-654 | gradeSession() | 採点プロンプト（ルブリック + evidence）、temperature:0                   |
| 656-702 | showResult()   | 10軸スコア表示、evidence/deduction表示、ドリルボタン                     |
| 729-789 | 利用管理       | saveUsage、showUsageDashboard、renderUsageDashboard、exportUsageCSV      |
| 792-818 | GAS連携        | fetchLossData、ドロップダウン、自動ペルソナ設定                          |

---

## 3. 実装済み機能

### 3.1 音声ロープレ

- Gemini Live API（WebSocket raw接続）
- モデル: `gemini-2.5-flash-native-audio-preview-12-2025`（推奨）
- 音声出力言語: 日本語（`language_code: 'ja-JP'`）、Voice: Puck
- バージイン（割り込み）対応: `data.serverContent?.interrupted` でキュークリア
- 沈黙ヒント: 15秒無言で弱点に応じたアドバイス表示

### 3.2 顧客ペルソナ（6種）

| ID      | 名前           | 初期セリフ                                             |
| ------- | -------------- | ------------------------------------------------------ |
| price   | 「高い」顧客   | 「ちょっと…正直、値段がちょっと気になってて。」        |
| compare | 比較検討       | 「他にも似たようなスクールあるじゃないですか？」       |
| busy    | 「忙しい」     | 「興味はあるんですけど、正直今すごく忙しくて…」        |
| spouse  | 「配偶者相談」 | 「いいとは思うんですけど、一応妻にも相談して…」        |
| doubt   | 「効果ある？」 | 「AIマーケティングって結局どのくらい使えるんですか？」 |
| custom  | 実商談トレース | GASから取得した失注データで顧客パターン再現            |

### 3.3 採点（10軸×10点 = 100点満点）

| 軸             | キー          | 評価内容                     |
| -------------- | ------------- | ---------------------------- |
| 権威性         | authority     | 「選ぶ側」のスタンス         |
| 損失回避       | loss_aversion | 現状リスク指摘               |
| リフレーミング | reframing     | スクール→社内研修 定義変換   |
| アンカリング   | anchoring     | 費用を「覚悟のフィルター」に |
| クロージング   | closing       | 「手続き進めます」言い切り   |
| ヒアリング     | hearing       | 課題の深堀り                 |
| ラポール       | rapport       | 信頼関係構築                 |
| 反論処理       | objection     | 切り返しの質                 |
| テンポ         | tempo         | 会話リズム・間               |
| 提案力         | proposal      | 具体的活用イメージ           |

- **ルブリック**: 0-2（行動なし）/ 3-4（不十分）/ 5-6（基本OK）/ 7-8（効果的）/ 9-10（卓越）
- **出力**: 各軸に `evidence`（根拠発言の引用）と `deduction`（減点理由）
- `temperature: 0` で決定的出力

### 3.4 弱点ピンポイントロープレ

- 採点結果の各軸横に🎯ボタン → クリックで弱点+シナリオ自動設定
- ワースト3の「ピンポイント練習」ボタン

### 3.5 月別利用状況管理

- `localStorage` に `rp_usage` として保存
- 担当者別の回数・平均スコア・合計時間・最終練習日
- CSVエクスポート対応

### 3.6 GAS連携

- サイドバーでGAS Web App URL入力 → 「失注データ取得」ボタン
- ドロップダウンから失注商談選択 → 自動でカスタムペルソナ設定

---

## 4. localStorage キー一覧

| キー         | 内容                 | 形式                                                               |
| ------------ | -------------------- | ------------------------------------------------------------------ |
| `rp_apikey`  | Gemini API Key       | string                                                             |
| `rp_history` | 練習履歴（直近50件） | `[{date, score, sub, wp, name}]`                                   |
| `rp_usage`   | 利用統計             | `[{name, date, month, score, sub, duration, scenario, weakPoint}]` |
| `rp_gasUrl`  | GAS Web App URL      | string                                                             |

---

## 5. 未実装・今後の開発項目

### 優先度高

1. **GAS側 doGet デプロイ**: walkthrough.md にコードあり。スプレッドシートの列名に合わせて調整が必要
2. **音声の文字起こし精度向上**: `outputTranscript` / `inputTranscript` がモデルによって対応状況が異なる
3. **利用データのGASスプレッドシート連携**: localStorageだけでなく、チーム共有のためGASにPOSTする仕組み

### 優先度中

4. **スコア推移グラフ**: 担当者ごとの週次・月次推移をChart.jsで可視化
5. **録音・再生機能**: MediaRecorderで商談音声を保存（基盤コードは `startRecording()` にあるが未活用）
6. **Slack通知**: 練習完了時にスコアをSlackに自動投稿
7. **自動再接続**: WebSocket切断時の自動リトライ

### 優先度低

8. **マルチ言語対応**: 英語版UI
9. **エフェメラルトークン**: API Keyの代わりにエフェメラルトークンでセキュリティ向上
10. **PWA対応**: オフラインキャッシュ + ホーム画面追加

---

## 6. 既知の問題

1. **音声モデルの英語出力**: `gemini-2.5-flash-native-audio` は内部推論を英語でテキスト出力することがある → `p.text` はUI非表示にしている
2. **音声クラッキング**: チャンクサイズが小さいと発生 → 80msバッチで軽減済み
3. **ファイル末尾の残骸**: `</html>` 後に古いコードの断片が残る場合がある（ブラウザは無視するので動作に影響なし）

---

## 7. 開発環境メモ

- macOS上のローカルHTMLファイル（サーバー不要）
- `file://` または `python3 -m http.server` で動作確認
- GitHub Pages: https://kitayama-ai.github.io/ai-sales-roleplay/
- API Key: ユーザーがUI上で入力（localStorageに保存オプションあり）
