# テンプレート設定リファレンス

## テンプレート全体設定

| パラメータ | 型 | デフォルト | 説明 |
|-----------|---|----------|------|
| `name` | string | 必須 | テンプレート名（最大200文字） |
| `description` | string | なし | 説明 |
| `is_active` | boolean | true | 有効/無効 |
| `evaluation_period_id` | integer | なし | 紐付ける評価期間 |
| `show_overall_comment` | boolean | false | 総合コメント欄の表示 |
| `evaluation_steps` | integer | 2 | 評価段階数（1-3） |
| `calculation_method` | string | "points" | 計算方法: "points" or "rank" |
| `rank_setting_id` | integer | なし | ランク設定ID（rank方式時） |
| `memos` | list[string] | [] | メモラベル（最大5個） |

## 評価軸（Axis）設定

### 基本情報

| パラメータ | 型 | デフォルト | 説明 |
|-----------|---|----------|------|
| `name` | string | 必須 | 軸名（最大100文字） |
| `weight` | integer | 0 | ウェイト（0-100、全軸合計100） |
| `sort_order` | integer | 0 | 表示順序 |

### 表示制御

| パラメータ | 型 | デフォルト | 説明 |
|-----------|---|----------|------|
| `show_category` | boolean | true | カテゴリ欄を表示 |
| `show_detail` | boolean | true | 詳細欄を表示 |
| `show_goal` | boolean | true | 目標欄を表示 |

### 評点設定

| パラメータ | 型 | デフォルト | 説明 |
|-----------|---|----------|------|
| `score_notation_id` | integer | 必須 | 評点表記マスタのID |
| `score_levels` | integer | 5 | 評点段階数（1-10） |
| `show_scale` | boolean | true | 評点スケールバーを表示 |
| `scale_labels` | dict | なし | カスタムラベル（例: `{"1": "D", "2": "C", "3": "B", "4": "A", "5": "S"}`） |

### 最終評価: 自己評価設定

| パラメータ | 型 | デフォルト | 説明 |
|-----------|---|----------|------|
| `final_has_self_score` | boolean | true | 自己評点の入力欄を表示 |
| `final_has_self_comment` | boolean | true | 自己コメント欄を表示 |

### 最終評価: 評価者設定

| パラメータ | 型 | デフォルト | 説明 |
|-----------|---|----------|------|
| `final_has_evaluator_score` | boolean | true | 評価者の評点入力欄を表示 |
| `final_has_evaluator_comment` | boolean | true | 評価者コメント欄を表示 |

### 中間評価設定

| パラメータ | 型 | デフォルト | 説明 |
|-----------|---|----------|------|
| `has_interim_evaluation` | boolean | false | 中間評価フェーズの有効化 |
| `interim_has_self_score` | boolean | true | 中間: 自己評点 |
| `interim_has_self_comment` | boolean | true | 中間: 自己コメント |
| `interim_has_evaluator_score` | boolean | true | 中間: 評価者評点 |
| `interim_has_evaluator_comment` | boolean | true | 中間: 評価者コメント |

### その他

| パラメータ | 型 | デフォルト | 説明 |
|-----------|---|----------|------|
| `allow_evaluatee_edit_all` | boolean | false | 被評価者が目標設定時に全項目を編集可能にする |
| `objective_type` | string | "other" | `outcome`(成果目標) / `behavior`(行動目標) / `other` |

## 評価項目（Item）設定

| パラメータ | 型 | デフォルト | 説明 |
|-----------|---|----------|------|
| `name` | string | 必須 | 項目名（最大255文字） |
| `category` | string | なし | カテゴリ |
| `detail` | string | なし | 詳細説明 |
| `goal` | string | なし | 目標テンプレート |
| `weight` | integer | 1 | 項目ウェイト |
| `sort_order` | integer | 0 | 表示順序 |
| `show_item_scale` | boolean | false | 項目レベルのスケール表示 |
| `item_scale_labels` | dict | なし | 項目レベルのスケールラベル |

### 項目レベルのスコア表示制御

| パラメータ | デフォルト | 説明 |
|-----------|----------|------|
| `show_self_score` | true | 自己評点の表示 |
| `show_1st_score` | true | 1次評価者の評点表示 |
| `show_2nd_score` | true | 2次評価者の評点表示 |
| `show_3rd_score` | true | 3次評価者の評点表示 |
| `show_interim_self_score` | true | 中間: 自己評点の表示 |
| `show_interim_1st_score` | true | 中間: 1次評価者評点 |
| `show_interim_2nd_score` | true | 中間: 2次評価者評点 |
| `show_interim_3rd_score` | true | 中間: 3次評価者評点 |
| `show_1st_comment` | true | 1次評価者コメント表示 |
| `show_2nd_comment` | true | 2次評価者コメント表示 |
| `show_3rd_comment` | true | 3次評価者コメント表示 |

