# Gemini CLI コマンドリファレンス

Gemini CLI v0.16.0 以上の完全リファレンスです。

## インストール

```bash
npm install -g @google/gemini-cli
# またはインストールせずに実行:
npx @google/gemini-cli
```

## 認証

```bash
# オプション 1: API キー
export GEMINI_API_KEY=your_key

# オプション 2: OAuth（対話式）
gemini  # 初回実行時に認証プロンプトが表示されます
```

## コマンドラインフラグ

### 必須フラグ

| フラグ | 短縮形 | 説明 |
|-------|--------|------|
| `--yolo` | `-y` | 全てのツール呼び出しを自動承認 |
| `--output-format` | `-o` | 出力形式: `text`、`json`、`stream-json` |
| `--model` | `-m` | モデル選択（例: `gemini-2.5-flash`） |

### セッション管理

| フラグ | 短縮形 | 説明 |
|-------|--------|------|
| `--resume` | `-r` | インデックスまたは "latest" でセッションを再開 |
| `--list-sessions` | | 利用可能なセッションの一覧 |
| `--delete-session` | | インデックスでセッションを削除 |

### 実行オプション

| フラグ | 短縮形 | 説明 |
|-------|--------|------|
| `--sandbox` | `-s` | 分離されたサンドボックスで実行 |
| `--approval-mode` | | `default`、`auto_edit`、または `yolo` |
| `--timeout` | | リクエストタイムアウト（ミリ秒） |
| `--checkpointing` | | ファイル変更のスナップショットを有効化 |

### コンテキストとツール

| フラグ | 説明 |
|-------|------|
| `--include-directories` | ワークスペースにディレクトリを追加 |
| `--allowed-tools` | 使用可能なツールを制限 |
| `--allowed-mcp-server-names` | MCP サーバーを制限 |

### その他のオプション

| フラグ | 短縮形 | 説明 |
|-------|--------|------|
| `--debug` | `-d` | デバッグ出力を有効化 |
| `--version` | `-v` | バージョン表示 |
| `--help` | `-h` | ヘルプ表示 |
| `--list-extensions` | `-l` | インストール済み拡張機能の一覧 |
| `--prompt-interactive` | `-i` | 初期プロンプト付きの対話モード |

## 出力形式

### テキスト（`-o text`）
```bash
gemini "プロンプト" -o text
# 返却: 人が読める形式のレスポンス
```

### JSON（`-o json`）
```bash
gemini "プロンプト" -o json
```

構造化データを返却：
```json
{
  "response": "実際のレスポンス内容",
  "stats": {
    "models": {
      "gemini-2.5-flash": {
        "api": {
          "totalRequests": 3,
          "totalErrors": 0,
          "totalLatencyMs": 5000
        },
        "tokens": {
          "prompt": 1500,
          "candidates": 500,
          "total": 2000,
          "cached": 800,
          "thoughts": 150,
          "tool": 50
        }
      }
    },
    "tools": {
      "totalCalls": 2,
      "totalSuccess": 2,
      "totalFail": 0,
      "byName": {
        "google_web_search": {
          "count": 1,
          "success": 1,
          "durationMs": 3000
        }
      }
    }
  }
}
```

### ストリーム JSON（`-o stream-json`）
長時間タスクの監視用に、改行区切りの JSON イベントをリアルタイムで出力します。

## モデル選択

### 利用可能なモデル

| モデル | 用途 | コンテキスト |
|-------|------|------------|
| `gemini-3-pro` | 複雑なタスク（デフォルト） | 100 万トークン |
| `gemini-2.5-flash` | 素早いタスク、低レイテンシ | 大 |
| `gemini-2.5-flash-lite` | 最速、最も単純なタスク | 中 |

### 使い方
```bash
# デフォルト（Pro）
gemini "複雑な分析" -o text

# 速度重視の Flash
gemini "単純なタスク" -m gemini-2.5-flash -o text
```

