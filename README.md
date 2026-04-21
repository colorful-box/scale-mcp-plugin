# Scale MCP Plugin for Claude Code

Scale人事評価システム（[scale-hr.io](https://scale-hr.io)）のMCPツールを効果的に活用するためのClaude Code Plugin。

## 機能

- **Agent Skill**: Scale MCPツールの使い方をClaude Codeに教えるガイド
- **MCP サーバー設定**: プラグインインストールでScale MCPサーバーが自動設定される
- **ロール別スキル**: 企業管理者向け（62ツール）と一般ユーザー向け（15ツール）を分離提供

### スキル一覧

| スキル | 対象ロール | ツール数 | 内容 |
|--------|-----------|---------|------|
| `scale-mcp-guide` | `company_admin` | 62 | テンプレート作成、評価シート配布、マスタ管理、データ分析、神の手操作 |
| `scale-mcp-guide-general` | `general` | 15 | 自分の評価シート確認・提出、1on1セッション、プロフィール管理、ダッシュボード |

MCPサーバーがログインユーザーのロールに応じて利用可能なツールを自動フィルタリングします。

## Agent Skills と MCP の通信の流れ

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {
  'primaryColor': '#E8F5EA',
  'primaryBorderColor': '#5CB88A',
  'primaryTextColor': '#2D3436',
  'lineColor': '#5CB88A',
  'signalColor': '#2D3436',
  'noteBkgColor': '#F7F8FA',
  'noteBorderColor': '#5CB88A',
  'noteTextColor': '#2D3436',
  'actorBkg': '#5CB88A',
  'actorBorder': '#4A9E75',
  'actorTextColor': '#FFFFFF',
  'sequenceNumberColor': '#FFFFFF'
}}}%%
sequenceDiagram
    participant User as ユーザー
    participant Agent as AI Agent
    participant Skill as Agent Skills<br/>(ワークフロー・設定ガイド)
    participant MCP as MCP サーバー
    participant API as Scale API

    User->>Agent: リクエスト<br/>「評価シートを全社員に配布して」

    Note over Agent,Skill: 1. Agent Skills からガイドを取得
    Agent->>Skill: scale-mcp-guide 呼び出し
    Skill-->>Agent: ワークフロー注入<br/>(ツール実行順序、パラメータ仕様)

    Note over Agent,MCP: 2. MCP Tool で API を実行
    Agent->>MCP: scale_preview_distribution 呼び出し<br/>template_id, user_ids
    MCP->>MCP: パラメータ検証
    MCP->>MCP: OAuth 認証トークン付与

    Note over MCP,API: 3. Scale API への通信
    MCP->>API: POST /api/distributions/preview<br/>Authorization: Bearer xxx
    API-->>MCP: JSON レスポンス

    MCP-->>Agent: 配布プレビュー結果
    Agent-->>User: 結果を整形して表示
```

## インストール

### 1. Marketplace を追加

```
/plugin marketplace add colorful-box/scale-mcp-plugin
```

### 2. Plugin をインストール

ロールに応じて必要なスキルをインストールしてください。

**企業管理者（company_admin）の場合:**

```
/plugin install scale-mcp-guide
```

**一般ユーザー（general）の場合:**

```
/plugin install scale-mcp-guide-general
```

インストール後、Scale MCPサーバー（`https://mcp.scale-hr.io/mcp`）が自動設定されます。
初回接続時にブラウザでOAuth認証が求められます。

## 使い方

プラグインをインストールすると、Scale関連の依頼時にClaude Codeが自動的にガイドを参照します。

### 企業管理者の例

> 2026年度上期の評価期間を作成して、全社員に標準テンプレートで評価シートを配布して

Claude Code が以下のツールを自動的に実行します:

1. `scale_create_evaluation_period` → 評価期間を作成
2. `scale_list_templates` → テンプレート一覧から「標準」を特定
3. `scale_get_template` → テンプレート設定を確認
4. `scale_get_distribute_users` → 配布対象の全社員を取得
5. `scale_preview_distribution` → 配布プレビューで確認
6. `scale_distribute_sheets` → 全社員に評価シートを配布

### 一般ユーザーの例

> 今の自分の評価状況を教えて

Claude Code が以下のツールを自動的に実行します:

1. `scale_whoami` → 認証状態・ロールを確認
2. `scale_dashboard_summary` → 全体の評価状況サマリー
3. `scale_dashboard_my_sheets` → 自分の評価シート一覧
4. `scale_get_evaluation_sheet` → シート詳細を表示

> 評価シートを提出したい

Claude Code が以下のツールを自動的に実行します:

1. `scale_dashboard_my_sheets` → 自分のシート一覧
2. `scale_get_evaluation_sheet` → 提出対象のシート内容を確認
3. `scale_submit_evaluation_sheet` → ワークフローに沿って提出

## スキル構成

### 企業管理者向け（`scale-mcp-guide`）

| ファイル | 説明 |
|---------|------|
| `SKILL.md` | メインガイド（ワークフロー概要、テンプレート作成プロセス、削除・復元） |
| `references/workflows.md` | 詳細なワークフローパターン |
| `references/template_settings.md` | テンプレート設定の全パラメータと典型パターン例 |
| `references/template_excel_format.md` | Excel入力フォーマット仕様 |
| `references/tool_reference.md` | 全79ツールのカテゴリ別リファレンス |

### 一般ユーザー向け（`scale-mcp-guide-general`）

| ファイル | 説明 |
|---------|------|
| `SKILL.md` | メインガイド（ダッシュボード、シート提出、1on1、プロフィール） |
| `references/workflows.md` | ワークフロー詳細とユースケース |

## ライセンス

MIT