## 典型パターン例

### 標準型（自己評価あり、中間評価なし、3次評価）

```json
{
  "name": "2026年度 標準評価テンプレート",
  "show_overall_comment": true,
  "axes": [
    {
      "name": "業績評価",
      "weight": 60,
      "score_notation_id": 1,
      "show_scale": true,
      "final_has_self_score": true,
      "final_has_self_comment": true,
      "final_has_evaluator_score": true,
      "final_has_evaluator_comment": true,
      "has_interim_evaluation": false,
      "objective_type": "outcome",
      "items": [
        {"name": "売上目標達成", "weight": 3},
        {"name": "新規顧客開拓", "weight": 2},
        {"name": "業務改善提案", "weight": 1}
      ]
    },
    {
      "name": "行動評価",
      "weight": 40,
      "score_notation_id": 1,
      "show_scale": true,
      "final_has_self_score": true,
      "final_has_self_comment": true,
      "final_has_evaluator_score": true,
      "final_has_evaluator_comment": true,
      "has_interim_evaluation": false,
      "objective_type": "behavior",
      "items": [
        {"name": "チームワーク", "weight": 1},
        {"name": "リーダーシップ", "weight": 1},
        {"name": "コミュニケーション", "weight": 1}
      ]
    }
  ]
}
```

### 管理職型（中間評価あり、コメント重視）

```json
{
  "name": "管理職評価テンプレート",
  "show_overall_comment": true,
  "axes": [
    {
      "name": "組織成果",
      "weight": 50,
      "score_notation_id": 1,
      "final_has_self_score": true,
      "final_has_self_comment": true,
      "final_has_evaluator_comment": true,
      "has_interim_evaluation": true,
      "interim_has_self_score": true,
      "interim_has_self_comment": true,
      "interim_has_evaluator_comment": true,
      "objective_type": "outcome",
      "items": [
        {"name": "部門目標達成率"},
        {"name": "人材育成"},
        {"name": "コスト管理"}
      ]
    },
    {
      "name": "マネジメント力",
      "weight": 50,
      "score_notation_id": 1,
      "final_has_self_score": true,
      "final_has_self_comment": true,
      "final_has_evaluator_comment": true,
      "has_interim_evaluation": true,
      "interim_has_self_comment": true,
      "objective_type": "behavior",
      "items": [
        {"name": "戦略立案"},
        {"name": "部下指導"},
        {"name": "組織風土づくり"}
      ]
    }
  ]
}
```

### 簡易型（自己評価なし、1次評価のみ）

```json
{
  "name": "簡易評価テンプレート",
  "show_overall_comment": false,
  "axes": [
    {
      "name": "総合評価",
      "weight": 100,
      "score_notation_id": 1,
      "final_has_self_score": false,
      "final_has_self_comment": false,
      "final_has_evaluator_score": true,
      "final_has_evaluator_comment": true,
      "has_interim_evaluation": false,
      "items": [
        {"name": "業務遂行能力", "weight": 1},
        {"name": "勤務態度", "weight": 1}
      ]
    }
  ]
}
```

### 目標管理型（MBO: 目標設定→最終評価）

```json
{
  "name": "目標管理（MBO）テンプレート",
  "show_overall_comment": true,
  "axes": [
    {
      "name": "個人目標",
      "weight": 70,
      "score_notation_id": 1,
      "show_goal": true,
      "final_has_self_score": true,
      "final_has_self_comment": true,
      "final_has_evaluator_score": true,
      "final_has_evaluator_comment": true,
      "has_interim_evaluation": false,
      "allow_evaluatee_edit_all": true,
      "objective_type": "outcome",
      "items": []
    },
    {
      "name": "コンピテンシー",
      "weight": 30,
      "score_notation_id": 1,
      "show_goal": false,
      "final_has_self_score": true,
      "final_has_self_comment": false,
      "has_interim_evaluation": false,
      "objective_type": "behavior",
      "items": [
        {"name": "問題解決力"},
        {"name": "対人関係力"},
        {"name": "自己成長"}
      ]
    }
  ]
}
```

## 制約事項

| 制約 | 上限 |
|------|------|
| 評価軸数 | 最大5軸/テンプレート |
| 評価項目数 | 最大100項目/軸 |
| メモ数 | 最大5個/テンプレート |
| 軸ウェイト合計 | 100（全軸の合計） |
| 評点段階数 | 1-10 |
| テンプレート名 | 最大200文字 |
| 軸名 | 最大100文字 |
| 項目名 | 最大255文字 |