## 設定ファイル

### 設定の配置場所
優先順位（高い順）：
1. `/etc/gemini-cli/settings.json`（システム）
2. `~/.gemini/settings.json`（ユーザー）
3. `.gemini/settings.json`（プロジェクト）

### 設定例
```json
{
  "security": {
    "auth": {
      "selectedType": "oauth-personal"
    }
  },
  "general": {
    "previewFeatures": true,
    "vimMode": false,
    "checkpointing": true
  },
  "mcpServers": {}
}
```

### プロジェクトコンテキスト（GEMINI.md）

プロジェクトルートに `.gemini/GEMINI.md` を作成：
```markdown
# プロジェクトコンテキスト

プロジェクトの説明とガイドライン。

## コーディング規約
- Gemini が従うべき規約

## 変更時の注意事項
- 変更に関するガイドライン
```

### 無視ファイル（.geminiignore）

`.gitignore` と同様に、コンテキストからファイルを除外：
```
node_modules/
dist/
*.log
.env
```

## セッション管理

### セッション一覧
```bash
gemini --list-sessions
```

出力：
```
Available sessions for this project (5):
  1. Create task manager (10 minutes ago) [uuid]
  2. Review code (20 minutes ago) [uuid]
  ...
```

### セッション再開
```bash
# インデックスで指定
echo "フォローアップの質問" | gemini -r 1 -o text

# 最新のセッション
echo "続き" | gemini -r latest -o text
```

## レート制限

### 無料枠の制限
- 60 リクエスト/分
- 1000 リクエスト/日

### レート制限時の動作
- CLI は指数バックオフで自動リトライ
- メッセージ: `"quota will reset after Xs"`
- 一般的な待ち時間: 1〜5 秒

### 緩和策
1. 単純なタスクには `gemini-2.5-flash` を使用
2. 操作を単一プロンプトにバッチ化
3. 長いタスクはバックグラウンドで実行

## インタラクティブコマンド

対話モードで使用できるスラッシュコマンド：

| コマンド | 目的 |
|---------|------|
| `/help` | 使用可能なコマンドを表示 |
| `/tools` | 使用可能なツールの一覧 |
| `/stats` | トークン使用量を表示 |
| `/compress` | トークン節約のためコンテキストを要約 |
| `/restore` | ファイルチェックポイントを復元 |
| `/chat save <タグ>` | 会話を保存 |
| `/chat resume <タグ>` | 会話を再開 |
| `/memory show` | GEMINI.md コンテキストを表示 |
| `/memory refresh` | コンテキストファイルを再読み込み |

## パイプとスクリプティング

### パイプ入力
```bash
echo "2+2 は?" | gemini -o text
cat file.txt | gemini "これを要約して" -o text
```

### ファイル参照構文
プロンプト内で `@` を使ってファイルを参照：
```bash
gemini "Review @./src/main.js for bugs" -o text
```

### シェルコマンド実行
対話モードでは `!` をプレフィックスとして使用：
```
> !git status
```

## キーボードショートカット（対話モード）

| ショートカット | 機能 |
|--------------|------|
| `Ctrl+L` | 画面クリア |
| `Ctrl+V` | クリップボードから貼り付け |
| `Ctrl+Y` | YOLO モードの切り替え |
| `Ctrl+X` | 外部エディタで開く |

## トラブルシューティング

### よくある問題

| 問題 | 解決方法 |
|------|---------|
| "API key not found" | `GEMINI_API_KEY` 環境変数を設定 |
| "Rate limit exceeded" | 自動リトライを待つか Flash を使用 |
| "Context too large" | `.geminiignore` を使うか具体的に指定 |
| "Tool call failed" | JSON の統計で詳細を確認 |

### デバッグモード
```bash
gemini "プロンプト" --debug -o text
```

### エラーレポート
完全なエラーレポートの保存先：
```
/var/folders/.../gemini-client-error-*.json
```
