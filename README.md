# Oil Market Risk Analysis

A data analysis project that builds a monthly and daily oil market dataset from financial, macroeconomic, and physical market variables, then cleans, explores, and statistically characterizes crude oil price behavior. The project also builds a candidate belief network of price drivers and compares it against a structure learned directly from the data.

## Project Overview

This notebook assembles a multi-source dataset centered on WTI and Brent crude oil prices and examines the statistical properties of returns, the relationships between oil prices and other financial and macro variables, and the underlying structure connecting supply, demand, and price. The work is organized as the first stage of a larger risk management study, with the discretized regimes and candidate belief network here forming the basis for structure learning and inference in a follow up project.

## Data Sources

| Group                   | Variables                                                            | Source                     | Notes                                                                                              |
| ----------------------- | -------------------------------------------------------------------- | -------------------------- | -------------------------------------------------------------------------------------------------- |
| Financial               | WTI, Brent, Henry Hub natural gas, S&P 500, long term interest rate  | Yahoo Finance via yfinance | Local cache used as fallback if the live request fails                                             |
| Macro (genuine)         | US headline CPI                                                      | FRED                       | Local cache used as fallback if the live request fails                                             |
| Macro (proxy)           | Industrial production, geopolitical risk                             | Seeded proxy series        | Not freely available at this granularity, reconstructed and documented                             |
| Physical market (proxy) | OPEC and non OPEC supply, OECD and non OECD demand, OECD inventories | Seeded proxy series        | Anchored to real episodes: 2008 crisis, 2014 to 2016 glut, 2020 demand collapse, 2022 supply shock |

## Notebook Structure

| Section                            | Description                                                                                                                                                                                               |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Setup                              | Library imports, plotting configuration, and helper functions                                                                                                                                             |
| Data Import and Structuring        | Fetching live financial and CPI data, and generating proxy series for the remaining variables                                                                                                             |
| Data Dictionary                    | A summary table describing every variable, its unit, frequency, source, and coverage                                                                                                                      |
| Data Cleaning                      | Outlier screening on daily returns, duplicate and stale price checks, and gap handling for the monthly panel                                                                                              |
| Sterilized Dataset and Removal Log | A final cleaned dataset along with a log of every exclusion or imputation and the reason for it                                                                                                           |
| Exploratory Data Analysis          | Return distributions, price history with major events marked, rolling volatility, driver scatter plots, a correlation matrix, and rolling correlation with equities                                       |
| About Oil Price                    | Statistical tests on returns (autocorrelation, volatility clustering, normality, stationarity), a discretized regime frame, and both an expert defined and a data learned belief network of price drivers |
| Conclusion                         | A short summary of the main findings                                                                                                                                                                      |

## Requirements

The notebook relies on the following Python packages:

| Package         | Purpose                                            |
| --------------- | -------------------------------------------------- |
| numpy           | Numerical computing and array operations           |
| pandas          | Data structures and time series handling           |
| matplotlib      | Plotting                                           |
| seaborn         | Statistical plot styling                           |
| yfinance        | Live market data from Yahoo Finance                |
| scipy           | Statistical distributions and tests                |
| statsmodels     | Time series diagnostics (ACF, ADF, Ljung-Box)      |
| pgmpy           | Bayesian network structure learning                |
| networkx        | Graph construction and drawing for belief networks |
| python-dateutil | Date utilities                                     |

Install them with:

```
pip install numpy pandas matplotlib seaborn yfinance scipy statsmodels pgmpy networkx python-dateutil
```

## Usage

1. Clone this repository.
2. Install the required packages listed above.
3. Open Oil Market Risk Analysis.ipynb in Jupyter or a compatible environment.
4. Run all cells from top to bottom. If a live data request fails, the notebook falls back to a local cache stored under a data folder; without a live connection or a cache, the affected cell will raise an error.

## Notes

- Descriptions of what each step of the code is doing are printed to the notebook output rather than written as inline code comments.
- The physical market and select macro variables are reconstructed proxies, not official releases, and are clearly labeled as such in the data dictionary.
- The genuine sub zero WTI settlement from 2020 is preserved in the price history but excluded from the return series, since a non positive price breaks log returns.

## License

This project is licensed under the MIT License
