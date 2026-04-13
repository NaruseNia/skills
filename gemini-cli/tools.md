# Gemini CLI 組み込みツール

Gemini の組み込みツールとその機能のリファレンスです。

## 独自ツール（Claude Code にないもの）

Gemini CLI でのみ利用可能なツールです：

### google_web_search

Google Search API を使用した Web 検索を実行します。

**機能:**
- リアルタイムのインターネット検索
- 最新情報（ニュース、リリース、ドキュメント）
- ソース付きのグラウンデッドなレスポンス

**使い方:**
```bash
gemini "What are the latest React 19 features? Use Google Search." -o text
```

**最適な用途:**
- 最新のイベントやニュース
- 最新のライブラリバージョン
- 最近のドキュメント更新
- コミュニティの意見やベンチマーク

**クエリ例:**
- "What are the security vulnerabilities in lodash 4.x? Use Google Search."
- "What's new in TypeScript 5.4? Use Google Search."
- "Best practices for Next.js 14 app router in November 2025."

---

### codebase_investigator

コードベースの深い分析に特化したツールです。

**機能:**
- アーキテクチャのマッピング
- 依存関係の分析
- ファイル間の関係検出
- システム全体のパターン特定

**使い方:**
```bash
gemini "Use the codebase_investigator tool to analyze this project" -o text
```

**出力内容:**
- 全体的なアーキテクチャの説明
- 主要ファイルの目的
- コンポーネント間の関係
- 依存関係チェーン
- 潜在的な問題/不整合

**最適な用途:**
- 新しいコードベースへのオンボーディング
- レガシーシステムの理解
- 隠れた依存関係の発見
- アーキテクチャドキュメントの作成

**例:**
```bash
gemini "Use codebase_investigator to map the authentication flow in this project" -o text
```

---

### save_memory

永続的な長期メモリに情報を保存します。

**機能:**
- セッション横断の永続化
- キーバリューストレージ
- 将来のセッションでの呼び出し

**使い方:**
```bash
gemini "Remember that this project uses Zustand for state management. Save this to memory." -o text
```

**最適な用途:**
- プロジェクトの規約
- ユーザーの好み
- 繰り返し使うコンテキスト
- カスタムインストラクション

---

## 標準ツール

Claude Code の機能に類似したツールです：

### list_directory

パス内のファイルとサブディレクトリを一覧表示します。

**パラメータ:**
- `path`: 一覧表示するディレクトリ
- `ignore`: 除外する glob パターン

**出力例:**
```
src/
  components/
  utils/
  index.js
package.json
README.md
```

---

### read_file

大きなファイルは切り詰めてファイル内容を読み取ります。

**対応形式:**
- テキストファイル（全種類）
- 画像（PNG、JPG、GIF、WEBP、SVG、BMP）
- PDF ドキュメント

**パラメータ:**
- `path`: ファイルパス
- `offset`: 開始行（大きなファイル用）
- `limit`: 行数

**大きなファイルの処理:**
ファイルが制限を超える場合、出力で切り詰めが示され、offset/limit を使った追加読み取りの手順が提供されます。

---

### search_file_content

ripgrep を使用した高速なコンテンツ検索です。

**grep に対する優位性:**
- 最適化されたパフォーマンス
- 自動出力制限（最大 2 万件のマッチ）
- 優れたパターンマッチング

**パラメータ:**
- `pattern`: 正規表現パターン
- `path`: 検索ルート
- 各種 ripgrep フラグ

---

### glob

パターンベースのファイル検索です。

**返却内容:**
- 絶対パス
- 更新日時順（新しい順）

**パターン例:**
- `src/**/*.ts` - src 内の全 TypeScript ファイル
- `**/*.test.js` - 全テストファイル
- `**/README.md` - 全 README

---

### web_fetch

URL からコンテンツを取得します。

