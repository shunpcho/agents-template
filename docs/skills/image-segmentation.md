# Skill: Image Segmentation

## Overview

画像セグメンテーションタスクを実装するためのスキルです。
意味的セグメンテーション（semantic）およびインスタンスセグメンテーション（instance）の両方をカバーします。

## Applicable Agents

- CV Data Agent（マスク付きデータセット構築）
- CV Training Agent（セグメンテーションモデルのトレーニング）
- CV Evaluation Agent（mIoU 算出・可視化）

## Task Definition

| タイプ | 出力 | 代表的なユースケース |
|--------|------|---------------------|
| Semantic Segmentation | ピクセル単位のクラスマスク | 自動運転、医療画像解析 |
| Instance Segmentation | 物体ごとの個別マスク | 群衆解析、製造ライン検査 |
| Panoptic Segmentation | Semantic + Instance の統合 | シーン理解 |

- **入力**: 単一画像 (`npt.NDArray[np.uint8]` または `torch.Tensor`)
- **出力**: ピクセル単位のセグメンテーションマスク (`npt.NDArray[np.int64]`)
- **損失関数**: `CrossEntropyLoss`（意味的）、`DiceLoss` + `CrossEntropyLoss`（医療画像等）

## Recommended Architectures

| ユースケース | 推奨アーキテクチャ |
|------------|-----------------|
| Semantic 精度重視 | `SegFormer-B4`, `DeepLabV3+ (ResNet-101)` |
| Semantic 軽量 | `SegFormer-B0`, `DeepLabV3+ (MobileNetV3)` |
| Instance | `Mask R-CNN`, `SOLOv2` |
| 医療画像 | `U-Net`, `U-Net++`, `Swin-UNet` |

`mmsegmentation` または `segmentation-models-pytorch` を推奨します。

## Data Format

マスク画像はクラス ID を値として持つグレースケール PNG 形式（`np.int64`）を推奨します。

```
dataset/
  images/
    img001.jpg
  masks/
    img001.png  # ピクセル値 = クラスID (0, 1, 2, ...)
```

COCO JSON 形式（ポリゴンアノテーション）でも対応可能。`pycocotools` でマスクに変換する。

## Preprocessing Pipeline

1. リサイズ（画像とマスクを同一サイズに揃える）
2. データ拡張（学習時のみ）→ `docs/skills/data-augmentation.md` を参照
   - マスクと画像に対して**同一の空間変換**を適用する（`albumentations` の `mask` ターゲットを使用）
3. 画像のテンソル変換・正規化（マスクは整数型を維持）

## Key Metrics

- **mIoU** (mean Intersection over Union)
- **Dice Coefficient** / **F1**（医療画像など二値セグメンテーション）
- **Pixel Accuracy**

`torchmetrics.JaccardIndex` または `torchmetrics.Dice` を使用して算出する。

## Implementation Checklist

- [ ] マスク画像の dtype を `np.int64` / `torch.long` に統一する
- [ ] `albumentations` でマスクと画像に**同一の空間変換**を適用する
- [ ] クラス不均衡時は `weight` 引数付き `CrossEntropyLoss` またはFocal Lossを使用する
- [ ] `torchmetrics.JaccardIndex` で mIoU を算出する
- [ ] 予測マスクを元の画像解像度にアップサンプリングして評価する
- [ ] `mlflow.log_params` でハイパーパラメータを記録する
- [ ] `mlflow.log_metrics(step=epoch)` で epoch ごとの loss / mIoU を記録する
- [ ] `mlflow.pytorch.log_model` でトレーニング済みモデルを保存する

## Common Pitfalls

- マスク画像の補間には **最近傍補間（nearest）** を使用する（線形補間は不可）。
- 画像には bilinear 補間を使用してよい。
- void/ignore クラス（ラベル 255 など）を損失計算から除外する設定を忘れない。
- バッチ正規化はバッチサイズが小さい場合に不安定になるため、Group Normalization への置換を検討する。
