# Gemini CLI プロンプトテンプレート

一般的な操作に使える再利用可能なプロンプトテンプレート集です。

## コード生成

### 単一ファイルアプリケーション
```bash
gemini "Create a [説明] with [機能]. Include [要件]. Output the complete file content." --yolo -o text
```

**例:**
```bash
gemini "Create a single-file HTML/CSS/JS calculator with: basic operations, history display, keyboard support, dark mode toggle, responsive design. Output the complete file content." --yolo -o text
```

### 複数ファイルプロジェクト
```bash
gemini "Create a [プロジェクト種別] with [技術スタック]. Include [機能]. Create all necessary files and make it runnable. Use modern best practices. START BUILDING NOW." --yolo -o text
```

**例:**
```bash
gemini "Create a REST API with Express, SQLite, and JWT auth. Include user CRUD, input validation, error handling. Create all necessary files and make it runnable. START BUILDING NOW." --yolo -o text
```

### コンポーネント/モジュール
```bash
gemini "Create a [コンポーネント種別] that [機能]. Follow [規約]. Include [要件]. Output the code." --yolo -o text
```

**例:**
```bash
gemini "Create a React hook useLocalStorage that syncs state with localStorage. Follow React 18 best practices. Include TypeScript types. Output the code." --yolo -o text
```

## コードレビュー

### 総合レビュー
```bash
gemini "Review [ファイル] and tell me:
1) What features it has
2) Any bugs or security issues
3) Suggestions for improvement
4) Code quality assessment" -o text
```

### セキュリティ重視のレビュー
```bash
gemini "Review [ファイル] for security vulnerabilities including:
- XSS (cross-site scripting)
- SQL injection
- Command injection
- Insecure data handling
- Authentication issues
Report findings with severity levels." -o text
```

### パフォーマンスレビュー
```bash
gemini "Analyze [ファイル] for performance issues:
- Inefficient algorithms
- Memory leaks
- Unnecessary re-renders
- Blocking operations
- Optimization opportunities
Provide specific recommendations." -o text
```

## バグ修正

### 特定されたバグの修正
```bash
gemini "Fix these bugs in [ファイル]:
1) [バグの説明]
2) [バグの説明]
3) [バグの説明]
Apply fixes now." --yolo -o text
```

### 自動検出と修正
```bash
gemini "Analyze [ファイル] for bugs, then fix all issues you find. Apply fixes immediately." --yolo -o text
```

## テスト生成

### ユニットテスト
```bash
gemini "Generate [フレームワーク] unit tests for [ファイル]. Cover:
- All public functions
- Edge cases
- Error handling
- [特定の領域]
Output the complete test file." --yolo -o text
```

**例:**
```bash
gemini "Generate Jest unit tests for utils.js. Cover:
- All exported functions
- Edge cases (empty input, null, undefined)
- Error handling
- Boundary conditions
Output the complete test file." --yolo -o text
```

### インテグレーションテスト
```bash
gemini "Generate integration tests for [コンポーネント/API]. Test:
- Happy path scenarios
- Error scenarios
- Edge cases
Use [フレームワーク]. Output complete test file." --yolo -o text
```

## ドキュメント

### JSDoc/TSDoc
```bash
gemini "Generate [JSDoc/TSDoc] documentation for all functions in [ファイル]. Include:
- Function descriptions
- Parameter types and descriptions
- Return types and descriptions
- Usage examples
Output as [形式]." --yolo -o text
```

### README 生成
```bash
gemini "Generate a README.md for this project. Include:
- Project description
- Installation instructions
- Usage examples
- API reference
- Contributing guidelines
Use the codebase to gather accurate information." --yolo -o text
```

### API ドキュメント
```bash
gemini "Document all API endpoints in [ファイル/ディレクトリ]. Include:
- HTTP method and path
- Request parameters
- Request body schema
- Response schema
- Example requests/responses
Output in [Markdown/OpenAPI] format." --yolo -o text
```

## コード変換

### リファクタリング
```bash
gemini "Refactor [ファイル] to:
- [具体的な改善点]
- [具体的な改善点]
Maintain all existing functionality. Apply changes now." --yolo -o text
```

### 言語変換
```bash
gemini "Translate [ファイル] from [ソース言語] to [ターゲット言語]. Maintain:
- Same functionality
- Similar code structure
- Idiomatic patterns for target language
Output the translated code." --yolo -o text
```

### フレームワーク移行
```bash
gemini "Convert [ファイル] from [旧フレームワーク] to [新フレームワーク]. Maintain all functionality. Use [新フレームワーク] best practices. Output the converted code." --yolo -o text
```

## Web 調査

### 最新情報
```bash
gemini "What are the latest [トピック] as of [日付]? Use Google Search to find current information. Summarize key points." -o text
```

### ライブラリ/API 調査
```bash
gemini "Research [ライブラリ/API] and provide:
- Latest version and changes
- Best practices
- Common patterns
- Gotchas to avoid
Use Google Search for current information." -o text
```

### 比較調査
```bash
gemini "Compare [選択肢 A] vs [選択肢 B] for [ユースケース]. Use Google Search for current benchmarks and community opinions. Provide recommendation." -o text
```

## アーキテクチャ分析

### プロジェクト分析
```bash
gemini "Use the codebase_investigator tool to analyze this project. Report on:
- Overall architecture
- Key dependencies
- Component relationships
- Potential issues" -o text
```

### 依存関係分析
```bash
gemini "Analyze dependencies in this project:
- Direct vs transitive
- Outdated packages
- Security vulnerabilities
- Bundle size impact
Use available tools to gather information." -o text
```

## 特殊なタスク

### Git コミットメッセージ
```bash
gemini "Analyze staged changes and generate a commit message following conventional commits format. Be concise but descriptive." -o text
```

### コード説明
```bash
gemini "Explain what [ファイル/関数] does in detail:
- Purpose and use case
- How it works step by step
- Key algorithms/patterns used
- Dependencies and side effects" -o text
```

### エラー診断
```bash
gemini "Diagnose this error:
[エラーメッセージ]
Context: [関連するコンテキスト]
Provide:
- Root cause
- Solution steps
- Prevention tips" -o text
```

## テンプレート変数

テンプレートで使用するプレースホルダー：

- `[ファイル]` - ファイルパスまたは名前
- `[ディレクトリ]` - ディレクトリパス
- `[説明]` - 簡単な説明
- `[機能]` - 機能の一覧
- `[要件]` - 具体的な要件
- `[フレームワーク]` - テスト/UI フレームワーク
- `[言語]` - プログラミング言語
- `[形式]` - 出力形式（Markdown、JSON など）
- `[日付]` - 時間に敏感なクエリの日付
- `[トピック]` - 調査対象のテーマ
