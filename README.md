# Scale MCP Plugin for Claude Code

Scale人事評価システム（[scale-hr.io](https://scale-hr.io)）のMCPツールを効果的に活用するためのClaude Code Plugin。

## 機能

- **Agent Skill**: 79個のScale MCPツールの使い方をClaude Codeに教えるガイド
- **MCP サーバー設定**: プラグインインストールでScale MCPサーバーが自動設定される
- **ワークフローパターン**: 評価制度セットアップ、テンプレート管理、配布、分析などの手順
- **テンプレート設定ガイド**: 30以上の設定パラメータと典型的な設定パターン例

## インストール

### 1. Marketplace を追加

```
/plugin marketplace add colorful-box/scale-mcp-plugin
```

### 2. Plugin をインストール

```
/plugin install scale-mcp-guide
```

インストール後、Scale MCPサーバー（`https://mcp.scale-hr.io/mcp`）が自動設定されます。
初回接続時にブラウザでOAuth認証が求められます。

## 使い方

プラグインをインストールすると、Scale関連の依頼時にClaude Codeが自動的にガイドを参照します。

### 例

```
ユーザー: 「2026年度上期の評価期間を作成して、全社員に標準テンプレートで評価シートを配布して」

Claude Code:
  1. scale_create_evaluation_period → 評価期間を作成
  2. scale_list_templates → テンプレート一覧から「標準」を特定
  3. scale_get_template → テンプレート設定を確認
  4. scale_get_distribute_users → 配布対象の全社員を取得
  5. scale_preview_distribution → 配布プレビューで確認
  6. scale_distribute_sheets → 全社員に評価シートを配布
```

## スキル内容

| ファイル | 説明 |
|---------|------|
| `SKILL.md` | メインガイド（ワークフロー概要、テンプレート設定、削除・復元） |
| `references/workflows.md` | 詳細なワークフローパターン |
| `references/template_settings.md` | テンプレート設定の全パラメータと典型パターン例 |
| `references/tool_reference.md` | 全79ツールのカテゴリ別リファレンス |

## ライセンス

MIT