**機能:**
- HTTP/HTTPS URL
- ローカルアドレス（localhost）
- 1 リクエストあたり最大 20 URL

**使い方:**
```bash
gemini "Fetch and summarize https://example.com/docs" -o text
```

---

### write_todos

内部的なタスク追跡です。

**機能:**
- 複雑なリクエストのサブタスクを追跡
- 複数ステップの作業を整理
- ステップの見落としを防止

**自動使用:**
Gemini は複雑なタスクでこれを内部的に使用します。

---

## ツール呼び出し

### 自動ツール選択

Gemini はプロンプトに基づいて適切なツールを自動選択します：

| プロンプトの種類 | 選択されるツール |
|----------------|----------------|
| "What files are in src/" | list_directory |
| "Find all TODO comments" | search_file_content |
| "Read package.json" | read_file |
| "Find all React components" | glob |
| "What's new in Vue 4?" | google_web_search |
| "Analyze this codebase" | codebase_investigator |

### 明示的なツール指定

ツールを明示的にリクエストできます：

```bash
gemini "Use the codebase_investigator tool to..." -o text
gemini "Search the web for..." -o text
gemini "Use glob to find all..." -o text
```

---

## JSON 出力でのツール統計

`-o json` 使用時、ツールの使用状況が報告されます：

```json
{
  "stats": {
    "tools": {
      "totalCalls": 3,
      "totalSuccess": 3,
      "totalFail": 0,
      "totalDurationMs": 5000,
      "totalDecisions": {
        "accept": 0,
        "reject": 0,
        "modify": 0,
        "auto_accept": 3
      },
      "byName": {
        "google_web_search": {
          "count": 1,
          "success": 1,
          "fail": 0,
          "durationMs": 3000,
          "decisions": {
            "auto_accept": 1
          }
        },
        "read_file": {
          "count": 2,
          "success": 2,
          "fail": 0,
          "durationMs": 2000,
          "decisions": {
            "auto_accept": 2
          }
        }
      }
    }
  }
}
```

---

## Claude Code ツールとの比較

| 機能 | Claude Code | Gemini CLI |
|------|-------------|------------|
| ファイル一覧 | LS, Glob | list_directory, glob |
| ファイル読み取り | Read | read_file |
| ファイル書き込み | Write, Edit | write_file（YOLO モード） |
| コード検索 | Grep | search_file_content |
| Web 取得 | WebFetch | web_fetch |
| Web 検索 | WebSearch | **google_web_search** |
| アーキテクチャ | Task (Explore) | **codebase_investigator** |
| メモリ | N/A | **save_memory** |
| タスク追跡 | TodoWrite | write_todos |

**太字** = Gemini の独自の強み

---

## ツール制限

### allowed-tools の使用

設定またはコマンドラインで、使用可能なツールを制限：

```bash
gemini --allowed-tools "read_file,glob" "Find config files" -o text
```

### 設定ファイルで指定
```json
{
  "security": {
    "allowedTools": ["read_file", "list_directory", "glob"]
  }
}
```

---

## ベストプラクティス

### 特定ツールの使用場面

**google_web_search:**
- 最新/最近の情報が必要な場合
- 最新バージョンの確認
- ドキュメント更新の調査
- 問題に対するコミュニティの解決策

**codebase_investigator:**
- 新しいコードベースに取り組む場合
- 複雑なシステムの理解
- 隠れた依存関係の発見
- ドキュメントの作成

**save_memory:**
- 繰り返し使うプロジェクトコンテキスト
- ユーザーの好み
- カスタム規約

### ツール組み合わせパターン

**調査→実装:**
```bash
gemini "Use Google Search to find best practices for [トピック], then implement them" --yolo -o text
```

**分析→レポート:**
```bash
gemini "Use codebase_investigator to analyze the project, then write a summary report" --yolo -o text
```

**検索→読み取り→修正:**
```bash
gemini "Find all files using deprecated API, read them, and suggest updates" -o text
```
