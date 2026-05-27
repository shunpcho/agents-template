# Skill: Object Detection

## Overview

物体検出タスクを実装するためのスキルです。
入力画像内に存在する物体のバウンディングボックスとクラスラベルを検出します。

## Applicable Agents

- CV Data Agent（アノテーション付きデータセット構築）
- CV Training Agent（検出モデルのトレーニング）
- CV Evaluation Agent（mAP 算出・可視化）

## Task Definition

- **入力**: 単一画像 (`npt.NDArray[np.uint8]` または `torch.Tensor`)
- **出力**: バウンディングボックス座標、クラスラベル、信頼スコアのリスト
- **損失関数**: アーキテクチャ依存（例: YOLO loss, Focal loss + L1/GIoU）

## Recommended Architectures

| ユースケース | 推奨アーキテクチャ |
|------------|-----------------|
| 精度重視 | `DINO`, `DETR`, `Faster R-CNN (ResNet-50 FPN)` |
| リアルタイム推論 | `YOLOv8`, `RT-DETR` |
| 軽量・エッジ向け | `YOLOv8n`, `MobileNet-SSD` |

`torchvision.models.detection` または `ultralytics` ライブラリを推奨します。

## Data Format

COCO JSON 形式を推奨します。

```json
{
  "images": [{"id": 1, "file_name": "img001.jpg", "width": 640, "height": 480}],
  "annotations": [
    {"id": 1, "image_id": 1, "category_id": 1,
     "bbox": [x, y, width, height], "area": 0.0, "iscrowd": 0}
  ],
  "categories": [{"id": 1, "name": "cat"}]
}
```

バウンディングボックスは `[x_min, y_min, width, height]`（COCO 形式）または
`[x_min, y_min, x_max, y_max]`（Pascal VOC 形式）のいずれかに統一する。

## Preprocessing Pipeline

1. リサイズ（アスペクト比を保持、例: 最長辺 640px）
2. データ拡張（学習時のみ）→ `docs/skills/data-augmentation.md` を参照
   - バウンディングボックスと連動した拡張が必須（`albumentations` の `BboxParams` を使用）
3. テンソル変換 (`ToTensor`)
4. 正規化

## Key Metrics

- **mAP** (mean Average Precision) @ IoU=0.50 および IoU=0.50:0.95
- **IoU** (Intersection over Union)
- **Precision / Recall** per class

`torchmetrics.detection.MeanAveragePrecision` を使用して算出する。

## Implementation Checklist

- [ ] アノテーションを COCO JSON 形式に統一する
- [ ] `albumentations` の `BboxParams` を設定してバウンディングボックスの変換を保証する
- [ ] `torchvision.ops.nms` または `batched_nms` で重複ボックスを除去する
- [ ] `torchmetrics.detection.MeanAveragePrecision` でmAPを算出する
- [ ] 信頼スコアの閾値をハイパーパラメータとして外部設定できるようにする

## Common Pitfalls

- バウンディングボックス座標系（COCO vs VOC）の混在に注意する。
- 拡張適用時にボックス座標も同様に変換されているかを確認する。
- 小物体の検出精度向上には FPN（Feature Pyramid Network）の利用を検討する。
- NMS の IoU 閾値はタスクや物体の密度に応じて調整する。
