# 私の轍 — Watashi no Wadachi

> 自分の後に路はできる。走った記録をプルで集め、自分像を浮かび上がらせる。

**QQ #038 / watashi-no-wadachi_neo-garmin-trucker**  
**QQ Series** by [qq-collective](https://github.com/qq-collective)

---

## 概要

RUNNETのレース戦歴・GarminのトレーニングCSV・睡眠データ・HRVを統合し、  
ランナーとしての自分を可視化するシングルHTMLアプリ。

去年の「入力して管理する」プッシュ型とは違う。  
**すでにある記録を集めて見せる、プル型の自己chronicle。**

> 道具は使う人のエネルギーを守るために存在する。  
> マラソンのための記録アプリが、マラソンの情熱を奪ってはいけない。

---

## 機能

| タブ | 内容 |
|------|------|
| 概観 | プロフィール統計・フルマラソン推移・全レースタイムライン |
| 練習×睡眠 | 月間走行距離×睡眠スコア・HRV日次推移・レース前4週分析・心拍推移 |
| レース戦歴 | 全出走記録テーブル（PB自動マーク） |
| 診断 | A/B/C/D判定＋データに基づく総合所見 |
| チャット | データ参照AI（Anthropic Claude API） |
| データ更新 | Garmin／睡眠／HRV CSVドロップ → グラフ即時反映・手動レース追加 |

---

## データソース

| データ | 取得方法 | 更新頻度 |
|--------|---------|---------|
| レース記録 | RUNNETマイページよりコピペ | レース後 |
| 練習ログ | Garmin Connect「CSVをエクスポート」 | 月1回 |
| 睡眠データ | Garmin Connect 睡眠ページ「CSVをエクスポート」 | 月1回 |
| HRV | Garmin Connect HRVステータス「CSVをエクスポート」 | 月1回 |

すべて手動エクスポート。外部APIへの自動連携なし。  
**セキュリティファースト：認証情報はアプリに渡さない設計。**

---

## 使い方

1. `watashi_no_wadachi.html` をブラウザで開く
2. チャット機能を使う場合は「チャット」タブでAnthropicのAPIキーを入力
3. データ更新は「データ更新」タブからCSVをドロップ → グラフ即時反映
4. RUNNETにないレースは手動追加フォームから登録

### データ更新ルーティン（月1回）

```
Garmin Connect → アクティビティ CSVエクスポート
  → パワークエリで整形（トレッドミル除外・日付YYYY-MM-DD統一）
  → wadachi_garmin.csv として保存 → アプリにドロップ

Garmin Connect 睡眠ページ → CSVエクスポート（1年タブ）
  → wadachi_sleep.csv として保存 → アプリにドロップ

Garmin Connect HRVステータス → CSVエクスポート
  ※年またぎ注意：1ファイル1年以内に収める
  → wadachi_hrv.csv として保存 → アプリにドロップ
```

### パワークエリ構成

```
📁 wadachi_data/
  📁 garmin_raw/   ← Garmin アクティビティCSV
  📁 sleep_raw/    ← 睡眠データCSV
  📁 hrv_raw/      ← HRVステータスCSV（年またぎNG）
  📄 wadachi_master.xlsx  ← クエリ本体（4クエリ）
      wadachi_garmin / wadachi_sleep / wadachi_hrv / wadachi_meta
```

詳細は `wadachi_powerquery.md` を参照。

### HRVファイル命名規則

```
✅ HRVステータス_20251101_1231.csv  ← 年内で完結
✅ HRVステータス_20260101_0504.csv  ← 年内で完結
❌ HRVステータス_20251216_0112.csv  ← 年またぎNG・2ファイルに分割する
```

### APIキーについて
- [Anthropic Console](https://console.anthropic.com/) で取得
- `sk-ant-` から始まるキー
- localStorageに保存（外部送信なし）

---

## 技術スタック

- Vanilla JS / 単一HTMLファイル
- Chart.js 4.4.1（CDN）
- Google Fonts（Noto Serif JP / Space Mono）
- Anthropic Claude API（`claude-sonnet-4-20250514`）
- GitHub Pages対応
- データ整形：Microsoft Excel パワークエリ（M言語）

---

## Garminデータ取得状況

| 期間 | 件数 | 状態 |
|------|------|------|
| 2018-05 〜 2021-03 | 580件 | 取得済み |
| 2021-03 〜 2024-10 | — | 未取得（スクロール問題） |
| 2024-10 〜 2026-04 | 280件 | 取得済み |

---

## ロードマップ

- [x] v0.1.0 — 6タブ基本動作・全データ統合・チャット
- [x] v0.2.0 — CSVドロップでグラフ即更新・手動レース追加・トレッドミル自動除外
- [x] v0.2.1 — HRV日次グラフ追加（ベースライン帯・割れ警告）
- [x] v0.2.2 — パワークエリ整備（wadachi_garmin / sleep / hrv / meta）・HRV年またぎ対応
- [ ] v0.3.0 — 主観タグ・トレラン分類・メモ機能
- [ ] v0.4.0 — 2021〜2024年の空白期間Garminデータ統合
- [ ] v1.0.0 — Webデータ検索連携チャット

---

## 設計哲学

> 「入力する」のではなく「浮かび上がらせる」。  
> データは走ることで自然に積まれている。  
> アプリはそれを見えるようにするだけでいい。

月1回の手動エクスポートはあえて残している。  
エクスポートの一瞬に「先月どうだったか」を感じることが、  
主観タグを入れる動機になり、次のレース戦略につながる。  
**全自動化は、この小さな振り返りを奪う。**

---

## QQ Seriesについて

セナリ学院発、日常の「もやもや」をAIとの対話でアプリに結晶化するプロジェクト。  
すべて単一HTMLファイル、Vanilla JS、GitHub Pagesで動く。

→ [qq-collective](https://github.com/qq-collective)
