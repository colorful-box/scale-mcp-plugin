# 評価シートステータス値とフロー遷移

評価シートの `status` フィールドは、評価フローのどの段階にあるかを表す16値の文字列。`scale_update_evaluation_sheet_status` / `scale_bulk_update_status`（神の手）の `status` パラメータ、および `scale_list_evaluation_sheets` の `statuses` フィルタで使用する。

## ステータス一覧

| # | 値 | 日本語名 | フェーズ | 編集権限 | 主な使用場面 |
|---|----|--------|--------|--------|----------|
| 1 | `before_goal_setting` | 目標設定前 | 配布直後 | - | 配布直後の初期状態。被評価者が目標入力を開始する前 |
| 2 | `goal_setting_evaluatee` | 目標設定中（被評価者） | 目標設定 | 被評価者 | 被評価者が目標・評価項目を入力中 |
| 3 | `goal_approval_1st` | 目標承認依頼中（1次評価者） | 目標設定 | 1次評価者 | 被評価者が提出後、1次評価者が承認待ち |
| 4 | `goal_approval_2nd` | 目標承認依頼中（2次評価者） | 目標設定 | 2次評価者 | 1次承認後、2次評価者が承認待ち |
| 5 | `goal_approval_3rd` | 目標承認依頼中（3次評価者） | 目標設定 | 3次評価者 | 2次承認後、3次評価者が承認待ち |
| 6 | `goal_completed` | 目標設定完了 | 目標設定 | - | 目標が確定し、評価期間開始待ち |
| 7 | `interim_self_evaluation` | 中間自己評価中 | 中間評価 | 被評価者 | 被評価者が中間の自己評価を入力中 |
| 8 | `interim_evaluation_1st` | 中間評価（1次評価者） | 中間評価 | 1次評価者 | 1次評価者が中間評価を入力中 |
| 9 | `interim_evaluation_2nd` | 中間評価（2次評価者） | 中間評価 | 2次評価者 | 2次評価者が中間評価を入力中 |
| 10 | `interim_evaluation_3rd` | 中間評価（3次評価者） | 中間評価 | 3次評価者 | 3次評価者が中間評価を入力中 |
| 11 | `interim_completed` | 中間評価完了 | 中間評価 | - | 中間評価確定。期末評価開始待ち |
| 12 | `self_evaluation` | 自己評価中（最終） | 最終評価 | 被評価者 | 被評価者が期末の自己評価を入力中 |
| 13 | `evaluation_1st` | 最終評価（1次評価者） | 最終評価 | 1次評価者 | 1次評価者が最終評価を入力中 |
| 14 | `evaluation_2nd` | 最終評価（2次評価者） | 最終評価 | 2次評価者 | 2次評価者が最終評価を入力中 |
| 15 | `evaluation_3rd` | 最終評価（3次評価者） | 最終評価 | 3次評価者 | 3次評価者が最終評価を入力中 |
| 16 | `completed` | 評価完了 | 最終評価 | - | 全評価確定。フロー終了 |

## 通常ワークフロー遷移

`scale_submit_evaluation_sheet`（提出）の正常遷移は以下。評価ステップ数（1〜3次）と評価軸の `has_interim_evaluation` / 自己評価フラグの設定により分岐する。

```
[配布]
  ↓
before_goal_setting
  ↓ (被評価者が目標入力開始)
goal_setting_evaluatee
  ↓ (提出)
goal_approval_1st → goal_approval_2nd → goal_approval_3rd → goal_completed
                                                              ↓
                                            (中間評価ありの場合)
                                                              ↓
                                          interim_self_evaluation
                                                              ↓ (提出)
                                          interim_evaluation_1st → ..._2nd → ..._3rd → interim_completed
                                                                                          ↓
                                                                                self_evaluation
                                                                                          ↓ (提出)
                                                                  evaluation_1st → ..._2nd → ..._3rd → completed
```

中間評価がない場合は `goal_completed` から直接 `self_evaluation` に遷移する。

## 神の手（強制変更）の使用ガイド

`scale_update_evaluation_sheet_status` と `scale_bulk_update_status` は通常ワークフローを迂回して任意のステータスに強制変更できる。以下の制約に注意。

### 評価軸設定との整合性

サーバー側で以下のバリデーションが行われ、矛盾するとHTTP 400エラーが返る。

- **中間評価ステータス（`interim_*`）への変更**: シートの評価軸に `has_interim_evaluation=true` が設定されている必要がある。
- **自己評価ステータス（`self_evaluation` / `interim_self_evaluation`）への変更**: シートの評価軸に「自己評点あり」または「本人コメントあり」のフラグがいずれか1つでも `true` である必要がある。具体的には `self_evaluation` の場合は `final_has_self_score` または `final_has_self_comment`、`interim_self_evaluation` の場合は `interim_has_self_score` または `interim_has_self_comment` のいずれかが `true`。

### 一括更新のトランザクション

`scale_bulk_update_status` は**全件成功または全件ロールバック**で動作する。1件でもバリデーションエラーがあれば変更は適用されない。エラーレスポンスには個別シートごとの失敗理由が含まれる。

### 典型的な使用例

| シナリオ | 推奨値 |
|--------|------|
| 配布直後の評価シートをやり直したい | `before_goal_setting` |
| 目標設定をスキップして中間評価から開始したい | `interim_self_evaluation`（中間評価有効時のみ） |
| 評価完了済みシートを差し戻したい | `evaluation_1st` 等の戻したい段階 |
| フローを終了状態にしたい | `completed` |

通常運用では `scale_submit_evaluation_sheet` / `scale_reject_evaluation_sheet` を優先し、神の手はイレギュラー対応のみに使うこと。
