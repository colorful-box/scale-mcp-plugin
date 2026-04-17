---
name: scale-mcp-guide-general
description: Scale人事評価システムの一般ユーザー向けMCPツール活用ガイド。自分の評価シート確認・提出、1on1セッション、プロフィール管理、ダッシュボード操作を案内する。ユーザーが「自分の評価」「評価シート提出」「1on1」「プロフィール」に言及したときに参照する。
---

# Scale MCP ツール活用ガイド（一般ユーザー向け）

Scale人事評価システム（scale-hr.io）の一般ユーザー（`general` ロール）が利用できる15個のMCPツールのガイド。

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

- ユーザーが自分の評価シートを確認・提出したいとき
- 評価の進捗状況やTODOを確認したいとき
- 1on1セッションの内容を確認したいとき
- プロフィール情報を確認・更新したいとき
- 掲示板の内容を確認したいとき

## 最初に実行すべきツール

1. `scale_whoami` — 認証状態・ユーザー情報・ロールを確認
2. `scale_dashboard_summary` — 現在の評価状況を把握

## 主要ワークフローパターン

### 評価状況の確認

```
scale_dashboard_summary          → 全体の評価状況サマリー
scale_dashboard_evaluation_todos → 未対応の評価TODO一覧
scale_dashboard_my_sheets        → 自分の評価シート一覧
scale_dashboard_action_required  → 対応が必要なシート
scale_dashboard_current_period   → 当期のシート
```

特定のシートの詳細を確認する場合:

```
scale_get_evaluation_sheet(sheet_id=X) → シート詳細（目標・評点・コメント等）
```

### 評価シートの提出・差し戻し

```
scale_submit_evaluation_sheet(sheet_id=X)   → ワークフローに沿って提出
scale_reject_evaluation_sheet(
  sheet_id=X,
  reason="修正理由"                          → 差し戻し（理由付き）
)
```

- 提出前に `scale_get_evaluation_sheet` でシート内容を確認すること
- 差し戻しは評価者として割り当てられたシートに対して行える

### 1on1セッションの確認

```
scale_list_one_on_one_sessions              → 自分のセッション一覧
scale_get_one_on_one_session(session_id=X)  → セッション詳細
```

### プロフィールの確認・更新

```
scale_get_profile     → 自分のプロフィール取得
scale_update_profile  → プロフィール更新
```

### 掲示板の確認

```
scale_get_bulletins_by_location → 掲示板の内容を表示場所ごとに取得
```

詳細は `references/workflows.md` を参照。

## ツール一覧（15ツール）

| カテゴリ | ツール | 説明 |
|---------|--------|------|
| 認証 | `scale_whoami` | 認証状態・ユーザー情報・ロールを取得 |
| ダッシュボード | `scale_dashboard_summary` | 評価状況の全体サマリー |
| ダッシュボード | `scale_dashboard_evaluation_todos` | 未対応の評価TODO |
| ダッシュボード | `scale_dashboard_my_sheets` | 自分の評価シート一覧 |
| ダッシュボード | `scale_dashboard_action_required` | 対応が必要なシート |
| ダッシュボード | `scale_dashboard_current_period` | 当期のシート |
| プロフィール | `scale_get_profile` | 自分のプロフィール取得 |
| プロフィール | `scale_update_profile` | プロフィール更新 |
| 評価シート | `scale_get_evaluation_sheet` | 評価シートの詳細取得 |
| 評価シート | `scale_list_my_evaluation_sheets` | 自分の評価シート一覧 |
| 評価シート | `scale_submit_evaluation_sheet` | 評価シートをワークフローに沿って提出 |
| 評価シート | `scale_reject_evaluation_sheet` | 評価シートを差し戻し |
| 1on1 | `scale_list_one_on_one_sessions` | 自分の1on1セッション一覧 |
| 1on1 | `scale_get_one_on_one_session` | 1on1セッション詳細 |
| 掲示板 | `scale_get_bulletins_by_location` | 掲示板の内容を表示場所ごとに取得 |

## 注意事項

- ダッシュボード系ツールはパラメータ不要。ログインユーザーの情報を自動取得する
- 管理者向けツール（テンプレート作成、配布、ユーザー管理等）は `general` ロールでは利用不可
- 権限エラー（403）が発生した場合は「管理者に問い合わせてください」と案内し、他のユーザー情報を開示しないこと
- `scale_whoami` でロールが `company_admin` と判明した場合は、管理者向けスキル（`scale-mcp-guide`）のワークフローを参照すること

## 詳細リファレンス

- `references/workflows.md` — ワークフロー詳細とユースケース
