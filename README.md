# Chips Category Review & Trial Store Analysis

A retail analytics case study for a supermarket chain, examining the **Chips (Snack Foods)** category and evaluating the impact of a **new store layout trial** on sales performance.

This project mirrors a real-world category manager / retail analytics workflow: clean and explore transactional data, profile customer segments, then run an experimental analysis to measure the uplift from a trial store layout change.

## The Data

Two raw datasets from a supermarket chain's loyalty program, covering **1 July 2018 – 30 June 2019**:

### `data/QVI_transaction_data.csv`
Transaction-level scan data for the **Chips / Snack Foods** category.

| Column | Description |
|---|---|
| `DATE` | Date of transaction (stored as an Excel serial integer in the raw file — converted to a proper datetime in the notebook) |
| `STORE_NBR` | Store identifier |
| `LYLTY_CARD_NBR` | Customer loyalty card number (joins to the behaviour dataset) |
| `TXN_ID` | Transaction identifier |
| `PROD_NBR` | Product identifier |
| `PROD_NAME` | Product description, e.g. `"Smiths Crinkle Cut Chips Chicken 170g"` |
| `PROD_QTY` | Units purchased in that transaction |
| `TOT_SALES` | Total dollar value of that line item |

- **264,836 transaction rows**, **272 stores**, **114 distinct products**
- No missing values, but the raw data included non-chip items (e.g. salsa/dip products misfiled under the category) and needed pack size / brand extracted from free-text product names

### `data/QVI_purchase_behaviour.csv`
Customer segment lookup, one row per loyalty card.

| Column | Description |
|---|---|
| `LYLTY_CARD_NBR` | Customer loyalty card number |
| `LIFESTAGE` | One of 7 life-stage segments: `NEW FAMILIES`, `YOUNG FAMILIES`, `OLDER FAMILIES`, `YOUNG SINGLES/COUPLES`, `MIDAGE SINGLES/COUPLES`, `OLDER SINGLES/COUPLES`, `RETIREES` |
| `PREMIUM_CUSTOMER` | Affluence tier: `Budget`, `Mainstream`, or `Premium` |

- **72,637 customers**, every one of which appears in the transaction data (a clean 1:1 join with no orphaned loyalty numbers)
- Together, `LIFESTAGE × PREMIUM_CUSTOMER` gives 21 customer segments used throughout the analysis

### `data/merged_chip_data.csv` *(generated, not raw)*
Produced by `01_category_analysis.ipynb` by joining the two datasets above on `LYLTY_CARD_NBR`, after cleaning. This is the dataset the trial store analysis is built on.

## Analysis & Findings

The analysis is split into two notebooks, run in order.

### 1. Chips Category Review (`notebooks/01_category_analysis.ipynb`)

**Data cleaning & feature engineering**
- Converted `DATE` from Excel serial integer to a proper datetime
- Inspected product name word frequencies and removed non-chip products (salsa/dip) that were miscategorized
- Found two abnormally large transactions (200 units each) from a single loyalty card with only those two transactions all year — identified as a likely commercial buyer rather than a retail customer, and excluded from further analysis
- Engineered `PACK_SIZE` (extracted the gram weight from the product name, ranging 70g–380g, centered around 175g) and `BRAND_NAME` (extracted from the first word of the product name)
- Deduplicated inconsistent brand naming (e.g. multiple spellings/abbreviations mapping to the same brand) down to **20 clean, unique brands**
- Verified the customer behaviour table had no missing values, then merged it with transactions on `LYLTY_CARD_NBR` — the merged row count confirmed every transaction matched to a known customer

**Exploratory analysis**
- Plotted transactions per day across the year: volume is stable, but spikes sharply in the days before Christmas (Dec 23–24 are the two highest-volume days), and Dec 25 shows zero transactions due to store closures on the public holiday

