# equity-random-walk-exploration

This short project aims to assess the viability of modelling equity prices as geometric Brownian motion (GBM) by first testing whether the underlying random walk assumptions actually hold in daily price data, then by fitting a GBM model to the data and evaluating the resulting simulations.

For this project, we focus on Procter and Gamble (PG) close prices over its entire history. This asset was chosen as one that has a rich and relatively stable price history.

## Background

A stochastic process $S_t$ follows GBM if the process $\log (S_t)$ is a Brownian motion with drift, i.e. 

$$ \log (S_t) = \log (S_0) + t(\mu - \tfrac{\sigma^2}{2}) + \sigma W_t$$

or, equivalently,

$$ S_t = S_0 \exp (t(\mu - \tfrac{\sigma^2}{2}) + \sigma W_t)$$

where $W_t$ is a standard Brownian motion, $\sigma$ is the standard deviation, or volatility, and $\mu$ is the expected instantaneous rate of return.

For the stochastic process to follow GBM, $\log (S_t)$ must have normally distributed, independent and stationary increments. In particular, these require the drift and volatility of $\log(S_t)$ to remain constant over time, and rule out autocorrelation in the return series. In [notebook 2](notebooks/02_return_distribution_analysis.ipynb), we test the extent to which these properties hold for PG's daily log returns.


## Motivation

The project is a short investigation into the core assumptions, their validity and how they affect the performance of the GBM model. It is not intended to be a comprehensive study. 

## Repo Structure and How to Run

If users wish to clone this repository either to customise it or simply run through it, then they should be aware that it uses Poetry to manage packages. Upon cloning of the repository, the user should run `poetry install` to initiate the environment and install the repository's dependencies. 

This project consists of three numbered notebooks in the `notebooks/` folder. [Notebook 1](notebooks/01_data_pull.ipynb) should be run first as this is where the data used for the rest of the project is downloaded (via the `yfinance` package), processed and saved into the `data/` folder. Beyond this, there is no strict requirement to run [notebook 2](notebooks/02_return_distribution_analysis.ipynb) before [notebook 3](notebooks/03_gbm_fitting_and_simulation.ipynb), however it is strongly recommended that they are read in the correct order to preserve the logical line of thought. 


## Methodology


### 1. Data

In [notebook 1](notebooks/01_data_pull.ipynb), we obtain data for PG via the `yfinance` package that pulls data from [Yahoo Finance](https://finance.yahoo.com). We pull this data from 1961-01-02 onwards (Earliest available data) and ensure to use the auto adjust setting to ensure prices are scaled correctly. We also do completeness checks and minor processing in this notebook to ensure that the data is usable straight out of the csv in later notebooks.

### 2. Assumption Testing

- We use the Jarque-Bera, Shapiro-Wilk and Kolmogorov-Smirnov tests to test whether log returns are normally distributed.
- We consider an ARCH model and use the Lagrange multiplier test to tets for homoskedasticity.
- Finally, we use the Ljung-Box test to test for autocorrelation.
- These tests are non-inferiority in nature meaning that for each of them, a positive result is failing to reject the null. As such we do not conduct any alpha correction, as this would simply make it harder for us to identify issues with our assumptions. 

### 3. Fitting, Simulating and Evaluating GBM 

- We estimate the volatility and drift using the sample standard deviation and sample mean respectively on a specifically designed "training" period. For a justification on the estimates, see [notebook 2](notebooks/02_return_distribution_analysis.ipynb).
- We simulate ahead for an out of sample simulation period, replacing the unkowns in the GBM model with their estimates, and by drawing from a normal distribution for the Brownian motion.
- We then compare the simulations to the actual data for that period, looking at terminal prices, log returns distribution and percentiles, and sample standard deviation distribution.

### 4. Known Limitations

- The ARCH test homoskedasticity assumes that the time series has normally distributed increments. If this is not true then the test is technically not valid. 
- Similarly, the LB test for autocorrelation assumes that the time series is homoskedastic. This creates a chain of dependencies where if one of our tests "fail" (undesired result) it technically invalidates the subsequent tests. This is the main reason the tests have been ordered this way.


## Findings

We found evidence to suggest that each of the three assumptions were violated. This realised as a poorly performing simulation. In particular, the actual sample standard deviation during the simulation period was found to fall below the first percentile of sample standard deviations from the simulations, suggesting that the actual variance was lower during the simulation period compared to the period the model was fitted on.  


## Next Steps and Future Work

Each of the failed tests in [notebook 2](notebooks/02_return_distribution_analysis.ipynb) point towards an area of future work:
- We may look to find a better distribution to model log returns, such as by using [Merton's Jump Diffusion model (1976)](https://doi.org/10.1016/0304-405X(76)90022-2).
- There is an interesting opportunity to try and accomodate heteroskedasticity in the model using ARCH or GARCH, or "zoom in" on periods of relative constant volatility (though this comes with issues regarding selection bias).
- We may revisit more rigorous ways to test for autocorrelation under the absence of the other two assumptions (e.g. the modified Q* test ([Lobato, Nankervis and Savin, 2001](https://doi.org/10.1111/1468-2354.00106))) and potentially look to accomodate it in the model also.