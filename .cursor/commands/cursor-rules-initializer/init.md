# cursor-rules-initializer

## 指示

あなたは、このリポジトリの構造とアーキテクチャを分析し、適切なCursor Rulesファイルを生成するAIアシスタントです。
以下の手順に従って、`.cursor/rules`ディレクトリとルールファイルを作成してください。

## ステップ1: Cursor Rulesの仕組みを理解する

### プロジェクトルールの基本

Cursorでは、プロジェクト固有のルールを定義する方法が2つあります：

1. **`AGENTS.md`ファイル（単一ファイル方式）**
   - プロジェクトルート直下に配置
   - シンプルなプロジェクトや少数のルールに適している
   - フロントマター不要、全ての内容が常に適用される

2. **`.cursor/rules/`ディレクトリ（複数ファイル方式）**
   - ルールをカテゴリごとに分割管理できる
   - フロントマターで適用条件を細かく制御可能
   - 大規模プロジェクトやチーム開発に適している
   - **このプロンプトではこちらの方式を採用**

### フロントマターの詳細

各ルールファイル（`.mdc`ファイル）の先頭には、YAMLフロントマターで適用条件を指定します：

**パターン1: グローバルルール（常に適用）**
```yaml
---
description: プロジェクト全体のアーキテクチャ概要
alwaysApply: true
---
```

**パターン2: コンテキスト依存ルール（特定ファイル編集時のみ適用）**
```yaml
---
description: domain層の実装に適用するルール
globs: app/domain/**/*.go
alwaysApply: false
---
```

#### `description`フィールド
- ルールの目的や適用範囲を簡潔に説明
- AIがルールをいつ適用すべきか判断する際のヒントになる
- 例: `"domain層の実装に適用するルール"`, `"TypeScript全般のコーディング規約"`

#### `globs`フィールド（ファイルパターンマッチング）

