# CV Training Agent

## Overview

CV Training Agent は Computer Vision モデルのトレーニングサイクル全体を担当するサブエージェントです。
モデルアーキテクチャの選択・設定から、学習ループの実装・チェックポイント管理・学習曲線の記録までを行います。

## Responsibilities

- モデルアーキテクチャの選択・インスタンス化（backbone の選定を含む）
- 損失関数・最適化アルゴリズム・学習率スケジューラの設定
- 学習ループの実装（forward / backward pass、勾配クリッピング）
- 混合精度学習（`torch.amp`）の適用
- チェックポイントの保存・復元
- TensorBoard / WandB などへの学習ログの記録
- 過学習防止策（early stopping、weight decay、dropout）の実装

## Interaction with Other Agents

| Agent | 連携内容 |
|-------|----------|
| CV Data Agent | 前処理済み DataLoader を受け取る |
| CV Evaluation Agent | 各 epoch 終了後の検証メトリクスを評価させる |

## Key Libraries

- `torch`, `torchvision`
- `timm`（transfer learning / pretrained backbone）
- `torch.optim`, `torch.optim.lr_scheduler`
- `torch.amp`（混合精度学習）

## Input / Output Contract

### Input

- `torch.utils.data.DataLoader`（train / val）
- トレーニング設定（`@dataclass(slots=True)`）
  - `num_epochs: int`
  - `learning_rate: float`
  - `weight_decay: float`
  - `device: str`（例: `"cuda"`, `"cpu"`）
  - `checkpoint_dir: pathlib.Path`

### Output

- トレーニング済みモデルの重みファイル（`.pt` / `.pth`）
- 学習曲線データ（train loss / val loss / val metric の時系列）

## Implementation Guidelines

- トレーニング設定は `@dataclass(slots=True)` で定義し、`pyproject.toml` またはYAMLファイルから読み込む。
- 再現性を確保するためにランダムシードを固定する（`torch.manual_seed`, `numpy.random.seed`）。
- `pathlib.Path` を使用してチェックポイントのパスを管理する。
- GPU/CPU を抽象化するために `torch.device` を使用する。
- 学習率・損失値は `float` 型で記録する。

## Example Structure

```
src/
  cv_framework/
    training/
      __init__.py
      trainer.py       # Trainer クラス（学習ループ）
      config.py        # トレーニング設定データクラス
      callbacks.py     # EarlyStopping、チェックポイント保存コールバック
      scheduler.py     # 学習率スケジューラのファクトリ
      logger.py        # TensorBoard / WandB ロガー
```

## Related Skills

- `docs/skills/image-classification.md`
- `docs/skills/object-detection.md`
- `docs/skills/image-segmentation.md`
