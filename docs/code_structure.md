# Code Structure

```
  src/{project_name}
  |-- configs      # Configuration files for datasets, models, logging, and experiments.
  |-- data         # Data loading, augmentation, preprocessing, and dataset utilities.
  |-- models       # Model definitions, layers, and model-building modules.
  |-- utils        # Shared utilities (training helpers, logging, losses, and common functions).
  |-- train.py     # Training entry point.
  `-- inference.py # Inference/evaluation entry point.
```

## Description

- `configs`: Centralized configuration files to control data, model, runtime, and logging behavior.
- `data`: Components for reading datasets and preparing inputs for training and inference.
- `models`: Core network architectures and reusable model components.
- `utils`: Common helper modules used across the project.
- `train.py`: Script to launch model training.
- `inference.py`: Script to run prediction or evaluation with trained weights.

## Agents (`.github/agents/`)

```
  .github/agents/
  |-- cv-data-agent.agent.md       # Pair dataset loading, synthetic degradation pipelines, and patch-based DataLoader.
  |-- cv-training-agent.agent.md   # Model training with pixel/perceptual/SSIM loss and MLflow experiment tracking.
  `-- cv-evaluation-agent.agent.md # PSNR / SSIM / LPIPS evaluation, side-by-side visualization, and MLflow logging.
```

### Description

- `cv-data-agent`: Loads and validates degraded/clean image pairs, builds synthetic degradation and augmentation pipelines, and provides `DataLoader` for training and evaluation.
- `cv-training-agent`: Selects model architecture, implements the training loop (iteration-based), manages checkpoints, and tracks experiments with MLflow.
- `cv-evaluation-agent`: Runs inference on the test set, computes PSNR / SSIM / LPIPS, generates side-by-side visualizations, and logs results to MLflow.

## Skills (`.github/skills/`)

```
  .github/skills/
  |-- data-augmentation/ # Pair-consistent spatial augmentation pipeline using albumentations.
  |-- image-denoising/   # Supervised denoising model for AWGN / Poisson / real-world noise (DnCNN, NAFNet, Restormer).
  |-- image-deblurring/  # Supervised deblurring model for motion / defocus / Gaussian blur (NAFNet, MPRNet, DeblurGAN-v2).
  `-- super-resolution/  # Single image super-resolution ×2/×3/×4 (EDSR, SwinIR, Real-ESRGAN).
```

### Description

- `data-augmentation`: Builds an augmentation pipeline that applies identical spatial transforms to both degraded and clean images using `albumentations`.
- `image-denoising`: Implements a supervised denoising model that restores clean images from noisy inputs; supports AWGN, Poisson, salt-and-pepper, and real-world noise.
- `image-deblurring`: Implements a supervised deblurring model that recovers sharp images from blurry inputs; supports motion, defocus, and Gaussian blur.
- `super-resolution`: Implements a single image super-resolution model that upscales low-resolution images to high-resolution outputs at ×2, ×3, and ×4 scales.