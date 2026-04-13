---
name: gemini
description: Google の Gemini CLI を、コード生成・レビュー・分析・Web 調査のための強力な補助ツールとして活用する。別の AI の視点、Google 検索による最新情報、コードベースアーキテクチャ分析、並列コード生成が有益なタスクに使用する。ユーザーが明示的に Gemini の操作を要求した場合にも使用する。
allowed-tools:
  - Bash
  - Read
  - Write
  - Grep
  - Glob
---

# Gemini CLI 連携スキル

このスキルにより、Claude Code は Gemini CLI（v0.16.0 以上）と Gemini 3 Pro を効果的に活用し、コード生成・レビュー・分析・特殊タスクを実行できるようになります。

## このスキルを使うべき場面

### 最適なユースケース

1. **セカンドオピニオン / クロスバリデーション**
   - コード記述後のレビュー（異なる AI の視点）
   - 代替的分析によるセキュリティ監査
   - Claude が見落としたバグの発見

2. **Google 検索グラウンディング**
   - 最新のインターネット情報が必要な質問
   - 最新のライブラリバージョン、API 変更、ドキュメント更新
   - 最新の出来事やリリース

3. **コードベースアーキテクチャ分析**
   - Gemini の `codebase_investigator` ツールを使用
   - 不慣れなコードベースの理解
   - ファイル間依存関係のマッピング

4. **並列処理**
   - 他の作業を続けながらタスクをオフロード
   - 複数のコード生成を同時実行
   - バックグラウンドでのドキュメント生成

5. **特殊な生成**
   - テストスイートの生成
   - JSDoc/ドキュメントの生成
   - プログラミング言語間のコード翻訳

### 使うべきでない場面

- 単純で素早いタスク（オーバーヘッドに見合わない）
- 即座の応答が必要なタスク（レート制限による遅延）
- コンテキストが既に読み込まれ理解されている場合
- 会話を通じたインタラクティブな改善が必要な場合

## コアインストラクション

### 1. インストール確認

```bash
command -v gemini || which gemini
```

### 2. 基本コマンドパターン

```bash
gemini "[プロンプト]" --yolo -o text 2>&1
```

主要フラグ：
- `--yolo` または `-y`：全てのツール呼び出しを自動承認
- `-o text`：人が読める形式の出力
- `-o json`：統計情報付きの構造化出力
- `-m gemini-2.5-flash`：単純なタスクにはより高速なモデルを使用

### 3. 重要な動作上の注意

**YOLO モードの動作**：ツール呼び出しを自動承認しますが、計画プロンプトは抑制しません。Gemini は計画を提示し「この計画でよろしいですか？」と尋ねる場合があります。強い表現を使ってください：
- "Apply now"（今すぐ適用）
- "Start immediately"（直ちに開始）
- "Do this without asking for confirmation"（確認なしで実行）

**レート制限**：無料枠は 60 リクエスト/分、1000 リクエスト/日。CLI は自動的にバックオフ付きでリトライします。「quota will reset after Xs」のようなメッセージが表示されます。

### 4. 出力の処理

JSON 出力（`-o json`）のパース：
```json
{
  "response": "実際のコンテンツ",
  "stats": {
    "models": { "tokens": {...} },
    "tools": { "byName": {...} }
  }
}
```

## クイックリファレンスコマンド

### コード生成
```bash
gemini "Create [説明] with [機能]. Output complete file content." --yolo -o text
```

### コードレビュー
```bash
gemini "Review [ファイル] for: 1) features, 2) bugs/security issues, 3) improvements" -o text
```

### バグ修正
```bash
gemini "Fix these bugs in [ファイル]: [リスト]. Apply fixes now." --yolo -o text
```

### テスト生成
```bash
gemini "Generate [Jest/pytest] tests for [ファイル]. Focus on [領域]." --yolo -o text
```

### ドキュメント
```bash
gemini "Generate JSDoc for all functions in [ファイル]. Output as markdown." --yolo -o text
```

### アーキテクチャ分析
```bash
gemini "Use codebase_investigator to analyze this project" -o text
```

### Web 調査
```bash
gemini "What are the latest [トピック]? Use Google Search." -o text
```

### 高速モデル（単純なタスク向け）
```bash
gemini "[プロンプト]" -m gemini-2.5-flash -o text
```

## エラーハンドリング

### レート制限超過
- CLI は自動的にバックオフ付きでリトライ
- 優先度の低いタスクには `-m gemini-2.5-flash` を使用
- 長時間の操作はバックグラウンドで実行

### コマンド失敗
- JSON 出力で詳細なエラー統計を確認
- Gemini の認証を確認：`gemini --version`
- `~/.gemini/settings.json` で設定の問題を確認

### 生成後のバリデーション
Gemini の出力は必ず検証してください：
- セキュリティ脆弱性のチェック（XSS、インジェクション）
- 機能が要件を満たしているかテスト
- コードスタイルの一貫性をレビュー
- 依存関係が適切か確認

## 連携ワークフロー

### 標準的な 生成→レビュー→修正 サイクル

```bash
# 1. 生成
gemini "Create [コード]" --yolo -o text

# 2. レビュー（Gemini が自分の成果をレビュー）
gemini "Review [ファイル] for bugs and security issues" -o text

# 3. 特定された問題を修正
gemini "Fix [問題] in [ファイル]. Apply now." --yolo -o text
```

### バックグラウンド実行

長いタスクはバックグラウンドで実行し監視：
```bash
gemini "[長いタスク]" --yolo -o text 2>&1 &
# BashOutput ツールで監視
```

## Gemini 独自の機能

Gemini でのみ利用可能なツール：

1. **google_web_search** - Google によるリアルタイムのインターネット検索
2. **codebase_investigator** - 深いアーキテクチャ分析
3. **save_memory** - セッション横断の永続メモリ

## 設定

### プロジェクトコンテキスト（オプション）

プロジェクトルートに `.gemini/GEMINI.md` を作成すると、Gemini が自動的に読み込む永続コンテキストになります。

### セッション管理

セッション一覧：`gemini --list-sessions`
セッション再開：`echo "フォローアップ" | gemini -r [インデックス] -o text`

## 関連ドキュメント

- `reference.md` - コマンドとフラグの完全リファレンス
- `templates.md` - 一般的な操作のプロンプトテンプレート
- `patterns.md` - 高度な連携パターン
- `tools.md` - Gemini の組み込みツールドキュメント
