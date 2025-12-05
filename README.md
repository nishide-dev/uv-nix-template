# uv-nix-template

**モダンで再現可能なPythonライブラリプロジェクト構築基盤**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![uv](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/uv/main/assets/badge/v0.json)](https://github.com/astral-sh/uv)

[Copier](https://copier.readthedocs.io/)ベースのテンプレートで、**uv**（高速Pythonパッケージマネージャー）と**Nix**を組み合わせた、モダンで保守性の高いPythonライブラリプロジェクトを生成します。

## ✨ 特徴

### 🚀 高速な開発体験

- **uvによる圧倒的な速度**: pip/poetryと比較して10-100倍高速な依存関係解決
- **Copy-on-Write**: グローバルキャッシュによりディスク使用量を最小化
- **決定論的ビルド**: `uv.lock`による厳密な依存関係の固定

### 🔒 再現性（Nix）

- **Nixによる環境管理**: Pythonバージョンとuvを固定
- **direnv統合**: ディレクトリに入ると自動で環境アクティベート
- **システムレベルの再現性**: チーム全体で同一の開発環境
- **uvバージョン選択**: nixpkgs版 or cargo最新版

### 🤖 AI開発支援

- **Node.js統合**: Claude Code CLI、Gemini CLI自動インストール
- **即座に利用可能**: Nix環境構築時に自動セットアップ

## 🔥 PyTorch/CUDAが必要な場合

**GPUを使った機械学習開発には、派生テンプレートがあります：**

👉 **[uv-torch-nix-template](https://github.com/nishide-dev/uv-torch-nix-template)** - PyTorch/CUDA特化版

- PyTorch + CUDA環境をプリセットで簡単設定（16種類の組み合わせ）
- CUDAバージョン管理（13.0/12.8/12.6/12.4/12.1/11.8）
- Ubuntu/Debian等でのCUDA有効化ガイド付き
- このテンプレートのすべての機能を継承

## 🚀 クイックスタート

### 前提条件

#### Option 1: Nix + uv（推奨）

Nixを使用すると、uvとPythonの両方が自動管理されます。

```bash
# Nixのインストール
sh <(curl -L https://nixos.org/nix/install) --daemon

# Flakesを有効化
mkdir -p ~/.config/nix
echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf

# direnvのインストール
# macOS
brew install direnv
# Ubuntu/Debian
sudo apt install direnv

# シェル統合
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc  # or zsh
```

#### Option 2: uvのみ

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### プロジェクト生成

#### 方法1: uvを使う（推奨）

```bash
# 新しいプロジェクトを生成
uvx copier copy --trust gh:nishide-dev/uv-nix-template my-awesome-project

# プロジェクトに移動
cd my-awesome-project

# 開発を開始
uv sync
```

**ヒント:** git設定から作者情報を自動設定：
```bash
uvx copier copy --trust gh:nishide-dev/uv-nix-template my-project \
  --data author_name="$(git config user.name)" \
  --data author_email="$(git config user.email)"
```

#### 方法2: ローカルテンプレートから

```bash
# テンプレートをクローン
git clone https://github.com/nishide-dev/uv-nix-template.git

# ローカルパスから生成
uvx copier copy --trust ./uv-nix-template my-awesome-project
```

## 📝 生成時の質問

Copierは以下の質問をします：

| 質問 | 説明 | デフォルト | 例 |
|-----|------|-----------|-----|
| `project_name` | プロジェクト名（PyPI登録名） | - | `my-awesome-lib` |
| `package_name` | パッケージ名（Pythonインポート名） | `project_name`のスネークケース | `my_awesome_lib` |
| `description` | プロジェクトの短い説明 | - | `A modern Python library` |
| `author_name` | 作者名 | - | `Your Name` |
| `author_email` | 作者のメールアドレス | - | `your.email@example.com` |
| `python_version` | 最小Pythonバージョン | `3.12` | `3.10`, `3.11`, `3.12`, `3.13` |
| `use_ruff` | Ruff（linter/formatter）を使うか | `true` | - |
| `use_ty` | ty（型チェッカー）を使うか | `true` | - |
| `use_pytest` | Pytest（テストフレームワーク）を使うか | `true` | - |
| `use_github_actions` | GitHub Actions CI/CDを設定するか | `true` | - |
| `include_docker` | Docker環境（開発用・本番用）を含めるか | `false` | - |
| `use_nix` | Nix + direnv環境管理を使うか | `true` | - |
| `setup_direnv_hook` | シェル設定ファイルにdirenv hookを自動追加するか（`use_nix=true`の場合のみ） | `false` | - |

## 📂 生成されるプロジェクト構造

```
my-awesome-project/
├── src/
│   └── my_awesome_lib/
│       ├── __init__.py
│       └── py.typed          # 型情報の公開マーカー
├── tests/
│   ├── __init__.py
│   └── test_my_awesome_lib.py
├── .github/
│   └── workflows/
│       └── test.yml          # CI/CDワークフロー
├── flake.nix                 # Nix環境定義（use_nix=trueの場合）
├── .envrc                    # direnv設定（use_nix=trueの場合）
├── Dockerfile.dev            # 開発用Dockerイメージ（include_docker=trueの場合）
├── Dockerfile.prod           # 本番用Dockerイメージ（include_docker=trueの場合）
├── docker-compose.yml        # Docker Compose設定（include_docker=trueの場合）
├── .dockerignore             # Docker除外設定（include_docker=trueの場合）
├── docs/
│   ├── NIX_SETUP.md          # Nixセットアップガイド
│   ├── AI_CLI_TOOLS.md       # AI CLIツールガイド
│   └── DOCKER.md             # Docker環境ガイド（include_docker=trueの場合）
├── pyproject.toml            # プロジェクト設定（PEP 621）
├── uv.lock                   # 依存関係のロックファイル
├── .gitignore
├── .python-version           # Pythonバージョン指定
├── LICENSE
└── README.md
```

## 🐳 Docker環境

デプロイやCI/CD、Nix非対応環境向けに、Docker環境を含めることができます（`include_docker=true`）。

### 2種類のDockerfile

- **Dockerfile.dev**: 開発用（ホットリロード、SSH対応、全ツール含む）
- **Dockerfile.prod**: 本番用（マルチステージビルド、最小依存関係、セキュリティ強化）

### クイックスタート

```bash
# 開発環境で起動
docker compose up -d dev

# インタラクティブシェル
docker compose run --rm dev bash

# テスト実行
docker compose run --rm dev pytest

# 本番イメージのビルド
docker compose build prod

# 本番環境で起動
docker compose up -d prod
```

### Nix環境との使い分け

| 用途 | 推奨環境 |
|------|---------|
| ローカル開発 | **Nix + direnv** |
| デプロイ | **Docker** |
| CI/CD | **Docker** または Nix |
| チーム共有 | **Docker** |

**重要**: `uv.lock` が両環境の依存関係を保証するため、Nix環境とDocker環境で全く同じバージョンがインストールされます。

詳細は [`docs/DOCKER.md`](docs/DOCKER.md) を参照してください。

## 💡 開発ワークフロー例

```bash
# プロジェクト生成
uvx copier copy --trust gh:nishide-dev/uv-nix-template ml-experiment
cd ml-experiment

# 依存関係の追加
uv add pandas numpy scikit-learn

# 開発依存関係の追加
uv add --dev ipython jupyter

# テストの実行
uv run pytest

# コードフォーマット
uv run ruff format .

# Lint
uv run ruff check .

# 型チェック
uv run ty check

# パッケージのビルド
uv build

# PyPIへの公開
uv publish
```

## 🔄 テンプレートの更新

既存のプロジェクトに対して、テンプレートの最新版を適用できます：

```bash
cd my-awesome-project

# テンプレートの更新を適用
uvx copier update

# コンフリクトが発生した場合は手動でマージ
git diff
```

## 🎯 派生テンプレート

このテンプレートをベースにした、特化版テンプレートも利用可能です：

### PyTorch/CUDA特化版

👉 **[uv-torch-nix-template](https://github.com/nishide-dev/uv-torch-nix-template)**

GPU機械学習開発向けに最適化されたテンプレート：
- PyTorch + CUDA環境をプリセットで簡単設定（16種類）
- CUDAバージョン管理（13.0/12.8/12.6/12.4/12.1/11.8）
- Ubuntu/Debian等でのCUDA有効化ガイド
- このテンプレートのすべての機能を継承

**使い方**：

```bash
# PyTorch/CUDA環境込みで1コマンドで生成
uvx copier copy --trust gh:nishide-dev/uv-torch-nix-template my-torch-project
```

## 📄 ライセンス

MIT License

## 🤝 コントリビューション

テンプレートに関するフィードバックは、以下のIssueテンプレートをご利用ください：
- バグレポート
- 機能リクエスト
- テンプレート改善提案
- ドキュメント改善
