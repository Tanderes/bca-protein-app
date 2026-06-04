# BCA Protein App

[![Launch App](https://img.shields.io/badge/Launch%20App-tanderes.github.io-4f8ef7?style=for-the-badge&logo=github)](https://tanderes.github.io/bca-protein-app/)

A browser-based tool for analysing Pierce BCA microplate protein assays. Upload two CSVs — absorbance readings and a plate layout map — and the app fits a standard curve, interpolates protein concentrations for every sample well, and renders an interactive box plot grouped by strain. No server, no installation, no data leaves your machine.

---

## Quick start

1. **[Open the app →](https://tanderes.github.io/bca-protein-app/)**
2. Upload your **absorbance CSV** and **plate layout CSV**
3. Set your wavelength, dilution factor, and parent strain in the config bar
4. Click **Run analysis**

---

## Input file format

Both files must be CSV with **rows A–H** in the first column and **columns 1–12** in the header row — matching the standard 96-well plate grid.

### Absorbance file
Raw absorbance readings from the plate reader, one value per well.

```
,1,2,3,4,5,6,7,8,9,10,11,12
A,0.850,0.779,0.815,0.788,...
B,0.863,0.802,0.792,0.770,...
...
H,2.100,1.829,1.276,1.042,...
```

### Plate layout file
Sample names for each well. Wells belonging to the standard curve should be labelled `STD`. Empty wells can be left blank.

```
,1,2,3,4,5,6,7,8,9,10,11,12
A,"Y293+EBB769,c1","Y293+EBB770,c1",...,Y293,Y293
B,"Y293+EBB769,c1","Y293+EBB770,c1",...,Y293,Y293
...
G,Y11,Y243,Y310,Y371,Y11,Y243,...
H,STD,STD,STD,STD,STD,STD,STD,STD,STD,STD,STD,STD
```

> **Standard curve row:** By default the app reads the standard curve from **row H** — the last row of the plate. This matches the layout above where all 12 wells of row H are labelled `STD`. You can change the standard row using the **Standards in row** dropdown in the config bar if your plate is set up differently.

---

## Test files

The following files are provided as a worked example and can be used to verify the app is running correctly. They are from a BCA assay run on 22 May 2026 measuring protein concentration in Y293 yeast strains expressing PDI variants.

| File | Description |
|------|-------------|
| `20260522_BCA_293PDIs.csv` | Absorbance readings at 526 nm from a 96-well microplate |
| `Transformation__Y293___PDIs_-_PC_layout1.csv` | Plate layout map identifying each well |

**Plate layout summary for the test files:**

| Rows | Contents |
|------|----------|
| A–C | Y293+EBB769–797 colony 1 replicates (`c1`) |
| D–F | Y293+EBB769–797 colony 2 replicates (`c2`) |
| G | Control strains: Y11, Y243, Y310, Y371 (3 replicates each) |
| **H** | **BSA standard curve — 9 concentrations across 12 wells (2000, 1500, 1000, 750, 500, 250, 125, 25, 0 µg/mL with replicates)** |

**Recommended settings for the test files:**

| Setting | Value |
|---------|-------|
| Wavelength | 526 nm |
| Dilution factor | 1× |
| Standards in row | **H** |
| Std concentrations | `2000,1500,1000,750,500,250,125,25,0` |
| Parent strain | `Y293` |

Expected output: R² ≥ 0.99, protein concentrations in the range ~500–700 µg/mL, dashed Y293 reference line at approximately 600 µg/mL.

---

## Config options

| Setting | Description |
|---------|-------------|
| **Wavelength** | Plate reader measurement wavelength (nm). Default 526. |
| **Dilution factor** | Multiplier applied to all interpolated concentrations. Use `1` for undiluted samples. |
| **Standards in row** | Which plate row contains the BSA standards. Default `H`. |
| **Std concentrations** | Comma-separated BSA concentrations (µg/mL) matching the standard wells left to right. Must include a `0` for the blank. |
| **Parent strain** | Sample name to use as reference. Sets the dashed line on the box plot and the grey colour group. |

---

## How it works

1. **Blank correction** — the mean absorbance of all `0 µg/mL` standard wells is subtracted from every well.
2. **Linear regression** — a least-squares fit is performed on the blank-corrected absorbances vs. BSA concentrations (0 µg/mL excluded from fit).
3. **Interpolation** — each sample well concentration is calculated as `conc = slope × (abs − blank) + intercept`, then multiplied by the dilution factor.
4. **Summary statistics** — mean, SD, and CV are computed per sample across all replicate wells.
5. **Box plot** — per-well values are plotted as a vertical box plot (IQR box, median line, whiskers to min/max, individual replicate dots). Samples are ordered by natural sort with the parent strain first.

---

## Export

After running the analysis, two export formats are available:

- **CSV** — summary table (sample, N, mean, SD, CV)
- **Excel (.xlsx)** — three sheets: Summary (with equation and R²), Per-Well Data (every well with raw and corrected absorbance), Standard Curve (standards table + regression statistics)

---

## Browser compatibility

Works in any modern browser (Chrome, Firefox, Safari, Edge). No internet connection required after the page has loaded — all processing runs locally in JavaScript. External dependencies (Chart.js, SheetJS, PapaParse) are loaded from CDN on first open.

---

## Assay reference

Pierce™ BCA Protein Assay Kit — microplate procedure (25 µL sample + 200 µL working reagent, 37 °C for 30 min, read at 562 nm). See `MAN0011430` (Thermo Fisher) for full protocol.
