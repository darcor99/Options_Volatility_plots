# Options Volatility Plots

Implied volatility surface and smile/smirk analysis for US equity options using the Black-Scholes model. Fetches live options chain data from Yahoo Finance and visualizes IV across strikes, expirations, and moneyness.

## What Are Volatility Plots?

**Implied volatility (IV)** is the market's expectation of future price movement embedded in an option's price. Unlike historical volatility (which looks backward), IV is forward-looking and derived by inverting the Black-Scholes pricing model.

Volatility plots reveal key patterns in how IV varies:

- **Volatility Smile**: IV tends to be higher for deep in-the-money (ITM) and deep out-of-the-money (OTM) options, forming a U-shape when plotted against strike price. This reflects the market pricing in a higher probability of extreme moves than the Black-Scholes model assumes.
- **Volatility Smirk (Skew)**: In equity markets, OTM puts often carry higher IV than OTM calls, creating an asymmetric smile. This reflects demand for downside protection.
- **Volatility Surface**: A 3D plot showing how IV varies across both strike price (or moneyness) and time to expiration simultaneously, revealing the full term structure of implied volatility.

## How Implied Volatility Is Calculated

1. **Fetch market option prices** — mid-price from bid/ask spread when available, otherwise last traded price
2. **Apply the Black-Scholes model** — standard closed-form pricing for European calls and puts:
   - `d1 = (ln(S/K) + (r + 0.5*sigma^2)*T) / (sigma*sqrt(T))`
   - `d2 = d1 - sigma*sqrt(T)`
   - Call price = `S*N(d1) - K*e^(-rT)*N(d2)`
   - Put price = `K*e^(-rT)*N(-d2) - S*N(-d1)`
3. **Numerically invert** — since there's no closed-form solution for sigma given a market price, we use `scipy.optimize.brentq` (Brent's root-finding method) to find the volatility value that makes the Black-Scholes price match the observed market price, searching within the range [0.0001, 5.0]
4. **Compare against Yahoo Finance IV** — the notebook also pulls Yahoo's own IV values for validation

Key parameters:
- `S` = current stock price (last close from historical data)
- `K` = option strike price
- `T` = time to expiration in years (days to expiry / 365)
- `r` = risk-free rate (default `0.05`)
- Moneyness = `S / K` (>1 = ITM for calls, <1 = OTM for calls)

## Visualizations

The notebook produces the following plots (saved as PNGs in `plots/`):

| Plot | Description |
|------|-------------|
| Candlestick + SMAs | 1-year price history with 5/20/50-day simple moving averages and volume |
| Strike vs IV (2D) | Scatter plot of implied volatility against strike price, colored by option type (call/put) and grouped by expiration date. Shows the volatility smile/smirk pattern. Current stock price marked with a vertical dashed line. |
| Strike vs T vs IV (3D) | Interpolated 3D volatility surface across strike prices and time to expiry, with actual call/put data points overlaid |
| Moneyness vs T vs IV (3D) | Volatility surface using moneyness (S/K) instead of raw strikes, with an ATM reference line at moneyness=1. Filtered to 0.5-1.5 moneyness range. |

3D surfaces are built using `scipy.interpolate.griddata` with cubic interpolation (linear fallback for gaps) on a 50x50 grid.

## Features

- Fetches 1-year historical stock data via Alpaca API (candlestick charts with SMA overlays)
- Retrieves full options chains (all available expirations) from Yahoo Finance
- Computes implied volatility using Black-Scholes pricing and Brent's method
- Compares calculated IV against Yahoo Finance's provided IV values
- Interactive Plotly visualizations with hover data (symbol, prices, IV values)

## Requirements

```
alpaca-py
alpaca-trade-api
yfinance
plotly
tensorflow
scikit-learn
scipy
pandas
numpy
matplotlib
seaborn
```

Install all dependencies (included as `!pip install` cells in the notebook):

```bash
pip install alpaca-py alpaca-trade-api yfinance plotly tensorflow scikit-learn scipy pandas numpy matplotlib seaborn
```

## Setup

### Alpaca API (for historical stock data)

The notebook expects Alpaca API credentials via Google Colab secrets:

- `alpaca_api_key`
- `alpaca_secret_key`

If running outside Colab, replace the `userdata.get()` calls with your credentials or environment variables.

### Yahoo Finance (for options data)

No API key required. Options chains are fetched via `yfinance.Ticker.option_chain()`.

## Usage

1. Open `Options_volatility.ipynb` in Jupyter or Google Colab
2. Set the ticker symbol:
   ```python
   symbol = "BITF"  # Replace with any US ticker
   ```
3. Run all cells sequentially

The risk-free rate for IV calculations defaults to `r = 0.05`.

## Observations

- Implied volatility tends to be higher for out-of-the-money options (both puts and calls), consistent with the expected smile pattern
- Near-term options exhibit more pronounced volatility smiles than longer-dated ones
- The volatility term structure is visible across different expiration dates on the 3D surface
- Calculated IV values diverge from Yahoo Finance's IV for very short-dated and deep ITM/OTM options, likely due to differences in the underlying model assumptions

## Potential Improvements

- **Remove Alpaca dependency**: Historical stock data could be fetched entirely from Yahoo Finance (`yf.Ticker.history()`), eliminating the need for API keys
- **Better IV solver bounds**: The current search range of [1e-6, 5.0] fails for some extreme options; adaptive bounds or a different solver (Newton-Raphson with Vega) could improve coverage
- **Filter illiquid options**: Options with zero volume or wide bid-ask spreads produce unreliable IV values; filtering by open interest and spread width would clean the surface
- **Use mid-price more robustly**: Fall back to last price only when bid/ask are both zero, and flag stale quotes
- **Add Greeks**: Delta, gamma, vega, and theta can be computed alongside IV from the same Black-Scholes framework

## Future Directions

- Compare IV patterns across different underlying assets and sectors
- Analyze historical volatility vs implied volatility spreads (volatility risk premium)
- Build separate call and put volatility surfaces for side-by-side comparison
- Track IV surface changes over time (volatility surface dynamics)
- Implement SABR or SVI parameterization for smoother surface fitting
- Use the ML components (TensorFlow/scikit-learn) already imported to predict IV surface movements

## Output

Generated plots are saved to the `plots/` directory as PNG files.
