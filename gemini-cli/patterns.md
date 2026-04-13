# Gemini CLI 連携パターン

Claude Code から Gemini CLI を効果的に活用するための高度なパターン集です。

## パターン 1: 生成→レビュー→修正サイクル

高品質なコード生成のための最も信頼できるパターンです。

```bash
# ステップ 1: コードを生成
gemini "Create [コードの説明]" --yolo -o text

# ステップ 2: Gemini に自身の成果をレビューさせる
gemini "Review [生成されたファイル] for bugs and security issues" -o text

# ステップ 3: 特定された問題を修正
gemini "Fix these issues in [ファイル]: [レビューからのリスト]. Apply now." --yolo -o text
```

### なぜ効果的か
- 生成とレビューで異なる「思考モード」が働く
- セルフコレクションで一般的なミスを検出
- セキュリティ脆弱性はレビューフェーズで発見されやすい

### 例
```bash
# 生成
gemini "Create a user authentication module with bcrypt and JWT" --yolo -o text

# レビュー
gemini "Review auth.js for security vulnerabilities" -o text
# 出力: "Found XSS risk, missing input validation, weak JWT secret"

# 修正
gemini "Fix in auth.js: XSS risk, add input validation, use env var for JWT secret. Apply now." --yolo -o text
```

## パターン 2: プログラム的処理のための JSON 出力

結果をプログラムで処理する必要がある場合に JSON 出力を使用します。

```bash
gemini "[プロンプト]" -o json 2>&1
```

### レスポンスのパース

```javascript
// Node.js または jq で使用
const result = JSON.parse(output);
const content = result.response;
const tokenUsage = result.stats.models["gemini-2.5-flash"].tokens.total;
const toolCalls = result.stats.tools.byName;
```

### ユースケース
- レスポンスからの特定データの抽出
- トークン使用量の監視
- ツール呼び出しの成功/失敗の追跡
- 自動化パイプラインの構築

## パターン 3: バックグラウンド実行

長時間のタスクはバックグラウンドで実行し、他の作業を続けます。

```bash
# バックグラウンドで開始
gemini "[長いタスク]" --yolo -o text 2>&1 &

# 後で使うプロセス ID を取得
echo $!

# BashOutput ツールで出力を段階的に監視
```

### 使用場面
- 大規模プロジェクトのコード生成
- ドキュメント生成
- 複数の Gemini タスクの並列実行

### 並列実行
```bash
# 複数のタスクを同時に実行
gemini "Create frontend" --yolo -o text 2>&1 &
gemini "Create backend" --yolo -o text 2>&1 &
gemini "Create tests" --yolo -o text 2>&1 &
```

## パターン 4: モデル選択戦略

タスクに適したモデルを選択します。

### 判断ツリー

```
タスクは複雑か？（アーキテクチャ、複数ファイル、深い分析）
├── はい → デフォルト（Gemini 3 Pro）を使用
└── いいえ → 速度が重要か？
    ├── はい → gemini-2.5-flash を使用
    └── いいえ → 些末なタスクか？（フォーマット、簡単なクエリ）
        ├── はい → gemini-2.5-flash-lite を使用
        └── いいえ → gemini-2.5-flash を使用
```

### 例
```bash
# 複雑: アーキテクチャ分析
gemini "Analyze codebase architecture" -o text

# 素早く: 単純なフォーマット
gemini "Format this JSON" -m gemini-2.5-flash -o text

# 些末: ワンライナー
gemini "What is 2+2?" -m gemini-2.5-flash -o text
```

## パターン 5: レート制限対応

レート制限内で作業するための戦略です。

### アプローチ 1: 自動リトライに任せる
デフォルトの動作 - CLI が自動的にバックオフ付きでリトライします。

### アプローチ 2: 優先度の低いタスクに Flash を使用
```bash
# 高優先度: Pro を使用
gemini "[重要なタスク]" --yolo -o text

# 低優先度: Flash を使用（別のクォータ）
gemini "[それほど重要でないタスク]" -m gemini-2.5-flash -o text
```

### アプローチ 3: バッチ操作
関連する操作を単一のプロンプトにまとめます：
```bash
# 複数回呼び出す代わりに:
gemini "Create file A" --yolo
gemini "Create file B" --yolo
gemini "Create file C" --yolo

# 一括で:
gemini "Create files A, B, and C with [仕様]. Create all now." --yolo
```

### アプローチ 4: ディレイ付きの逐次実行
自動スクリプトの場合、ディレイを追加：
```bash
gemini "[タスク 1]" --yolo -o text
sleep 2
gemini "[タスク 2]" --yolo -o text
```

