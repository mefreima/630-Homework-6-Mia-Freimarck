# Plan: Is There Really a "Lipstick Effect"?

## 1. Question and Expectation

**Question:** Does spending at beauty, cosmetics and perfume stores hold up better than overall retail spending during economic downturns?

**Background:** The "lipstick effect" says that when the economy weakens, people cut big purchases but keep buying small, affordable luxuries like cosmetics.

**Expectation:** I expect beauty store sales to fall less than total retail during the 2008-09 recession, so beauty's share of total retail should hold steady or rise. I expect 2020 to look different. Store closures hit beauty stores directly, so the lipstick effect may not show up in that downturn. With the data now at annual frequency (see Section 2), I can also check the 1990-91 and 2001 recessions as additional, independent tests of the same idea.

## 2. Data Sources (3 series, 2 sources)

**Note on frequency change:** I originally planned to pull beauty-store sales from the Census Monthly Retail Trade Survey (MARTS) at NAICS 446120/456120. I verified this directly against the live Census API (`api.census.gov/data/timeseries/eits/marts`) and confirmed that MARTS does **not** publish a cosmetics/beauty/perfume-specific series at any frequency — the finest breakout available monthly is the 3-digit subsector "446 Health and Personal Care Stores," which is dominated by pharmacies and isn't a clean beauty measure. The only public cosmetics-specific series I could find is **annual**, from BLS via FRED, so the analysis below moves to annual frequency.

| Series | Source | How I pull it |
|---|---|---|
| Cosmetics, beauty supplies and perfume store output, current dollars (NAICS 44612), annual, 1987-2025 | BLS Sectoral Output for Retail Trade, via FRED (`IPUHN44612T300000000`) | FRED API, `fredapi` package |
| Total retail trade output, current dollars (NAICS 44-45), annual, 1987-2025, same BLS methodology | BLS Sectoral Output for Retail Trade, via FRED (`IPUHN4445T300000000`) | FRED API, `fredapi` package |
| Unemployment rate (UNRATE), annual average | FRED | FRED API, `fredapi` package, resampled monthly → annual mean |

**Credentials:** The FRED API key goes in a `.env` file as `FRED_API_KEY`. It is loaded with `python-dotenv`, and `.env` is listed in `.gitignore`. (No Census key is needed now that the plan no longer pulls from MARTS.)

**To verify before coding:**
- Confirm both BLS sectoral-output series use the same units (millions of current dollars) and vintage so "beauty share of total retail" is a valid ratio — spot-checked already: both are in millions of current dollars, not seasonally adjusted (annual data has no seasonality to adjust).
- Pull the cosmetics-specific CPI (`CUUR0000SEGB02`: Cosmetics, Perfume, Bath, Nail Preparations and Implements, from FRED) so output can optionally be deflated to real dollars using a category-specific price index rather than economy-wide CPI-U (see Limitations).

## 3. Cleaning Steps

- Pull both BLS output series and UNRATE from 1987 (earliest beauty-series year) to the latest available year.
- Convert to an annual (year) index and values to numeric.
- Resample monthly UNRATE to an annual average to match the other two series' frequency.
- Merge the three series on year.
- Check for missing years or gaps between the two BLS series (they should share the same vintage/coverage since both come from the same BLS release). Drop or note any I find.

**Transformations:**
- Year-over-year percent growth for beauty and total retail output.
- Beauty share of total retail: beauty output divided by total retail output.
- Recession-year indicator (1 if any month of that calendar year falls inside an NBER recession: 1990, 1991, 2001, 2007, 2008, 2009, 2020), using NBER dates.
- Year-over-year change in the annual-average unemployment rate.
- Optional: deflate both output series by CPI to check whether results hold in real terms.

## 4. Charts and Tests

**Charts**
- Line chart of year-over-year growth for beauty and total retail output, 1988-2025, with recession years shaded.
- Line chart of beauty's share of total retail output, with recession years shaded.
- Scatter plot of beauty minus total retail growth against the year-over-year change in annual unemployment rate.
- Bar chart of average growth in beauty vs. total retail across all recession years compared to non-recession years.

**Tests**
- Compare average growth gap (beauty minus total retail) in recession years vs. non-recession years, using a t-test.
- Regress the growth gap on the change in unemployment, plus a recession-year dummy, using `statsmodels` with HAC (Newey-West) standard errors (`cov_type='HAC'`), since this is a time series and residuals may be autocorrelated year to year.
- Run each recession episode (1990-91, 2001, 2008-09, 2020) separately as well as pooled, so one does not hide the others. With annual data there are now four episodes instead of two, which helps statistical power somewhat, but each episode still only contributes 1-3 annual observations — treat any single-episode result as suggestive, not conclusive, and lean on the pooled result for the headline finding.

## 5. What Supports or Contradicts the Expectation

**Supports the lipstick effect:**
- Beauty growth falls less than total retail growth in recessions.
- Beauty's share of total retail rises or stays flat during 2008-09.
- The regression coefficient on unemployment change is negative and statistically significant (beauty gains ground as unemployment rises).

**Contradicts it:**
- Beauty falls as much as or more than total retail in downturns.
- Beauty's share drops in recessions.
- The coefficient is zero or positive.

**Limitations to discuss in the conclusion:**
- The data is annual, not monthly, because no public source publishes a monthly cosmetics/beauty-specific sales series (confirmed by directly querying the Census MARTS API — see Section 2). Annual bins blur the timing within a recession year and can't isolate a 3-month episode like 2020's.
- The recession-year indicator is crude: a year is flagged "recession" if any month in it falls in an NBER window, so a year that was only in recession for one month (e.g., 2007, 2020) is coded the same as a year that was in recession for all 12 (e.g., 2008, 2009).
- 2020 was driven by lockdowns, not income shocks, so it is a weaker substantive test of the idea even though it's now coded the same as any other recession year.
- This BLS sectoral-output series measures output for the cosmetics/beauty/perfume store industry, not retail sales in the exact Census MARTS sense — the two are closely related but not identical concepts, and output excludes cosmetics sold through other channels (online, department stores, drugstores).
- The series is in nominal dollars, so price changes could affect the results — I will check this with a CPI-deflated version if time allows.

## 6. Repo and Workflow Checklist

- [ ] Public GitHub repo with `.gitignore` excluding `.env`
- [ ] Commits in stages: plan, data pull, cleaning, charts/tests, conclusion
- [ ] One Jupyter notebook that pulls all data from the APIs, loads keys from `.env`, and runs top to bottom
- [ ] Markdown cells explaining each step
- [ ] Code reviewed by me: clear names, functions for repeated steps, no unused code
