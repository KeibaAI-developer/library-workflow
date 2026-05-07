# library-workflow

KeibaAIの各Pythonライブラリが共有するGitHub Actions reusable workflowを管理するリポジトリ。

## ワークフロー

### `.github/workflows/ci.yml`

lint・型チェック・テストを実行するCIワークフロー。

**ジョブ構成**

1. `lint-and-type-check`: isort / flake8 / mypy を実行
2. `test`: 単体テスト・結合テストを実行（`lint-and-type-check` 完了後）

## 使用方法

各ライブラリの `.github/workflows/ci.yml` から以下のように呼び出す。

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  ci:
    uses: KeibaAI-developer/library-workflow/.github/workflows/ci.yml@main
    with:
      package_name: my_package
      repo_name: my-repo
```

## 引数

| 引数 | 型 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| `package_name` | string | ✅ | - | パッケージのディレクトリ名（isort / flake8 / mypy / pytest のターゲット） |
| `repo_name` | string | ✅ | - | リポジトリ名（pipキャッシュキーのプレフィックスに使用） |
| `run_unit_test` | boolean | | `true` | 単体テストを実行するかどうか |
| `run_integration_test` | boolean | | `false` | 結合テストを実行するかどうか |
| `dependency_library` | string | | `''` | 依存ライブラリのリスト（スペース区切り） |

### `dependency_library` について

GitHubの `KeibaAI-developer` organization配下のリポジトリ名をスペース区切りで指定する。
指定されたリポジトリは `pip install git+https://github.com/KeibaAI-developer/<lib>.git` でインストールされる。

```yaml
with:
  package_name: my_package
  repo_name: my-repo
  dependency_library: 'keiba-scraping mykeibadb-python'
```

## 設定例

### 依存ライブラリなし・単体テストのみ

```yaml
jobs:
  ci:
    uses: KeibaAI-developer/library-workflow/.github/workflows/ci.yml@main
    with:
      package_name: discord_logger
      repo_name: discord-logger
```

### 依存ライブラリあり

```yaml
jobs:
  ci:
    uses: KeibaAI-developer/library-workflow/.github/workflows/ci.yml@main
    with:
      package_name: rating
      repo_name: rating
      dependency_library: 'keiba-data-interface db-client race-data'
```

### 単体テスト・結合テストの両方を実行

```yaml
jobs:
  ci:
    uses: KeibaAI-developer/library-workflow/.github/workflows/ci.yml@main
    with:
      package_name: keiba_data_interface
      repo_name: keiba-data-interface
      run_integration_test: true
      dependency_library: 'keiba-scraping mykeibadb-python'
```
