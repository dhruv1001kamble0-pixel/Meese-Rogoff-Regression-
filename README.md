USDINR and Macro Factors README
Does Meese-Rogoff still stand.
This project investigates the macroeconomic drivers of USDINR, the exchange rate between the US Dollar and Indian Rupee, a major EM currency pair. Using monthly data from 2006 to 2025, the project tests whether fundamental variables including interest rate differentials, inflation differentials, money supply, and global risk (CBOE VIX as proxy) can explain and predict monthly USDINR returns.
The analysis is framed around Meese and Rogoff's (1983) finding that structural exchange rate models fail to outperform a random walk forecast of 0, even with realised values of explanatory variables. This project replicates and extends their test to USDINR specifically, examining whether their result holds across multiple macroeconomic channels and whether any variable provides genuine out-of-sample predictive power.
Methodology
Data for bond yields, inflation rates, money supply, and industrial production indices for both the US and India was sourced from the St. Louis Federal Reserve FRED database. VIX and oil prices were downloaded from Yahoo Finance. The dataset covers January 2006 to November 2024 at monthly frequency. 
All price and index series were converted to percentage returns. Yield series were transformed to percentage point changes rather than percentage returns, as a yield moving from 1% to 2% represents a 1pp change, not a 100% change. CPI and yield data from both countries were combined into differentials, India minus US to capture relative policy and price dynamics rather than absolute levels.
An Augmented Dickey-Fuller (ADF) test was applied to all transformed series to confirm stationarity prior to regression. The Indian CPI differential was non-stationary at first difference (p = 0.499), reflecting India's structural inflation phases over the sample period, and required second differencing to achieve stationarity. OLS regression with Newey-West HAC standard errors was used throughout to account for potential autocorrelation and heteroscedasticity in monthly financial data. Model diagnostics included the Breusch-Pagan test for heteroscedasticity, Ljung-Box test for residual autocorrelation, and Jarque-Bera test for normality of residuals.
ADF Table
Variable	ADF p-value	Stationary
USDINR_ret	0.000	Yes
OIL_ret	0.000	Yes
VIX_ret	0.000	Yes
US_M2_ret	0.000	Yes
US_IP_ret	0.000	Yes
CPI_diff2	0.000	Yes
rate_diff	0.000	Yes
Regression Table
Variable	Coefficient	p-value
Constant	0.401	0.020
Oil Returns	0.005	0.773
VIX Returns	0.033	0.000***
US M2 Growth	-0.290	0.130
US Industrial Production	-0.198	0.042**
Inflation Differential Change	-0.226	0.029**
Interest Rate Differential	-0.026	0.941
Model statistics
Statistic	Value
Observations	227
R²	0.128
Adjusted R²	0.104
F-statistic p-value	0.000006

