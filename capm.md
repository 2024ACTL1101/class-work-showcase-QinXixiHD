
# CAPM Analysis

## Introduction

In this assignment, you will explore the foundational concepts of the Capital Asset Pricing Model (CAPM) using historical data for AMD and the S&P 500 index. This exercise is designed to provide a hands-on approach to understanding how these models are used in financial analysis to assess investment risks and returns.

## Background

The CAPM provides a framework to understand the relationship between systematic risk and expected return, especially for stocks. This model is critical for determining the theoretically appropriate required rate of return of an asset, assisting in decisions about adding assets to a diversified portfolio.

## Objectives

1. **Load and Prepare Data:** Import and prepare historical price data for AMD and the S&P 500 to ensure it is ready for detailed analysis.
2. **CAPM Implementation:** Focus will be placed on applying the CAPM to examine the relationship between AMD's stock performance and the overall market as represented by the S&P 500.
3. **Beta Estimation and Analysis:** Calculate the beta of AMD, which measures its volatility relative to the market, providing insights into its systematic risk.
4. **Results Interpretation:** Analyze the outcomes of the CAPM application, discussing the implications of AMD's beta in terms of investment risk and potential returns.

## Instructions

### Step 1: Data Loading

- We are using the `quantmod` package to directly load financial data from Yahoo Finance without the need to manually download and read from a CSV file.
- `quantmod` stands for "Quantitative Financial Modelling Framework". It was developed to aid the quantitative trader in the development, testing, and deployment of statistically based trading models.
- Make sure to install the `quantmod` package by running `install.packages("quantmod")` in the R console before proceeding.

```r
# Set start and end dates
start_date <- as.Date("2019-05-20")
end_date <- as.Date("2024-05-20")

# Load data for AMD, S&P 500, and the 1-month T-Bill (DTB4WK)
amd_data <- getSymbols("AMD", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
gspc_data <- getSymbols("^GSPC", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
rf_data <- getSymbols("DTB4WK", src = "FRED", from = start_date, to = end_date, auto.assign = FALSE)

# Convert Adjusted Closing Prices and DTB4WK to data frames
amd_df <- data.frame(Date = index(amd_data), AMD = as.numeric(Cl(amd_data)))
gspc_df <- data.frame(Date = index(gspc_data), GSPC = as.numeric(Cl(gspc_data)))
rf_df <- data.frame(Date = index(rf_data), RF = as.numeric(rf_data[,1]))  # Accessing the first column of rf_data

# Merge the AMD, GSPC, and RF data frames on the Date column
df <- merge(amd_df, gspc_df, by = "Date")
df <- merge(df, rf_df, by = "Date")
```

#### Data Processing 
```r
colSums(is.na(df))
# Fill N/A RF data
df <- df %>%
  fill(RF, .direction = "down") 
```

### Step 2: CAPM Analysis

The Capital Asset Pricing Model (CAPM) is a financial model that describes the relationship between systematic risk and expected return for assets, particularly stocks. It is widely used to determine a theoretically appropriate required rate of return of an asset, to make decisions about adding assets to a well-diversified portfolio.

#### The CAPM Formula
The formula for CAPM is given by:

$$
E(R_i) = R_f + \beta_i (E(R_m) - R_f)
$$

Where:

- $E(R_i)$ is the expected return on the capital asset,
- $R_f$ is the risk-free rate,
- $\beta_i$ is the beta of the security, which represents the systematic risk of the security,
- $E(R_m)$ is the expected return of the market.



#### CAPM Model Daily Estimation

- **Calculate Returns**: First, we calculate the daily returns for AMD and the S&P 500 from their adjusted closing prices. This should be done by dividing the difference in prices between two consecutive days by the price at the beginning of the period.
  
