# Brazilian Real Estate Market Analysis

Data analysis of Brazil's housing market using Python — exploring regional price differences, property size distributions, and the relationship between property size and price across 22,000+ listings.

---

## Overview

This project combines and cleans two real estate datasets covering properties across Brazil, then uses exploratory data analysis (EDA) and visualizations to surface patterns in the housing market. A focused deep-dive into the Southern region quantifies how strongly property size drives price in each of the three southern states.

**Dataset size:** ~22,844 records after cleaning  
**Price range:** $74,892 – $525,659 USD  
**Average property size:** 115 m²

---

## Key Findings

- Housing prices vary significantly by region — the South and Southeast command higher average prices than the North and Northeast
- Price distributions are right-skewed: most properties cluster in a moderate range, with a long tail of high-value listings
- Property size (area m²) has a moderate positive correlation with price across all southern states:
  - Rio Grande do Sul: **r = 0.577**
  - Paraná: **r = 0.544**
  - Santa Catarina: **r = 0.507**
- Location (region and state) is at least as important a pricing driver as property size

---

## Project Structure

```
brazilian-real-estate-analysis/
├── README.md
├── requirements.txt
├── data/
│   ├── brasil-real-estate-1.csv   # 12,834 records (price in USD)
│   └── brasil-real-estate-2.csv   # 10,010 records (price in BRL → converted)
├── notebook/
│   └── notebook.ipynb             # Full analysis notebook
└── images/
    ├── Mean_Price_by_Region_plot.png
    └── Price_Distribution_plot.png
```

---

## Visualizations

**Price Distribution** — right-skewed histogram showing most properties in the $100k–$300k range  
**Property Size Distribution** — box plot of area in m²  
**Mean Price by Region** — bar chart comparing average prices across Brazil's five regions  
**Property Location Map** — interactive scatter map (Plotly) with colour = price, size = area m²  
**Price vs Area (Rio Grande do Sul)** — scatter plot for the most-represented southern state

---

## Tech Stack

| Library | Purpose |
|---|---|
| pandas | Data loading, cleaning, transformation |
| matplotlib | Static charts (histogram, box plot, bar chart, scatter) |
| plotly | Interactive geographic map |

---

## Setup

**1. Clone the repo**
```bash
git clone https://github.com/Nathi247/brazilian-real-estate-analysis.git
cd brazilian-real-estate-analysis
```

**2. Install dependencies**
```bash
pip install -r requirements.txt
```

**3. Launch the notebook**
```bash
jupyter notebook brazilian-real-estate-analysis/notebook/notebook.ipynb
```

---

## Data Notes

- Dataset 1 (`brasil-real-estate-1.csv`): prices already in USD; lat/lon stored as a combined `lat-lon` column
- Dataset 2 (`brasil-real-estate-2.csv`): prices in BRL, converted at a fixed rate of 1 USD = 3.19 BRL
- Rows with missing coordinates were dropped (~1,283 records from dataset 1)
- Both datasets were concatenated into a single DataFrame for combined analysis

---

## Interactive Version

Full interactive notebook on DataCamp DataLab:  
https://www.datacamp.com/datalab/w/6b4f715c-7b02-47a6-b589-320fcd087b26/edit

---

## Author

**Nkosinathi Nduli**  
BSc Applied Mathematics & Applied Statistics  
Aspiring Data Analyst
