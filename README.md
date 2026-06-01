# HEp-2 Cell Pattern Classification

An automated cell-level classification system for HEp-2 fluorescence microscopy images using hand-crafted image features and a Random Forest classifier. Built as part of a Computer-Aided Diagnosis (CAD) pipeline for the ANA (Antinuclear Antibody) test.

---

## Background

The ANA test is the standard laboratory method for diagnosing autoimmune diseases such as Lupus, Rheumatoid Arthritis, and Scleroderma. A patient's blood sample is applied to HEp-2 cells, stained with fluorescent dye, and examined under a fluorescence microscope. A specialist identifies the staining pattern to determine the diagnosis.

This manual process is time-consuming, subjective, and highly dependent on the operator's experience. This project automates the pattern recognition step using Digital Image Processing (DIP) and Machine Learning techniques.

---

## Task

**Cell-level classification** — each individual cropped cell image is classified independently into one of 7 staining pattern classes.

| Pattern | Cells | Associated Disease |
|---------|-------|--------------------|
| Homogeneous | 330 | Lupus (SLE) |
| Nucleolar | 241 | Scleroderma |
| Centromere | 212 | CREST syndrome |
| Coarse speckled | 210 | Mixed connective tissue disease |
| Fine speckled | 208 | Sjögren's syndrome |
| Centromeric | 145 | Scleroderma |
| Cytoplasmatic | 110 | Liver diseases |

---

## Dataset

**MIVIA HEp-2 Image Dataset**
- 28 patients, one fluorescence slide image per patient
- 1,455 individually labeled and cropped cell images
- Binary cell masks provided
- Labels provided in per-patient CSV files (semicolon-separated)

Download: https://mivia.unisa.it/datasets/biomedical-image-datasets/hep2-image-dataset/

### Dataset folder structure expected
```
Main_Dataset/
├── Images/
│   ├── 01/
│   │   ├── <slide>.bmp         # full fluorescence slide image
│   │   ├── <mask>.bmp          # black and white slide mask
│   │   └── <labels>.csv        # cell labels (semicolon-separated)
│   └── 02/ ... 28/
└── Cells/
    ├── Cell_Images/
    │   ├── 01/                 # individual cropped cell PNGs (001.png, 002.png ...)
    │   └── 02/ ... 28/
    └── Cell_Masks/
        ├── 01/                 # binary mask PNGs per cell
        └── 02/ ... 28/
```

---

## Pipeline

```
Cell Images (PNG)
      ↓
Preprocessing — CLAHE contrast enhancement
      ↓
Feature Extraction — 40 features per cell
  ├── Intensity statistics     (6 features)
  ├── GLCM texture descriptors (6 features)
  ├── LBP local patterns       (26 features)
  └── Sobel gradient features  (2 features)
      ↓
Label Encoding + StandardScaler normalization
      ↓
Random Forest Classifier (200 trees, 80/20 train-test split)
      ↓
Predicted staining pattern class
```

---

## Feature Extraction Details

### Intensity Statistics (6 features)
Basic brightness measurements: mean, standard deviation, min, max, 25th and 75th percentile of pixel values.

### GLCM — Gray Level Co-occurrence Matrix (6 features)
Captures spatial texture by analyzing pixel neighborhood relationships. Extracts: contrast, dissimilarity, homogeneity, energy, correlation, and ASM at distances [1, 3] and angles [0°, 45°, 90°].

### LBP — Local Binary Pattern (26 features)
For each pixel, compares intensity with 8 surrounding neighbors (radius=3). Encodes the comparison as a binary number and builds a 26-bin histogram over the image. Captures local micro-texture patterns effectively.

### Gradient Features (2 features)
Applies the Sobel operator to detect intensity edges. Extracts mean and standard deviation of gradient magnitude. Cells with sharp bright spots (e.g. centromere) produce high gradient values.

---

## Results

| Metric | Score |
|--------|-------|
| Accuracy | 89.35% |
| Precision (weighted) | 89.56% |
| Recall (weighted) | 89.35% |
| F1 Score (weighted) | 89.39% |

**Train samples:** 1,164 | **Test samples:** 291

### Per-class performance

| Class | Precision | Recall | F1 |
|-------|-----------|--------|----|
| Centromere | 0.86 | 0.90 | 0.88 |
| Centromeric | 0.97 | 0.97 | 0.97 |
| Coarse speckled | 0.95 | 0.90 | 0.93 |
| Cytoplasmatic | 1.00 | 0.86 | 0.93 |
| Fine speckled | 0.79 | 0.81 | 0.80 |
| Homogeneous | 0.88 | 0.91 | 0.90 |
| Nucleolar | 0.90 | 0.90 | 0.90 |

---

## How to Run

### 1. Open in Google Colab
Upload `HEp2_Classification.ipynb` to [Google Colab](https://colab.research.google.com).

### 2. Upload dataset to Google Drive
Upload the `Main_Dataset` folder to your Google Drive root (`My Drive`).

### 3. Mount Drive and set path
Run the first cells. In Step 3, set:
```python
BASE_PATH = '/content/drive/MyDrive/Main_Dataset'
```
If your folder is inside a subfolder, adjust accordingly:
```python
BASE_PATH = '/content/drive/MyDrive/YourFolder/Main_Dataset'
```

### 4. Run all cells in order
All libraries used (OpenCV, scikit-learn, scikit-image) are pre-installed in Colab. No additional installation needed.

---

## Dependencies

All available by default in Google Colab:

```
numpy
pandas
opencv-python (cv2)
scikit-learn
scikit-image
matplotlib
```

---

## Outputs Generated

- `sample_cells.png` — sample cell images per pattern class
- `confusion_matrix.png` — confusion matrix heatmap
- `feature_importance.png` — top 20 most important features
- `sample_predictions.png` — 10 test cells with true vs predicted labels

---

## Limitations

- Small dataset (28 patients, 1,455 cells) limits generalizability
- Hand-crafted features may not capture all complex spatial morphology
- Fine speckled and coarse speckled are visually similar causing residual misclassification
- Random 80/20 split may include cells from the same patient in both train and test sets
- No data augmentation applied

## Future Work

- CNN-based end-to-end feature learning for larger datasets
- Patient-level train/test split for stricter evaluation
- Apply cell masks as ROI to exclude background pixels during feature extraction
- Data augmentation (rotation, flipping) to increase effective dataset size

---

## Project Info

**Course:** EE433 Digital Image Processing  
**Batch:** BESE-28  
**Dataset:** MIVIA HEp-2 Image Dataset — University of Salerno  
**Environment:** Google Colab (Python 3)
