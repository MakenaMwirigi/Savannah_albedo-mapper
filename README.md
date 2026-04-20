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