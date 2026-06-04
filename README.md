# BCA Protein App

<a href="https://tanderes.github.io/bca-protein-app/" target="_blank" rel="noopener">
  <img src="https://img.shields.io/badge/Launch%20App-tanderes.github.io-4f8ef7?style=for-the-badge&logo=github" alt="Launch App"/>
</a>

<a href="https://tanderes.github.io/bca-protein-app/" target="_blank" rel="noopener"><strong>Open the app in a new tab →</strong></a>

A browser-based tool for analysing Pierce BCA microplate protein assays. Upload two CSVs — absorbance readings and a plate layout map — and the app fits a BSA standard curve, interpolates protein concentrations for every sample well, and renders an interactive box plot. No server, no installation, no data leaves your machine.

---

## Quick start

1. <a href="https://tanderes.github.io/bca-protein-app/" target="_blank" rel="noopener"><strong>Open the app →</strong></a>
2. Upload your **absorbance CSV** and **plate layout CSV**
3. Set your wavelength, dilution factor, and reference sample in the config bar
4. Click **Run analysis**

---

## Input file format

Both files must be CSV with **rows A–H** in the first column and **columns 1–12** in the header row — matching a standard 96-well plate grid.

### Absorbance file
Raw absorbance readings from the plate reader, one value per well.

```
,1,2,3,4,5,6,7,8,9,10,11,12
A,0.872,0.851,0.863,0.748,0.731,...
B,0.868,0.879,0.855,0.742,0.761,...
...
H,2.104,1.831,1.274,1.038,0.762,...
```

### Plate layout file
Sample name for each well. Standard curve wells must be labelled `STD`. Empty wells can be left blank. Sample names containing commas should be wrapped in double quotes.

```
,1,2,3,4,5,6,7,8,9,10,11,12
A,SampleA,SampleB,SampleC,SampleD,SampleE,SampleF,SampleG,SampleH,SampleI,SampleJ,WT,WT
B,SampleA,SampleB,SampleC,...
...
G,Ctrl1,Ctrl2,Ctrl3,Ctrl4,...
H,STD,STD,STD,STD,STD,STD,STD,STD,STD,STD,STD,STD
```

> **Standard curve row:** By default the app reads the standard curve from **row H** — the last row of the plate. Change this using the **Std row** dropdown in the config bar if your plate is set up differently.

---

## Template files

Generic template files are provided to help you format your own data correctly. Download them from the app's About panel or directly below.

| File | Description |
|------|-------------|
| `BCA_absorbance_template.csv` | Example absorbance readings — rows A–G as samples, row H as BSA standard curve |
| `BCA_layout_template.csv` | Example plate layout — generic sample names, controls in row G, STD in row H |

The templates use `WT` as the reference sample (cols 11–12) — type `WT` in the **Reference sample** field to see the dashed reference line on the box plot.

---

## Config options

| Setting | Description |
|---------|-------------|
| **Wavelength** | Plate reader measurement wavelength (nm). Default 526. |
| **Dilution factor** | Multiplier applied to all interpolated concentrations. Use `1` for undiluted samples. |
| **Std row** | Which plate row contains the BSA standards. Default `H`. |
| **Std concentrations** | Comma-separated BSA concentrations (µg/mL) left to right. Must include a `0` for the blank. Default: `2000,1500,1000,750,500,250,125,25,0`. |
| **Reference sample** | Sample name to use as reference. Sets the dashed line on the box plot and the grey colour group. |
| **CV flag (%)** | Samples with CV above this threshold are flagged in amber. Default 10%. |

---

## How it works

1. **Blank correction** — the mean absorbance of all `0 µg/mL` standard wells is subtracted from every well.
2. **Linear regression** — a least-squares fit is performed on the blank-corrected absorbances vs. BSA concentrations (0 µg/mL excluded from fit).
3. **Interpolation** — each sample well concentration is calculated as `conc = slope × (abs − blank) + intercept`, then multiplied by the dilution factor.
4. **Summary statistics** — mean, SD, and CV are computed per sample across all replicate wells.
5. **Box plot** — per-well values plotted as a vertical box plot (IQR box, median line, whiskers to min/max, individual replicate dots). Samples are ordered by natural sort with the reference sample first.

---

## Export

| Format | Contents |
|--------|----------|
| **CSV** | Summary table — sample, N, mean, SD, CV, flag |
| **Excel (.xlsx)** | Three sheets: Summary (equation + R²), Per-Well Data (raw and corrected absorbance per well), Standard Curve (standards table + regression statistics) |
| **PNG** | Box plot downloaded at 2× resolution |
| **Copy table** | Summary copied as tab-separated values — pastes directly into Excel or Google Sheets |

---

## Flexible file parsing

The app handles a range of layout file formats:

- Sample names containing commas (e.g. `"SampleA,rep1"`) — wrap in double quotes
- Numeric row labels (1–8 mapped to A–H automatically)
- Extra header or blank rows — skipped automatically
- Rows in any order
- Extra columns beyond 12 — ignored
- Leading/trailing whitespace in cell values — trimmed

---

## Browser compatibility

Works in any modern browser (Chrome, Firefox, Safari, Edge). No internet connection required after the page has loaded — all processing runs locally in JavaScript. External dependencies (Chart.js, SheetJS, PapaParse) are loaded from CDN on first open.

---

## Assay reference

Pierce™ BCA Protein Assay Kit — microplate procedure (25 µL sample + 200 µL working reagent, 37 °C for 30 min, read at 562 nm). See `MAN0011430` (Thermo Fisher) for full protocol.