**Segment-level analysis (by Lifestage × Premium/Mainstream/Budget)**
- **Total sales by segment**: driven mainly by Budget–Older Families, Mainstream–Young Singles/Couples, and Mainstream–Retirees
- **Number of customers by segment**: Mainstream–Young Singles/Couples and Mainstream–Retirees have the largest customer counts — explaining most of their sales contribution. Budget–Older Families' high sales, however, are *not* mainly explained by customer count
- **Average units per transaction**: Older and Young Families buy noticeably more chips per basket than other segments — a signal of larger overall grocery baskets, not just chip-specific preference
- **Average price per unit**: Mainstream Young and Midage Singles/Couples pay more per pack than their Budget and Premium counterparts. A two-sample t-test confirmed this difference is statistically significant (**p ≈ 3.48 × 10⁻³⁰⁶**) — likely reflecting impulse buying rather than deliberate value-shopping, since Premium shoppers in the same life stages buy chips less often (favoring healthier snacks) and treat chips more as an occasional/entertainment purchase

**Affinity analysis (deep dive on Mainstream Young Singles/Couples, the top segment)**
- **23% more likely** to buy Tyrrells chips than the rest of the population
- **56% less likely** to buy Burger Rings than the rest of the population
- **27% more likely** to buy 270g packs — but investigation showed only one brand (Twisties) sells that pack size, so this is really a Twisties preference showing up as a pack-size signal

**Conclusions**
- Sales are concentrated in three segments: Budget–Older Families, Mainstream–Young Singles/Couples, and Mainstream–Retirees
- High spend from Mainstream young/midage singles & couples is driven by both larger customer counts *and* a willingness to pay more per pack — consistent with impulse-driven purchasing
- **Recommendation**: off-locate Tyrrells and smaller chip packs in discretionary/high-traffic space frequented by young singles and couples to capture more impulse purchases

### 2. Trial Store Analysis (`notebooks/02_trial_store_analysis.ipynb`)

Three stores (77, 86, 88) trialed a new store layout. The goal: measure whether the layout drove a real sales uplift, using a **matched control store** methodology (a standard causal-inference approach for retail A/B-style tests where randomization isn't possible).

**Control store selection**
- For each trial store, every other store was scored on how closely its pre-trial monthly trends (total sales and total customers) matched the trial store — combining a **correlation score** with a **standardized magnitude-of-difference score**
- Best matches: **Store 77 → Control 233**, **Store 86 → Control 155**, **Store 88 → Control 237**
- Verified visually that each control store's pre-trial trend lines closely tracked its trial store before drawing any conclusions

**Uplift testing methodology**
- Scaled the control store's sales to the same baseline level as the trial store (correcting for any pre-existing scale differences between the two stores)
- Compared trial-period performance (Feb–Apr) against the control store's expected range, using a **95% confidence interval** and a **t-test** against the t-distribution to check statistical significance

**Findings by store**
- **Store 77**: Sales significantly higher than its control store in 2 of 3 trial months (t-value exceeded the 95th-percentile threshold) — a real, statistically supported uplift
- **Store 86**: Sales were *not* significantly different from its control store, despite customer counts being significantly higher in all 3 trial months. This mismatch (more customers, no matching sales lift) suggests the layout may have driven foot traffic without lifting basket value — possibly due to promotional pricing in the trial store that needs to be confirmed with the Category Manager
- **Store 88**: Sales significantly higher than its control store in 2 of 3 trial months, and customer counts also significantly higher in 2 of 3 months — a clear positive result

**Conclusion**
- 2 of the 3 trial stores (77 and 88) showed a statistically significant sales uplift from the new layout
- Store 86 is the exception and warrants follow-up with the client to understand implementation differences
- **Overall, the trial supports a positive case for rolling out the new store layout**, with a note to investigate what happened differently at store 86

### Presentation
`reports/chips_category_review_presentation.pdf` — the executive summary deck built from the above findings, presenting recommendations to the Category Manager: promotional displays ahead of the Christmas peak, targeting Young/Older Families to grow basket size, and rolling out the new store layout following its measured uplift.

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