Three variables reached statistical significance at the 5% level: VIX returns, US Industrial Production, and the CPI differential. Oil returns, US money supply growth, and the interest rate differential showed no significant relationship with USDINR returns. CBOE fear index is the dominant predictor of USDINR returns (p < 0.000, coef = 0.033). A 1% increase in VIX is associated with a 0.033% depreciation of the rupee in the same month. During crisis episodes, a VIX move from 20 to 8 leads to approximately 2% depreciation from the VIX alone as per the model. This is consistent with the risk premium hypothesis: when global uncertainty rises, capital flees emerging markets, reducing demand for rupee-denominated assets and depreciating the currency. 
US Industrial Production was found significant at the 5% level, with a negative coefficient suggesting rupee appreciation when US economic activity rises. However, when tested independently against a model excluding this variable, the R² difference was marginal (12.8% vs 11.0%), indicating that US IP partially proxies for the same risk-off dynamics captured by VIX. Both variables tend to move together as strong US growth typically coincides with low VIX and US IP was not retained as a robust independent predictor.
The CPI differential change was significant (p = 0.029, coef = -0.226), with rupee appreciation associated with accelerating Indian inflation relative to the US. This seemingly counterintuitive result is consistent with a carry trade mechanism as rising Indian inflation typically precedes RBI rate hikes, and investors front-run this by purchasing rupee assets for yield pickup, causing short-term appreciation ahead of the tightening cycle but another reason could be inflation signalling higher growth rates in India assets such as equities creating inflows for the EM currency.
Oil returns, money supply growth, and the interest rate differential showed no significant contemporaneous relationship with USDINR. The absence of oil significance is consistent with the finding that oil's effect on INR operates indirectly through global risk channels captured by VIX and has significant lag on USDINR. The insignificance of the interest rate differential is consistent with two competing mechanisms that operate simultaneously and offset each other at monthly frequency. First, UIP predicts that a higher interest rate differential should depreciate the rupee over the holding period, but this adjustment occurs gradually over 12 to 24 months, well outside the contemporaneous window a monthly regression captures. Second, in practice higher Indian rates attract carry inflows that appreciate the rupee, directly contradicting UIP, while simultaneously signalling rising risk premium that can trigger capital flight and depreciate it. These opposing forces net to approximately zero at monthly frequency, producing an insignificant coefficient that reflects genuine complexity in the transmission mechanism rather than an absence of any relationship.
The overall R² of 0.128 indicates the model explains approximately 13% of monthly USDINR return variance. While modest, this is meaningfully above the near-zero explanatory power typically found at daily frequency and represents a replication of the Meese-Rogoff result at a more favourable frequency.
Meese Rogoff Random Walk on USDINR

To test whether in-sample relationships generalise, the model was trained on data from 2006 to 2018 and used to forecast monthly USDINR returns from 2019 to 2025. Forecasts were compared against a random walk benchmark, the simplest possible forecast, which predicts 0% return every month.
Metric	Value
Model RMSE (2019–2025)	1.8000
Random Walk RMSE (2019–2025)	1.2000
Random walk outperforms model	True

The random walk produced lower prediction error than the estimated model in the out-of-sample period, with RMSE of 1.20 versus 1.80. This confirms the Meese-Rogoff result for USDINR at monthly frequency. The model's in-sample coefficients, learned from 2006 to 2018 failed to generalise the 2019–2025 period, which included COVID-19, post-pandemic inflation, and the Palestine conflict War structural break of 2024.
This parameter instability across regimes is itself consistent with Meese and Rogoff's original critique that even when fundamentals show in-sample significance, the structural relationships between macro variables and exchange rates shift across different economic environments, making reliable out-of-sample prediction impossible.
Model Diagnostics
Test	Result
Breusch-Pagan (heteroscedasticity)	p = 0.33 — No heteroscedasticity
Ljung-Box (residual autocorrelation)	No significant autocorrelation at any lag
Jarque-Bera (normality)	p < 0.001 — Non-normal, kurtosis = 6.80
Skewness	0.50 — Positive skew toward depreciation

Conclusion
This project finds that macroeconomic fundamentals have limited and unstable predictive power over USDINR returns at monthly frequency. VIX is the only robust predictor across the full sample, consistent with risk premium dynamics dominating over the monetary and interest rate channels proposed by classical exchange rate theory.
Together, these findings suggest USDINR is not predictable from macroeconomic fundamentals at any practically useful frequency, consistent with the broader FX literature. Risk sentiment i.e. VIX represents the most tractable short-run signal, operating through the capital flow channel that is the subject of the next stage of this research.
Data and code tools
Python | statsmodels | yfinance | FRED API | pandas | numpy | matplotlib | Claude Code
FRED series: M2SL, CPIAUCSL, GS10, INDPRO, INDCPIALLMINMEI, INDIRLTLT01STM
yfinance tickers: USDINR=X, CL=F, ^VIX
Author: Dhruv Kamble, BSc Business Economics with Financial Markets, Queen's University Belfast








