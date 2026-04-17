# ワークフロー詳細（一般ユーザー向け）

## 1. 評価状況の確認

### 全体の流れ

まず全体サマリーで状況を把握し、必要に応じて詳細を確認する。

```
scale_dashboard_summary
  → 評価期間の進捗、提出状況、未対応件数の概要

scale_dashboard_evaluation_todos
  → 自分が対応すべき評価タスク（目標設定、自己評価、評価者評価など）

scale_dashboard_action_required
  → アクションが必要なシート（差し戻し対応、期限切れなど）
```

### 自分の評価シートを一覧で確認

```
scale_dashboard_my_sheets
  → 自分が被評価者として割り当てられたシート一覧

scale_list_my_evaluation_sheets
  → 同様に自分のシート一覧（詳細なフィルタが可能）
```

### 特定のシートの詳細を確認

```
scale_get_evaluation_sheet(sheet_id=X)
  → 評価軸、項目、目標、評点、コメント、ステータス等の全情報
```

### 典型的なユースケース

**「今の自分の評価状況を教えて」**
1. `scale_dashboard_summary` で全体把握
2. `scale_dashboard_my_sheets` で自分のシート一覧
3. 必要に応じて `scale_get_evaluation_sheet` で詳細確認

**「何か対応すべきものはある？」**
1. `scale_dashboard_evaluation_todos` でTODO確認
2. `scale_dashboard_action_required` で要対応シート確認

## 2. 評価シートの提出

### 提出フロー

```
scale_get_evaluation_sheet(sheet_id=X)
  → 提出前にシート内容を確認（目標・自己評価が記入済みか等）

scale_submit_evaluation_sheet(sheet_id=X)
  → ワークフローに沿って次のステップへ進める
```

- 提出するとステータスが変わり、次の評価者（1次→2次→…）へ回る
- 提出後は内容の編集ができなくなる（差し戻しされない限り）

### 差し戻し

評価者として割り当てられたシートに対して差し戻しが可能:

```
scale_reject_evaluation_sheet(
  sheet_id=X,
  reason="目標の定量指標が不足しています。KPIを具体的に記載してください。"
)
```

- `reason` に差し戻し理由を必ず記載する
- 差し戻されたシートは被評価者に戻り、修正→再提出の流れになる

### 典型的なユースケース

**「評価シートを提出したい」**
1. `scale_dashboard_my_sheets` でシート一覧を確認
2. `scale_get_evaluation_sheet` で提出対象のシート内容を確認
3. 内容に問題なければ `scale_submit_evaluation_sheet` で提出

**「部下のシートを差し戻したい」**
1. `scale_dashboard_evaluation_todos` で評価待ちシートを確認
2. `scale_get_evaluation_sheet` で内容を確認
3. `scale_reject_evaluation_sheet` で理由を添えて差し戻し

## 3. 1on1セッションの確認

```
scale_list_one_on_one_sessions
  → 自分が参加する1on1セッション一覧
    （上司との面談、部下との面談の両方が含まれる）

scale_get_one_on_one_session(session_id=X)
  → セッション詳細（議題、メモ、次回アクション等）
```

### 典型的なユースケース

**「次の1on1の予定を教えて」**
1. `scale_list_one_on_one_sessions` で一覧取得
2. 必要に応じて `scale_get_one_on_one_session` で詳細確認

## 4. プロフィールの確認・更新

```
scale_get_profile
  → 自分のプロフィール情報（氏名、所属、職位、職種など）

scale_update_profile
  → プロフィール情報の更新
```

- 更新可能なフィールドはシステム設定による（管理者が制限している場合あり）
- 所属・職位・職種などの変更は管理者が行う場合が多い

## 5. 掲示板の確認

```
scale_get_bulletins_by_location
  → 掲示板の内容を表示場所ごとに取得（全社通知、評価に関するお知らせなど）
```
