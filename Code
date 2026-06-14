
import yfinance as yf
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import matplotlib.dates as mdates
import statsmodels.api as sm
from statsmodels.tsa.stattools import adfuller
from statsmodels.stats.diagnostic import het_breuschpagan, acorr_ljungbox
from statsmodels.stats.outliers_influence import variance_inflation_factor
from statsmodels.stats.stattools import jarque_bera
from fredapi import Fred
# ── FRED Setup ────────────────────────────────────────────────────
fred = Fred(api_key="ad15764e513aae03870c133f6f491d0f")

START = "2006-01-01"
END   = "2024-11-01"

# ── yfinance ──────────────────────────────────────────────────────
USDINR = yf.download("USDINR=X", start=START, end=END)["Close"].squeeze().resample("MS").last()
OIL    = yf.download("CL=F",     start=START, end=END)["Close"].squeeze().resample("MS").last()
VIX    = yf.download("^VIX",     start=START, end=END)["Close"].squeeze().resample("MS").mean()

# ── FRED — copy these series IDs exactly ─────────────────────────
US_M2   = fred.get_series("M2SL",            START, END)
US_CPI  = fred.get_series("CPIAUCSL",        START, END)
US_10Y  = fred.get_series("GS10",            START, END)
US_IP   = fred.get_series("INDPRO",          START, END)
IN_CPI  = fred.get_series("INDCPIALLMINMEI", START, END)
IN_10Y  = fred.get_series("INDIRLTLT01STM",  START, END)

# ── Combine ───────────────────────────────────────────────────────
df_raw = pd.concat([
    USDINR.rename("USDINR"),
    OIL.rename("OIL"),
    VIX.rename("VIX"),
    US_M2.rename("US_M2"),
    US_CPI.rename("US_CPI"),
    US_10Y.rename("US_10Y"),
    US_IP.rename("US_IP"),
    IN_CPI.rename("IN_CPI"),
    IN_10Y.rename("IN_10Y"),
], axis=1)

df_raw.index = pd.to_datetime(df_raw.index)
df_raw = df_raw.sort_index()

print(df_raw.shape)
print(df_raw.isnull().sum())
df_raw.to_excel("USDINR_macro_raw4.xlsx")
print("Saved")
# Read complete combined file
df_raw = pd.read_excel("USDINR_macro_combined.xlsx",
                        index_col=0,
                        parse_dates=True)

df_raw.index = pd.to_datetime(df_raw.index)
df_raw = df_raw.sort_index()
df_raw = df_raw.resample("MS").last()

print(f"Shape: {df_raw.shape}")
print(f"Date range: {df_raw.index.min()} to {df_raw.index.max()}")
print(f"\nMissing values:")
print(df_raw.isnull().sum())

df = pd.DataFrame()

# Percentage returns for prices and indices
df["USDINR_ret"] = df_raw["USDINR"].pct_change() * 100
df["OIL_ret"]    = df_raw["OIL"].pct_change() * 100
df["VIX_ret"]    = df_raw["VIX"].pct_change() * 100
df["US_M2_ret"]  = df_raw["US_M2"].pct_change() * 100
df["US_CPI_ret"] = df_raw["US_CPI"].pct_change() * 100
df["IN_CPI_ret"] = df_raw["IN_CPI"].pct_change() * 100
df["US_IP_ret"]  = df_raw["US_IP"].pct_change() * 100

# Yield changes in percentage points not pct_change
df["IN_10Y_chg"] = df_raw["IN_10Y"].diff()
df["US_10Y_chg"] = df_raw["US_10Y"].diff()

# Differentials — India minus US
df["CPI_diff"]   = df["IN_CPI_ret"] - df["US_CPI_ret"]
df["rate_diff"]  = df["IN_10Y_chg"] - df["US_10Y_chg"]

df = df.dropna()

print(f"Shape after returns: {df.shape}")

print('=== Augmented Dickey-Fuller Tests ===')
print('p-value < 0.05 = stationary = safe to use\n')

for col in df.columns:
    result = adfuller(df[col].dropna())
    status = "STATIONARY" if result[1] < 0.05 else "NON-STATIONARY"
    print(f"{col:<20} ADF: {result[0]:>8.4f}   p-value: {result[1]:.4f}   {status}")

