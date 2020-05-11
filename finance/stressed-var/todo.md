# TODO

## Improve Input Files

* Remove AsOfDate column
* Consider rearranging data:
    * base store: trade, risk-factor, P&L Vector
    * mapping stores:
        * trade -> desk, product type, etc
        * risk-factor -> risk class
* Create dates store, to label indices into P&L Vectors
* Better sample data set

## Structure

Investigate import alternative measures (i.e. import the kernel)

e.g.
```
%run es.ipynb
```

## Move VaR Calculations to Atoti

Combine `sort_indices` vector with `kernel` to produce $(n - $ `window_size` $) \times $ `window_size` 2-dimensional array, with a row for each window representing the kernel rearranged for the sorted P&L vector (for that window).  So that for each window, the addative decomposition value can be calculated by taking the dot product of the P&L vector window with the row.

i.e.
```
sorted_kernel = np.array([np.zeros(window_size) for i in period_indices])
for i in period_indices:
    for j in range(window_size):
        sorted_kernel[i][sort_indices[i][j]] = kernel[j]
```
So that we could redefine:
```
def VaR_decomp_period(pl, i): return -pl[i:i+window_size] @ sorted_kernel[i]
```

Then push this `sorted_kernel` back into Atoti (e.g. a store with rows indexed by period and each row containing the vector `sorted_kernel[i]`), so that this VaR calculation can be done in Atoti.

i.e. the process is:

* (atoti) aggregate to top-of-house P&L vector
* extract agggregated P&L vector from atoti into python
* (python) sort P&L vector for each window, generate 2-D array for calculating decomposition values based on kernel and window sort orders
* push array back into atoti
* (atoti) calculate decomposition values while slicing and dicing.

*Note:* for 260 P&L values per year and 13 years history (back to 2007), `sorted_kernel` will have about 800k entries (6MB).