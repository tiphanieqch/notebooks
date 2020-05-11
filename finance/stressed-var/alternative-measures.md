# Alternative Measures

Alternative Tail Measure calculations

## Interpolated VaR

Instead of taking the next lowest P&L value, interpolate between two P&L values

```
kernel = np.zeros(window_size)
quantile_rank = quantile * window_size
quantile_rank_ceil = math.ceil(quantile_rank)
weight = quantile_rank - quantile_rank_ceil + 1
kernel[quantile_rank_ceil - 1] = 1 - weight
kernel[quantile_rank_ceil] = weight
```

## Expected Shortfall (ES)

Replace VaR kernel with ES kernel (e.g. first $k$ values are $\frac{1}{k}$)

```
kernel = np.zeros(window_size)
quantile_rank = math.floor(quantile * window_size)
kernel[0:quantile_rank] = np.ones(quantile_rank) / quantile_rank
```

*Note: this may also benefit from interpolation at the cut-off P&L value.*

## Harrell-Davis Quantile Estimator

Use $Beta$ function, to produce a kernel that can be used for the Harrell-Davis Quantile Estimator.

```
alpha = quantile * (window_size + 1)
beta = (1 - quantile) * (window_size + 1)

beta_x = np.arange(window_size + 1)/float(window_size)
cdf = stats.beta.cdf(beta_x, alpha, beta)

# difference between consecutive CDF values
kernel = cdf[1:] - cdf[:-1]
```