## パターン 6: コンテキストの充実

より良い結果のために豊富なコンテキストを提供します。

### ファイル参照の使用
```bash
gemini "Based on @./package.json and @./src/index.js, suggest improvements" -o text
```

### GEMINI.md の使用
自動的に含まれるプロジェクトコンテキストを作成：
```markdown
# .gemini/GEMINI.md

## プロジェクト概要
TypeScript を使用した React アプリです。

## コーディング規約
- 関数コンポーネントを使用
- クラスよりフックを優先
- 全ての関数に JSDoc が必要
```

### プロンプトでの明示的なコンテキスト
```bash
gemini "Given this context:
- Project uses React 18 with TypeScript
- State management: Zustand
- Styling: Tailwind CSS

Create a user profile component." --yolo -o text
```

## パターン 7: バリデーションパイプライン

Gemini の出力は使用前に必ず検証します。

### バリデーション手順

1. **構文チェック**
   ```bash
   # JavaScript の場合
   node --check generated.js

   # TypeScript の場合
   tsc --noEmit generated.ts
   ```

2. **セキュリティスキャン**
   - ユーザー入力を含む innerHTML のチェック（XSS）
   - eval() や Function() 呼び出しの確認
   - 入力バリデーションの確認

3. **機能テスト**
   - 生成されたテストの実行
   - 手動スモークテスト

4. **スタイルチェック**
   ```bash
   eslint generated.js
   prettier --check generated.js
   ```

### 自動バリデーションパターン
```bash
# 生成
gemini "Create utility functions" --yolo -o text

# バリデーション
node --check utils.js && eslint utils.js && npm test
```

## パターン 8: 段階的改善

複雑な出力をステージに分けて構築します。

```bash
# ステージ 1: コア構造
gemini "Create basic Express server with routes for /api/users" --yolo -o text

# ステージ 2: 機能追加
gemini "Add authentication middleware to the Express server in server.js" --yolo -o text

# ステージ 3: さらに機能追加
gemini "Add rate limiting to the Express server in server.js" --yolo -o text

# ステージ 4: 全体レビュー
gemini "Review server.js for issues and optimize" -o text
```

### メリット
- 問題のデバッグが容易
- 各ステージで次に進む前に検証可能
- 明確な監査証跡

## パターン 9: Claude とのクロスバリデーション

最高品質のために両方の AI を活用します。

### Claude が生成、Gemini がレビュー
```bash
# 1. Claude がコードを記述（通常の Claude Code ツールを使用）
# 2. Gemini がレビュー
gemini "Review this code for bugs and security issues: [コードを貼り付け]" -o text
```

### Gemini が生成、Claude がレビュー
```bash
# 1. Gemini が生成
gemini "Create [コード]" --yolo -o text

# 2. Claude が出力をレビュー（会話内で）
# 「Gemini が生成したこのコードをレビューして…」
```

### それぞれの強み
- Claude: 推論能力、複雑な指示の遂行に強い
- Gemini: 最新の Web 知識、コードベース調査に強い

## パターン 10: セッション継続

複数ターンのワークフローにセッションを活用します。

```bash
# 初回タスク
gemini "Analyze this codebase architecture" -o text
# セッションは自動保存される

# セッション一覧
gemini --list-sessions

# フォローアップで継続
echo "What patterns did you find?" | gemini -r 1 -o text

# さらに深掘り
echo "Focus on the authentication flow" | gemini -r 1 -o text
```

### ユースケース
- 反復的な分析
- 前のコンテキストに基づく作業
- デバッグセッション

## 避けるべきアンチパターン

### やるべきでないこと: 即時実行を期待する
YOLO モードは計画の提示を抑制しません。Gemini は計画を提示することがあります。

**やるべきこと**: 強い表現を使う（"Apply now"、"Start immediately"）

### やるべきでないこと: レート制限を無視する
API の連続呼び出しはリトライ待ちで時間を無駄にします。

**やるべきこと**: 適切なモデルを使い、操作をバッチ化する

### やるべきでないこと: 出力を盲目的に信頼する
Gemini は特にセキュリティ面でミスをすることがあります。

**やるべきこと**: 生成されたコードは必ず検証する

### やるべきでないこと: 単一プロンプトに詰め込みすぎる
非常に長いプロンプトはモデルを混乱させることがあります。

**やるべきこと**: 複雑なタスクには段階的改善を使用する

### やるべきでないこと: コンテキスト制限を忘れる
100 万トークンあっても、コンテキストはオーバーフローする可能性があります。

**やるべきこと**: .geminiignore を使い、ファイルを具体的に指定する