$$
\text{Daily Return} = \frac{\text{Today's Price} - \text{Previous Trading Day's Price}}{\text{Previous Trading Day's Price}}
$$

```r
# Calculate daily returns and add to the dataframe
df <- df %>%
  mutate(AMD_daily_return = (AMD - lag(AMD)) / lag(AMD),
         SP500_daily_return = (GSPC - lag(GSPC)) / lag(GSPC))
```

- **Calculate Risk-Free Rate**: Calculate the daily risk-free rate by conversion of annual risk-free Rate. This conversion accounts for the compounding effect over the days of the year and is calculated using the formula:
  
$$
\text{Daily Risk-Free Rate} = \left(1 + \frac{\text{Annual Rate}}{100}\right)^{\frac{1}{360}} - 1
$$

```r
# Calculate the daily risk-free rate and add to the dataframe
df <- df %>%
  mutate(daily_RF = (1 + RF / 100) ^ (1 / 360) - 1)
```


- **Calculate Excess Returns**: Compute the excess returns for AMD and the S&P 500 by subtracting the daily risk-free rate from their respective returns.

```r
# Calculate excess returns and add to the dataframe
df <- df %>%
  mutate(AMD_excess = AMD_daily_return - daily_RF,
         SP500_excess = SP500_daily_return - daily_RF)
```


- **Perform Regression Analysis**: Using linear regression, we estimate the beta (\(\beta\)) of AMD relative to the S&P 500. Here, the dependent variable is the excess return of AMD, and the independent variable is the excess return of the S&P 500. Beta measures the sensitivity of the stock's returns to fluctuations in the market.

```r
# Remove any rows with na values
df <- df %>%
  na.omit()

# Perform linear regression
CAPM_model <- lm(AMD_excess ~ SP500_excess, data = df)
summary(CAPM_model)
```


#### Interpretation

What is your \(\beta\)? Is AMD more volatile or less volatile than the market?

**Answer:**

The \(\beta\) value is indicated in the summary of the CAPM_model above as the value 1.5699987, which is displayed as the estimate of SP500_excess. This means that for a one unit increase in the excess return for the S&P 500 will result in an approximate 1.57 unit increase in excess return for the AMD. Since \(\beta\) > 1, that means that AMD is sharing a bigger proportion of the market risk, so it is more volatile than the market. This greater risk generates potential for higher returns due to the greater fluctuations of the AMD share price, however, this consequently results in greater potential losses. This higher volatility in AMD shares would incentive more 'risk lover' investors, while 'risk adverse' investors would be more inclined to invest in other shares within the market.

#### Plotting the CAPM Line
Plot the scatter plot of AMD vs. S&P 500 excess returns and add the CAPM regression line.

```r
# Plot the data points and CAPM line
ggplot(df,aes(x = SP500_excess, y = AMD_excess)) +
  geom_point() +
  geom_smooth(method = "lm", col = "red", se = TRUE) +
  theme(text = element_text(family = "Palatino")) +
  labs(title = "AMD vs S&P 500",
       x = "S&P 500 excess return",
       y = "AMD excess return")
```

### Step 3: Predictions Interval
Suppose the current risk-free rate is 5.0%, and the annual expected return for the S&P 500 is 13.3%. Determine a 90% prediction interval for AMD's annual expected return.



**Answer:**

```r
# Define values
Rf <- 0.05
annual_E_Rm <- 0.133
beta_i <- coef(CAPM_model)[2]

# Calculate the forecast AMD return from the CAPM Formula
AMD_E_Ri <- Rf + beta_i * (annual_E_Rm - Rf)

# Extract and calculate the standard error from the regression model
s_f <- summary(CAPM_model)$sigma
annual_s_f <- s_f * sqrt(252)

# Determine the quantile of the critical value
quantile <- 1 - (1 - 0.90) / 2 # The t test is two sided

# Find the t-score(critical value)
t_score <- qt( quantile, df = nrow(df) - 1 )

# Calculate the prediction interval
lower_bound <- AMD_E_Ri - t_score * annual_s_f
upper_bound <- AMD_E_Ri + t_score * annual_s_f
cat("Prediction interval:", lower_bound, "~", upper_bound)
```
