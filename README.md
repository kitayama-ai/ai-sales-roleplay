# 🎯 AI営業ロープレ v2

> Gemini Live APIを使った音声ベースの営業ロールプレイ練習ツール

**🔗 [デモ（GitHub Pages）](https://kitayama-ai.github.io/ai-sales-roleplay/)**

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![Gemini API](https://img.shields.io/badge/Gemini_Live_API-4285F4?style=flat&logo=google&logoColor=white)

---

## 💡 概要

AIが顧客役を演じ、営業担当者がマイクで会話。終了後に**10軸×10点の100点満点**で自動採点し、弱点をピンポイント練習できます。

### こんな人に

- 営業チームのクロージング力を底上げしたい
- 新人営業の練習環境が欲しい
- 失注商談を再現して改善点を掴みたい

---

## ✨ 主な機能

| 機能                    | 説明                                                                                                         |
| ----------------------- | ------------------------------------------------------------------------------------------------------------ |
| 🎙️ **音声ロープレ**     | Gemini Live APIでリアルタイム音声会話。AIが日本語で顧客役を演じる                                            |
| 📊 **10軸採点**         | 権威性・損失回避・リフレーミング・アンカリング・クロージング・ヒアリング・ラポール・反論処理・テンポ・提案力 |
| 🔬 **evidence付き採点** | 各軸に「根拠発言の引用」と「減点理由」を明示。`temperature:0`で同じログなら同じ点数                          |
| 🎯 **弱点ドリル**       | 採点結果のワースト3からワンクリックで弱点ロープレ開始                                                        |
| 👥 **6種のペルソナ**    | 「高い」「比較したい」「忙しい」「配偶者相談」「効果ある？」「実商談トレース」                               |
| 📋 **GAS連携**          | 失注商談データをGASから自動取得→顧客パターン再現ロープレ                                                     |
| 📈 **利用管理**         | 担当者別の月間練習回数・平均スコア・合計時間をダッシュボード表示                                             |
| 📥 **CSVエクスポート**  | 利用データをCSVダウンロード                                                                                  |

---

## 🚀 使い方

### 1. API Keyを取得

[Google AI Studio](https://aistudio.google.com/apikey) でGemini API Keyを取得

### 2. アクセス

https://kitayama-ai.github.io/ai-sales-roleplay/

### 3. 設定して開始

1. 担当者名を入力
2. 弱点・ペルソナ・難易度・シナリオを選択
3. API Keyを入力
4. **「🎙️ ロープレ開始」** をクリック
5. マイク許可 → AIと音声で会話開始！

### 4. 終了＆採点

1. **「⏹ 終了」** で会話終了
2. **「📊 採点」** で10軸自動採点
3. 弱点の **「🎯」** ボタンでピンポイント練習

---

## 📋 GAS連携（オプション）

失注商談データを自動取得してロープレに活用できます。

### セットアップ

1. GASエディタで `doGet.gs` を追加（コードは `HANDOFF.md` 参照）
2. 「デプロイ」→「新しいデプロイ」→ ウェブアプリ → アクセス: 全員
3. URLをコピー
4. ロープレアプリの「GAS Web App URL」に貼り付け
5. **「📋 失注データ取得」** をクリック

---

## 🏗️ 技術スタック

- **音声入出力**: Web Audio API + AudioWorklet（PCM 16kHz）
- **AI対話**: Gemini Live API（WebSocket）
- **採点**: Gemini 2.0 Flash REST API（`temperature: 0`）
- **データ保存**: localStorage
- **デプロイ**: GitHub Pages（サーバー不要）

---

## 📁 ファイル構成

```
index.html    ← 全機能入りの単一ファイル（HTML+CSS+JS）
HANDOFF.md    ← 技術引き継ぎ書（詳細仕様）
README.md     ← このファイル
```

---

## 📝 採点ルブリック

| 点数 | レベル   | 基準                             |
| ---- | -------- | -------------------------------- |
| 0-2  | 行動なし | 該当する行動が全くない           |
| 3-4  | 不十分   | 試みはあるが顧客に響いていない   |
| 5-6  | 基本OK   | 基本はできているが深みがない     |
| 7-8  | 効果的   | 顧客の反応を引き出せている       |
| 9-10 | 卓越     | 顧客の感情が明確に動いた証拠あり |

---

## 📄 ライセンス

Private Use
