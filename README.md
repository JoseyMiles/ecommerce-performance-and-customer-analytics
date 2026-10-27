# E-commerce Business Intelligence: Insights, Forecasts & Strategic Recommendations

An end-to-end analytics project on 25,000 e-commerce orders (Jan 2020 to 30 Mar 2026). Python handles the cleaning, validation and statistical testing. Power BI delivers a 7-page dashboard on a star-schema model.

The project asks business questions and tests every answer before reporting it. Most tested factors showed **no measurable effect**, and the report says so.

![Executive Summary page](https://github.com/JoseyMiles/ecommerce-performance-and-customer-analytics/blob/a1ab35fd1d7ae64e0bebb7b15b49e59e1df40db1/Executive%20Summary.png)

## Headline results

| Question | Answer | Evidence |
|---|---|---|
| Is revenue growing? | No. It is flat at about **$34.5k a month** ($396k to $425k a year) | Trend p = 0.15 |
| Is there seasonality or a weekday pattern? | No | p = 0.98 and p = 0.76 |
| Who churns? (8.9% of customers) | Customers with fewer orders, lower spend and fewer reviews. Not country, channel, device, payment method, gender or age | Adjusted p < 0.01 for orders, spend and reviews |
| Where is the customer value? | **At Risk** customers are 24% of the base but hold **35%** of lifetime spend. The top 20% of customers hold 61% | Customers-table RFM segmentation |
| Do discounts pay off? | No visible lift. They cost **$149k (5.45% of sales)** and units per order stay at about 1.7 | p = 0.26 for quantity |
| Can we predict each customer's future spend? | No. Ranking is no better than chance, but the **total** is predicted well (93% of actual) | 12-month hold-out |
| Which channel, category, product or delivery speed is better? | No significant difference found | Channel p = 0.46, delivery p = 0.79 |

## Business questions and methods

| # | Analysis | Business question | Methods |
|---|---|---|---|
| 0 | Validation | Can the numbers be trusted? | Arithmetic checks, key checks, reconciliation to summary tables |
| 1 | Revenue | Where does growth come from: volume or basket size? | Volume/basket decomposition, trend test, Kruskal-Wallis |
| 2 | Churn | Who leaves and why? | Chi-square, Mann-Whitney, logistic regression, 5-fold cross-validation |
| 3 | RFM and CLV | Which customers are worth retaining? | Quintile RFM, BG/NBD, gradient boosting benchmark, 12-month hold-out |
| 4 | Channels | Which channel brings the best customers? | Kruskal-Wallis |
| 5 | Discounts | Do discounts create sales or give margin away? | Spearman, chi-square, cost per order by band |
| 6 | Returns | What drives returns and what do they cost? | Chi-square, logistic regression |
| 7 | Products | Which products carry the business? | Pareto / ABC, rating and return tests |
| 8 | Forecast | What revenue should we plan for? | Exponential smoothing variants against a naive benchmark, 6-month hold-out |
| 9 | Delivery | Does slow delivery hurt satisfaction? | Spearman, Kruskal-Wallis |
| 10 | Behaviour | Does browsing behaviour, device or payment affect order value? | Spearman, Kruskal-Wallis |
| 11 | Demographics | Do age, gender or country affect spend? | Kruskal-Wallis |

Because about 40 tests were run, p-values are corrected with the **Benjamini-Hochberg** method within each analysis. A result counts as real only if the adjusted p-value is below 0.05.

## Dashboard (Power BI)

| Page | Question it answers |
|---|---|
| 1. Executive Summary | How is the business doing? |
| 2. Revenue | Where does growth come from? |
| 3. Customers and CLV | Who are the most valuable customers? |
| 4. Retention and churn | Who leaves, and how much value is at risk? |
| 5. Products and discounts | Do discounts work, and which products matter? |
| 6. Returns and delivery | What do returns and slow delivery cost? |
| 7. Forecast and audit | Do the numbers reconcile, and what was tested? |

A PDF export is in [`powerbi/ecommerce_dashboard.pdf`](https://github.com/JoseyMiles/ecommerce-performance-and-customer-analytics/blob/198b738d1d61ce8f114ca2923101617d4dffef09/ecommerce_dashboard.pdf). The DAX measures are in [`powerbi/dax_measures.md`](powerbi/dax_measures.md).

**Model:** star schema with `fact_orders`, `dim_customer`, `dim_product` and a DAX `dim_date`.

| From | To | Active |
|---|---|---|
| `dim_date[Date]` | `fact_orders[order_date]` | Yes |
| `dim_date[Date]` | `fact_orders[delivery_date]` | No |
| `dim_date[Date]` | `dim_customer[first_order_date]` | No |
| `dim_customer[customer_id]` | `fact_orders[customer_id]` | Yes |
| `dim_product[product_key]` | `fact_orders[product_key]` | Yes |

`monthly_revenue`, `product_summary`, `forecast`, `stat_tests` and `data_quality_log` are loaded as unconnected audit and reference tables. An audit page reconciles the model to the official monthly table with a **difference of 0** in every month.

## Data

| Table | Rows | One row is | Used for |
|---|---|---|---|
| `orders` | 25,000 | an order | Sales, discounts, returns, delivery |
| `customers` | 8,000 | a customer | Attributes, churn flag, lifetime totals |
| `monthly_revenue` | 75 | a month | Reconciling the model |
| `product_summary` | 140 | a product | Reconciling the model |

**Source and licence:** `[add where the data comes from and its licence]`

## Data quality findings and decisions

Every decision is logged to `data_quality_log.csv`, which feeds the audit page.

| Finding | Decision |
|---|---|
| `monthly_revenue` did not match the orders at first | Tested combinations of order status, revenue column and date. It reconciles exactly to **Delivered orders, `total_amount_usd`**, so revenue is defined that way everywhere |
| `product_summary` uses a different rule | It reconciles to all **non-cancelled** orders ("booked"). The model carries both `Revenue` and `Booked Sales` |
| 86% of customers registered **after** their first order | Registration date is replaced by the first order date where it is later. The original is kept in `registration_date_original`, and every changed row is flagged |
| `customers` lifetime totals do not match `orders` (132,362 lifetime orders against 25,000 order rows) | The orders table looks like a slice of the full history. Revenue comes from `orders`, and customer segments and churn use the full-history `customers` table |
| `customer_rating` is blank on 63% of orders | Left as null (not rated), never filled with 0 |
| No cost or marketing spend data | No profit or ROI claims. The 30% margin used for CLV is an assumption |

## What did not work (and why it is in the report)

- **Customer-level CLV prediction.** A BG/NBD model and a gradient-boosting benchmark were tested on a hidden 12-month window. Ranking was no better than chance (Spearman 0.02, p = 0.13), because customers in this table behave almost identically. The earlier simple CLV estimate, which suggested large differences between segments, was discarded. The model **is** accurate in total, so it is used for planning, not for targeting individuals.
- **Complex forecasting.** Trend and seasonal models (15% error) did not beat a flat level (14.1%) or a simple 12-month average (13.4%), which matches the finding of no trend or seasonality.
- **Product quadrants.** Rating and return rate do not differ significantly between products, so the quadrant chart was dropped.

## Repository structure

```
.
├── README.md
├── requirements.txt
├── data/                       # input CSVs (see Data section)
├── notebooks/
│   └── ecommerce_analysis.ipynb   # cleaning, validation, analysis, export
├── powerbi_inputs/             # tables exported by the notebook for Power BI
├── powerbi/
│   ├── ecommerce_dashboard.pbix
│   ├── ecommerce_dashboard.pdf
│   └── dax_measures.md
└── docs/images/                # dashboard screenshots
```

## How to run

1. Install Python 3.10 or newer, then `pip install -r requirements.txt`.
2. Put the four CSV files in `data/`.
3. Open `notebooks/ecommerce_analysis.ipynb` and set `DATA` in the setup cell to your data folder.
4. Restart the kernel and run all cells from top to bottom. The last cell writes eight files to `powerbi_inputs/`.
5. In Power BI Desktop, load those files, build the relationships above, create `dim_date` and add the measures from `powerbi/dax_measures.md`. Or open the `.pbix` and refresh the data source path.

## Limitations

- The orders table appears to hold about 19% of the orders the customers table reports, so per-customer order histories are thin.
- Recency and the churn flag are measured at the same point in time, so churn findings show association, not cause.
- Discount results show whether discounts change basket size. They cannot show whether discounts bring extra visits. That needs a controlled experiment.
- Revenue includes tax and shipping.
- Many factors were tested and found to make no difference. This is a result, not a gap.

## Recommended actions (from the analysis)

1. Plan on flat revenue of about $34.5k a month (range $27.7k to $41.3k).
2. Run a win-back test on the At Risk group with a comparison group.
3. Test discounts in a controlled experiment before changing them.
4. Do not move marketing budget between channels on value grounds.
5. Treat returns as a company-wide issue, not a category issue.
6. Collect cost and acquisition spend data so profit and ROI can be measured.

## Skills demonstrated

Python (pandas, SciPy, statsmodels, scikit-learn), data validation and reconciliation, hypothesis testing with multiple-comparison correction, churn modelling, probabilistic CLV (BG/NBD), time-series forecasting, Power BI data modelling, DAX (time intelligence, reconciliation measures), dashboard design and written communication of results and limits.

## Author
`Ayodele Adeniyi` · `https://www.linkedin.com/in/the-ayodele-adeniyi/` · `deniyiayodele@gmail.com`
`
