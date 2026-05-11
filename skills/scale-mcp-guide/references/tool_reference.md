# ツールリファレンス

## 認証（1ツール）

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_whoami` | general | 認証状態・ユーザー情報・ロールを取得 |

## ユーザー管理（5ツール）

| ツール | 説明 | 主要パラメータ |
|--------|------|--------------|
| `scale_list_users` | 一覧（フィルタ・ページネーション） | `search_name`, `search_email`, `occupation_id`, `position_id`, `affiliation_id`, `roles` |
| `scale_get_user` | 詳細 | `user_id` |
| `scale_create_user` | 作成 | `employee_code`(必須), `name`(必須), `password`(必須), `roles`(必須), `email`, `occupation_id`, `position_id`, `affiliation_id` |
| `scale_update_user` | 更新（退職処理は `is_retired=True` を指定） | `user_id`(必須), 変更フィールドのみ指定。`is_retired` で退職処理（ログイン不可・退職者検索ヒット） |
| `scale_delete_user` | 論理削除（誤登録の取り消し用、退職処理ではない） | `user_id` |

全て `company_admin` ロール。パスワードはランダム生成を推奨（チャット履歴に残るため）。

**退職と論理削除の使い分け:**
- 退職処理 → `scale_update_user(user_id, is_retired=True)`（データ保持・退職者検索でヒット・ログイン拒否）
- 誤登録の取り消し → `scale_delete_user(user_id)`（一覧から消える論理削除）

## ダッシュボード（5ツール）

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_dashboard_summary` | general | 評価状況の全体サマリー |
| `scale_dashboard_evaluation_todos` | general | 未対応の評価TODO |
| `scale_dashboard_my_sheets` | general | 自分の評価シート一覧 |
| `scale_dashboard_action_required` | general | 対応が必要なシート |
| `scale_dashboard_current_period` | general | 当期のシート |

パラメータなし。ログインユーザーの情報を自動取得。

