# Options Volatility Plots

Implied volatility surface and smile/smirk analysis for US equity options using the Black-Scholes model. Fetches live options chain data from Yahoo Finance and visualizes IV across strikes, expirations, and moneyness.

## Features

- Fetches 1-year historical stock data via Alpaca API (candlestick charts with SMA overlays)
- Retrieves full options chains (all available expirations) from Yahoo Finance
- Computes implied volatility using Black-Scholes pricing and `scipy.optimize.brentq` solver
- Compares calculated IV against Yahoo Finance's provided IV values
- Interactive Plotly visualizations:
  - Candlestick chart with 5/20/50-day SMAs
  - 2D scatter: Strike Price vs Implied Volatility (calls and puts, colored by type and expiration)
  - 3D surface: Strike Price vs Time to Expiry vs IV (cubic-interpolated)
  - 3D surface: Moneyness (S/K) vs Time to Expiry vs IV with ATM reference line

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

## Output

Generated plots are saved to the `plots/` directory as PNG files.
