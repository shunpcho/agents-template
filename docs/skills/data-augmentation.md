# Skill: Data Augmentation

## Overview

Computer Vision モデルの汎化性能を高めるためのデータ拡張スキルです。
`albumentations` ライブラリを主軸に、タスクに応じた拡張パイプラインを構築します。

## Applicable Agents

- CV Data Agent（拡張パイプラインの組み込み）

## Library Choice

`albumentations` を第一選択とします。

- 画像分類・物体検出・セグメンテーションの全タスクに対応
- `torchvision.transforms` より高速かつ豊富な変換を提供
- `BboxParams` / `KeypointParams` でアノテーションと連動した変換が可能

## Augmentation Categories

### 空間変換（Spatial Transforms）

ラベル（バウンディングボックス・マスク）と連動して変換されます。

| 変換 | `albumentations` クラス |
|------|------------------------|
| ランダムクロップ | `RandomResizedCrop` |
| 水平反転 | `HorizontalFlip` |
| 垂直反転 | `VerticalFlip` |
| 回転 | `Rotate`, `ShiftScaleRotate` |
| アフィン変換 | `Affine` |
| エラスティック変換 | `ElasticTransform` |

### ピクセル変換（Pixel Transforms）

ラベルに影響を与えない変換です。

| 変換 | `albumentations` クラス |
|------|------------------------|
| 明度・コントラスト調整 | `RandomBrightnessContrast` |
| 色相・彩度調整 | `HueSaturationValue` |
| ガウシアンノイズ付加 | `GaussNoise` |
| ガウシアンブラー | `GaussianBlur` |
| グレースケール変換 | `ToGray` |
| カットアウト / CoarseDropout | `CoarseDropout` |

### 高度な拡張

| 手法 | 説明 | 実装ライブラリ |
|------|------|----------------|
| MixUp | 2枚の画像とラベルを線形補間 | カスタム実装または `torchvision` |
| CutMix | 画像の一部を別画像で置換 | カスタム実装または `timm` |
| RandAugment | ランダムに拡張を選択・適用 | `albumentations.auto_augment.RandAugment` |
| AugMix | 複数の拡張を組み合わせ混合 | `albumentations.augmentations.domain_adaptation` |

## Task-Specific Pipeline Examples

### 画像分類

```python
import albumentations as albu
from albumentations.pytorch import ToTensorV2

train_transform = albu.Compose([
    albu.RandomResizedCrop(height=224, width=224, scale=(0.08, 1.0)),
    albu.HorizontalFlip(p=0.5),
    albu.RandomBrightnessContrast(p=0.2),
    albu.HueSaturationValue(p=0.2),
    albu.Normalize(mean=(0.485, 0.456, 0.406), std=(0.229, 0.224, 0.225)),
    ToTensorV2(),
])
```

### 物体検出

```python
train_transform = albu.Compose([
    albu.RandomResizedCrop(height=640, width=640),
    albu.HorizontalFlip(p=0.5),
    albu.RandomBrightnessContrast(p=0.2),
    albu.Normalize(mean=(0.485, 0.456, 0.406), std=(0.229, 0.224, 0.225)),
    ToTensorV2(),
], bbox_params=albu.BboxParams(format="coco", label_fields=["class_labels"]))
```

### セグメンテーション

```python
train_transform = albu.Compose([
    albu.RandomResizedCrop(height=512, width=512),
    albu.HorizontalFlip(p=0.5),
    albu.RandomBrightnessContrast(p=0.2),
    albu.Normalize(mean=(0.485, 0.456, 0.406), std=(0.229, 0.224, 0.225)),
    ToTensorV2(),
])
# マスクには interpolation=cv2.INTER_NEAREST が自動適用される
```

## Implementation Guidelines

- 拡張パイプラインはトレーニング時のみ適用する。検証・テスト時はリサイズと正規化のみ。
- 拡張の確率（`p`）と強度はハイパーパラメータとして外部設定できるようにする。
- 再現性のために `albumentations` の設定を JSON / YAML にシリアライズ可能にする（`A.to_dict()` / `A.from_dict()`）。
- 強すぎる拡張はデータの意味を壊す場合があるため、`p` 値は小さめ（0.2〜0.5）から始める。

## Common Pitfalls

- セグメンテーションマスクの補間は必ず **最近傍補間（nearest）** にする。
- 物体検出では空間変換後にボックスが画像外に出る場合があるため、`min_visibility` を設定して除外する。
- 正規化は `albumentations.Normalize` でパイプライン内で行い、手動計算と混在させない。