## プロフィール（2ツール）

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_get_profile` | general | 自分のプロフィール |
| `scale_update_profile` | general | プロフィール更新 |

## 企業設定（2ツール）

| ツール | 説明 |
|--------|------|
| `scale_get_company_settings` | 企業設定の取得 |
| `scale_update_company_settings` | 企業設定の更新 |

全て `company_admin` ロール。

## 評価期間（5ツール）

| ツール | 説明 | 主要パラメータ |
|--------|------|--------------|
| `scale_list_evaluation_periods` | 一覧 | `search_name`, `is_active`, `sort_by`, `sort_order` |
| `scale_get_evaluation_period` | 詳細 | `period_id` |
| `scale_create_evaluation_period` | 作成 | `name`(必須), `start_date`(必須), `end_date`(必須), `goal_setting_deadline`, `interim_evaluation_deadline`, `final_evaluation_deadline` |
| `scale_update_evaluation_period` | 更新 | `period_id`(必須), 変更フィールドのみ |
| `scale_delete_evaluation_period` | 削除 | `period_id` |

日付は `YYYY-MM-DD` 形式。全て `company_admin` ロール。

## マスタデータ: 職位・職種・所属（各5ツール、計15ツール）

各マスタ共通のCRUDパターン:

| 操作 | 職位 | 職種 | 所属 |
|------|------|------|------|
| 一覧 | `scale_list_positions` | `scale_list_occupations` | `scale_list_affiliations` |
| 詳細 | `scale_get_position` | `scale_get_occupation` | `scale_get_affiliation` |
| 作成 | `scale_create_position` | `scale_create_occupation` | `scale_create_affiliation` |
| 更新 | `scale_update_position` | `scale_update_occupation` | `scale_update_affiliation` |
| 削除 | `scale_delete_position` | `scale_delete_occupation` | `scale_delete_affiliation` |

削除はユーザーに紐付いている場合は不可。全て `company_admin` ロール。

## ピッチ金額（3ツール）

| ツール | 説明 |
|--------|------|
| `scale_get_pitch_amount_setting` | 現在のピッチ金額設定（mode + 職位×金額一覧）を取得。未設定時は null |
| `scale_get_pitch_amount_matrix` | 職種×職位のマトリックス形式で取得（編集UI向け） |
| `scale_update_pitch_amount_setting` | 一括更新（完全置換）。`mode`(`all`/`individual`) と `amounts: list[{occupation_id, position_id, amount}]` を指定 |

1企業につき設定は1件のみ。`mode='all'`（全職種共通）の場合は `amounts[].occupation_id` を `null`、`mode='individual'`（職種別）の場合は職種IDを指定する。全て `company_admin` ロール。

## ランク設定（5ツール）

| ツール | 説明 | 主要パラメータ |
|--------|------|--------------|
| `scale_list_ranks` | ランク設定一覧 | — |
| `scale_get_rank` | ランク設定詳細 | `rank_setting_id` |
| `scale_create_rank` | ランク設定作成 | `name`(必須), `levels`(必須), `description`(任意・最大500文字) |
| `scale_update_rank` | ランク設定更新（部分更新） | `rank_setting_id`(必須), `name`, `description`, `levels`, `is_active` |
| `scale_delete_rank` | ランク設定削除（評価シートに紐付いていると不可） | `rank_setting_id` |

全て `company_admin` ロール。

### `levels` の指定形式

`list[dict]` 形式。各要素は以下:

| キー | 型 | 必須 | 説明 |
|-----|----|----|------|
| `label` | string | ○ | ランク名（例: `"S"`, `"A"`） |
| `min_score` | number | ○ | 範囲の下限 |
| `max_score` | number | ○ | 範囲の上限 |
| `sort_order` | integer | × | 表示順 |
| `salary_step_change` | integer | × | 号俸増減 |

**バリデーション制約（`levels` 指定時）:**
- 最下位ランクの `min_score` は必ず `0`
- 最上位ランクの `max_score` は必ず `100`
- ランク範囲に隙間や重複があってはならない

例:
```python
levels=[
  {"label": "D", "min_score": 0,  "max_score": 60},
  {"label": "C", "min_score": 60, "max_score": 80},
  {"label": "B", "min_score": 80, "max_score": 90},
  {"label": "A", "min_score": 90, "max_score": 100},
]
```

## 評点表記（5ツール）

| ツール | 説明 |
|--------|------|
| `scale_list_score_notations` | 評点表記一覧 |
| `scale_get_score_notation` | 評点表記詳細 |
| `scale_create_score_notation` | 評点表記作成（`name`, `score_levels`(1-10), `use_custom_labels`, `labels`） |
| `scale_update_score_notation` | 評点表記更新（`labels`, `use_custom_labels`, `is_active` 等） |
| `scale_delete_score_notation` | 評点表記削除（評価軸に紐付いていると不可） |

全て `company_admin` ロール。

### `scale_create_score_notation` のパラメータ詳細

| パラメータ | 必須 | 説明 |
|-----------|-----|------|
| `name` | ○ | 評点表記名 |
| `score_levels` | ○ | 段階数（1〜10の整数） |
| `use_custom_labels` | - | カスタムラベル利用フラグ。未指定なら `labels` の有無から自動推論される |
| `labels` | 条件付 | `dict`（推奨: `{"1": "S", "2": "A"}`）または `list[str]`（`["S","A"]` は 1-origin の dict に自動変換）。`use_custom_labels=True` のとき必須 |

`use_custom_labels=False`（または labels なし＆自動推論で false）なら数値表記（1,2,3…）として作成される。`is_active` は Create には存在せず Update でのみ指定可能。

## テンプレート（6ツール）

| ツール | 説明 | 主要パラメータ |
|--------|------|--------------|
| `scale_list_templates` | 一覧 | `search_name`, `is_active`, `evaluation_period_id` |
| `scale_get_template` | 詳細（軸・項目含む） | `template_id` |
| `scale_create_template` | 作成 | `name`(必須), `axes`(評価軸配列), `show_overall_comment` |
| `scale_update_template` | 更新 | `template_id`(必須), `axes`指定時は全軸置換 |
| `scale_delete_template` | 削除 | 評価シートに使用中は不可 |
| `scale_copy_template` | コピー | `template_id` → 新テンプレート生成 |

`axes` の詳細は `references/template_settings.md` を参照。全て `company_admin` ロール。

## 評価シート（18+ツール）

### 基本操作

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_list_evaluation_sheets` | admin | 一覧（フィルタ・ソート対応） |
| `scale_get_evaluation_sheet` | general | 詳細（`admin_mode=true`で管理者権限取得可） |
| `scale_submit_evaluation_sheet` | general | ワークフロー提出 |
| `scale_reject_evaluation_sheet` | general | 差戻し（`reason`指定可） |
| `scale_calculate_scores` | admin | スコア再計算 |

