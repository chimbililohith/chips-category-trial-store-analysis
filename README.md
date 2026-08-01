# Chips Category Review & Trial Store Analysis

A retail analytics case study for a supermarket chain, examining the **Chips (Snack Foods)** category and evaluating the impact of a **new store layout trial** on sales performance.

This project mirrors a real-world category manager / retail analytics workflow: clean and explore transactional data, profile customer segments, then run an experimental analysis to measure the uplift from a trial store layout change.

## Project Overview

The analysis is split into two parts:

### 1. Chips Category Review (`notebooks/01_category_analysis.ipynb`)
- Cleaned and prepared ~264K transaction records and ~72K customer records
- Engineered features: pack size and brand name extracted from product descriptions
- Removed non-chip products (e.g. salsa) and an outlier commercial-purchase customer
- Analyzed weekly transaction trends, finding a sharp spike in the week before Christmas
- Segmented customers by **lifestage** and **premium/mainstream/budget** status to identify:
  - Who buys the most chips (Mainstream Young Singles/Couples, Mainstream Retirees)
  - Who buys the most per transaction (Older & Young Families)
  - Who pays the most per unit, with statistical significance confirmed via a t-test (p ≈ 3.48e-306)
- Used affinity/likelihood analysis to find brand and pack-size preferences by segment

### 2. Trial Store Analysis (`notebooks/02_trial_store_analysis.ipynb`)
- Identified the best-matching **control store** for each of 3 trial stores (77, 86, 88) using correlation of sales and customer count trends
- Matches found: Store 77 → 233, Store 86 → 155, Store 88 → 237
- Compared trial vs. control store performance before and after the layout change
- Found that trial stores significantly outperformed their control stores during the trial period (Feb–Apr), confirming the new layout drove a real uplift in sales

### Presentation
`reports/chips_category_review_presentation.pdf` — executive summary deck presenting key findings and recommendations to stakeholders (holiday promotional displays, targeting Young/Older Families for basket growth, rollout of the new store layout).

## Repository Structure

```
.
├── data/
│   ├── QVI_transaction_data.csv       # Raw transaction-level data
│   └── QVI_purchase_behaviour.csv     # Customer lifestage & affluence segments
│   └── merged_chip_data.csv           # Cleaned and Merged Transaction and Purchase data
├── notebooks/
│   ├── 01_category_analysis.ipynb     # Data cleaning, EDA, segment profiling
│   └── 02_trial_store_analysis.ipynb  # Control store selection & uplift testing
├── reports/
│   └── chips_category_review_presentation.pdf
└── README.md
```

## Key Findings

- **Seasonality**: Chip transactions are stable year-round with a clear spike the week before Christmas — an opportunity for promotional displays/gondola ends.
- **Core shopper**: Mainstream Young Singles & Couples are the largest chip-buying segment.
- **Growth opportunity**: Young and Older Families (26% of chip shoppers) purchase larger baskets on average, offering room to grow spend per customer.
- **Trial success**: The new store layout produced a measurable sales uplift in trial stores relative to matched control stores from February through April.

## Tech Stack

- Python (pandas, numpy)
- matplotlib, seaborn for visualization
- scipy for statistical testing
- Jupyter notebooks (originally built in Google Colab)