df["IN_CPI_ret2"]  = df["IN_CPI_ret"].diff()
df["US_CPI_ret2"]  = df["US_CPI_ret"].diff()
df["CPI_diff2"]    = df["IN_CPI_ret2"] - df["US_CPI_ret2"]

df = df.dropna()

# ==========================================================
# RERUN ADF ON DIFFERENCED CPI SERIES
# ==========================================================

for col in ["IN_CPI_ret2", "US_CPI_ret2", "CPI_diff2"]:
    result = adfuller(df[col].dropna())
    status = "STATIONARY" if result[1] < 0.05 else "NON-STATIONARY"

    print(
        f"{col:<20} "
        f"ADF: {result[0]:>8.4f} "
        f"p-value: {result[1]:.4f} "
        f"{status}"
    )

# ==========================================================
# REGRESSION
# ==========================================================

X_cols = [
    "OIL_ret",
    "VIX_ret",
    "US_M2_ret",
    "US_IP_ret",
    "CPI_diff2",
    "rate_diff"
]

X = sm.add_constant(df[X_cols])
y = df["USDINR_ret"]

model = sm.OLS(
    y,
    X
).fit(
    cov_type="HAC",
    cov_kwds={"maxlags": 3}
)

print(model.summary())

# ==========================================================
# RANDOM WALK TEST
# ==========================================================

train = df[df.index < "2019-01-01"]
test = df[df.index >= "2019-01-01"]

X_train = sm.add_constant(train[X_cols])
y_train = train["USDINR_ret"]

model_train = sm.OLS(
    y_train,
    X_train
).fit(
    cov_type="HAC",
    cov_kwds={"maxlags": 3}
)

X_test = sm.add_constant(test[X_cols])
y_test = test["USDINR_ret"]

y_pred = model_train.predict(X_test)

# Random walk forecast = 0 return
y_rw = pd.Series(0, index=test.index)

rmse_model = np.sqrt(((y_test - y_pred) ** 2).mean())
rmse_rw = np.sqrt(((y_test - y_rw) ** 2).mean())

print(f"Model RMSE: {rmse_model:.4f}")
print(f"Random Walk RMSE: {rmse_rw:.4f}")
print(f"Random Walk Wins: {rmse_rw < rmse_model}")

# ==========================================================
# EXPORT RESULTS TO WORD
# ==========================================================

from docx import Document

doc = Document()

doc.add_heading(
    "USD/INR Exchange Rate Regression Results",
    level=1
)

# Regression table
doc.add_heading(
    "OLS Regression with HAC Standard Errors",
    level=2
)

doc.add_paragraph(model.summary().as_text())

# Forecast results
doc.add_heading(
    "Forecast Comparison",
    level=2
)

doc.add_paragraph(
    f"Model RMSE: {rmse_model:.4f}"
)

doc.add_paragraph(
    f"Random Walk RMSE: {rmse_rw:.4f}"
)

doc.add_paragraph(
    f"Random Walk Wins: {rmse_rw < rmse_model}"
)


# ==========================================================
# DIAGNOSTIC TESTS
# ==========================================================

resid = model.resid

# Jarque-Bera
jb_stat, jb_pvalue, skew, kurtosis = jarque_bera(resid)

# Breusch-Pagan
bp_stat, bp_pvalue, f_stat, f_pvalue = het_breuschpagan(resid, X)

# Ljung-Box
lb = acorr_ljungbox(
    resid,
    lags=[12],
    return_df=True
)

lb_stat = lb["lb_stat"].iloc[0]
lb_pvalue = lb["lb_pvalue"].iloc[0]

# ==========================================================
# VIF TABLE
# ==========================================================

vif = pd.DataFrame()
vif["Variable"] = X.columns
vif["VIF"] = [
    variance_inflation_factor(X.values, i)
    for i in range(X.shape[1])
]
doc.save("USDINR_Results.docx")

print("Word document saved as USDINR_Results.docx")







<img width="451" height="684" alt="image" src="https://github.com/user-attachments/assets/814d78a2-ce4b-45ef-91d0-60e9612771a0" />