### 神の手（配布済みシート修正）

配布後の評価シートを管理者権限で強制編集する機能。通常ワークフローを迂回するため慎重に使用。

| ツール | 説明 | 主要パラメータ |
|--------|------|--------------|
| `scale_update_evaluation_sheet_status` | ステータス強制変更 | `sheet_id`, `status`（有効値16値は `references/sheet_statuses.md` 参照） |
| `scale_bulk_update_status` | 一括ステータス変更 | `sheet_ids`, `status`（有効値16値は `references/sheet_statuses.md` 参照） |
| `scale_god_hand_update_axis` | 評価軸の編集 | `sheet_id`, `axis_id`, 任意: `name`, `weight`(0-100), `show_scale`, `scale_labels`, `objective_type`(`outcome`/`behavior`/`other`) |
| `scale_god_hand_update_item` | 評価項目の編集 | `sheet_id`, `item_id`, 任意: `category`(最大100文字), `name`, `detail`, `goal`, `weight`, `show_item_scale`, `item_scale_labels` |
| `scale_god_hand_update_profile` | 被評価者・評価者・期間の変更 | `sheet_id`, 任意: `evaluatee_user_id`, `evaluator_1st/2nd/3rd_user_id`, `evaluation_period_id`, 職種・職位・所属名 |
| `scale_god_hand_create_memo` | メモ追加 | `sheet_id`, `label`(1-100文字), `sort_order` |
| `scale_god_hand_update_memo` | メモ編集 | `sheet_id`, `memo_id`, 任意: `label`, `sort_order` |
| `scale_god_hand_delete_memo` | メモ削除 | `sheet_id`, `memo_id` |

### 削除・復元

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_bulk_delete_evaluation_sheets` | admin | 一括削除（論理削除、`reason`指定可） |
| `scale_list_deleted_sheets` | admin | 削除済みシート一覧 |
| `scale_restore_deleted_sheet` | admin | 削除済みシート復元 |

### CSV エクスポート

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_export_items_csv` | admin | 項目データCSV（`sheet_ids`カンマ区切り） |
| `scale_export_scores_csv` | admin | 評点データCSV |
| `scale_export_memos_csv` | admin | メモデータCSV |

### フィルタパラメータ（scale_list_evaluation_sheets）

`person_name`, `person_type`, `template_id`, `period_id`, `occupation`, `position`, `affiliation`, `statuses`, `incomplete_only`, `overdue`, `due_soon`, `sort_by`, `sort_order`

## 配布（3ツール）

| ツール | 説明 | 主要パラメータ |
|--------|------|--------------|
| `scale_get_distribute_users` | 配布対象ユーザー | `template_id`(必須), `occupation_id`, `position_id`, `affiliation_id`, `exclude_already_distributed` |
| `scale_preview_distribution` | 配布プレビュー | `template_id`(必須), `user_ids`(必須) |
| `scale_distribute_sheets` | 配布実行 | `template_id`(必須), `user_ids`(必須) |

**配布前に必ずプレビューすること。** 全て `company_admin` ロール。

## 個人評価シート（1ツール）

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_list_my_evaluation_sheets` | general | 自分の評価シート一覧 |

## 掲示板（3ツール）— 閲覧のみ

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_list_bulletins` | admin | 掲示板一覧 |
| `scale_get_bulletin` | admin | 掲示板詳細 |
| `scale_get_bulletins_by_location` | general | 表示場所別取得 |

作成・更新・削除は Web UI から行う。

## 1on1（7ツール）— 閲覧のみ

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_list_one_on_one_schedules` | admin | スケジュール一覧 |
| `scale_list_one_on_one_sessions_admin` | admin | セッション一覧（管理者用） |
| `scale_list_one_on_one_sessions` | general | セッション一覧（ユーザー用） |
| `scale_get_one_on_one_session` | general | セッション詳細 |
| `scale_one_on_one_dashboard` | admin | 1on1ダッシュボード統計 |
| `scale_list_one_on_one_templates` | admin | テンプレート一覧 |
| `scale_get_one_on_one_template` | admin | テンプレート詳細 |

作成・更新・削除は Web UI から行う。

## 分析（1ツール）

| ツール | ロール | 説明 |
|--------|--------|------|
| `scale_get_evaluation_analysis` | admin | AI評価結果分析（非同期、最大180秒） |

パラメータ: `sheet_ids`(必須), `analysis_type`
