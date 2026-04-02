# 🔬 Breast Cancer Mitosis Detection on Whole Slide Images

> **Published Paper:** [Web based Mitosis Detection on Breast Cancer Whole Slide Images using Faster R-CNN and YOLOv5](https://dx.doi.org/10.14569/IJACSA.2022.0131268)  
> *International Journal of Advanced Computer Science and Applications (IJACSA), Vol. 13, No. 12, 2022*  
> Rajasekaran Subramanian, R. Devika Rubi, Rohit Tapadia, Katakam Karthik, Mohammad Faseeh Ahmed, Allam Manudeep

---

## Overview

Mitotic count is a core component of the **Nottingham Grading System** for breast cancer classification. Manual counting under a microscope is slow, semi-quantitative, and prone to inter-observer variability. This project automates mitosis detection on WSIs using **Faster R-CNN** (via Detectron2), trained on 56,258 annotated tiles, achieving an **F1 score of 84%**.

The system is integrated into **CADD4MBC** — a web-based platform for WSI upload, annotation, and diagnostic visualization.

---

## Workflow

```
WSI (.tiff)
    │
    ▼
[1. Tile Extraction]
    Break WSI into tiles (e.g. 992×992 px), named {x}_{y}.png
    │
    ▼
[2. Annotation & Mask Generation]  ◄── MaskGenerator-Copy1.ipynb
    Load CADD4MBC JSON annotations per tile
    Generate binary masks using polygon fill (cv2.fillPoly)
    │
    ▼
[3. Dataset Pipeline]  ◄── Pipeline_notebook_for_preparing_dataset-video-final.ipynb
    Organize tiles + masks across slides (T98, T120, T121, ...)
    Convert annotations to COCO format for Detectron2
    Build train/test split text files
    │
    ▼
[4. Training — Faster R-CNN]  ◄── Mitosis_Final.ipynb / KmitMitosDetectron-KmitData_test.ipynb
    Train Faster R-CNN X-101-FPN on annotated mitosis tiles
    Evaluate on KMIT dataset, MITOS-12, MITOS-14, Madhubushi
    │
    ▼
[5. Inference on WSI]
    Run predictor tile-by-tile across the full slide
    Apply score threshold, draw bounding boxes
    Compute Precision, Recall, F1
    │
    ▼
[6. Visualization via CADD4MBC Web Platform]
    Upload WSI → view detections overlaid on slide in browser
```

---

## Project Structure

```
MitosisDetection/
├── MaskGenerator-Copy1.ipynb                                    # JSON annotation → binary mask generation
├── Pipeline_notebook_for_preparing_dataset-video-final.ipynb    # Dataset prep & COCO format conversion
├── Mitosis_Final.ipynb                                          # Main training & evaluation notebook
├── KmitMitosDetectron-KmitData_test.ipynb                       # Testing on KMIT institutional dataset
│
└── Dataset/
    ├── cadd_json_files/
    │   └── <slide_id>/<batch>/<x>_<y>.json   # Per-tile CADD4MBC annotations
    ├── filtered_images/
    │   └── <slide_id>/                        # Extracted WSI tiles (.jpg / .png)
    ├── filtered_image_masks/
    │   └── <slide_id>/                        # Generated binary masks
    └── json_list/
        └── <slide_id>_json.txt                # Index of annotated tile filenames
```

---

## Step-by-Step Guide

### Step 1 — Tile Extraction

Break the WSI `.tiff` into fixed-size tiles. Tiles are named by their grid position:

```
{x}_{y}.png     →    e.g.   43_94.png   (column 43, row 94)
```

Annotations from the CADD4MBC viewer are saved as per-tile JSON files matching this naming.

---

### Step 2 — Mask Generation

`MaskGenerator-Copy1.ipynb` reads each CADD4MBC annotation JSON and generates a binary mask for the corresponding tile.

Each JSON contains polygon point coordinates for annotated mitotic figures. The notebook:
1. Loads the tile image and its annotation JSON
2. Draws filled polygons onto a blank canvas using `cv2.fillPoly`
3. Applies binary thresholding
4. Saves the mask alongside the tile

---

### Step 3 — Dataset Pipeline

`Pipeline_notebook_for_preparing_dataset-video-final.ipynb` handles the full multi-slide data organization:

- Scans annotation JSONs across slides (T98, T120, T121, T104, T118, ...)
- Copies corresponding tile images to working directories
- Converts polygon annotations into **COCO format** (bounding boxes + segmentation)
- Builds `train.txt` / `test.txt` split files for Detectron2

**Dataset scale:** 56,258 annotated tiles across multiple WSI slides.

---

### Step 4 — Training (Faster R-CNN)

Training uses **Detectron2** with a pretrained **Faster R-CNN X-101-32x8d-FPN** backbone fine-tuned on the mitosis dataset.

```bash
# Launch Jupyter and open Mitosis_Final.ipynb
jupyter notebook Mitosis_Final.ipynb
```

**Key configuration:**

| Parameter | Value |
|---|---|
| Backbone | ResNeXt-101-32x8d + FPN |
| Pretrained weights | COCO model zoo |
| LR Scheduler | WarmupMultiStepLR |
| Warmup iterations | 1,000 |
| Max iterations | 5,000 |
| Base LR | 0.001 |
| Score threshold (eval) | 0.5 – 0.8 |
| Output | `model_final.pth` |

**Evaluation datasets:**
- KMIT institutional dataset
- MITOS-12 (public benchmark)
- MITOS-14 (public benchmark)
- Madhubushi dataset

---

### Step 5 — Inference & Evaluation

After training, the model is run tile-by-tile over the WSI. Custom IoU-based TP/FP/FN matching is used to compute metrics:

```python
precision = TP / (TP + FP)
recall    = TP / (TP + FN)
F1        = 2 * precision * recall / (precision + recall)
```

**Result: F1 = 84%** on the KMIT dataset.

Predictions are visualized as bounding boxes overlaid on tiles, with confidence scores displayed.

---

### Step 6 — Web Platform (CADD4MBC)

Detections are served through the **CADD4MBC** web-based imaging platform, which supports:
- WSI upload and tile serving
- Annotation viewing and editing
- Overlay of model predictions on the slide viewer

---

## Setup

### Requirements

```bash
pip install torch torchvision
pip install detectron2 -f https://dl.fbaipublicfiles.com/detectron2/wheels/cu102/torch1.10/index.html
pip install opencv-python pillow numpy matplotlib tqdm pycocotools
```

> GPU required. Tested with PyTorch 1.10, CUDA 10.2, Detectron2.

---

## Results

| Dataset | F1 Score |
|---|---|
| KMIT Dataset | **84%** |
| MITOS-12 | *(see paper)* |
| MITOS-14 | *(see paper)* |

---

## Citation

If you use this work, please cite:

```bibtex
@article{Subramanian2022,
  title   = {Web based Mitosis Detection on Breast Cancer Whole Slide Images using Faster R-CNN and YOLOv5},
  journal = {International Journal of Advanced Computer Science and Applications},
  doi     = {10.14569/IJACSA.2022.0131268},
  url     = {http://dx.doi.org/10.14569/IJACSA.2022.0131268},
  year    = {2022},
  volume  = {13},
  number  = {12},
  author  = {Rajasekaran Subramanian and R. Devika Rubi and Rohit Tapadia and
             Katakam Karthik and Mohammad Faseeh Ahmed and Allam Manudeep}
}
```

---

## References

- [Faster R-CNN — Ren et al., 2015](https://arxiv.org/abs/1506.01497)
- [Detectron2 — Facebook AI Research](https://github.com/facebookresearch/detectron2)
- [MITOS-12 / MITOS-14 Benchmarks](http://ludo17.free.fr/mitos_2012/)
- Nottingham Histologic Grading System for breast cancer

---

## License

MIT License.
