# Nix環境のセットアップガイド

このプロジェクトは、NixとuvのHybridアプローチを採用しています。

## 概要

### アーキテクチャ

```
┌─────────────────────────────────────────┐
│  Nix Flake (flake.nix)                  │
│  ┌─────────────────────────────────┐   │
│  │ Python Interpreter (python3.12) │   │
│  │ uv (Package Manager)            │   │
│  └─────────────────────────────────┘   │
│              ↓                          │
│  ┌─────────────────────────────────┐   │
│  │ PyPI Packages via uv            │   │
│  │  └─ pandas, numpy, etc.         │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

**Nixが提供するもの:**
- Pythonインタープリタ（特定バージョン固定）
- uv（パッケージマネージャー）
- Node.js + npm（AI CLIツール用）
- Git、curl等の開発ツール

**npmが提供するもの:**
- Claude Code CLI（@anthropic-ai/claude-code）
- Gemini CLI（@google/gemini-cli）

**uvが提供するもの:**
- Pythonパッケージ（PyPI経由）

## 前提条件

### 1. Nixのインストール

```bash
# Nix Flakes対応版のインストール
sh <(curl -L https://nixos.org/nix/install) --daemon

# Flakesを有効化
mkdir -p ~/.config/nix
echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf
```

### 2. direnvのインストール（推奨）

```bash
# macOS (Homebrew)
brew install direnv

# Ubuntu/Debian
sudo apt install direnv

# Arch Linux
sudo pacman -S direnv

# シェル統合の設定
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc  # bash
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc   # zsh

# 設定を反映
source ~/.bashrc  # or source ~/.zshrc
```

## プロジェクトのセットアップ

### 方法1: direnvを使う（推奨）

```bash
cd /path/to/your-project

# 初回のみ許可が必要
direnv allow

# 自動的にNix環境が構築される
# Python + uv Development Environment
# ...
```

ディレクトリに入ると自動的に環境がロードされ、出ると自動的にアンロードされます。

### 方法2: 手動でNix環境に入る

```bash
cd /path/to/your-project

# Nix開発環境に入る
nix develop

# Python + uv Development Environment
# ...
```

## uvのバージョン管理

### nixpkgs版（デフォルト）

デフォルトではnixpkgsからuvをインストールします。安定版で、ビルド時間がかかりません。

```nix
uv = uv-from-nixpkgs;
```

### cargo版（最新版）

最新版のuvが必要な場合は、`flake.nix`を以下のように変更してください：

```nix
# flake.nix の該当部分を変更
uv = uv-from-cargo;
```

**注意点:**
- 初回ビルドに10-20分程度かかる場合があります
- バージョンとハッシュ値を手動で更新する必要があります

#### ハッシュ値の更新方法

1. まず仮のハッシュ値で試す:

```nix
uv-from-cargo = pkgs.rustPlatform.buildRustPackage rec {
  pname = "uv";
  version = "0.5.1";

  src = pkgs.fetchFromGitHub {
    owner = "astral-sh";
    repo = "uv";
    rev = version;
    hash = ""; # 空にする
  };

  cargoHash = ""; # 空にする
  # ...
};
```

2. ビルドを試す:

```bash
nix build .#
```

3. エラーメッセージから正しいハッシュ値をコピーして`flake.nix`に貼り付ける

## 開発ワークフロー

```bash
# プロジェクトに移動（direnvが自動起動）
cd your-project

# Python依存関係のインストール
uv sync

# パッケージの追加
uv add pandas numpy

# スクリプト実行
uv run python script.py

# テスト実行
uv run pytest

# AI CLIツールの使用
claude  # Claude Code CLI
gemini  # Gemini CLI

# ディレクトリから出ると自動的に環境がアンロード
cd ..
```

## 環境変数

### UV_PYTHON

```bash
UV_PYTHON = "${python}/bin/python"
```

uvに対してNixが提供するPythonインタープリタを使用するよう指示します。

### UV_PYTHON_DOWNLOADS

```bash
UV_PYTHON_DOWNLOADS = "never"
```

uvが勝手にPythonをダウンロードするのを防ぎます。常にNix提供のPythonを使用します。

## トラブルシューティング

### direnvが動作しない

```bash
# direnvが正しくインストールされているか確認
which direnv

# シェル統合が設定されているか確認
echo $DIRENV_DIR  # 空でなければOK

# 設定を再読み込み
direnv allow
```

### Nix Flakesが有効でない

```bash
# Flakes設定を確認
cat ~/.config/nix/nix.conf

# 含まれていない場合は追加
echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf

# Nixデーモンを再起動
sudo systemctl restart nix-daemon  # Linux
# macOSの場合は再ログインが必要
```

### uvのバージョンが古い

nixpkgs版のuvが古い場合は、cargo版に切り替えてください（上記参照）。

または、nixpkgsのバージョンを`nixos-unstable`から最新のコミットに変更します：

```nix
inputs = {
  nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  # または特定のコミット
  # nixpkgs.url = "github:NixOS/nixpkgs/[コミットハッシュ]";
};
```

## 参考資料

- [Nix Flakes](https://nixos.wiki/wiki/Flakes)
- [direnv](https://direnv.net/)
- [nix-direnv](https://github.com/nix-community/nix-direnv)
- [uv](https://docs.astral.sh/uv/)
- [元記事: Nix + uv での Python 開発環境構築](https://zenn.dev/shundeveloper/articles/36307d821d40f7)
