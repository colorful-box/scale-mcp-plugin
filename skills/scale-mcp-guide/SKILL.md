---
name: scale-mcp-guide
description: Scale人事評価システムの企業管理者（company_admin）向けMCPツール活用ガイド。テンプレート作成、評価シート配布、マスタ管理、データ分析など65個のscale_*ツールの使い方とワークフローを提供する。ユーザーがScale、人事評価、テンプレート、配布、評価期間、マスタ管理などの管理操作を依頼したときに参照する。
---

# Scale MCP ツール活用ガイド

Scale人事評価システム（scale-hr.io）の82個のMCPツールを効果的に使うためのガイド。

## MCP サーバー接続設定

`.claude/mcp.json` に以下を設定する:

```json
{
  "mcpServers": {
    "scale": {
      "type": "http",
      "url": "https://mcp.scale-hr.io/mcp"
    }
  }
}
```

初回接続時にブラウザで OAuth 認証画面が開く。承認後は自動認証。

## 使用タイミング

- ユーザーが「Scale」「人事評価」「評価シート」「配布」「評価期間」に言及したとき
- 評価業務のワークフローを組み立てるとき
- テンプレートの設定を確認・調整するとき
- 評価データの分析やレポート作成を依頼されたとき

## ロール別ガイド

| ロール | ツール数 | 操作範囲 |
|--------|---------|---------|
| `company_admin` | 65 | 企業内の全データCRUD |
| `general` | 15 | 自分の評価シート・プロフィール・ダッシュボード |

サーバーがロールに応じてツール一覧を自動フィルタリングする。権限エラー時は他のユーザー情報を開示せず「管理者に問い合わせてください」と案内すること。

## 最初に実行すべきツール

1. `scale_whoami` — 認証状態・ユーザー情報・ロールを確認
2. `scale_dashboard_summary` — 現在の評価状況を把握

## 主要ワークフローパターン

### 評価制度セットアップ（準備→確認→配布）

**準備フェーズ:**
1. マスタ作成: `scale_create_position` / `scale_create_occupation` / `scale_create_affiliation`
2. 評点表記作成: `scale_create_score_notation`（5段階評価など）
3. 評価期間作成: `scale_create_evaluation_period`
4. テンプレート作成: `scale_create_template`（axes に評価軸・項目を含む）

**確認フェーズ（最重要）:**
5. テンプレート確認: `scale_get_template` で全設定を確認
   - 自己評価/コメント設定、中間評価設定、評価者コメント設定
   - 軸ウェイト合計が100%か
   - 評点表記が正しいか
6. 配布プレビュー: `scale_preview_distribution`

**実行フェーズ:**
7. 配布: `scale_distribute_sheets`

詳細は `references/workflows.md` を参照。

### 提出状況の確認

`scale_dashboard_summary` → `scale_dashboard_evaluation_todos` → `scale_list_evaluation_sheets`（ステータスフィルタ）

### 評価データ分析

`scale_export_scores_csv` / `scale_export_items_csv` → `scale_get_evaluation_analysis`

### 1on1 実施状況

`scale_one_on_one_dashboard` → `scale_list_one_on_one_sessions_admin`

## テンプレート作成時の必須プロセス（最重要）

テンプレートは評価シートの「型」であり、設定が複雑。曖昧なまま作成すると誤ったテンプレートになるため、**必ず以下のプロセスに従って作成前にユーザーと確認する**。

### ステップ1: 入力情報の収集

ユーザーから以下のいずれかで情報を受け取る:

- **Excel 提供**: `references/template_excel_format.md` の仕様に従って読み取る。主に `テンプレート設定` シートから軸設定を取得し、項目は `成果目標（KGI・KPI）`, `バリュー評価４段階`, `WOOP目標設定` 等のシートから取得する
- **口頭説明**: 不足情報を質問で補う

