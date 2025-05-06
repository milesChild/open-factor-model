# Factor Model Glossary

## Introduction

<TODO>

---

# OFMUSLC01 — Open Factor Model: US Large Cap v1.0  

*(All exposures are computed once per month using data available as of the final trading day of the prior month and are subsequently held constant until the next rebalance. Fundamental (non-trading) characteristics are obtained consciously of the filing date as reported by the SEC and fiscal dates are not used to protect against look-ahead bias.)*

---

## 1 Market (Intercept)  

### 1.1 Description  
The Market (or **Intercept**) factor represents the common return that is shared by every asset in the universe after style‑ and industry‑specific effects have been stripped out. It is mathematically identical to an intercept term in a cross‑sectional regression: each stock receives an exposure of 1. Including this factor ensures that style and industry premia are estimated relative to the broad‑market return rather than in absolute‑price space, preserving additivity of coefficients and preventing the grand mean from leaking into the other factors.

### 1.2 Calculation Details  
| Step | Procedure |
|------|-----------|
|Data|No inputs—exposure is a constant **1** for every security.|
|Weighting| N/A |
|Standardisation| N/A |

---

## 2 Industry  

### 2.1 Description  
Industry factors proxy for systematic risk embedded in a firm’s line of business—supply‑chain shocks, regulatory events, commodity swings, and technology cycles that cluster within sectors but cut across style characteristics. Academic evidence (e.g., Fama & French 1997) shows that industry membership explains a sizable share of cross‑sectional return variance even after accounting for market beta, size, and value.

### 2.2 Calculation Details  
| Step | Procedure |
|------|-----------|
|Industry schema|Use the **Financial Modeling Prep (FMP) “industry” field** as the taxonomy. The full list of industry strings is available in the FMP documentation.<br>• Tickers lacking an industry tag are mapped to `Unknown`.|
|Exposure construction|Binary (0/1) dummy variables: a stock receives **1** for its reported industry and **0** otherwise.|
|Standardisation| N/A |

---

## 3 Style Factors  

### 3.1 Growth  

#### 3.1.1 Description  
The Growth factor captures the notion that firms with superior historical and forecasted expansion in fundamentals command valuation premia and may deliver differential risk‑adjusted returns. Empirical work (e.g., Lakonishok, Shleifer & Vishny 1994) suggests that growth signals can forecast both earnings surprises and future cash‑flow momentum.

#### 3.1.2 Calculation Details  
| Step | Procedure |
|------|-----------|
|Inputs|Compound annual growth rates (CAGR) in **sales** and **diluted EPS** over the latest **three fiscal years**.|
|Computation|`Growth_raw = ½·CAGR_sales + ½·CAGR_EPS`.|
|Winsorisation|Two‑sided at the **1st / 99th percentiles** of the cross‑section.|
|Standardisation|Z‑score (subtract cross‑sectional mean, divide by cross‑sectional std).|

---

### 3.2 Value  

#### 3.2.1 Description  
Value factors reward cheap securities relative to fundamentals, echoing the evidence of Basu (1977) and Fama & French (1992). The economic rationale is either a risk premium for distressed firms or a behavioural mispricing that corrects over time.

#### 3.2.2 Calculation Details  
| Step | Procedure |
|------|-----------|
|Inputs|**Book‑to‑Price (B/P)**: common equity from the most recent annual report ÷ month‑end market cap.<br>**Earnings Yield (E/P)**: trailing twelve‑month earnings ÷ market cap.|
|Computation|`Value_raw = ½·B/P + ½·E/P`.|
|Winsorisation|1 % / 99 %.|
|Standardisation|Z‑score.|

---

### 3.3 Size  

#### 3.3.1 Description  
Smaller firms have historically out‑performed larger peers on a risk‑adjusted basis (Banz 1981), possibly reflecting liquidity risk, limited analyst coverage, or greater optionality. Size also acts as a scaling variable for several other characteristics. Size is generally thought to be a weak factor that would be better explained by a culmination of other factors ([link](https://chatgpt.com/share/681a3711-f48c-8009-8b5e-53f9b257671c)). Nonetheless, it is frequently used and discussed in practice.

#### 3.3.2 Calculation Details  
| Step | Procedure |
|------|-----------|
|Input|Month‑end **float‑adjusted market capitalisation**.|
|Transform|`Size_raw = ln(Market Cap)`.|
|Standardisation|Z‑score (ln does winsorisation's job).|

---

### 3.4 Momentum  

#### 3.4.1 Description  
Momentum exploits the empirical finding (Jegadeesh & Titman 1993) that stocks with strong recent performance tend to keep outperforming over intermediate horizons, arguably due to under‑reaction or gradual information diffusion.

#### 3.4.2 Calculation Details  
| Step | Procedure |
|------|-----------|
|Input|**Total shareholder return (TSR)** over the **past 252 trading days**, inclusive of dividends.|
|Computation|Cumulative geometric return. (Some people subtract the recent month, but we do *not*; users can experiment in forks.)|
|Winsorisation|1 % / 99 %|
|Standardisation|Z‑score|

---

## 4 Market Sensitivity  

### 4.1 Description  
Market Sensitivity (β) quantifies each stock’s co‑movement with the broad market, representing exposure to systematic macro shocks.  CAPM and multifactor research alike treat beta as a baseline risk measure; controlling for it isolates style premia from pure market risk. The distinction between Market Sensitivity and Market is an important one - Market Sensitivity measures Beta whereas Market is simply the bias term in the OLS regression.

### 4.2 Calculation Details  
| Step | Procedure |
|------|-----------|
|Input data|**Weekly total returns** for each security and the **S&P 500** benchmark, rolling 104 weeks (≈ 2 years)|
|Estimation|OLS regression $r_i = α_i + β_i·r_m + ε_i$. Scholes–Williams adjustments could be made in future releases|
|Winsorisation|1 % / 99 %|
|Standardisation|Z‑score|

---

## 5 Volatility  

### 5.1 Description  
Volatility measures CAPM-idiosyncratic variability in returns, distinct from systematic beta risk.  High‑volatility stocks often under‑perform on a risk‑adjusted basis (Ang, Hodrick, Xing & Zhang 2006).  Because volatility and beta are naturally correlated, we orthogonalise Volatility to ensure it captures *pure* residual risk rather than leverage to the market factor.

### 5.2 Calculation Details  
| Step | Procedure |
|------|-----------|
|Raw metric| $σ_i$ = stdev( daily total returns ) over the **past 252 trading days**|
|Orthogonalisation|Run a cross‑sectional regression each month: $σ_raw = γ₀ + γ₁·β + ε$. The **residual ε** is the preliminary volatility exposure|
|Winsorisation|Apply 1 % / 99 % to the residuals|
|Standardisation|Z‑score of the winsorised residuals|

---

## Appendix  

| Abbreviation | Meaning |
|--------------|---------|
|B/P|Book‑to‑Price ratio|
|E/P|Earnings Yield|
|β|Market Sensitivity (beta)|
|TSR|Total Shareholder Return|

> **Version history**: 1.0 (2025‑05‑06) — initial public release of OFMUSLC01.