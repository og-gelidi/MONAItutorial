## how to run:

### Google Colab (recommended):
Easiest method is by using Google Colab (need to add func + instructions)

### VSCode / Own Device:
Activate venv using:
`.\.venv\Scripts\Activate.ps1`
in the terminal, from the root directory

--- 
stuff to write for tutorial 
- build pipeline:
- - 
- download dependencies
- run...?

--- 

## MICCAI 2026 Dataset 

#### Directory structure:
```text
CINE_MULTI/
├── SAX_TR/
│   ├── image/                     # Short-axis CINE images (.nii.gz)
│   └── anno/                      # Segmentation labels (.nii.gz)
├── SAX_VAL/
│   ├── image/                     # Validation short-axis images (.nii.gz)
│   └── anno/                      # Validation segmentation labels (.nii.gz)
├── 2CH_TR/
│   ├── image/                     # 2-chamber view CINE images (.nii.gz)
│   └── anno/                      # Segmentation labels (.nii.gz)
├── 2CH_VAL/
│   ├── image/                     # Validation 2-chamber images (.nii.gz)
│   └── anno/                      # Validation segmentation labels (.nii.gz)
├── 4CH_TR/
│   ├── image/                     # 4-chamber view CINE images (.nii.gz)
│   └── anno/                      # Segmentation labels (.nii.gz)
├── 4CH_VAL/
│   ├── image/                     # Validation 4-chamber images (.nii.gz)
│   └── anno/                      # Validation segmentation labels (.nii.gz)
├── dataset_train.xlsx             # Metadata for training cohorts
├── dataset_valid.xlsx             # Metadata for validation cohorts
└── id_slice_info_valid.json       # Slice index metadata for validation sets
```
<small>Note: there are also _TST directories for each view (e.g.: SAX_TST), however these are only useful in the challenge itself, as they are unlabelled.</small>

#### Label Definitions:

| View | Label ID | Description |
| :--- | :---: | :--- |
| **SAX_TR** | 1 | Left Ventricle Myocardium |
| | 2 | Left Ventricle Cavity |
| | 3 | Right Ventricle Cavity |
| **2CH_TR** | 1 | Left Ventricle Cavity |
| | 2 | Left Ventricle Myocardium |
| **4CH_TR** | 1 | Left Ventricle Cavity |
| | 2 | Left Ventricle Myocardium |
| | 3 | Right Ventricle Cavity |
| | 4 | Right Atrium |
| | 5 | Left Atrium |

---

## Glossary:

### 1. Dataset Naming Conventions & Views

| Code / Term | Full Name | Plane / Orientation | Description & Key Structures | Typical Dimensions |
| :--- | :--- | :--- | :--- | :--- |
| **`SAX`** | **Short-Axis** | Perpendicular to the LV long axis (transverse cuts) | Slices from heart **base to apex**. Shows the circular Left Ventricle (LV) blood pool, donut-shaped Myocardium (MYO), and crescent-shaped Right Ventricle (RV). Primary view for volume and mass quantification. | 3D stack over time: `(H, W, slices, phases)` |
| **`2CH`** | **2-Chamber** | Vertical Long-Axis (VLA) | Visualizes the **left side only**: Left Atrium (LA), Left Ventricle (LV), anterior/inferior myocardial walls, and mitral valve plane. The RV is not visible in this view. | Single slice over time: `(H, W, 1, phases)` |
| **`4CH`** | **4-Chamber** | Horizontal Long-Axis (HLA) | Cuts across the whole heart to show **all four chambers** (LV, RV, LA, RA), the interventricular septum, and both atrioventricular valves (mitral and tricuspid). | Single slice over time: `(H, W, 1, phases)` |
| **`_TR`** | **Training Set** | Metadata tag | Suffix denoting training split acquisitions with ground-truth segmentation masks available. | N/A |
| **`_TS`** | **Test / Validation Set** | Metadata tag | Suffix denoting validation or test split acquisitions used for benchmarking. | N/A |


### 2. Anatomical Target Structures & Segmentation Labels

| Label ID | Structure | Acronym | Clinical Significance |
| :---: | :--- | :--- | :--- |
| **0** | **Background** | `BG` | Surrounding chest wall, lungs, spine, and non-cardiac tissue. |
| **1** | **Left Ventricular Cavity** | `LV` (or `LVBP`) | The blood pool inside the main pumping chamber; measured to determine end-diastolic and end-systolic volumes. |
| **2** | **Left Ventricular Myocardium** | `LVM` / `MYO` | The thick muscular wall of the left ventricle; used to compute myocardial mass and detect thinning/hypertrophy. |
| **3** | **Right Ventricular Cavity** | `RV` (or `RVBP`) | The crescent blood pool pumping to the pulmonary circulation; essential for biventricular functional assessment. |

### 3. Cardiac Cycle & Temporal Terminology

* **Cine CMR:** Time-resolved MRI sequence capturing the heart beating across 25–30 time phases per cardiac cycle.
* **End-Diastole (ED):** The frame in the cardiac cycle where the ventricles are fully filled with blood and reach their maximum volume (relaxed heart).
* **End-Systole (ES):** The frame where the ventricles have finished contracting and reach their minimum volume (maximally squeezed).
* **Simpson's Rule (Disk Summation):** The standard clinical method of calculating total ventricular cavity volume by multiplying the segmented 2D blood-pool area on each short-axis slice by the slice thickness plus inter-slice gap, then summing across all slices from base to apex:
  $$\text{Volume} = \sum_{i=1}^{N} \left( \text{Area}_i \times (t_{\text{slice}} + t_{\text{gap}}) \right)$$


### 4. Key Clinical Metrics

* **End-Diastolic Volume (EDV):** Total cavity volume at the ED phase ($mL$).
* **End-Systolic Volume (ESV):** Total cavity volume at the ES phase ($mL$).
* **Stroke Volume (SV):** Blood volume pumped out per beat:
  $$\text{SV} = \text{EDV} - \text{ESV}$$
* **Left Ventricular Ejection Fraction (LVEF):** The primary diagnostic metric of cardiac pumping function:
  $$\text{LVEF} = \frac{\text{SV}}{\text{EDV}} \times 100\% = \frac{\text{EDV} - \text{ESV}}{\text{EDV}} \times 100\%$$
* **Dice Similarity Coefficient (DSC):** Overlap metric used to benchmark segmentation predictions against ground truth masks ($X$ and $Y$):
  $$\text{DSC} = \frac{2 |X \cap Y|}{|X| + |Y|}$$