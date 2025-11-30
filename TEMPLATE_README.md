# uv-nix-template

**モダンで再現可能なPythonライブラリプロジェクト構築基盤**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![uv](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/uv/main/assets/badge/v0.json)](https://github.com/astral-sh/uv)

このテンプレートは、**uv**（高速Pythonパッケージマネージャー）を使用した、モダンで保守性の高いPythonライブラリプロジェクトの基盤を提供します。

## 特徴

### 🚀 高速な開発体験

- **uvによる圧倒的な速度**: pip/poetryと比較して10-100倍高速な依存関係解決
- **Copy-on-Write**: グローバルキャッシュによりディスク使用量を最小化
- **決定論的ビルド**: `uv.lock`による厳密な依存関係の固定

### 🛠️ モダンな開発ツール統合

- **Ruff**: 極めて高速なlinter/formatter（Rust製）
- **ty**: 超高速型チェッカー（Rust製、Astral社製）
- **Pytest**: テストフレームワーク + カバレッジ
- **GitHub Actions**: 自動CI/CD

### 📦 ベストプラクティスの実装

- **Src Layout**: インポートパスの競合を防ぐ推奨構造
- **PEP 621準拠**: pyproject.tomlによる標準的なメタデータ管理
- **PEP 735準拠**: dependency-groupsによる依存関係グループ管理
- **型情報の公開**: py.typedマーカーによる型ヒント配布

### 🔒 完全な再現性（Nix）

- **Nixによる環境管理**: Pythonバージョンとuvを固定
- **direnv統合**: ディレクトリに入ると自動で環境アクティベート
- **システムレベルの再現性**: チーム全体で同一の開発環境
- **uvバージョン選択**: nixpkgs版 or cargo最新版

## テンプレートの使用方法

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

### 生成時の質問

Copierは以下の質問をします：

| 質問 | 説明 | 例 |
|-----|------|-----|
| `project_name` | プロジェクト名（PyPI登録名） | `my-awesome-lib` |
| `package_name` | パッケージ名（Pythonインポート名） | `my_awesome_lib` |
| `description` | プロジェクトの短い説明 | `A modern Python library` |
| `author_name` | 作者名 | `Your Name` |
| `author_email` | 作者のメールアドレス | `your.email@example.com` |
| `python_version` | 最小Pythonバージョン | `3.12` |
| `use_ruff` | Ruffを使うか | `true` |
| `use_ty` | ty（型チェッカー）を使うか | `true` |
| `use_pytest` | Pytestを使うか | `true` |
| `use_github_actions` | GitHub Actionsを設定するか | `true` |
| `use_nix` | Nix + direnv環境管理を使うか | `true` |

## プロジェクト構造

生成されるプロジェクトは以下の構造を持ちます：

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

## 開発ワークフロー例

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

# 型チェック
uv run ty check

# パッケージのビルド
uv build

# PyPIへの公開
uv publish
```

## テンプレートの更新

既存のプロジェクトに対して、テンプレートの最新版を適用できます：

```bash
cd my-awesome-project

# テンプレートの更新を適用
uvx copier update

# コンフリクトが発生した場合は手動でマージ
git diff
```

これにより、CI設定の改善や開発ツールの最新設定を既存プロジェクトに反映できます。

## Copierについて

本テンプレートは[Copier](https://copier.readthedocs.io/)を使用しています。Copierは以下の特徴を持つテンプレートエンジンです：

- **更新機能**: 既存プロジェクトへのテンプレート変更の適用
- **回答の記憶**: `.copier-answers.yml`による設定の永続化
- **マージサポート**: テンプレート更新時の差分マージ

### GitHubテンプレートとの違い

| 機能 | GitHub Template | Copier |
|-----|-----------------|--------|
| 初回生成 | ✅ | ✅ |
| 変数の置換 | ❌ | ✅ |
| テンプレート更新の追従 | ❌ | ✅ |
| 複数テンプレートの合成 | ❌ | ✅ |

## 拡張テンプレート

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

## 設計思想

このテンプレートは、以下の原則に基づいて設計されています：

1. **Src Layout**: インポートパスの競合を防ぎ、配布時のバグを削減
2. **PEP準拠**: PEP 621 (pyproject.toml), PEP 735 (dependency-groups)
3. **ツールの最小化**: uv一つでvirtualenv/pip/twineの役割を担う
4. **型安全性**: py.typedによる型情報の配布

## 関連プロジェクト

- **uv-cuda-nix-template**: CUDA/Nix環境を追加する拡張テンプレート
- **uv**: https://github.com/astral-sh/uv
- **Copier**: https://copier.readthedocs.io/

## ライセンス

MIT License

## コントリビューション

Issue/PRは大歓迎です！

## 参考資料

- [uvドキュメント](https://docs.astral.sh/uv/)
- [Python Packaging User Guide](https://packaging.python.org/)
- [PEP 621 – Storing project metadata in pyproject.toml](https://peps.python.org/pep-0621/)
- [PEP 735 – Dependency Groups in pyproject.toml](https://peps.python.org/pep-0735/)
