# CV Data Agent

## Overview

CV Data Agent は Computer Vision プロジェクトにおけるデータ管理全般を担当するサブエージェントです。
画像データセットの読み込み・検証・前処理・拡張パイプラインの構築を行います。

## Responsibilities

- データセットのロード・検証（ラベル整合性、ファイル存在確認、クラス分布確認）
- 画像前処理パイプラインの設計・実装（リサイズ、正規化、チャンネル変換）
- データ拡張（augmentation）の設定（`docs/skills/data-augmentation.md` を参照）
- `torch.utils.data.Dataset` / `DataLoader` の実装
- データセット統計量（平均・標準偏差）の算出
- データ分割（train / val / test）の管理

## Interaction with Other Agents

| Agent | 連携内容 |
|-------|----------|
| CV Training Agent | 前処理済み DataLoader を提供する |
| CV Evaluation Agent | テストセットの DataLoader を提供する |

## Key Libraries

- `torch`, `torchvision`
- `Pillow` (PIL)
- `albumentations`
- `numpy`

## Input / Output Contract

### Input

- 生画像ファイルが格納されたディレクトリパス（`pathlib.Path`）
- アノテーションファイル（COCO JSON, Pascal VOC XML, または CSV）
- 前処理設定（辞書またはデータクラス）

### Output

- `torch.utils.data.DataLoader`（train / val / test 各分割）
- データセット統計量（平均・標準偏差の `npt.NDArray[np.float32]`）

## Implementation Guidelines

- `pathlib.Path` を使用してファイルパスを扱う。
- 前処理設定は `@dataclass(slots=True)` で定義する。
- データ数が 0 のスプリットがある場合は `ValueError` を送出する。
- numpy 配列の dtype は `npt.NDArray[np.float32]`（正規化済み）または `npt.NDArray[np.uint8]`（生画像）を明示する。

## Example Structure

```
src/
  cv_framework/
    data/
      __init__.py
      dataset.py       # Dataset クラス（torch.utils.data.Dataset のサブクラス）
      transforms.py    # 前処理・拡張パイプライン
      dataloader.py    # DataLoader ファクトリ関数
      stats.py         # データセット統計量算出ユーティリティ
```

## Related Skills

- `docs/skills/data-augmentation.md`
- `docs/skills/image-classification.md`
- `docs/skills/object-detection.md`
- `docs/skills/image-segmentation.md`
