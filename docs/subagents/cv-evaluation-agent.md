# CV Evaluation Agent

## Overview

CV Evaluation Agent は Computer Vision モデルの評価・分析・可視化を担当するサブエージェントです。
テストセットに対する推論実行、各種メトリクスの算出、結果の可視化、モデル分析（混同行列、Grad-CAM等）を行います。

## Responsibilities

- テストセットに対する推論の実行
- タスクに応じたメトリクスの算出
  - 分類: Accuracy, Precision, Recall, F1-score, AUC-ROC
  - 物体検出: mAP (mean Average Precision), IoU
  - セグメンテーション: mIoU, Dice coefficient, Pixel Accuracy
- 混同行列の生成・可視化
- Grad-CAM / SHAP などによる予測根拠の可視化
- モデルの推論速度（latency）・スループット測定
- エラーケースの分析・サンプリング

## Interaction with Other Agents

| Agent | 連携内容 |
|-------|----------|
| CV Data Agent | テストセットの DataLoader を受け取る |
| CV Training Agent | 評価対象のモデル重みを受け取る |

## Key Libraries

- `torch`, `torchvision`
- `torchmetrics`
- `matplotlib`, `seaborn`
- `scikit-learn`（混同行列、分類レポート）
- `grad-cam`（`pytorch-grad-cam`）

## Input / Output Contract

### Input

- 評価対象モデル（`torch.nn.Module`）
- テストセットの `torch.utils.data.DataLoader`
- 評価設定（`@dataclass(slots=True)`）
  - `device: str`
  - `output_dir: pathlib.Path`（レポート・可視化の保存先）

### Output

- メトリクス辞書（`dict[str, float]`）
- 混同行列 (`npt.NDArray[np.int64]`)
- 可視化画像（PNG ファイル、`pathlib.Path` で管理）
- 評価レポート（Markdown または JSON）

## Implementation Guidelines

- メトリクスは `torchmetrics` を優先して使用し、独自実装を避ける。
- 可視化ファイルは `pathlib.Path` で管理し、出力先を設定ファイルで制御する。
- 推論時は `torch.inference_mode()` コンテキストを使用する。
- バッチ処理で OOM を防ぐため、テストデータは DataLoader 経由で処理する。

## Example Structure

```
src/
  cv_framework/
    evaluation/
      __init__.py
      evaluator.py     # Evaluator クラス（推論 + メトリクス算出）
      metrics.py       # メトリクス計算ユーティリティ
      visualizer.py    # 混同行列・Grad-CAM 可視化
      reporter.py      # 評価レポート生成（Markdown / JSON）
```

## Related Skills

- `docs/skills/image-classification.md`
- `docs/skills/object-detection.md`
- `docs/skills/image-segmentation.md`
