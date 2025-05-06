# Introduction

The tools directory of this project contains a collection of methods that make the economic and fundamental factor models useful.

______

# Glossary

## Fundamental Factor Model Tools

### `get_characteristic_factor_exposures`

Retrieves the characteristic factor exposures for a given period and symbol if provided, otherwise for all symbols.

Parameters:

- `from`: The start date of the period to compute the characteristic factor exposures.
- `to`: The end date of the period to compute the characteristic factor exposures.
- `symbol?`: Optional, the symbol to compute the characteristic factor exposures for. Defaults to None.
- `universe?`: Optional, the universe of symbols to compute the characteristic factor exposures for. Defaults to S&P 500.

Returns:

- A pandas DataFrame with the characteristic factor exposures.

Pseudocode:

```
TODO
```

### `get_cross_sectional_return`

Retrieves the cross-sectional return for a given period and symbol if provided, otherwise for all symbols.

Parameters:

- `from`: The start date of the period to compute the cross-sectional return.
- `to`: The end date of the period to compute the cross-sectional return.
- `symbol?`: Optional, the symbol to compute the cross-sectional return for. Defaults to None.
- `universe?`: Optional, the universe of symbols to compute the cross-sectional return for. Defaults to S&P 500.

Returns:

- A pandas DataFrame with the cross-sectional return.

---

Pseudocode:

```
TODO
```

### `compute_cross_sectional_return_decomposition`

**Parameters**

- `from`  
  The start date of the period over which to compute the cross-sectional return decomposition.  
- `to`  
  The end date of the period over which to compute the cross-sectional return decomposition.  
- `symbol`  
  The ticker symbol for which to compute the cross-sectional return decomposition.  
- `universe?` *(optional)*  
  The universe of symbols to use in the decomposition (defaults to S&P 500).

**Returns**

- A mapping with:
  - `characteristic_returns`: vector of per-factor contributions
  - `systematic`: the factor return (scalar)
  - `specific`: the residual return (scalar)

---

**Pseudocode**

1. **Load data**  
   Let `universe_characteristic_factor_exposures ← get_characteristic_factor_exposures(from, to, universe)`  
   → returns a mapping `{ ticker → [exposure₁, exposure₂, …, exposureₖ] }`  
   Let `universe_returns ← get_cross_sectional_return(from, to, universe)`
   → returns a mapping `{ ticker → return }`

2. **Build matrices & vectors** (sorted by ticker)  
   Let $\Phi$ be a matrix of size $N \times K$ containing the exposures  
   Let $y$ be a vector of length $N$ containing the returns

3. **Estimate universe-wide factor risk premia via OLS**  
   $$
   \text{factor\_risk\_premia} = (\Phi^\top \Phi)^{-1} \Phi^\top y
   $$

4. **Extract exposures for the stock of interest**  
   Let $w = \text{universe\_characteristic\_factor\_exposures}[symbol]$  
   → a $K$-vector of factor exposures for the given stock

5. **Compute the stock’s systematic return**  
   $$
   \text{this\_stock\_factor\_return} = w^\top \cdot \text{factor\_risk\_premia}
   $$

6. **Compute the residual (specific) return**  
   Let $\text{tot\_ret} = \text{universe\_returns}[symbol]$  
   Let $\text{factor\_ret} = \text{this\_stock\_factor\_return}$  
   $$
   \text{residual} = \text{tot\_ret} - \text{factor\_ret}
   $$

7. **Return the decomposition**  
   Return a mapping with:
   - `characteristic_returns`: vector of per-factor contributions  
   - `systematic`: the factor return (scalar)  
   - `specific`: the residual return (scalar)

Here’s the full documentation for `compute_time_series_return_decomposition` in markdown with properly embedded LaTeX (no code blocks), styled consistently with your previous section:

---

### `compute_time_series_return_decomposition`

**Parameters**

- `from`  
  The start date of the period over which to compute the time-series return decomposition.  
- `to`  
  The end date of the period over which to compute the time-series return decomposition.  
- `symbol`  
  The ticker symbol for which to compute the time-series return decomposition.  
- `universe?` *(optional)*  
  The universe of symbols to use in the cross-sectional regression. Defaults to S&P 500.

**Returns**

- A pandas DataFrame with the cumulative time-series return decomposition, including systematic, specific, and per-characteristic contributions.

---

**Pseudocode**

1. **Initialize parameters and date range**  
   Let `interval = daily`  
   Let `dates` be the list of trading days between `from` and `to`
   ... check for future dates, from > to, etc

2. **Initialize time series storage**
   Let the following be dictionaries indexed by date:
   - `characteristic_return[char][date]`: cross-sectional return due to a specific characteristic  
   - `systematic_return[date]`: total factor-driven return  
   - `specific_return[date]`: stock-specific residual return

3. **For each date, run the cross-sectional decomposition**  
   For each `date ∈ dates`:
   - Let `returns = compute_cross_sectional_return_decomposition(symbol, date, date, universe)`
   - For each factor `char`, set:
     `characteristic_return[char][date] = returns.characteristic_returns[char]`
   - Set:  
     `systematic_return[date] = returns.systematic`  
     `specific_return[date] = returns.specific`

4. **Convert returns to log space**  
   For each `date ∈ dates`, compute:
   - $\log\_r_{\text{char}}[char][date] = \log(1 + \text{characteristic\_return}[char][date])$
   - $\log\_r_{\text{systematic}}[date] = \log(1 + \text{systematic\_return}[date])$
   - $\log\_r_{\text{specific}}[date] = \log(1 + \text{specific\_return}[date])$

5. **Compute cumulative log returns over time**  
   Let `cumulative_log_characteristic_return`, `cumulative_log_systematic_return`, and `cumulative_log_specific_return` be initialized dictionaries.  
   For the first date:
   - Set each cumulative return to the corresponding log return  
   For each subsequent `date`:
   - Add the current log return to the previous cumulative value:
     $$
     \text{cumulative\_log}[t] = \text{cumulative\_log}[t-1] + \log(1 + r[t])
     $$

6. **Final output**  
   Return a DataFrame with columns:
   - `date`
   - `cumulative_systematic_return = \exp(\text{cumulative\_log\_systematic\_return}) - 1`
   - `cumulative_specific_return = \exp(\text{cumulative\_log\_specific\_return}) - 1`
   - One column for each characteristic:
     $$
     \text{cumulative\_characteristic\_return}_{i} = \exp(\text{cumulative\_log\_characteristic\_return}_{i}) - 1
     $$

---