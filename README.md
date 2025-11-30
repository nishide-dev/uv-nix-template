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
| `use_nix` | Nix + direnv環境管理を使うか | `true` | - |

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
├── docs/
│   ├── NIX_SETUP.md          # Nixセットアップガイド
│   └── AI_CLI_TOOLS.md       # AI CLIツールガイド
├── pyproject.toml            # プロジェクト設定（PEP 621）
├── uv.lock                   # 依存関係のロックファイル
├── .gitignore
├── .python-version           # Pythonバージョン指定
├── LICENSE
└── README.md
```

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

## 🎯 拡張テンプレート

このベーステンプレートに加えて、以下の拡張テンプレートも利用可能です：

- **[uv-cuda-nix-template](https://github.com/nishide-dev/uv-cuda-nix-template)**: CUDA/GPU環境とNixによるシステムレベル再現性を追加

拡張テンプレートの適用方法：

```bash
# ベーステンプレートでプロジェクト生成
uvx copier copy --trust gh:nishide-dev/uv-nix-template my-gpu-project
cd my-gpu-project

# CUDA/Nix拡張を追加適用
uvx copier copy --trust gh:nishide-dev/uv-cuda-nix-template .
```

## 📄 ライセンス

MIT License

## 🤝 コントリビューション

テンプレートに関するフィードバックは、以下のIssueテンプレートをご利用ください：
- バグレポート
- 機能リクエスト
- テンプレート改善提案
- ドキュメント改善