**Excel に `マスター設定` シートがある場合は必ず最初に読み取る**。このシートにはマスター（評点表記・職位・職種・所属・ランク等）の登録方針に関する特記事項が記述される。記述があればテンプレート作成前にその内容に従ってマスターを作成・選択し、他シートの読み取り方針にも反映する（例: 「バリュー評価４段階」シートの詳細列に A/B/C/D の評価内容が書かれていて、それを 4点=A, 3点=B, 2点=C, 1点=D として評点表記マスターに登録する、など）。特記内容を解釈した結果もユーザーに提示して承認を取る。

### ステップ2: 軸設定を表形式で提示して確認

`scale_create_template` を実行する**前に必ず**、各評価軸の設定を以下の表形式でユーザーに提示して承認を取る:

```
### 評価軸の設定内容

| 軸名 | ウェイト | 目標表示 | 軸尺度 | 詳細表示 | 中間評価 | 自己評点 | 自己コメント | 評価者評点 | 評価者コメント | 目標タイプ |
|-----|--------|---------|-------|---------|---------|---------|-----------|----------|-------------|----------|
| 成果 | 70 | ○ | × | ○ | ○ | ○ | ○ | ○ | × | outcome |
| バリュー | 20 | × | × | ○ | ○ | ○ | ○ | ○ | ○ | behavior |
| プロセス | 10 | ○ | × | × | ○ | ○ | ○ | ○ | ○ | other |

ウェイト合計: 100 ✓
この内容で作成してよろしいですか？
```

### ステップ3: 項目設定を表形式で提示して確認

各軸の評価項目も表形式で提示:

```
### 成果軸の評価項目

| # | 項目名 | 目標 | ウェイト | 項目尺度 |
|---|-------|------|---------|---------|
| 1 | 部署目標売上高 | 60000万円 | 50 | 6段階（75%未満〜120%以上） |
| 2 | 部署AEOLUSタイヤ販売本数 | 13,000本 | 25 | 6段階（11,000本未満〜15,000本以上） |
| 3 | 部署新規顧客獲得件数 | 60件以上 | 25 | 6段階（20件以下〜100件以上） |
```

### ステップ4: 確認後に作成、作成後に再確認

ユーザー承認後に `scale_create_template` を実行。作成後は `scale_get_template` で結果を取得し、設定が正しく反映されたかを同じ表形式で最終確認する。

### 典型的な誤りと質問すべきこと

ユーザーが曖昧な指示（例:「営業向けの標準的な評価テンプレートを作って」）をした場合、以下を**必ず質問**する:

| 判断が必要な設定 | 質問例 |
|----------------|--------|
| 軸ごとの目標表示 | 「〇〇軸には目標項目を設定しますか？（成果系はあり、バリュー系はなしが多い）」 |
| 軸尺度 vs 項目尺度 | 「評点スケールは軸全体で共通ですか、項目ごとに違う尺度を使いますか？」 |
| 中間評価 | 「中間評価フェーズは実施しますか？」 |
| 評価者コメント | 「1次・2次評価者のコメント欄は必要ですか？（軸ごとに異なる場合あり）」 |
| 評価者の段階数 | 「評価者は何段階ですか？（1次のみ / 1次・2次 / 1次・2次・3次）」 |

### バリュー系軸の典型パターン（要注意）

「バリュー」「コンピテンシー」「行動評価」のような軸は以下のパターンが多い:
- `show_goal: false`（項目ごとの目標ではなく、詳細に行動基準を記述）
- `show_scale: false` / 項目尺度も使わない（A/B/C/D評価を詳細テキストに記述）
- `objective_type: behavior`
- 詳細（`detail`）に「期待される行動」「評価内容（A/B/C/D）」を記述

ユーザーがバリュー系の軸を指示した場合、これらがデフォルトで良いか確認する。

## テンプレート設定ガイド

テンプレートは評価シートの「型」であり、設定が複雑。作成後は必ず `scale_get_template` で全設定を確認すること。

### 評価軸の主要設定

