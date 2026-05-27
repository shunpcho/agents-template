# Skill: Image Classification

## Overview

画像分類タスクを実装するためのスキルです。
入力画像に対してクラスラベルを割り当てるモデルを構築します。

## Applicable Agents

- CV Data Agent（データセット構築）
- CV Training Agent（モデルトレーニング）
- CV Evaluation Agent（精度評価）

## Task Definition

- **入力**: 単一画像 (`npt.NDArray[np.uint8]` または `torch.Tensor`)
- **出力**: クラスラベルと信頼スコア (`dict[str, float]`)
- **損失関数**: `CrossEntropyLoss`（多クラス）または `BCEWithLogitsLoss`（マルチラベル）

## Recommended Architectures

| ユースケース | 推奨アーキテクチャ |
|------------|-----------------|
| 精度重視 | `EfficientNet-B4`, `ViT-B/16`, `ConvNeXt-Small` |
| 速度・軽量 | `MobileNetV3-Small`, `EfficientNet-B0` |
| 転移学習ベースライン | `ResNet-50` (ImageNet pretrained) |

`timm` ライブラリ経由でロードすることを推奨します。

## Data Format

```
dataset/
  train/
    class_a/
      img001.jpg
      ...
    class_b/
      ...
  val/
    class_a/
    class_b/
  test/
    class_a/
    class_b/
```

`torchvision.datasets.ImageFolder` が利用可能なディレクトリ構成を推奨します。

## Preprocessing Pipeline

1. リサイズ（モデルの入力サイズに合わせる、例: 224×224）
2. データ拡張（学習時のみ）→ `docs/skills/data-augmentation.md` を参照
3. テンソル変換 (`ToTensor`)
4. 正規化（ImageNet 統計: `mean=[0.485, 0.456, 0.406]`, `std=[0.229, 0.224, 0.225]`）

## Key Metrics

- **Accuracy** (Top-1 / Top-5)
- **Precision / Recall / F1** (macro / weighted)
- **AUC-ROC**（二値分類またはマルチラベル分類時）

## Implementation Checklist

- [ ] `@dataclass(slots=True)` でモデル設定を定義する
- [ ] `timm.create_model(model_name, pretrained=True, num_classes=N)` でモデルを生成する
- [ ] 最終層の出力次元を `num_classes` に合わせる
- [ ] 学習率スケジューラに `CosineAnnealingLR` を使用する
- [ ] 評価時に `torch.inference_mode()` を使用する
- [ ] `torchmetrics.Accuracy` でメトリクスを計測する
- [ ] `mlflow.log_params` でハイパーパラメータを記録する
- [ ] `mlflow.log_metrics(step=epoch)` で epoch ごとの loss / accuracy を記録する
- [ ] `mlflow.pytorch.log_model` でトレーニング済みモデルを保存する

## Common Pitfalls

- クラス不均衡がある場合は `WeightedRandomSampler` または `class_weight` を使用する。
- 転移学習では最初の数 epoch は backbone を凍結（freeze）し、その後全体を fine-tuning する。
- 入力正規化をトレーニング時と推論時で統一する。
