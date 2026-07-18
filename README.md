# Crude Oil Risk Modeling

A two part data analysis project on crude oil price risk. Part 1 builds a monthly and daily oil market dataset from financial, macroeconomic, and physical market variables, then cleans, explores, and statistically characterizes crude oil price behavior. Part 2 detects bull, bear, and stagnant regimes in that same dataset using hidden Markov models built from scratch, then learns a belief network structure linking those regimes to WTI crude oil price movements.

## Project Overview

Both notebooks share the same underlying dataset and data loading approach, so Part 2 can be read as a direct continuation of Part 1 rather than a separate project. Part 1 focuses on assembling the dataset and understanding its statistical properties. Part 2 focuses on regime detection and structure learning built on top of that dataset.

## Notebooks

| Notebook                 | Focus                                                                                                |
| ------------------------ | ---------------------------------------------------------------------------------------------------- |
| 01_risk_management.ipynb | Data assembly, cleaning, exploratory analysis, and statistical characterization of oil price returns |
| 02_belief_network.ipynb  | Hidden Markov model implementation, regime detection, and belief network structure learning          |

## Data Sources

| Group                   | Variables                                                            | Source                     | Notes                                                                                              |
| ----------------------- | -------------------------------------------------------------------- | -------------------------- | -------------------------------------------------------------------------------------------------- |
| Financial               | WTI, Brent, Henry Hub natural gas, S&P 500, long term interest rate  | Yahoo Finance via yfinance | Local cache used as fallback if the live request fails                                             |
| Macro (genuine)         | US headline CPI                                                      | FRED                       | Local cache used as fallback if the live request fails                                             |
| Macro (proxy)           | Industrial production, geopolitical risk                             | Seeded proxy series        | Not freely available at this granularity, reconstructed and documented                             |
| Physical market (proxy) | OPEC and non OPEC supply, OECD and non OECD demand, OECD inventories | Seeded proxy series        | Anchored to real episodes: 2008 crisis, 2014 to 2016 glut, 2020 demand collapse, 2022 supply shock |

Both notebooks fetch this data independently using the same live fetch with local cache fallback approach, so either notebook can run on its own once the data folder with cached fallbacks is in place.

## Notebook Structure

### Part 1: risk_management.ipynb

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

### Part 2: belief_network.ipynb

| Section                                               | Description                                                                                          |
| ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Environment Setup                                     | Library imports, plotting configuration, and helper functions                                        |
| Discrete Hidden Markov Model Class                    | A from scratch implementation of forward, backward, Viterbi, and Baum-Welch                          |
| Forward and Backward Algorithm Toy Example            | Validating the forward and backward pass on a known two state weather example                        |
| Viterbi Algorithm Toy Example                         | Validating decoded state paths against a brute force search                                          |
| Baum-Welch Algorithm Toy Example                      | Recovering known generating parameters from simulated sequences                                      |
| Data Import and Structuring                           | Fetching live financial and CPI data with cache fallback, and building the proxy and master datasets |
| Bull, Bear, and Stagnant Regime Illustration          | A motivating visual example of the three regimes on the WTI price history                            |
| Transforming the Time Series into Emission Sequences  | Converting each variable into an up and down emission sequence                                       |
| Learning Regime Parameters with Baum-Welch            | Training a three state HMM per variable                                                              |
| Decoding Regimes with Viterbi                         | Decoding the most likely regime path per variable                                                    |
| Multicolored Regime Plots                             | Visualizing decoded regimes against the underlying series                                            |
| Identifying the Latent Meaning of Each Hidden State   | Labeling each hidden state by its mean change and time spent in state                                |
| Train, Validation, and Test Split                     | A chronological 80:10:10 split of the regime data                                                    |
| Minimal Working Example of Hill Climb Search          | A small synthetic demonstration of structure learning before applying it to real data                |
| Training and Validating the Oil-Market Belief Network | Learning and fitting the belief network, then evaluating prediction accuracy                         |
| Conclusion                                            | A short summary of the main findings                                                                 |

## Requirements

| Package         | Purpose                                            | Used in |
| --------------- | -------------------------------------------------- | ------- |
| numpy           | Numerical computing and array operations           | Both    |
| pandas          | Data structures and time series handling           | Both    |
| matplotlib      | Plotting                                           | Both    |
| seaborn         | Statistical plot styling                           | Part 1  |
| yfinance        | Live market data from Yahoo Finance                | Both    |
| scipy           | Statistical distributions and tests                | Part 1  |
| statsmodels     | Time series diagnostics (ACF, ADF, Ljung-Box)      | Part 1  |
| pgmpy           | Belief network structure learning and inference    | Both    |
| networkx        | Graph construction and drawing for belief networks | Both    |
| python-dateutil | Date utilities                                     | Both    |

Install them with:

```
pip install numpy pandas matplotlib seaborn yfinance scipy statsmodels pgmpy networkx python-dateutil
```

## Usage

1. Clone this repository.
2. Install the required packages listed above.
3. Open 01_risk_management.ipynb in Jupyter or a compatible environment and run all cells top to bottom.
4. Open 02_belief_network.ipynb and run all cells top to bottom. It rebuilds the dataset independently, so it does not require having run Part 1 first, though reading Part 1 first gives useful context on the data and its statistical properties.
5. If a live data request fails in either notebook, it falls back to a local cache stored under a data folder; without a live connection or a cache, the affected cell will raise an error.

## Notes

- Descriptions of what each step of the code is doing are printed to the notebook output rather than written as inline code comments.
- The physical market and select macro variables are reconstructed proxies, not official releases, and are clearly labeled as such in the Part 1 data dictionary.
- The genuine sub zero WTI settlement from 2020 is preserved in the price history but excluded from the return series, since a non positive price breaks log returns.
- The Part 2 hidden Markov model implementation is a direct NumPy reimplementation rather than a third party HMM library, since the reference library used in the original coursework does not build in current environments.
- Brent and the WTI-Brent spread are deliberately excluded from the belief network's candidate variables in Part 2, since Brent moves in near lockstep with WTI and would let the network predict WTI from a twin of itself rather than from genuine macro and physical market drivers.
- Validation and test error rates in Part 2 should be read against the chance level baseline for three classes.

## License

Add a license of your choice for this repository.