[minimatch](https://github.com/isaacs/minimatch)形式でファイルパターンを指定します。

**⚠️ このフィールドは`alwaysApply: false`の場合のみ指定します。`alwaysApply: true`の場合は指定不要です。**

**⚠️ 重要な記法ルール**
- **クォーテーションで囲まない**: ダブルクォーテーション（`"`）やシングルクォーテーション（`'`）で囲んではいけません
- **複数パターンはカンマ区切り**: 配列形式（`-`を使ったリスト）ではなく、カンマ（`,`）で区切ります
- **スペースは入れない**: カンマの前後にスペースを入れないでください

**基本パターン**
```yaml
# ✅ 正しい例: 特定のディレクトリ配下の全Goファイル
globs: app/domain/**/*.go

# ✅ 正しい例: 特定のディレクトリ配下の全TypeScript/TSXファイル
globs: src/components/**/*.{ts,tsx}

# ✅ 正しい例: 複数のディレクトリパターン（カンマ区切り）
globs: lib/view/**/*.dart,lib/viewmodel/**/*.dart,lib/mixin/**/*.dart

# ✅ 正しい例: テストファイル（複数拡張子対応）
globs: **/*_test.go,**/*_test.ts

# ❌ 間違った例: クォーテーションで囲んでいる
globs: "app/domain/**/*.go"

# ❌ 間違った例: 配列形式（リスト記法）を使用
globs:
  - app/domain/**/*.go
  - app/model/**/*.go

# ❌ 間違った例: カンマの後にスペースがある
globs: app/domain/**/*.go, app/model/**/*.go
```

**パターン記法**
- `**`: 任意の深さのディレクトリマッチ（`app/**/domain`は`app/domain`, `app/foo/domain`, `app/foo/bar/domain`など）
- `*`: 任意の文字列（ただし`/`は含まない）
- `?`: 任意の1文字
- `{a,b}`: aまたはb（例: `*.{js,ts}`は`.js`または`.ts`）
- `[0-9]`: 数字の範囲
- `[!a-z]`: a-z以外
- `,`: 複数パターンの区切り（スペースなし）

**複数パターンの指定例**
```yaml
# Go言語のdomain層とmodel層
globs: app/domain/**/*.go,app/model/**/*.go

# TypeScriptのコンポーネントとフック
globs: src/components/**/*.{ts,tsx},src/hooks/**/*.{ts,tsx}

# 複数のテストファイルパターン
globs: **/*_test.go,**/*.test.ts,**/*.spec.ts

# Dartのプレゼンテーション層
globs: lib/view/**/*.dart,lib/viewmodel/**/*.dart,lib/mixin/**/*.dart
```

#### `alwaysApply`フィールド

ルールの適用タイミングを制御します：

- **`alwaysApply: true`** - グローバルルール
  - ファイル編集に関係なく常に適用される
  - プロジェクト全体の方針、アーキテクチャ概要、言語ガイドラインなどに使用
  - 例: `architecture.mdc`, `go-guidelines.mdc`, `general-rule.mdc`
  - ⚠️ パフォーマンスへの影響を考慮し、必要最小限にすること
  - **`globs`フィールドは不要**（指定しても無視されます）

- **`alwaysApply: false`** - コンテキスト依存ルール（デフォルト）
  - `globs`パターンに一致するファイルを編集している時のみ適用される
  - レイヤー固有、モジュール固有のルールに使用
  - 例: `domain-rule.mdc`（`app/domain/**/*.go`を編集中のみ）
  - 📌 ほとんどのルールはこちらを使用

### ルール適用の仕組み

1. **ファイル編集開始時**: 編集中のファイルパスがチェックされる
2. **パターンマッチング**: 各ルールファイルの`globs`と照合
3. **ルール適用**:
   - `alwaysApply: true`のルールは常に適用
   - 編集ファイルが`globs`パターンに一致するルールが追加で適用
4. **複数ルール**: 該当する全てのルールがAIのコンテキストに含まれる

**例：** `app/domain/user/user_entity.go`を編集する場合
- ✅ `base/architecture.mdc` (alwaysApply: true)
- ✅ `base/go-guidelines.mdc` (alwaysApply: true)
- ✅ `app/domain-rule.mdc` (globs: `app/domain/**/*.go`)
- ❌ `app/usecase-rule.mdc` (globs: `app/usecase/**/*.go` - マッチしない)

### ルールの分類（推奨ディレクトリ構成）

```
.cursor/rules/
├── base/          # プロジェクト全体の基本ルール（alwaysApply: true推奨）
├── app/           # レイヤー/モジュール固有ルール（alwaysApply: false）
└── testing/       # テスト関連ルール（alwaysApply: false）
```

#### 1. **`base/`** - プロジェクト全体に適用される基本ルール

常に適用される基盤となるルール群：

- **`architecture.mdc`** (alwaysApply: true)
  - アーキテクチャ全体の概要、設計思想、依存関係の方向など
  - 全ての実装判断の基準となる最重要ドキュメント

- **`{言語名}-guidelines.mdc`** (alwaysApply: true)
  - 使用言語固有のコーディング規約
  - 例: `go-guidelines.mdc`, `typescript-guidelines.mdc`, `python-guidelines.mdc`
  - 命名規則、フォーマット、エラーハンドリング、ベストプラクティス

- **`general-rule.mdc`** (alwaysApply: true)
  - プロジェクト固有の全般的なルール
  - タスク実行のプロセス、品質管理、レポート形式など

- **その他の特定用途ルール** (alwaysApply: true/false)
  - `api-rule.mdc`: API設計のルール
  - `security-rule.mdc`: セキュリティガイドライン

#### 2. **`app/`** - アプリケーションのレイヤー/モジュールごとのルール

アーキテクチャのレイヤーに対応した詳細ルール（全てalwaysApply: false推奨）：

**クリーンアーキテクチャの例：**
- `domain-rule.mdc` (globs: `app/domain/**/*.{go,ts}`)
- `usecase-rule.mdc` (globs: `app/usecase/**/*.{go,ts}`)
- `infrastructure-rule.mdc` (globs: `app/infrastructure/**/*.{go,ts}`)
- `presentation-rule.mdc` (globs: `app/presentation/**/*.{go,ts}`)

**MVC/MVVMの例：**
- `model-rule.mdc` (globs: `src/models/**/*.ts`)
- `view-rule.mdc` (globs: `src/views/**/*.{vue,tsx}`)
- `controller-rule.mdc` (globs: `src/controllers/**/*.ts`)

**機能モジュール別の例：**
- `auth-rule.mdc` (globs: `src/modules/auth/**/*`)
- `payment-rule.mdc` (globs: `src/modules/payment/**/*`)

#### 3. **`testing/`** - テストに関するルール

テスト実装の規則（alwaysApply: false）：

- **`test-rule.mdc`**
  ```yaml
  globs: **/*_test.go,**/*.test.ts,**/*.spec.ts
  alwaysApply: false
  ```
  - テストファイルの命名規則
  - テスト構造（テーブル駆動テストなど）
  - モック使用方法、アサーション方法
  - カバレッジ目標、レイヤー別テスト戦略

### ベストプラクティス

#### ✅ 推奨事項

1. **ディレクトリベースのルール管理を使用**
   - `.cursor/rules/`ディレクトリで管理
   - カテゴリ分けにより可読性と保守性が向上

2. **alwaysApply: trueは最小限に**
   - グローバルルールが多いとパフォーマンスに影響
   - 本当に常時必要なルールのみに限定（通常3-5ファイル程度）

3. **globsパターンは具体的に**
   - `**/*`のような広すぎるパターンは避ける
   - 適用範囲を明確にする（`app/domain/**/*.go`など）

4. **ルールファイル名は内容を反映**
   - `domain-rule.mdc`, `user-service-rule.mdc`のように明確に
   - チームメンバーが内容を推測できる名前にする

5. **実装例リンクを活用**
   - 標準のMarkdownリンク形式で実際のコードにリンク
   - ルールの具体的な適用例を示す

#### ❌ 避けるべきパターン

1. **過度に細分化されたルール**
   - ファイルごとに別々のルールを作成（管理が煩雑）
   - 関連する内容は1つのファイルにまとめる

2. **重複するルール**
   - 同じ内容が複数のファイルに記述される
   - 矛盾するルールが存在する

3. **曖昧なglobsパターン**
   - 意図しないファイルにマッチしてしまう
   - テストで実際にマッチするか確認する

### トラブルシューティング

#### ルールが適用されない場合

1. **globsパターンの確認**
   ```bash
   # ファイルパスがパターンに一致するか確認
   # 例: app/domain/user/user.go が app/domain/**/*.go にマッチするか
   ```

2. **ファイル拡張子の確認**
   - `.mdc`拡張子を使用しているか
   - フロントマターの形式が正しいか（YAMLの構文エラーがないか）

3. **alwaysApplyの設定**
   - 常に適用したい場合は`alwaysApply: true`
   - 特定ファイルのみの場合は`alwaysApply: false`と適切な`globs`

#### パフォーマンスが気になる場合

1. **alwaysApply: trueのルールを見直す**
   - 本当に常時必要か再検討
   - レイヤー固有のルールはalwaysApply: falseに変更

2. **ルールの内容を簡潔に**
   - 不要に長いルールはAIの処理時間を増やす
   - 要点を絞った記述を心がける

## ステップ2: リポジトリの分析

以下の情報を徹底的に調査してください：

### 2-1. 基本情報の収集
- **プロジェクト名**: リポジトリ名やpackage.json、go.mod等から特定
- **使用言語**: 主要な言語とバージョン
- **フレームワーク**: 使用しているフレームワークやライブラリ
- **技術スタック**: データベース、API形式（REST/gRPC）、その他の主要技術

### 2-2. ディレクトリ構造の把握

`list_dir`ツールを使用してディレクトリ構造を段階的に調査してください：

**調査手順：**

1. **ルートディレクトリの確認**
   - プロジェクトルートで`list_dir`を実行し、主要なディレクトリを特定
   - アプリケーションコードの配置場所（`app/`, `src/`, `lib/`など）を確認

2. **主要ディレクトリの深掘り**
   - 特定した主要ディレクトリに対して`list_dir`を実行
   - レイヤー構造の有無を確認（domain, usecase, infrastructure, presentationなど）
   - モジュール/フィーチャー構造の有無を確認（features/, modules/, services/など）

3. **重要なサブディレクトリの確認**
   - アーキテクチャパターンを特定するために必要な階層まで確認
   - 通常は2-3階層まで確認すれば十分

**特に以下の点に注目：**
- アプリケーションコードの配置場所（`app/`, `src/`, `lib/`など）
- レイヤー構造の有無（domain, usecase, infrastructure, presentationなど）
- モジュール/フィーチャー構造の有無（features/, modules/など）
- テストファイルの配置パターン（`**/*_test.go`, `**/*.test.ts`など）
- CLIや特殊なコマンドの有無（`cmd/`, `cli/`, `scripts/`など）

**効率的な調査のコツ：**
- 全てのディレクトリを確認する必要はありません
- アーキテクチャパターンを特定するために必要な情報に集中してください
- 不明確な場合は、主要なディレクトリを優先的に確認してください

### 2-3. アーキテクチャパターンの特定

リポジトリのアーキテクチャパターンを詳細に特定してください。これは`app/`ルール作成の基盤となる重要なステップです。

#### レイヤードアーキテクチャ系

**クリーンアーキテクチャ / オニオンアーキテクチャ**
- 特徴: domain, usecase, infrastructure, presentationなどのレイヤーに明確に分離
- 依存関係: 外側から内側（presentation → usecase → domain）
- ディレクトリ例: `app/domain/`, `app/usecase/`, `app/infrastructure/`, `app/presentation/`

**ヘキサゴナルアーキテクチャ（Ports & Adapters）**
- 特徴: core（ドメイン）、ports（インターフェース）、adapters（実装）
- ディレクトリ例: `src/core/`, `src/ports/`, `src/adapters/`

**従来型3層アーキテクチャ**
- 特徴: プレゼンテーション層、ビジネスロジック層、データアクセス層
- ディレクトリ例: `src/controllers/`, `src/services/`, `src/repositories/`

**MVC / MVVM**
- 特徴: Model, View, Controller（またはViewModel）に分離
- ディレクトリ例: `src/models/`, `src/views/`, `src/controllers/`

#### 機能/モジュールベースアーキテクチャ

**フィーチャーベースアーキテクチャ**
- 特徴: 技術的なレイヤーではなく、機能（フィーチャー）ごとにディレクトリ分割
- 各フィーチャー内に独自のmodel/view/controller等を持つ
- ディレクトリ例: `src/features/auth/`, `src/features/payment/`, `src/features/user/`

**モジュラーモノリス**
- 特徴: 独立したモジュール（アプリケーション）が1つのリポジトリに共存
- 各モジュールが独自のドメイン、ユースケースなどを持つ
- ディレクトリ例: `app/modules/cms/`, `app/modules/table/`, `app/modules/pickup/`

**マイクロサービス風構造**
- 特徴: 各サービスがディレクトリで分離され、独立してデプロイ可能
- ディレクトリ例: `services/user-service/`, `services/order-service/`

#### フレームワーク固有のアーキテクチャ

**Rails風（Ruby on Rails）**
- ディレクトリ例: `app/models/`, `app/controllers/`, `app/views/`, `app/helpers/`

**Django風（Python）**
- ディレクトリ例: `apps/blog/`, `apps/shop/`（各アプリに`models.py`, `views.py`等）

**Next.js風（React）**
- ディレクトリ例: `app/`, `pages/`, `components/`, `lib/`

**NestJS風（Node.js/TypeScript）**
- ディレクトリ例: `src/modules/users/`, `src/modules/auth/`（各モジュールに`controller.ts`, `service.ts`等）

#### その他のパターン

**シンプルな機能別分類**
- 特徴: 明確なレイヤーやモジュールはなく、機能カテゴリで分類
- ディレクトリ例: `src/api/`, `src/utils/`, `src/helpers/`, `src/services/`

**フラット構造**
- 特徴: 深い階層がなく、src/配下に直接ファイル/ディレクトリ
- ディレクトリ例: `lib/auth.ts`, `lib/db.ts`, `lib/utils.ts`

#### 特定方法のチェックリスト

以下の質問に答えて、アーキテクチャパターンを特定してください：

1. **レイヤー構造があるか？**
   - domain, usecase, infrastructure, presentationのような技術的レイヤーの分離
   - YES → レイヤードアーキテクチャ系

2. **機能/モジュールで分割されているか？**
   - features/, modules/, services/のようなディレクトリ構造
   - 各ディレクトリが独立した機能を持つ
   - YES → 機能/モジュールベースアーキテクチャ

3. **フレームワークの規約に従っているか？**
   - Rails, Django, Next.jsなどのフレームワーク特有のディレクトリ構造
   - YES → フレームワーク固有のアーキテクチャ

4. **明確な構造がないか？**
   - シンプルな分類またはフラット構造
   - YES → その他のパターン

**注意**: 複数のパターンが混在している場合もあります（例: モジュラーモノリス + 各モジュールがクリーンアーキテクチャ）

### 2-4. コーディング規約の確認
既存のコードから以下を確認：
- 命名規則（ファイル、クラス、関数、変数）
- ディレクトリ命名パターン（snake_case, kebab-case, PascalCaseなど）
- エラーハンドリングのパターン
- 依存性注入の方式
- テスト戦略（ユニットテスト、統合テスト）

## ステップ3: サンプルの参照

`sample/rules/`ディレクトリには、参考となるサンプルルールが含まれています。
このサンプルは Go言語 + クリーンアーキテクチャ のプロジェクト例です。

**サンプルを参照する際の注意点：**
- サンプルはGo言語のプロジェクトですが、あなたのリポジトリが別の言語の場合は適切に変換してください
- ディレクトリ構造が異なる場合は、globs設定を調整してください
- サンプルをそのまま使用せず、分析したあなたのリポジトリに合わせた内容にすること
  - 特にサンプルで使用されているパッケージ構成や名称、ツールなどをそのまま使用しない

## ステップ4: ルールファイルの生成

以下の手順でルールファイルを生成してください：

### 4-1. ディレクトリ構造の作成
```
.cursor/
└── rules/
    ├── base/
    ├── app/
    └── testing/
```

### 4-2. base/ルールの作成

#### `base/architecture.mdc`
- **alwaysApply: true**
- 内容に含めるべき項目：
  - プロジェクト概要
  - 技術スタック
  - アーキテクチャの説明
  - ディレクトリ構造
  - レイヤー構造と責務
  - データフロー
  - 依存性の方向
  - 開発プラクティス
  - AI開発のためのガイドライン

#### `base/{言語名}-guidelines.mdc`
- **alwaysApply: true**
- 言語固有のコーディング規約：
  - フォーマット規則
  - 命名規則
  - エラーハンドリング
  - 言語特有のベストプラクティス
  - 並行処理やメモリ管理など

#### `base/general-rule.mdc`
- **alwaysApply: true**
- プロジェクト全体に適用される一般的なルール：
  - タスク分析と計画の方法
  - タスク実行の手順
  - 品質管理
  - 結果報告のフォーマット
  - 重要な注意事項

### 4-3. app/ルールの作成

**重要**: `app/`ルールは、ステップ2-3で特定したアーキテクチャパターンに応じて作成します。
画一的に「レイヤー」として扱うのではなく、リポジトリの実際の構造に合わせてください。

#### 基本方針

1. **明確な構造がある場合のみ作成**
   - レイヤー、モジュール、フィーチャーなど、明確な分割がある場合
   - その構造に応じた適切なルールファイルを作成

2. **構造が不明確な場合は作成しない**
   - フラット構造やシンプルな分類の場合
   - `base/`のルールで十分カバーできる場合は、`app/`ディレクトリ自体を作らない

3. **ルールファイルの粒度**
   - 小規模プロジェクト: 大まかなルールで十分
   - 大規模プロジェクト: 詳細なルールが必要

#### アーキテクチャパターン別のルール作成指針

##### パターン1: レイヤードアーキテクチャ（DDD/クリーンアーキテクチャ等）

**特徴**: domain, usecase, infrastructure, presentationなどのレイヤーに分離

**ルールファイル例**:
```
.cursor/rules/app/
├── domain-rule.mdc
├── usecase-rule.mdc
├── infrastructure-rule.mdc
└── presentation-rule.mdc
```

**`domain-rule.mdc`の例**:
```yaml
---
description: domain層の実装に適用するルール
globs: app/domain/**/*.go
alwaysApply: false
---

# Domain層の実装ルール

## 役割
- ビジネスロジックの中心
- エンティティと値オブジェクトの定義
- リポジトリインターフェースの定義
- ドメインサービスの実装

## 詳細構造と命名規則
[実際のディレクトリ構造に基づいて記述]

## 実装ポイント
- ドメイン駆動設計のプリンシパルに従う
- エンティティは状態と振る舞いを持つ
- 値オブジェクトは不変
[...]

## 実装例
- Entity: [user_entity.go](../../../app/domain/entity/user_entity.go)
- Value Object: [user_id.go](../../../app/domain/value_object/user_id.go)
- Repository Interface: [user_repository.go](../../../app/domain/repository/user_repository.go)
```

**各レイヤーのルールに含める内容**:
- **役割**: レイヤーの責務と境界
- **依存関係**: 許可される依存の方向（例: domain層への依存のみ）
- **禁止事項**: 他のレイヤーへの不正な依存など
- **命名規則**: ファイル、クラス、関数の命名パターン
- **実装規則**: 基本構造、エラーハンドリング、トランザクション管理など
- **アンチパターン**: 避けるべき実装
- **実装例**: 既存コードへのリンク（標準のMarkdownリンク形式）

##### パターン2: フィーチャーベース/モジュラーモノリス

**特徴**: 機能やモジュールごとにディレクトリが分かれ、各ディレクトリが独立

**ルールファイル例**:

**オプション1: 共通のフィーチャールール**
```
.cursor/rules/app/
└── feature-rule.mdc
```

```yaml
---
description: 各フィーチャー/モジュールの実装ルール
globs: src/features/**/*
alwaysApply: false
---

# フィーチャー実装ルール

## 役割
各フィーチャーは独立したビジネス機能を提供

## フィーチャー内の推奨構造
src/features/{feature-name}/
├── components/  # UI components
├── hooks/       # React hooks
├── services/    # Business logic
├── types/       # Type definitions
└── index.ts     # Public exports

## モジュール間の依存関係ルール
- 他のフィーチャーへの直接的な依存は避ける
- 共通機能は`src/shared/`に配置
[...]
```

**オプション2: 主要フィーチャーごとに個別ルール**
```
.cursor/rules/app/
├── auth-feature-rule.mdc
├── payment-feature-rule.mdc
└── user-feature-rule.mdc
```

各ファイルには、そのフィーチャー固有のビジネスルールや制約を記載。

**各ルールに含める内容**:
- **フィーチャーの責務範囲**: 何を扱い、何を扱わないか
- **モジュール内の構造**: 推奨されるディレクトリ/ファイル構成
- **他モジュールとの境界**: 依存関係のルール、公開インターフェース
- **共通機能の使い方**: shared/commonディレクトリの利用方法

##### パターン3: 従来型MVC/3層アーキテクチャ

**特徴**: models, views, controllers/handlersに分離

**ルールファイル例**:
```
.cursor/rules/app/
├── model-rule.mdc
├── controller-rule.mdc
└── view-rule.mdc
```

**各ルールに含める内容**:
- **各層の責務**: モデルはビジネスロジック、コントローラーはルーティング等
- **設計方針**: Fat ModelかFat Controllerか（プロジェクトの方針に従う）
- **データフロー**: Controller → Model → Viewのデータの流れ
- **命名規則**: `UserController`, `user_model.py`など

##### パターン4: フレームワーク固有のアーキテクチャ

**Rails, Django, Next.js等の場合**: フレームワークの規約に従ったルールを作成

**例（Next.js）**:
```
.cursor/rules/app/
├── app-router-rule.mdc      # app/ディレクトリのルール
├── api-route-rule.mdc       # API routesのルール
└── components-rule.mdc      # components/のルール
```

##### パターン5: シンプル/フラット構造

**判断基準**: 以下の場合は`app/`ディレクトリを作成しない
- 明確なレイヤーやモジュール構造がない
- 小規模プロジェクトで`base/`のルールで十分
- ディレクトリが浅く、機能カテゴリも不明確

**対処法**:
- `base/`のルールで全体をカバー
- 必要に応じて`.cursor/rules/api-rule.mdc`など、最小限のルールのみ追加

#### ルール作成の判断フロー

```
1. 明確なレイヤー/モジュール構造があるか？
   ├─ YES → 各レイヤー/モジュールごとにルールファイルを作成
   └─ NO → 次へ

2. 機能カテゴリで分類可能か？（api/, utils/, services/など）
   ├─ YES → 主要な機能カテゴリごとにルールファイルを作成
   └─ NO → 次へ

3. app/ルールは必要か？
   ├─ 明確な構造がなく、base/のルールで十分 → app/ディレクトリを作成しない
   └─ 特定の領域に固有のルールが必要 → 最小限のルールファイルを作成
```

#### ルールファイルの共通フォーマット

各ルールファイルには以下の要素を含めてください（アーキテクチャに応じて調整）：

**必須項目**:
- **フロントマター**: description, alwaysApply（`alwaysApply: false`の場合は`globs`も記載）
- **役割**: この領域の責務
- **実装規則**: 基本的な実装方法、命名規則

**推奨項目**:
- **詳細構造と命名規則**: ディレクトリ構造とファイル命名
- **実装ポイント**: 実装時の注意点、ベストプラクティス
- **依存関係**: 他の領域との依存関係ルール
- **アンチパターン**: 避けるべき実装
- **テスト規則**: テストの書き方（該当する場合）
- **実装例**: 既存コードへのリンク（標準のMarkdownリンク形式）

#### 実装例リンクの重要性

各ルールファイルには、必ず実際のコードへのリンクを含めてください。
**重要**: リンクは`.cursor/rules/`配下から見た相対パスで、必ず`../../../`から始めます。

```markdown
## 実装例
- Entity: [user_entity.go](../../../app/domain/entity/user_entity.go)
- Repository Interface: [user_repository.go](../../../app/domain/repository/user_repository.go)
- Repository Implementation: [user_repository.go](../../../app/infrastructure/repository/user_repository.go)
```

これにより、AIが具体的なコーディングスタイルを参照できます。
詳細な記述方法はステップ5を参照してください。

### 4-4. testing/ルールの作成

#### `testing/test-rule.mdc`
- **globs**: テストファイルのパターン
  - 複数パターンはカンマ区切りで指定
  - 例: `**/*_test.go,**/*.test.ts,**/*.spec.ts`
- **alwaysApply: false**
- 内容：
  - テストファイルの命名規則
  - テスト関数の命名規則
  - テスト構造（テーブル駆動テストなど）
  - モックの使用方法
  - アサーション方法
  - テストカバレッジの目標
  - レイヤー別テスト戦略

## ステップ5: 実装例リンクの追加

各ルールファイルには、実際のコードへのリンクを含めてください。
これにより、AIが具体的なコーディングスタイルを参照できます。

### リンク記述の重要ルール

#### ⚠️ 必ず守るべき記法

1. **標準のMarkdownリンク形式のみを使用**
   - `[表示名](相対パス)`の形式を使用
   - **「mdc:」などの接頭辞は絶対に付けない**

2. **相対パスの正確な計算**
   - ルールファイルの場所からプロジェクトルートへ戻る相対パスを使用
   - `.cursor/rules/`のディレクトリ構造に応じて`../`の数を調整

#### 相対パスの計算方法

`.cursor/rules/`配下のルールファイルからプロジェクトルートに戻るには、以下の階層数の`../`が必要です：

- `.cursor/rules/base/xxx.mdc` → プロジェクトルート: `../../../`
- `.cursor/rules/app/xxx.mdc` → プロジェクトルート: `../../../`
- `.cursor/rules/testing/xxx.mdc` → プロジェクトルート: `../../../`

**計算式**: ルールファイルの深さに応じて`../`を追加
- `.cursor/` = 1階層
- `rules/` = 2階層
- `base/`または`app/`または`testing/` = 3階層
- **合計3階層 = `../../../`**

#### 正しいリンク記述例

**ケース1: `.cursor/rules/app/domain-rule.mdc`から`app/domain/entity/user.go`へのリンク**
```markdown
## 実装例
- Entity: [user.go](../../../app/domain/entity/user.go)
- Value Object: [user_id.go](../../../app/domain/value_object/user_id.go)
```

**ケース2: `.cursor/rules/base/architecture.mdc`から各種ファイルへのリンク**
```markdown
## 参考実装
- Domain層: [user_entity.go](../../../app/domain/entity/user_entity.go)
- Usecase層: [user_usecase.go](../../../app/usecase/user/user_usecase.go)
- Infrastructure層: [user_repository.go](../../../app/infrastructure/repository/user_repository.go)
```

**ケース3: `.cursor/rules/testing/test-rule.mdc`からテストファイルへのリンク**
```markdown
## テスト実装例
- Entity Test: [user_test.go](../../../app/domain/entity/user_test.go)
- Usecase Test: [user_usecase_test.go](../../../app/usecase/user/user_usecase_test.go)
```

#### ❌ 間違った記述例

```markdown
# 間違い1: mdc:接頭辞を使用している
- Entity: [user.go](mdc:app/domain/entity/user.go)

# 間違い2: 相対パスの階層数が不足
- Entity: [user.go](app/domain/entity/user.go)
- Entity: [user.go](../app/domain/entity/user.go)

# 間違い3: 絶対パスを使用
- Entity: [user.go](/app/domain/entity/user.go)

# 間違い4: プロジェクトルートから遷移できない記述
- Entity: [user.go](../../app/domain/entity/user.go)  # ../が2つしかない
```

### リンク追加のチェックリスト

実装例リンクを追加する際は、以下を確認してください：

- [ ] 標準のMarkdownリンク形式`[表示名](相対パス)`を使用
- [ ] 「mdc:」などの接頭辞を付けていない
- [ ] ルールファイルの階層に応じた正確な相対パス（通常は`../../../`）
- [ ] リンク先のファイルが実際に存在する
- [ ] プロジェクトルートから正しく遷移できる相対パス

## ステップ6: 検証とフィードバック

生成したルールファイルについて、以下を報告してください：

1. **生成したファイルのリスト**
2. **各ファイルの目的と適用範囲**
3. **特記事項**：
   - このリポジトリ特有の設計パターン
   - サンプルから大きく変更した点
   - 追加で必要になる可能性のあるルール

## 重要な注意事項

### アーキテクチャに関する注意

- **アーキテクチャに合わせた柔軟な設計**: サンプルはDDD/オニオンアーキテクチャですが、あなたのリポジトリのアーキテクチャが異なる場合は、その構造に合わせてルールを設計してください
- **app/ディレクトリは必須ではない**: 明確なレイヤー/モジュール構造がない場合、`app/`を作らず`base/`と`testing/`のみで構成することも有効です
- **ルールの粒度は状況次第**: 小規模プロジェクトでは大まかなルールで十分、大規模プロジェクトでは詳細なルールが必要です
- **実際の構造を優先**: サンプルの構造に無理に合わせるのではなく、リポジトリの実際の構造を尊重してください

### 実装に関する注意

- **プロジェクト固有の用語は適切に置き換える**: サンプルに含まれる固有名詞（"ToGo", "CMS", "Table"等）は、あなたのリポジトリに合わせて変更してください
- **globs設定は正確に**: ファイルパターンが実際のディレクトリ構造と一致することを確認してください
- **説明は日本語で**: ルールの内容は日本語で記述してください（プロジェクトの言語設定に応じて調整可）
- **実装例は実在するファイルへ**: リンクは実際に存在するファイルを参照してください
- **段階的に生成**: まずbase/を作成し、確認後にapp/とtesting/を作成する流れが推奨されます

### 実装例リンクに関する注意

- **標準Markdownリンクのみ使用**: `[表示名](相対パス)`の形式で、「mdc:」などの接頭辞は絶対に使用しない
- **相対パスは必ず`../../../`から開始**: `.cursor/rules/`配下（3階層目）から見たプロジェクトルートへの相対パスは常に`../../../`
- **リンクの動作確認**: 生成後、実際にリンクが機能することを確認する（ファイルが存在し、相対パスが正しいか）
- **ステップ5の詳細を参照**: 実装例リンクの正確な記述方法はステップ5に詳述されているため、必ず参照すること

### 判断に迷った場合

- **シンプルに保つ**: 不要なルールは作らない方が良い。迷ったら最小限のルールから始める
- **段階的に追加**: 最初は大まかなルールで運用し、必要に応じて詳細化する
- **サンプルは参考**: サンプルは1つの例に過ぎません。あなたのリポジトリに最適な形を追求してください

## 開始の合図

準備ができたら、「リポジトリの分析を開始します」と宣言し、ステップ2から順に実行してください。
