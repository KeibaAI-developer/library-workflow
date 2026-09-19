# library-workflow

KeibaAIの各Pythonライブラリが共有するGitHub Actions reusable workflowを管理するリポジトリ。

## ワークフロー

### `.github/workflows/ci.yml`

lint・型チェック・テストを実行するCIワークフロー。

**ジョブ構成**

1. `lint-and-type-check`: ruff / mypy を実行
2. `test`: 単体テスト・結合テストを実行（`lint-and-type-check` 完了後）

**静的解析ツールのバージョン**

ruffのバージョンはこのワークフローの `env.RUFF_VERSION` で集中管理しており、各ライブラリの
`pyproject.toml` には書かない。リポジトリごとに版がずれると同じコードでも判定結果が変わるため。
ruffはASTだけで動きパッケージ本体を必要としないので、`uvx` で直接実行している。

mypyは型スタブと組み合わせて結果が決まるため、各ライブラリの `dev` extras で管理する。

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
    secrets: inherit
    with:
      package_name: my_package
      repo_name: my-repo
```

`secrets: inherit` を指定することで、呼び出し元リポジトリ（またはOrganization）のシークレットがこのワークフローに引き継がれる。

## 引数

| 引数 | 型 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| `package_name` | string | ✅ | - | パッケージのディレクトリ名（ruff / mypy / pytest のターゲット） |
| `repo_name` | string | ✅ | - | リポジトリ名（uvキャッシュのサフィックスに使用） |
| `run_unit_test` | boolean | | `true` | 単体テストを実行するかどうか |
| `run_integration_test` | boolean | | `false` | 結合テストを実行するかどうか |
| `dependency_library` | string | | `''` | 依存ライブラリのリスト（スペース区切り、`lib@ref` 形式でrefを指定可能） |

### `dependency_library` について

`KeibaAI-developer` organization配下のリポジトリ名をスペース区切りで指定する。
`lib@ref` の形式でブランチ・タグ・コミットSHAを指定できる。`@ref` を省略した場合は `develop` を使用する。
`owner/lib` の形式にすると、`KeibaAI-developer` 以外のorganizationのリポジトリも指定できる。
プライベートリポジトリを含む場合は `GH_PAT` シークレットが必要（後述）。

**指定した順にインストールする。** 依存される側を先に並べること。
モノレポ内ライブラリはPyPIに公開していないため、依存されるライブラリが未インストールのまま先に依存する側をインストールすると、PyPIへの問い合わせで解決に失敗する。

```yaml
with:
  package_name: my_package
  repo_name: my-repo
  dependency_library: 'keiba-scraping@main mykeibadb-python@v1.2.0 Kubo-Tech/discord-logger@develop'
```

## シークレット

| シークレット | 必須 | 説明 |
|---|---|---|
| `GH_PAT` | `dependency_library` にプライベートリポジトリが含まれる場合は必須 | プライベートリポジトリへのアクセストークン。`repo` スコープを持つPersonal Access Tokenを `KeibaAI-developer` OrganizationのシークレットまたはリポジトリのシークレットとしてGitHubに登録する。 |

## 設定例

### 依存ライブラリなし・単体テストのみ

```yaml
jobs:
  ci:
    uses: KeibaAI-developer/library-workflow/.github/workflows/ci.yml@main
    secrets: inherit
    with:
      package_name: discord_logger
      repo_name: discord-logger
```

### 依存ライブラリあり

```yaml
jobs:
  ci:
    uses: KeibaAI-developer/library-workflow/.github/workflows/ci.yml@main
    secrets: inherit
    with:
      package_name: rating
      repo_name: rating
      dependency_library: 'keiba-data-interface db-client race-data feature-value-utils'
```

### 単体テスト・結合テストの両方を実行

```yaml
jobs:
  ci:
    uses: KeibaAI-developer/library-workflow/.github/workflows/ci.yml@main
    secrets: inherit
    with:
      package_name: keiba_data_interface
      repo_name: keiba-data-interface
      run_integration_test: true
      dependency_library: 'keiba-scraping mykeibadb-python'
```
