# ワークフローパターン

## 1. 評価制度セットアップ（準備→確認→配布）

### 準備フェーズ

#### Step 1: マスタデータ作成

```
scale_create_position      → 職位（部長、課長、一般等）
scale_create_occupation    → 職種（営業、開発、管理等）
scale_create_affiliation   → 所属（営業部、開発部等）
scale_create_score_notation → 評点表記（5段階、10段階等）
scale_create_rank          → ランク設定（S/A/B/C/D等）
```

各マスタの一覧確認: `scale_list_positions` / `scale_list_occupations` / `scale_list_affiliations`

#### Step 2: 評価期間作成

```
scale_create_evaluation_period
  name: "2026年度上期"
  start_date: "2026-04-01"
  end_date: "2026-09-30"
  goal_setting_deadline: "2026-04-30"      # 目標設定期限
  interim_evaluation_deadline: "2026-07-15" # 中間評価期限（任意）
  final_evaluation_deadline: "2026-09-30"   # 最終評価期限
```

#### Step 3: テンプレート作成

既存テンプレートをコピーして調整するか、新規作成する:

- コピー: `scale_copy_template` → `scale_update_template` で調整
- 新規: `scale_create_template` で axes（評価軸）・items（項目）を含めて作成

テンプレート設定の詳細は `references/template_settings.md` を参照。

### 確認フェーズ（最重要）

#### Step 4: テンプレート確認

```
scale_get_template(template_id=X)
```

以下を必ず確認する:

| 確認項目 | 確認方法 |
|---------|---------|
| 評価軸のウェイト合計が100% | 全軸の `weight` を合計 |
| 自己評価設定 | `final_has_self_score` / `final_has_self_comment` |
| 中間評価設定 | `has_interim_evaluation` と関連フラグ |
| 評価者コメント設定 | `final_has_evaluator_comment` |
| 評点表記 | `score_notation_id` が正しいマスタを参照しているか |
| 評価項目の過不足 | 各軸の `items` の内容 |

問題がある場合は `scale_update_template` で修正。

#### Step 5: 配布プレビュー

```
scale_get_distribute_users(template_id=X)          → 対象ユーザーの確認
scale_preview_distribution(template_id=X, user_ids=[...]) → 配布内容のプレビュー
```

- `exclude_already_distributed=true` で既に配布済みのユーザーを除外可能
- `occupation_id` / `position_id` / `affiliation_id` でフィルタリング可能

### 実行フェーズ

#### Step 6: 配布

```
scale_distribute_sheets(template_id=X, user_ids=[...])
```

配布後は `scale_list_evaluation_sheets` で配布結果を確認。

## 2. テンプレート確認・調整

テンプレートの設定を確認し、必要に応じて調整するフロー:

```
scale_list_templates                 → テンプレート一覧
scale_get_template(template_id=X)    → 詳細確認
scale_update_template(template_id=X, axes=[...]) → 軸・項目の更新
scale_get_template(template_id=X)    → 更新後の再確認
```

`scale_update_template` で `axes` を指定すると、既存の軸・項目は完全に置き換えられる。部分更新ではないため注意。

## 3. 削除・復元操作

### 評価シートの削除

```
scale_list_evaluation_sheets(...)           → 削除対象の特定
scale_bulk_delete_evaluation_sheets(
  sheet_ids=[1, 2, 3],
  reason="テスト配布のため削除"             → 論理削除（削除理由付き）
)
scale_list_deleted_sheets                   → 削除済みシートの確認
scale_restore_deleted_sheet(sheet_id=1)      → 誤削除時の復元
```

### テンプレートの削除

```
scale_list_evaluation_sheets(template_id=X)  → テンプレートが使用中か確認
scale_delete_template(template_id=X)         → 使用中でなければ削除可能
```

使用中のテンプレートはエラーになるため、先に評価シートを削除する必要がある。

### マスタデータの削除

職位・職種・所属・ランク・評点表記は、ユーザーや評価軸に紐付いていると削除不可:

```
scale_list_users(position_id=X)              → 職位を使用中のユーザーを確認
scale_delete_position(position_id=X)         → 使用中でなければ削除可能
```

### 評価期間の削除

```
scale_delete_evaluation_period(period_id=X)  → 削除前に使用状況を確認
```

## 4. 日常管理

### 提出状況の確認

```
scale_dashboard_summary               → 全体サマリー（提出率等）
scale_dashboard_evaluation_todos       → 未対応シートの一覧
scale_list_evaluation_sheets(
  statuses="self_evaluating",          → ステータスでフィルタ
  incomplete_only=true                 → 未完了のみ
)
```

### 差戻し対応

```
scale_get_evaluation_sheet(sheet_id=X)        → シートの詳細確認
scale_reject_evaluation_sheet(
  sheet_id=X,
  reason="目標の記述が不十分です"              → 差戻し（理由付き）
)
```

### 一括ステータス変更（神の手）

通常のワークフローを迂回するため慎重に:

```
scale_bulk_update_status(
  sheet_ids=[1, 2, 3],
  status="completed"
)
```

## 5. データ分析・レポート

### CSVエクスポート

```
scale_export_items_csv(sheet_ids="1,2,3")   → 項目データ
scale_export_scores_csv(sheet_ids="1,2,3")  → 評点データ
scale_export_memos_csv(sheet_ids="1,2,3")   → メモデータ
```

`sheet_ids` を省略すると全シートが対象。カンマ区切りの文字列で指定。

### AI評価分析

```
scale_get_evaluation_analysis(
  sheet_ids=[1, 2, 3],
  analysis_type="summary"              → 非同期ジョブ（最大180秒）
)
```

### 横断集計パターン

```
scale_list_evaluation_sheets(...)      → 全シートデータ
scale_list_users(...)                  → ユーザー情報（所属・職位）
scale_list_affiliations(...)           → 部門マスタ
→ データを組み合わせて部門別・ランク別などの集計
```

## 6. 一般ユーザー向け

### 自分のシートを確認

```
scale_dashboard_my_sheets              → 自分の評価シート一覧
scale_get_evaluation_sheet(sheet_id=X) → シートの詳細
```

### 評価シートの提出

```
scale_submit_evaluation_sheet(sheet_id=X) → ワークフローに沿って提出
```

### 1on1の確認

```
scale_list_one_on_one_sessions         → 自分のセッション一覧
scale_get_one_on_one_session(session_id=X) → セッション詳細
```
