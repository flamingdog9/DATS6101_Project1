# DATS6101_Project1
This is where the code for the DATS6101 Project 1 will go
Using listing prices across all metro areas from Realtor.com, which sources its information from MLS-listed for-sale homes in the USA, questions we want to explore are:
1. How much does the listing price vary based on the number of days it's on the market? If it’s listed for longer will the listing price fluctuate more?
2. How much does the listing prices vary based on its listed metro area? How much does it vary over the average?
3. Does the metro area relate to the number of price increases/reductions made by a realtor? Are there more increases in more densely populated areas?
4. Does the total amount of listings in a metro area fluctuate during the year? Is there an increase during important times (Ex. start of school/summer)?

## Claude's suggestions and changes to the questions 
**(If we use AI suggestions, we just have to understand them, put it in our own words in the project)**
Although we are using technically the same questions we came up, with Claude here is just:
1. Suggesting changes needed to make them SMART.
2. What Featured Columns would have to be made to answer.
3. What graphs and tests would be best for them.


All four use your **Jun 2023–Aug 2026** window.

### 1. Days on market vs. price variation

**Fix needed:** the data can't follow a single listing over time, so ask this at the metro level.

**SMART version:** From Jun 2023 to Aug 2026, do metros with a longer average `median_days_on_market` have more month-to-month variation in their median listing price?

- **Existing columns:** `cbsa_code`, `month_date_yyyymm`, `median_days_on_market`, `median_listing_price_mm` (the month-over-month price change already in the file; check whether it's stored as a percent or a decimal)
- **New columns:**
  - average days on market for each metro
  - price volatility for each metro (the standard deviation of its monthly price changes)
  - optional: a days-on-market tier (short, medium or long, splitting metros into equal thirds)
- **Tests:**
  - Pearson or Spearman correlation, after a normality check
  - or ANOVA comparing volatility across the three tiers
  - Levene's test if you're comparing spread between tiers
- **Graphs:** a scatterplot of days on market vs. volatility with a fitted line; box plots of volatility by tier

### 2. Price differences across metros

**Fix needed:** comparing about 900 metros one by one will show "they differ", which isn't useful. Group them first.

**SMART version:** From Jun 2023 to Aug 2026, does the average median listing price differ across the four Census regions, and how far above or below the national average does each metro sit?

- **Existing columns:** `cbsa_title` (to get the state), `median_listing_price`, and optionally `median_listing_price_per_square_foot` (adjusts for home size)
- **New columns:**
  - region, from the state abbreviation (use the first state for multi-state metros)
  - average price for each metro
  - % difference from the national average
  - log price, because prices are right-skewed
- **Tests:**
  - ANOVA + Tukey on log price (or Kruskal-Wallis if normality fails)
  - standard deviation and coefficient of variation for each region
  - Levene's test to check whether some regions vary more than others
- **Graphs:**  
  - box plots by region;
  - a histogram of % difference from the national average;
  - a bar chart of the 10 most and 10 least expensive metros

### 3. Price increases and reductions by metro

**Fix needed:** the dataset has no population-density column. `HouseholdRank` measures metro **size** (number of households), not density. Also:

- Use **shares**, not counts. Big metros will always have more price changes simply because they have more listings.
- The data counts listings whose price changed, not what individual realtors did.

**SMART version:** From Jun 2023 to Aug 2026, does the average share of listings with price reductions (or increases) differ by metro size?

- **Existing columns:** `price_reduced_share`, `price_increased_share`, `HouseholdRank`
- **New columns:**
  - size group (quartiles of `HouseholdRank`, or your Top 100 / Bottom 100 groups)
  - average shares for each metro
  - a yes/no flag for whether the metro's price-cut share is above the overall median
- **Tests:**
  - ANOVA + Tukey across size groups (or a t-test for Top vs. Bottom 100)
  - chi-square test of size group × the high-cut flag
  - Spearman correlation between rank and share
- **Graphs:**  
  - box plots of each share by size group;
  - a stacked proportion bar for the chi-square test
**Note:** price increases are rare, so `price_increased_share` will be small and skewed. Check normality before running tests on it.

### 4. Seasonal changes in listings

**Verdict:** the strongest of the four, and the dataset suits it well.

**SMART version:** From Jun 2023 to Aug 2026, do listings vary by season, and are they higher in summer (Jun–Aug) than in winter (Dec–Feb)?

- **Existing columns:** `month_date_yyyymm`, `cbsa_code`, `active_listing_count`, and `new_listing_count` (better for "homes coming onto the market")
- **New columns:**
  - calendar month (1–12)
  - season label, plus a "school start" (Aug–Sep) label if you want to test that period
  - market year (Jun–May), which gives three full years inside your window
  - **indexed listings:** each month's count divided by that metro's average for the same market year, so big metros don't dominate and the overall upward inventory trend doesn't hide the seasonal pattern
- **Tests:**
  - paired t-test of summer vs. winter within each metro (the cleanest option, because each metro is compared with itself)
  - or ANOVA + Tukey across seasons or months
- **Graphs:** ;
  - a line chart of monthly listings across the whole window (the three yearly cycles should show clearly)
  - box plots of indexed listings by month

**How the set covers the rubric:**

- Q1 → correlation
- Q2 → ANOVA and measures of variance
- Q3 → chi-square
- Q4 → t-test

