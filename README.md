# Project Aether-Savannah: Technical Trial
**Git Repo Name:** `savannah-albedo-mapper`
**Description:** A multi-spectral processing pipeline to calculate the physical albedo and vegetation indices of the savannah canopy.

### The Problem
We need to know how much solar radiation is reflected by the canopy (Albedo) versus how much is absorbed by vegetation.

### The Task
Pull one Sentinel-2 L2A scene from the MPC STAC API for the Minna bounding box `[6.40, 9.40, 6.70, 9.80]` (format: `[lon_min, lat_min, lon_max, lat_max]`) from October 2023. Select the least-cloudy available scene.

Calculate the **Broadband Surface Albedo ($\alpha$)** using the Liang (2001) empirical weighting adapted for Sentinel-2:

$$\alpha \approx 0.356B_2 + 0.130B_4 + 0.373B_{8A} + 0.085B_{11} + 0.072B_{12} - 0.0018$$

All band values must be in physical reflectance units $[0, 1]$. Sentinel-2 L2A DN values are scaled by 10,000 — divide before applying the formula. Resample all bands to a common 20m resolution before computation (B2 and B4 are native 10m; B8A, B11, B12 are native 20m).

Also compute **NDVI** per pixel:

$$NDVI = \frac{B_{8A} - B_4}{B_{8A} + B_4}$$

### The Constraints

1. **Cloud masking:** Use the Scene Classification Layer (SCL) to mask invalid pixels before computation. Exclude the following SCL classes:

| SCL Value | Class |
|---|---|
| 0 | No data |
| 1 | Saturated / defective |
| 3 | Cloud shadow |
| 8 | Cloud (medium probability) |
| 9 | Cloud (high probability) |
| 10 | Thin cirrus |

Set masked pixels to `NaN` and exclude from all downstream calculations.

2. **Comparison:** Compute per-pixel $\alpha$ and $NDVI$ over the valid (unmasked) scene extent.

### Strict Anti-AI Policy
You must **not use AI coding assistants**. The objective is to verify your ability to process multi-spectral data and apply radiative transfer concepts from scratch. Evidence of AI-generated code will result in automatic disqualification.

### Submission Requirements
1. **Git repo** containing your processing script and a `requirements.txt`.
2. A two-panel figure:
   - **Left:** Spatial map of $\alpha$ across the bounding box.
   - **Right:** Scatter plot of $NDVI$ (x-axis) vs $\alpha$ (y-axis), one point per valid pixel, showing the relationship between vegetation density and canopy reflectance.


   ## Results

- **Selected scene**: S2B_MSIL2A_20231029T095029_R079_T31PHL_20231029T142618  
- **Date**: 29 October 2023  
- **AOI cloud cover**: 32.4% (global scene: 45.3%)  
- **Processing baseline**: 5.09 (offset correction applied)  
- **Mean Albedo (α)**: 0.2166 ± 0.0371  
- **Mean NDVI**: 0.5748 ± 0.1688  
- **NDVI–Albedo correlation**: r = −0.21  

The spatial albedo map shows a homogeneous low-reflectance surface.
The NDVI vs albedo scatter shows a moderate negative relationship, confirming that denser vegetation is associated with slightly lower broadband albedo.

## Notes on the brief

|     Item     |  What the brief said   |   What was done and why    |

| Cloud filter | `eo:cloud_cover < 20%` | No scene met this threshold over the AOI. Switched to pixel-level SCL scoring within the bbox and selected the least cloudy scene (32.4% AOI cloud cover). |
| Reflectance conversion | `DN / 10000` | Applied `(DN + 1000) / 10000` for processing baseline ≥ 4.0, as required for scenes since January 2022. |
| Resampling | Not specified | Used `Resampling.average` instead of nearest-neighbour when downsampling B02 and B04 from 10m to 20m. |
