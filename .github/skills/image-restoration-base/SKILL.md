---
name: image-restoration-base
description: >
  Image Restoration プロジェクトの共通基盤を構築する。
  uv によるプロジェクト管理・PyTorch 環境のセットアップ・プロジェクト共通構造の設定を行う。
  タスク固有スキル（denoising / deblurring / super-resolution）を使用する前にこのスキルで基盤を整える。
---

# Skill: Image Restoration Base

## Overview

Image Restoration プロジェクト全体の共通基盤スキルです。
`uv` によるパッケージ管理、PyTorch 環境のセットアップ、プロジェクト共通構造・規約の定義を担います。

## Applicable Agents

- CV Data Agent
- CV Training Agent
- CV Evaluation Agent

## Project Setup

### uv によるプロジェクト初期化

```bash
uv init --package <project_name>
cd <project_name>

# 本番依存パッケージ
uv add torch torchvision torchaudio \
    torchmetrics[image] \
    mlflow \
    albumentations \
    Pillow numpy scipy imageio \
    timm lpips pytorch-msssim

# 開発・テスト・lint パッケージ
uv add --group dev ipython pyright ruff
uv add --group test pytest pytest-cov pytest-mock
uv add --group lint ruff pyright flake8 codespell
```

### pyproject.toml

`templates/pyproject.toml` をベースに以下の依存関係を追加する。
`ruff` / `pyright` / `build-system` の設定は `templates/pyproject.toml` の内容をそのまま使用する。

```toml
[project]
name = "project_name"
version = "0.1.0"
description = "Image Restoration project"
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "torch>=2.3",
    "torchvision>=0.18",
    "torchaudio>=2.3",
    "torchmetrics[image]>=1.4",
    "mlflow>=2.14",
    "albumentations>=1.4",
    "Pillow>=10.4",
    "numpy>=1.26",
    "scipy>=1.14",
    "imageio>=2.35",
    "timm>=1.0",
    "lpips>=0.1.4",
    "pytorch-msssim>=1.0",
]

[project.scripts]
train = "project_name.train:main"
```

## Key Libraries

| ライブラリ | 用途 |
|-----------|------|
| `torch`, `torchvision` | モデル実装・画像変換の基盤 |
| `torch.amp` | 混合精度学習 |
| `torchmetrics` | PSNR / SSIM 計算 |
| `mlflow` | 実験トラッキング（パラメータ・メトリクス・モデル保存） |
| `albumentations` | データ拡張（degraded / clean ペア整合） |
| `timm` | 事前学習済み backbone |
| `lpips` | 知覚的品質評価（LPIPS） |
| `pytorch-msssim` | SSIM Loss |
| `Pillow`, `imageio` | 画像 I/O（HDR・16bit 画像対応） |

## PyTorch 基本設定

```python
import torch

# デバイス設定
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 再現性確保
import numpy as np

torch.manual_seed(seed)
np.random.seed(seed)
```

推論時は必ず `torch.inference_mode()` コンテキストを使用する。

## Project Structure

`docs/code_structure.md` に定義された構造に従う。

```
src/{project_name}/
  configs/
    train.yaml          # トレーニング設定（ハイパーパラメータ・実験設定）
  data/
    __init__.py
    dataset.py          # RestorationDataset（degraded/clean ペア管理）
    degradation.py      # 合成劣化パイプライン
    transforms.py       # データ拡張（albumentations ベース）
    dataloader.py       # DataLoader ファクトリ
    stats.py            # データセット統計量算出
  models/
    __init__.py
    network.py          # モデルアーキテクチャ定義
  utils/
    __init__.py
    trainer.py          # Trainer クラス（学習ループ）
    loss.py             # 損失関数（Charbonnier, Perceptual, SSIM 等）
    evaluator.py        # Evaluator クラス（推論 + メトリクス算出）
    metrics.py          # PSNR / SSIM / LPIPS 計算ユーティリティ
    visualizer.py       # 可視化（degraded / restored / GT）
    logger.py           # MLflow ロガー
  train.py              # 学習エントリーポイント
  inference.py          # 推論・評価エントリーポイント
tests/
```

## Common Conventions

プロジェクト全体のコーディング規約は `.github/copilot-instructions.md` に定義されています。Image Restoration 固有の補足事項を以下に示します。

- 設定クラスは `@dataclass(slots=True)` で定義し、YAML ファイルから読み込む。
- ファイルパスは常に `pathlib.Path` を使用する。
- テンソルの値域は `[0, 1]` に統一する。
- 学習ループはエポックではなく **イテレーション（ステップ）** で管理する。
- バッチ正規化は Restoration タスクで不安定になることがあるため、Layer Normalization を優先する。

## Development Workflow

```bash
# 依存パッケージのインストール
uv sync --all-groups

# コードフォーマット
uv run ruff format .

# Lint チェック
uv run ruff check .
uv run pyright

# テスト実行
uv run pytest

# トレーニング実行
uv run train

# 推論・評価実行
uv run python -m project_name.inference
```

## Related Skills

- [data-augmentation](../data-augmentation/SKILL.md)
- [image-denoising](../image-denoising/SKILL.md)
- [image-deblurring](../image-deblurring/SKILL.md)
- [super-resolution](../super-resolution/SKILL.md)
