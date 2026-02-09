# Floor Classification Report  
**ADE Challenge 2016 – Floor vs Non-Floor Segmentation**

---

## 1. Performance Overview

Three different floor–non-floor segmentation approaches were implemented and
evaluated on the ADE Challenge 2016 dataset. Quantitative results from the final
runs are summarized below.

### Table 1: Quantitative Performance Comparison

| Approach                    | Accuracy | Precision | Recall | Train Time | Inference Time |
|-----------------------------|----------|-----------|--------|------------|----------------|
| Pixel-based (RGB)           | 93.92%   | 0.0000    | 0.0000 | 0.0161 s   | 0.0006 s       |
| RGB + Spatial (X, Y)        | 93.62%   | 0.0548    | 0.0033 | 0.0395 s   | 0.0004 s       |
| Region-based (KMeans + SVM) | 94.90%   | 0.0000    | 0.0000 | 186.95 s  | 0.0486 s       |

### Observation on Metrics

Despite relatively high accuracy values, some approaches exhibit extremely low
precision and recall for the *floor* class. This indicates a strong class
imbalance in the dataset, where the majority of pixels belong to the background
(non-floor) class.

As a result, models tend to predict the dominant class for most inputs. Accuracy
alone is therefore misleading and does **not** reflect true segmentation quality
for the minority (floor) class without additional techniques such as:

- Class weighting  
- Resampling  
- Alternative evaluation metrics (IoU, F1-score)

---

## 2. Approach-wise Comparison

### 2.1 Performance Characteristics

#### Region-based Approach (KMeans + SVM)

- Uses KMeans clustering to group pixels into coherent regions before
  classification.
- Reduces pixel-level noise and enforces spatial consistency.
- When a region contains a majority of floor pixels, the entire region is labeled
  as floor, eliminating salt-and-pepper artifacts common in pixel-based methods.

**Region Features:**
- Mean RGB values
- Color variance
- Spatial position
- Region area fraction

This approach produces smoother, more visually consistent segmentation results.

---

#### Pixel-based (RGB Only)

- Achieves competitive accuracy (93.92%) despite its simplicity.
- Relies solely on RGB values of individual pixels.
- Extremely fast but lacks contextual understanding.

**Limitations:**
- Cannot distinguish visually similar regions (e.g., white floor vs white wall).
- Sensitive to lighting variations.

---

#### Pixel-based (RGB + Spatial Coordinates)

- Adds normalized spatial coordinates (X, Y) to RGB features.
- Provides positional awareness (e.g., floors often appear near the bottom).

**Trade-off:**
- Can improve segmentation when layouts are consistent.
- Risk of overfitting to dataset-specific spatial distributions.

---

### 2.2 Speed and Computational Cost

#### Pixel-based Methods

- Sub-second training and inference times.
- Powered by `LinearSVC`, optimized for large-scale linear classification.
- Suitable for real-time or resource-constrained applications.

---

#### Region-based Method

Significantly slower due to:
- Image downscaling
- KMeans clustering
- Region-level feature extraction
- SVM training on region features

Once trained, inference remains reasonably fast since classification operates on
regions instead of individual pixels.

---

### 2.3 Robustness and Generalization

| Method                | Robustness |
|----------------------|------------|
| Pixel-based (RGB)    | Low        |
| RGB + Spatial (XY)   | Moderate   |
| Region-based         | High       |

- **Pixel-based (RGB):** Highly sensitive to color ambiguity and lighting.
- **RGB + XY:** Learns layout priors but may fail on unseen distributions.
- **Region-based:** Most robust due to spatial coherence and aggregated features.

---

## 3. Implementation Details

### Dataset
- **Dataset:** ADE Challenge 2016
- **Training Images:** 50
- **Validation Images:** 20
- **Floor Class ID:** 4

---

### Method 1: Pixel-based (RGB)

- **Features:** 3 (R, G, B normalized to [0, 1])
- **Classifier:** `LinearSVC`
- **C:** 1.0
- **Max Iterations:** 2000
- **Training Samples:** 50,000
- **Validation Samples:** 30,000

---

### Method 2: Pixel-based (RGB + Spatial)

- **Features:** 5 (R, G, B, X, Y normalized to [0, 1])
- **Classifier:** `LinearSVC`
- **C:** 1.0
- **Max Iterations:** 3000
- **Training Samples:** 100,000
- **Validation Samples:** 20,000

---

### Method 3: Region-based (KMeans + SVM)

- **Max Image Dimension:** 300 px
- **Clusters per Image:** 200
- **Features (7):**
  - Mean R
  - Mean G
  - Mean B
  - Color standard deviation (sum)
  - Mean X
  - Mean Y
  - Area fraction
- **Classifier:** `SVC` (Linear Kernel)
- **C:** 1.0

---

## 4. Summary

Each approach presents a clear trade-off between speed, robustness, and accuracy:

1. **Pixel-based (RGB):**
   - Fastest and simplest.
   - Suitable for real-time applications.
   - Struggles with visual ambiguity.

2. **RGB + Spatial:**
   - Adds positional awareness at minimal cost.
   - May overfit to dataset-specific layouts.

3. **Region-based (KMeans + SVM):**
   - Best spatial coherence and robustness.
   - Significantly higher training time.
   - Suitable for offline processing.

### Key Takeaway

High accuracy alone is misleading in segmentation tasks with severe class
imbalance. Precision, recall, and region-level consistency provide a more
meaningful evaluation.

**Future Improvements:**
- Class weighting
- Data augmentation
- Deep learning-based segmentation models

---