| 設定 | パラメータ | 説明 |
|------|-----------|------|
| 自己評価 | `final_has_self_score` | 最終評価で自己評点あり/なし |
| 自己コメント | `final_has_self_comment` | 最終評価で自己コメントあり/なし |
| 中間評価 | `has_interim_evaluation` | 中間評価フェーズの有無 |
| 中間自己評価 | `interim_has_self_score` | 中間で自己評点あり/なし |
| 中間自己コメント | `interim_has_self_comment` | 中間で自己コメントあり/なし |
| 評価者コメント | `final_has_evaluator_comment` | 評価者コメント表示 |
| 評点表記 | `score_notation_id` | 評点の表記方法（5段階/10段階等） |
| スケール表示 | `show_scale` | 評点スケール表示の有無 |
| 目標タイプ | `objective_type` | `outcome`(成果) / `behavior`(行動) / `other` |

### 典型パターン

- **標準型**: 自己評価あり、中間評価なし、3次評価
- **管理職型**: 自己評価あり、中間評価あり、コメント重視
- **簡易型**: 自己評価なし、1次評価のみ

各パターンのJSON例は `references/template_settings.md` を参照。

### 制約

- 評価軸: 最大5軸、ウェイト合計100
- 評価項目: 最大100項目/軸
- メモ: 最大5個

## 削除・復元ガイド

### 評価シートの削除

- `scale_bulk_delete_evaluation_sheets` — 論理削除（削除理由を指定可能）
- `scale_list_deleted_sheets` — 削除済みシートの確認
- `scale_restore_deleted_sheet` — 復元可能

### テンプレート・マスタの削除

- `scale_delete_template` — 評価シートに使用中のテンプレートは削除不可
- 職位/職種/所属/ランク/評点表記 — ユーザーや評価軸に紐付いていると削除不可
- **削除前に必ず使用状況を確認すること**

## ツールカテゴリ早見表

| カテゴリ | ツール数 | プレフィックス |
|---------|---------|--------------|
| 認証 | 1 | `scale_whoami` |
| ユーザー管理 | 5 | `scale_*_user` |
| ダッシュボード | 5 | `scale_dashboard_*` |
| プロフィール | 2 | `scale_*_profile` |
| 企業設定 | 2 | `scale_*_company_settings` |
| 評価期間 | 5 | `scale_*_evaluation_period` |
| 職位/職種/所属 | 15 | `scale_*_position/occupation/affiliation` |
| ランク/評点表記 | 10 | `scale_*_rank/score_notation` |
| ピッチ金額 | 3 | `scale_*_pitch_amount_setting/matrix` |
| テンプレート | 6 | `scale_*_template` |
| 評価シート | 12+ | `scale_*_evaluation_sheet` |
| 配布 | 3 | `scale_*_distribute/distribution` |
| 個人シート | 1 | `scale_list_my_evaluation_sheets` |
| 掲示板 | 3 | `scale_*_bulletin` |
| 1on1 | 7 | `scale_*_one_on_one_*` |
| 分析 | 1 | `scale_get_evaluation_analysis` |

## 注意事項

- 配布前に必ず `scale_preview_distribution` でプレビューすること
- `scale_update_evaluation_sheet_status` / `scale_bulk_update_status` は「神の手」機能（通常ワークフローを迂回）。慎重に使用すること。`status` の有効値（16値）は `references/sheet_statuses.md` を参照
- CSVエクスポートの `sheet_ids` はカンマ区切り文字列（例: `"1,2,3"`）
- `scale_get_evaluation_analysis` は非同期ジョブで最大180秒かかる場合がある
- ページネーション対応ツールは `page` と `page_size` を適切に使うこと
- 削除前に使用状況を確認し、削除理由を記録すること

## 詳細リファレンス

- `references/workflows.md` — 詳細なワークフローパターン
- `references/template_settings.md` — テンプレート設定の全パラメータと典型パターン
- `references/template_excel_format.md` — Excel 入力フォーマット仕様
- `references/tool_reference.md` — カテゴリ別ツール詳細リファレンス
- `references/sheet_statuses.md` — 評価シートステータス値（16値）の意味とフロー遷移
