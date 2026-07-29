# Crude Oil Risk Modeling

A single notebook that assembles and cleans an oil market dataset, detects bull, bear, and stagnant regimes with hidden Markov models built from scratch, learns a belief network linking those regimes to WTI price movements, then evaluates it with a robustness check, a replication comparison, and a simple trading simulation.

## Data Sources

| Group                   | Variables                                                            | Source                     | Notes                                                                                              |
| ----------------------- | -------------------------------------------------------------------- | -------------------------- | -------------------------------------------------------------------------------------------------- |
| Financial               | WTI, Brent, Henry Hub natural gas, S&P 500, long term interest rate  | Yahoo Finance via yfinance | Local cache used as fallback if the live request fails                                             |
| Macro (genuine)         | US headline CPI                                                      | FRED                       | Local cache used as fallback if the live request fails                                             |
| Macro (proxy)           | Industrial production, geopolitical risk                             | Seeded proxy series        | Not freely available at this granularity, reconstructed and documented                             |
| Physical market (proxy) | OPEC and non OPEC supply, OECD and non OECD demand, OECD inventories | Seeded proxy series        | Anchored to real episodes: 2008 crisis, 2014 to 2016 glut, 2020 demand collapse, 2022 supply shock |

The data is fetched once, near the top of the notebook, using a live fetch with local cache fallback. Every later section reuses that same dataset directly from memory.

## Notebook Structure

| Section                                                              | Description                                                                                                                                                                                                 |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Environment Setup                                                    | Library imports, plotting configuration, and shared helper functions used throughout the notebook                                                                                                           |
| Data Import and Structuring                                          | Fetching live financial and CPI data with cache fallback, and building the proxy and master datasets                                                                                                        |
| Data Dictionary                                                      | A summary table describing every variable, its unit, frequency, source, and coverage                                                                                                                        |
| Data Cleaning                                                        | Outlier screening on daily returns, duplicate and stale price checks, and gap handling for the monthly panel                                                                                                |
| Sterilized Dataset and Removal Log                                   | A final cleaned dataset along with a log of every exclusion or imputation and the reason for it                                                                                                             |
| Exploratory Data Analysis                                            | Return distributions, price history with major events marked, rolling volatility, driver scatter plots, a correlation matrix, and rolling correlation with equities                                         |
| About Oil Price                                                      | Statistical tests on returns (autocorrelation, volatility clustering, normality, stationarity), a discretized regime preview, and both an expert defined and a data learned belief network of price drivers |
| Data Assembly Summary                                                | A short recap of the data assembly and exploratory findings                                                                                                                                                 |
| Discrete Hidden Markov Model Class                                   | A from scratch implementation of forward, backward, Viterbi, and Baum-Welch                                                                                                                                 |
| Forward and Backward Algorithm Toy Example                           | Validating the forward and backward pass on a known two state weather example                                                                                                                               |
| Viterbi Algorithm Toy Example                                        | Validating decoded state paths against a brute force search                                                                                                                                                 |
| Baum-Welch Algorithm Toy Example                                     | Recovering known generating parameters from simulated sequences                                                                                                                                             |
| Bull, Bear, and Stagnant Regime Illustration                         | A motivating visual example of the three regimes on the WTI price history                                                                                                                                   |
| Transforming the Time Series into Emission Sequences                 | Converting each variable into an up and down emission sequence                                                                                                                                              |
| Learning Regime Parameters with Baum-Welch                           | Training a three state HMM per variable                                                                                                                                                                     |
| Decoding Regimes with Viterbi                                        | Decoding the most likely regime path per variable                                                                                                                                                           |
| Multicolored Regime Plots                                            | Visualizing decoded regimes against the underlying series, with shaded regime bands                                                                                                                         |
| Identifying the Latent Meaning of Each Hidden State                  | Labeling each hidden state by its mean change and time spent in state                                                                                                                                       |
| Train, Validation, and Test Split                                    | A chronological 80:10:10 split of the regime data                                                                                                                                                           |
| Minimal Working Example of Hill Climb Search                         | A small synthetic demonstration of structure learning before applying it to real data                                                                                                                       |
| Training and Validating the Oil-Market Belief Network                | Learning and fitting the belief network with an expert seeded, K2 scored Hill Climbing search, then evaluating prediction accuracy                                                                          |
| Regime Detection and Network Summary                                 | A short recap of the regime detection and structure learning results                                                                                                                                        |
| Robustness Check: Re-Running the Full Pipeline Across Multiple Seeds | Repeating the full regime detection and network fitting pipeline under five different random seeds to check how stable the learned structure and error rates are                                            |
| Comparing Our Results to the Dissertation's Reported Results         | A direct numeric comparison of validation and test error against the reference dissertation's own figures                                                                                                   |
| Reporting the Accuracy of Forecasting the Price of Crude Oil         | Confusion matrices and accuracy figures for the validation and test sets                                                                                                                                    |
| A Graphical Way to Display the Results                               | Confusion matrix heatmaps and a predicted versus actual regime timeline                                                                                                                                     |
| A Simple Trading Simulation                                          | A long, short, and hold strategy driven by the predicted regimes, compared against buy and hold                                                                                                             |
| Conclusion                                                           | A summary of the full notebook, from data assembly through the trading simulation                                                                                                                           |

## Requirements

| Package         | Purpose                                            |
| --------------- | -------------------------------------------------- |
| numpy           | Numerical computing and array operations           |
| pandas          | Data structures and time series handling           |
| matplotlib      | Plotting                                           |
| seaborn         | Statistical plot styling                           |
| yfinance        | Live market data from Yahoo Finance                |
| scipy           | Statistical distributions and tests                |
| statsmodels     | Time series diagnostics (ACF, ADF, Ljung-Box)      |
| pgmpy           | Belief network structure learning and inference    |
| networkx        | Graph construction and drawing for belief networks |
| python-dateutil | Date utilities                                     |

Install them with:

```
pip install numpy pandas matplotlib seaborn yfinance scipy statsmodels pgmpy networkx python-dateutil
```

## Usage

1. Clone this repository.
2. Install the required packages listed above.
3. Open crude_oil_risk_modeling.ipynb in Jupyter or a compatible environment.
4. Run all cells top to bottom. Each section builds on state already in memory from the sections above it, so the notebook is meant to be run in order rather than from an arbitrary cell.
5. If the live data request fails, the notebook falls back to a local cache stored under a data folder; without a live connection or a cache, the affected cell will raise an error.

## Notes

- Descriptions of what each step of the code is doing are printed to the notebook output rather than written as inline code comments.
- The physical market and select macro variables are reconstructed proxies, not official releases, and are clearly labeled as such in the data dictionary.
- The genuine sub zero WTI settlement from 2020 is preserved in the price history but excluded from the return series, since a non positive price breaks log returns.
- The hidden Markov model implementation is a direct NumPy reimplementation rather than a third party HMM library, since the reference library used in the original coursework does not build in current environments.
- Brent and the WTI-Brent spread are deliberately excluded from the belief network's candidate variables, since Brent moves in near lockstep with WTI and would let the network predict WTI from a twin of itself rather than from genuine macro and physical market drivers.
- Validation and test error rates should be read against the chance level baseline for three classes.
- The comparison against a reference dissertation's reported figures is expected to differ in exact numbers, since this project uses reconstructed macro and physical proxy series rather than the genuine EIA and FRED pulls used in that dissertation. What is checked is whether the qualitative pattern of results replicates, not an exact numeric match.
- The trading simulation is illustrative only. Its result is discussed in the notebook as a small sample, concentration driven effect rather than evidence of a reliable trading edge, and should not be read as investment advice.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
