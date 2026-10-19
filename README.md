# Determinants of Inventory Turnover in U.S. Consumer Retail (2011–2024)

An econometric study of whether the classical drivers of inventory turnover — **gross profit margin, capital intensity, and sales growth** — still hold in the contemporary U.S. consumer retail sector. The analysis revisits the log-log panel framework of Gaur, Fisher & Raman (2005) with recent data and dynamic panel methods.

![R](https://img.shields.io/badge/R-276DC3?logo=r&logoColor=white)
![Panel Data](https://img.shields.io/badge/method-panel%20econometrics-blue)
![Data](https://img.shields.io/badge/data-WRDS%20Compustat-lightgrey)

---

## Research Question

Have the relationships between inventory turnover and its classical financial determinants survived a decade of structural change in retail (e-commerce, omnichannel fulfillment, asset-light strategies)? We expect the relationships to remain significant but anticipate shifts in magnitude and direction, especially for capital intensity.

## Data

| Item | Detail |
|---|---|
| Source | Wharton Research Data Services (WRDS), annual financial statements |
| Sample | 51 publicly traded U.S. consumer retailers, selected by simple random sampling |
| Industries | 14 SIC codes (5311, 5399, 5531, 5600, 5621, 5651, 5661, 5700, 5731, 5734, 5900, 5940, 5944, 5945) |
| Period | Fiscal years 2011–2024 (unbalanced panel, T = 7–14 per firm) |
| Observations | ~580 firm-years |

> **Note:** The cleaned dataset (`df_cleaned2.csv`) is derived from WRDS and is not redistributed here due to licensing terms. Users with WRDS access can reconstruct it from Compustat annual fundamentals using the variable definitions below.

## Variables

All variables enter the model in natural logs.

| Variable | Definition |
|---|---|
| Inventory Turnover (DV) | COGS / Average Inventory, where Average Inventory = (Beginning + Ending) / 2 |
| Gross Profit Margin | (Revenue − COGS) / Revenue |
| Capital Intensity | Working Assets / Total Assets (linear + quadratic term) |
| Sales Growth | Revenue<sub>t</sub> / Revenue<sub>t−1</sub> |
| Annual Sales | Revenue (proxy for firm size / economies of scale) |
| Lagged Turnover | ln(Inventory Turnover)<sub>t−1</sub>, controls for serial correlation |

## Methodology

**Main specification — two-way fixed effects dynamic panel model:**

$$
\ln ITO_{it} = \gamma_1 \ln GM_{it} + \gamma_2 \ln CI_{it} + \gamma_5 (\ln CI_{it})^2 + \gamma_3 \ln SG_{it} + \gamma_4 \ln S_{it} + \phi_1 \ln ITO_{i,t-1} + \delta_t + c_i + \varepsilon_{it}
$$

**Robustness / comparison models:**
- Pooled OLS
- Random effects (Wallace–Hussain)
- Prais–Winsten with AR(1) errors and panel-heteroskedastic SEs (`panelAR`)

**Diagnostics:**
- Maddala–Wu panel unit-root tests (lag order by AIC) — all variables stationary (p < 0.0001)
- Panel Breusch–Godfrey test — residual serial correlation detected
- Breusch–Pagan test — heteroskedasticity detected → **cluster-robust standard errors** (HC1, clustered by firm)
- Hausman test — rejects RE (p < 0.0001) → fixed effects preferred
- Variance inflation factors — no severe multicollinearity
- AIC model comparison — FE model has the lowest AIC

## Key Findings

Coefficients from the two-way FE model with clustered SEs (log-log, so coefficients are elasticities):

| Determinant | Coefficient | p-value | Interpretation |
|---|---|---|---|
| ln(Capital Intensity) | −0.422 | < 0.0001 | Concave, non-linear relationship |
| ln(Capital Intensity)² | −0.104 | < 0.0001 | Turnover peaks at CI ≈ 0.13, then declines |
| ln(Sales Growth) | 0.213 | 0.025 | Faster growth → faster turnover |
| ln(Gross Profit Margin) | −0.323 | 0.023 | Earns-turns trade-off confirmed |
| ln(Annual Sales) | 0.069 | 0.044 | Larger firms turn inventory faster |

**Takeaways**
1. **Capital intensity** departs from earlier literature: instead of a positive linear effect, it shows a concave relationship — consistent with asset-light retailers being better positioned, though it may also reflect sample composition and the 2019 ASC 842 lease accounting change, which inflated reported assets.
2. **Sales growth** remains a positive driver, suggesting retailers often lack the capacity to fully capture demand upswings.
3. **Gross margin** retains its negative earns-turns relationship (Gaur et al., 2005).
4. **Firm size** has a positive effect on turnover, consistent with economies of scale.

Coefficient directions are consistent across FE, RE, pooled, and Prais–Winsten specifications.

## Repository Structure

```
.
├── Determinant of Inventory Turnover.Rmd   # Full analysis and write-up
├── df_cleaned2.csv                         # Cleaned panel data (not included — see Data)
└── README.md
```

## Reproducing the Analysis

1. Install R (≥ 4.2) and the required packages:

   ```r
   install.packages(c(
     "lubridate", "dplyr", "plm", "car", "DescTools", "panelAR",
     "sandwich", "lmtest", "readr", "panelView", "ggplot2",
     "corrplot", "patchwork", "knitr", "stargazer"
   ))
   ```

   > `panelAR` has been archived on CRAN; install it from the archive if needed:
   > `install.packages("https://cran.r-project.org/src/contrib/Archive/panelAR/panelAR_0.1.tar.gz", repos = NULL, type = "source")`

2. Place `df_cleaned2.csv` in the project root. Required columns include `tic`, `fyear`, `gvkey`, `year`, `conm`, `sic`, `at`, `invt`, and the log-transformed variables (`ln_invt_turnover`, `ln_cap_int`, `ln_sale_growth`, `ln_gpm`, `ln_sale`).

3. Knit the R Markdown file to PDF (requires a LaTeX distribution such as TinyTeX):

   ```r
   rmarkdown::render("Determinant of Inventory Turnover.Rmd")
   ```

## Limitations

- **Dynamic panel (Nickell) bias:** including a lagged DV in a fixed effects model can bias the autoregressive coefficient downward when T is small. A GMM estimator (Arellano–Bond / Blundell–Bond) is a natural extension.
- **Unbalanced panel:** firms with longer histories carry more weight in estimation.
- **Sample scope:** only publicly traded firms; small, private, and distressed retailers are absent.
- **Variable selection:** domain-driven forward selection; LASSO or dimension reduction could surface less intuitive predictors.
- **Volatility clustering:** possible ARCH effects in financial data may violate Prais–Winsten assumptions.

## References

- Breivik, J., Larsen, N., Thyholdt, S., & Myrland, Ø. (2021). Measuring inventory turnover efficiency using stochastic frontier analysis. *International Journal of Systems Science: Operations & Logistics.* https://doi.org/10.1080/23302674.2021.1964635
- Financial Accounting Standards Board. (2016). *ASU 2016-02 — Leases (Topic 842).*
- Gaur, V., Fisher, M., & Raman, A. (2005). An econometric analysis of inventory turnover performance in retail services. *Management Science.*
- Gaur, V., & Kesavan, S. (2007). The effects of firm size and sales growth rate on inventory turnover performance in the U.S. retail sector. Springer.
- Grubor, A., Milicevic, N., & Mijic, K. (2013). Empirical analysis of inventory turnover ratio in FMCG retail sector — evidence from the Republic of Serbia. *Engineering Economics.* https://doi.org/10.5755/j01.ee.24.5.3546
- Yousaf, M., & Dehning, B. (2023). The effects of sales surprise on inventory turnover: An empirical study. *Cogent Economics & Finance.* https://doi.org/10.1080/23322039.2023.2258696

## Author

**Chi Hou (Bob) Lok** — M.S. Applied Economics and Management, Cornell University
<!-- Add co-authors here if this was a team project -->
