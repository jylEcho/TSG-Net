# TSG-Net

**A Temporal Semantic-Guided Network with Dual-Domain Reversible Interaction for Multiphase Contrast-Enhanced CT Analysis**

TSG-Net integrates arterial (ART), portal venous (PV), and delayed (DL) phases for liver and tumor segmentation in contrast-enhanced CT.

## 📰 News
- ### 2026.10 — All codes will be released immediately upon acceptance!✅


## 🧠 Overview

The network consists of two main components:

- **Multi-Phase Dynamic Semantic Aggregation (MDSA):** combines SAM semantic priors with adaptive phase attention to aggregate complementary information.
- **Dual-Domain Reversible Interaction Transformer (DRIT):** models interactions between spatial and frequency representations.

This repository includes implementations for **MPLL** and **PLC-CECT**, with two SAM backbone options:

| Version | Backbone | Model file |
| :--- | :--- | :--- |
| Paper version | SAM ViT-B | `MADFNetV6_SAMV3.py` |
| Lightweight version | MobileSAM | `MADFNetV6_SAMV3_thinV1.py` |

## 🏗️ Architecture

### Overall framework

coming soon~

### Attention mechanism

coming soon~

## 📁 Repository Structure

```text
.
├── AutoSAM/
│   └── segment_anything/       # SAM implementation used by TSG-Net
├── MobileSAM/
│   └── mobile_sam/             # MobileSAM implementation
├── multi_phase/
│   ├── bash/                  # Training script examples
│   ├── datasets/              # Dataset loading
│   ├── lists/                 # Training and testing split lists
│   ├── networks/              # TSG-Net models and supporting networks
│   ├── optimizers/            # Learning rate scheduling
│   ├── trainer/               # Training and validation
│   ├── utils/                 # Losses, inference, and metrics
│   ├── train_7metrics_MPLL_GY.py
│   └── train_7metrics_PLCCECT_GY.py
├── COPY_AUDIT.json
└── README.md
```

Datasets, pretrained weights, training logs, and experiment outputs are not included.

## ⚙️ Environment

The original project uses Python 3.10 and PyTorch.

Install a PyTorch and torchvision combination compatible with your CUDA environment, then install the additional dependencies:

```bash
pip install numpy scipy h5py einops imgaug medpy monai \
    nibabel SimpleITK tensorboardX tqdm matplotlib pandas \
    timm opencv-python
```

### Pretrained weights

Place the backbone checkpoints in the following directory:

```text
AutoSAM/
└── Pth_files/
    ├── sam_vit_b_01ec64.pth    # Paper version
    └── mobile_sam.pt          # Lightweight version
```

Backbone resources:

- [Segment Anything](https://github.com/facebookresearch/segment-anything)
- [MobileSAM](https://github.com/ChaoningZhang/MobileSAM)

Use the corresponding checkpoint for each model version.

## 📦 Data Preparation

Organize the prepared datasets as follows:

```text
data/
├── MPLL/
│   ├── train/
│   └── test/
└── PLCCECT/
    ├── train/
    └── test/
```

The provided split lists are located in:

```text
multi_phase/lists/
├── lists_liver-MPLL/
│   ├── train.txt
│   └── test_vol.txt
└── lists_liver-PLCCECT/
    ├── train.txt
    └── test_vol.txt
```

### Expected format

| Split | File format | Image organization |
| :--- | :--- | :--- |
| Training | `.npz` | Prepared 2D slices |
| Testing | `.h5` | Prepared 3D volumes |

The data loader uses the following fields:

- `image_A`, `image_V`, `image_D`: ART, PV, and DL images.
- `label`: segmentation annotations.
- Crop metadata required by the data loader and inference functions.

Testing phase images must be aligned scalar volumes with shape `(H, W, Z)`.

Label definitions:

| Value | Class |
| :---: | :--- |
| 0 | Background |
| 1 | Liver |
| 2 | Tumor |

Refer to `multi_phase/datasets/dataset_multiphase.py` for the complete data-loading format.

## 🚀 Training

Run the following commands from the `multi_phase` directory:

```bash
cd multi_phase
```

### MPLL — paper version

```bash
CUDA_VISIBLE_DEVICES=0 python train_7metrics_MPLL_GY.py \
    --dataset Multiphase \
    --module networks.MADFNetV6_SAMV3.HAformerSpatialFrequency \
    --model_name MADFNetV6 \
    --root_path ../data/MPLL/train \
    --test_path ../data/MPLL/test \
    --list_dir lists/lists_liver-MPLL \
    --output_dir ../runs/MPLL \
    --n_gpu 1 \
    --batch_size 6 \
    --max_epochs 100 \
    --base_lr 0.01 \
    --crop_inference standard_crop_inference \
    --eval_interval 6
```

### PLC-CECT — paper version

```bash
CUDA_VISIBLE_DEVICES=0 python train_7metrics_PLCCECT_GY.py \
    --dataset Multiphase \
    --module networks.MADFNetV6_SAMV3.HAformerSpatialFrequency \
    --model_name MADFNetV6 \
    --root_path ../data/PLCCECT/train \
    --test_path ../data/PLCCECT/test \
    --list_dir lists/lists_liver-PLCCECT \
    --output_dir ../runs/PLCCECT \
    --n_gpu 1 \
    --batch_size 6 \
    --max_epochs 100 \
    --base_lr 0.01 \
    --crop_inference standard_crop_inference \
    --eval_interval 6
```

Adjust the batch size to your available GPU memory.

### MobileSAM version

To use the lightweight model, replace the `--module` argument with:

```bash
--module networks.MADFNetV6_SAMV3_thinV1.HAformerSpatialFrequency
```

The shell examples under `multi_phase/bash/` select this MobileSAM variant.

## 📊 Evaluation

The training pipeline performs periodic inference according to `--eval_interval`.

The implementation reports seven segmentation metrics:

| Metric | Preferred direction |
| :--- | :---: |
| Dice Similarity Coefficient (DSC) | ↑ |
| Jaccard Index | ↑ |
| 95th-percentile Hausdorff Distance (HD95) | ↓ |
| Average Symmetric Surface Distance (ASSD) | ↓ |
| Root Mean Square Surface Distance (RMSD) | ↓ |
| Volumetric Overlap Error (VOE) | ↓ |
| Relative Volume Difference (RVD) | Closer to 0 |

Inference and metric calculations are implemented in:

```text
multi_phase/utils/utils_multiphase_7metrics_JSC.py
```

## 🙏 Acknowledgements

This project builds upon [Segment Anything](https://github.com/facebookresearch/segment-anything) and [MobileSAM](https://github.com/ChaoningZhang/MobileSAM).

Third-party license files are retained in their respective directories.
