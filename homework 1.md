```python
!pip install pandas lxml pandas_datareader setuptools plotly yfinance
```

Question 1: [Index] S&P 500 Stocks Added to the Index
Which year had the highest number of additions?

Using the list of S&P 500 companies from Wikipedia's S&P 500 companies page, download the data including the year each company was added to the index.

Hint: you can use pandas.read_html to scrape the data into a DataFrame.

Steps:

Create a DataFrame with company tickers, names, and the year they were added.
Extract the year from the addition date and calculate the number of stocks added each year.
Which year had the highest number of additions (1957 doesn't count, as it was the year when the S&P 500 index was founded)? Write down this year as your answer (the most recent one, if you have several records).
Context:

"Following the announcement, all four new entrants saw their stock prices rise in extended trading on Friday" - recent examples of S&P 500 additions include DASH, WSM, EXE, TKO in 2025 (Nasdaq article).

Additional: How many current S&P 500 stocks have been in the index for more than 20 years? When stocks are added to the S&P 500, they usually experience a price bump as investors and index funds buy shares following the announcement.

```python
# IMPORTS
import numpy as np
import pandas as pd

#Fin Data Sources
import yfinance as yf
import pandas_datareader as pdr

#Data viz
import plotly.graph_objs as go
import plotly.express as px

import time
from datetime import date

```

```python
import pandas as pd

# Scrape the S&P 500 companies table from Wikipedia
url = "https://en.wikipedia.org/wiki/List_of_S%26P_500_companies"
tables = pd.read_html(url)
sp500_df = tables[0]  # The first table contains the list of S&P 500 companies

# Display the first few rows
sp500_df.head()

```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Symbol</th>
      <th>Security</th>
      <th>GICS Sector</th>
      <th>GICS Sub-Industry</th>
      <th>Headquarters Location</th>
      <th>Date added</th>
      <th>CIK</th>
      <th>Founded</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MMM</td>
      <td>3M</td>
      <td>Industrials</td>
      <td>Industrial Conglomerates</td>
      <td>Saint Paul, Minnesota</td>
      <td>1957-03-04</td>
      <td>66740</td>
      <td>1902</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AOS</td>
      <td>A. O. Smith</td>
      <td>Industrials</td>
      <td>Building Products</td>
      <td>Milwaukee, Wisconsin</td>
      <td>2017-07-26</td>
      <td>91142</td>
      <td>1916</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ABT</td>
      <td>Abbott Laboratories</td>
      <td>Health Care</td>
      <td>Health Care Equipment</td>
      <td>North Chicago, Illinois</td>
      <td>1957-03-04</td>
      <td>1800</td>
      <td>1888</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ABBV</td>
      <td>AbbVie</td>
      <td>Health Care</td>
      <td>Biotechnology</td>
      <td>North Chicago, Illinois</td>
      <td>2012-12-31</td>
      <td>1551152</td>
      <td>2013 (1888)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ACN</td>
      <td>Accenture</td>
      <td>Information Technology</td>
      <td>IT Consulting &amp; Other Services</td>
      <td>Dublin, Ireland</td>
      <td>2011-07-06</td>
      <td>1467373</td>
      <td>1989</td>
    </tr>
  </tbody>
</table>
</div>

```python
sp500_df['Year added'] = pd.to_datetime(sp500_df['Date added'], errors='coerce').dt.year
```

```python
#quick check
year_added = sp500_df['Year added']
valid_years = year_added.dropna()
valid_years = valid_years[(valid_years >= 1900) & (valid_years <= 2025)]
if len(valid_years) == len(sp500_df):
    print("data valid")
else:
    print("check your data")

```

    data valid

```python
# Count the number of companies added each year (excluding 1957)
year_counts = sp500_df[sp500_df['Year added'] != 1957]['Year added'].value_counts()

# Get the top years with the most additions
year_counts.nlargest(1)
print("Q: Year had the highest number of addition\nA:", year_counts.nlargest(1).index[0])

```

    Q: Year had the highest number of addition
    A: 2017

```python
#add the number of years
sp500_df['Years in sp500'] = abs((datetime.today() - pd.to_datetime(sp500_df['Date added'], errors='coerce')).dt.days) // 365

# Count the number of companies been index for more than 20 years
more_than_20_years = sp500_df[sp500_df['Years in sp500'] > 20]
print("Q: How many current S&P 500 stocks have been in the index for more than 20 years?\nA:", len(more_than_20_years))
```

    Q: How many current S&P 500 stocks have been in the index for more than 20 years?
    A: 215

Question 2. [Macro] Indexes YTD (as of 1 May 2025)
How many indexes (out of 10) have better year-to-date returns than the US (S&P 500) as of May 1, 2025?

Using Yahoo Finance World Indices data, compare the year-to-date (YTD) performance (1 January-1 May 2025) of major stock market indexes for the following countries:

United States - S&P 500 (^GSPC)
China - Shanghai Composite (000001.SS)
Hong Kong - HANG SENG INDEX (^HSI)
Australia - S&P/ASX 200 (^AXJO)
India - Nifty 50 (^NSEI)
Canada - S&P/TSX Composite (^GSPTSE)
Germany - DAX (^GDAXI)
United Kingdom - FTSE 100 (^FTSE)
Japan - Nikkei 225 (^N225)
Mexico - IPC Mexico (^MXX)
Brazil - Ibovespa (^BVSP)
Hint: use start_date='2025-01-01' and end_date='2025-05-01' when downloading daily data in yfinance

Context:

Global Valuations: Who's Cheap, Who's Not? article suggests "Other regions may be growing faster than the US and you need to diversify."

Reference: Yahoo Finance World Indices - https://finance.yahoo.com/world-indices/

Additional: How many of these indexes have better returns than the S&P 500 over 3, 5, and 10 year periods? Do you see the same trend? Note: For simplicity, ignore currency conversion effects.)

```python
from datetime import date, timedelta

def get_report_by_date(data,ticker_to_name, start=(date.today() - timedelta(days=1)).strftime("%Y-%m-%d"), end=date.today().strftime("%Y-%m-%d")):
    #set the start date jan
    print(f'Period for indexes: {start} to {end} ')
    data_prepped = data[(data.index >= start) & (data.index <= end)]

    # Rename columns in data for easier reading
    data_renamed = data_prepped.rename(columns=ticker_to_name)

    #data_renamed.head()

    # Calculate YTD return for each index
    ytd_returns = (data_renamed.iloc[-1] - data_renamed.iloc[0]) / data_renamed.iloc[0] * 100

    # Sort by return descending
    ytd_returns_sorted = ytd_returns.sort_values(ascending=False)
    ytd_returns_ranked = ytd_returns_sorted.reset_index()
    ytd_returns_ranked.columns = ['Index', 'YTD Return (%)']
    ytd_returns_ranked['Rank'] = ytd_returns_ranked.index + 1
    ytd_returns_ranked = ytd_returns_ranked[['Rank', 'Index', 'YTD Return (%)']]
    ytd_returns_ranked['YTD Return (%)'] = ytd_returns_ranked['YTD Return (%)'].map('{:.2f}%'.format)
    print(ytd_returns_ranked.to_string(index=False))
```

```python
import yfinance as yf
#get data from yahoo finance
index_tickers = {
    "United States - S&P 500": "^GSPC",
    "China - Shanghai Composite": "000001.SS",
    "Hong Kong - HANG SENG INDEX": "^HSI",
    "Australia - S&P/ASX 200": "^AXJO",
    "India - Nifty 50": "^NSEI",
    "Canada - S&P/TSX Composite": "^GSPTSE",
    "Germany - DAX": "^GDAXI",
    "United Kingdom - FTSE 100": "^FTSE",
    "Japan - Nikkei 225": "^N225",
    "Mexico - IPC Mexico": "^MXX",
    "Brazil - Ibovespa": "^BVSP"
}

tickers = list(index_tickers.values())
data = yf.download(tickers=tickers, interval='1d')['Close']

#most index closed on the 1st of jan, so using ffill to add extra data
data_filled = data.ffill(axis=0)

# Reverse the index_tickers dictionary to map ticker to name
ticker_to_name = {v: k for k, v in index_tickers.items()}
```

    [*********************100%***********************]  11 of 11 completed

```python

# get data from 1st jan 2025 to 1st may 2025
get_report_by_date(data_filled,ticker_to_name,'2025-01-01','2025-05-01')
```

    Period for indexes: 2025-01-01 to 2025-05-01
     Rank                       Index YTD Return (%)
        1         Mexico - IPC Mexico         13.62%
        2               Germany - DAX         13.00%
        3           Brazil - Ibovespa         12.29%
        4 Hong Kong - HANG SENG INDEX         10.27%
        5   United Kingdom - FTSE 100          3.96%
        6            India - Nifty 50          2.49%
        7  Canada - S&P/TSX Composite          0.27%
        8     Australia - S&P/ASX 200         -0.17%
        9  China - Shanghai Composite         -2.17%
       10     United States - S&P 500         -4.72%
       11          Japan - Nikkei 225         -8.63%

Bonus question

```python
# 3 years ago
today = date.today().strftime("%Y-%m-%d")
three_years_ago = (date.today() - timedelta(days=3*365)).strftime("%Y-%m-%d")
get_report_by_date(data_filled,ticker_to_name,three_years_ago,today)
```

    Period for indexes: 2022-06-08 to 2025-06-07
     Rank                       Index YTD Return (%)
        1               Germany - DAX         68.24%
        2            India - Nifty 50         52.87%
        3     United States - S&P 500         45.79%
        4          Japan - Nikkei 225         33.67%
        5  Canada - S&P/TSX Composite         27.11%
        6           Brazil - Ibovespa         25.59%
        7     Australia - S&P/ASX 200         19.58%
        8         Mexico - IPC Mexico         16.54%
        9   United Kingdom - FTSE 100         16.40%
       10 Hong Kong - HANG SENG INDEX          8.08%
       11  China - Shanghai Composite          3.72%

```python
# 5 years ago
today = date.today().strftime("%Y-%m-%d")
three_years_ago = (date.today() - timedelta(days=5*365)).strftime("%Y-%m-%d")
get_report_by_date(data_filled,ticker_to_name,three_years_ago,today)
```

    Period for indexes: 2020-06-08 to 2025-06-07
     Rank                       Index YTD Return (%)
        1            India - Nifty 50        145.91%
        2               Germany - DAX         89.59%
        3     United States - S&P 500         85.63%
        4  Canada - S&P/TSX Composite         65.44%
        5          Japan - Nikkei 225         62.83%
        6         Mexico - IPC Mexico         45.32%
        7     Australia - S&P/ASX 200         41.96%
        8           Brazil - Ibovespa         39.38%
        9   United Kingdom - FTSE 100         36.54%
       10  China - Shanghai Composite         15.24%
       11 Hong Kong - HANG SENG INDEX         -3.97%

```python
# 10 years ago
today = date.today().strftime("%Y-%m-%d")
three_years_ago = (date.today() - timedelta(days=10*365)).strftime("%Y-%m-%d")
get_report_by_date(data_filled,ticker_to_name,three_years_ago,today)
```

    Period for indexes: 2015-06-10 to 2025-06-07
     Rank                       Index YTD Return (%)
        1            India - Nifty 50        207.75%
        2     United States - S&P 500        185.03%
        3           Brazil - Ibovespa        152.62%
        4               Germany - DAX        115.74%
        5          Japan - Nikkei 225         88.27%
        6  Canada - S&P/TSX Composite         77.51%
        7     Australia - S&P/ASX 200         55.44%
        8         Mexico - IPC Mexico         30.25%
        9   United Kingdom - FTSE 100         29.39%
       10 Hong Kong - HANG SENG INDEX        -10.85%
       11  China - Shanghai Composite        -33.70%

Question 3. [Index] S&P 500 Market Corrections Analysis
Calculate the median duration (in days) of significant market corrections in the S&P 500 index.

For this task, define a correction as an event when a stock index goes down by more than 5% from the closest all-time high maximum.

Steps:

Download S&P 500 historical data (1950-present) using yfinance
Identify all-time high points (where price exceeds all previous prices)
For each pair of consecutive all-time highs, find the minimum price in between
Calculate drawdown percentages: (high - low) / high × 100
Filter for corrections with at least 5% drawdown
Calculate the duration in days for each correction period
Determine the 25th, 50th (median), and 75th percentiles for correction durations
Context:

Investors often wonder about the typical length of market corrections when deciding "when to buy the dip" (Reddit discussion).
A Wealth of Common Sense - How Often Should You Expect a Stock Market Correction?
Hint (use this data to compare with your results): Here is the list of top 10 largest corrections by drawdown:

2007-10-09 to 2009-03-09: 56.8% drawdown over 517 days
2000-03-24 to 2002-10-09: 49.1% drawdown over 929 days
1973-01-11 to 1974-10-03: 48.2% drawdown over 630 days
1968-11-29 to 1970-05-26: 36.1% drawdown over 543 days
2020-02-19 to 2020-03-23: 33.9% drawdown over 33 days
1987-08-25 to 1987-12-04: 33.5% drawdown over 101 days
1961-12-12 to 1962-06-26: 28.0% drawdown over 196 days
1980-11-28 to 1982-08-12: 27.1% drawdown over 622 days
2022-01-03 to 2022-10-12: 25.4% drawdown over 282 days
1966-02-09 to 1966-10-07: 22.2% drawdown over 240 days

```python
import numpy as np
from datetime import datetime
```

```python
# Download S&P 500 historical data (1950-present)
sp500 = yf.download("^GSPC", start="1950-01-01")['Close']

# Find all-time highs
all_time_highs = sp500.cummax()

# Find the points where a new all-time high is reached
ath_mask = sp500 == all_time_highs
ath_dates = sp500[ath_mask].dropna().index #= sp500[ath_mask].index

# For each pair of consecutive ATHs, find the minimum price in between
corrections = []
for i in range(len(ath_dates) - 1):
    start = ath_dates[i]
    end = ath_dates[i+1]
    # Only consider periods where there is at least one day between ATHs
    if (end - start).days > 1:
        period = sp500.loc[start:end]
        min_price = period.min().iloc[0]
        min_date = period.idxmin().iloc[0]
        max_price = period.max().iloc[0]
        drawdown = (max_price - min_price) / max_price * 100
        duration = (min_date - start).days
        if drawdown >= 5:
            corrections.append({
                'start': start,
                'min_date': min_date,
                'end': end,
                'drawdown_%': drawdown,
                'duration_days': duration
            })

# Convert to DataFrame
corrections_df = pd.DataFrame(corrections)

# Calculate percentiles
percentiles = corrections_df['duration_days'].quantile([0.25, 0.5, 0.75]).astype(int)
print("Correction duration percentiles (in days):")
print(percentiles)

# Show the median duration
print(f"\nMedian duration (days): {percentiles.loc[0.5]}")

# Show top 10 largest corrections by drawdown
print("\nTop 10 largest corrections by drawdown:")
print(corrections_df.sort_values('drawdown_%', ascending=False).head(10))
```

    [*********************100%***********************]  1 of 1 completed


    Correction duration percentiles (in days):
    0.25    19
    0.50    34
    0.75    76
    Name: duration_days, dtype: int64

    Median duration (days): 34

    Top 10 largest corrections by drawdown:
            start   min_date        end  drawdown_%  duration_days
    66 2007-10-09 2009-03-09 2013-03-28   56.886671            517
    64 2000-03-24 2002-10-09 2007-05-30   49.239002            929
    26 1973-01-11 1974-10-03 1980-07-17   48.715417            630
    23 1968-11-29 1970-05-26 1972-03-06   36.296770            543
    78 2020-02-19 2020-03-23 2020-08-18   33.995720             33
    39 1987-08-25 1987-12-04 1989-07-26   33.761276            101
    29 1980-11-28 1982-08-12 1982-11-03   28.312451            622
    16 1961-12-12 1962-06-26 1963-09-03   27.993398            196
    81 2022-01-03 2022-10-12 2024-01-19   26.091520            282
    19 1966-02-09 1966-10-07 1967-05-04   22.391860            240

Question 4. [Stocks] Earnings Surprise Analysis for Amazon (AMZN)
Calculate the median 2-day percentage change in stock prices following positive earnings surprises days.

Steps:

1. Load earnings data from CSV (ha1_Amazon.csv) containing earnings dates, EPS estimates, and actual EPS. Make sure you are using the correct delimiter to read the data, such as in this command python pandas.read_csv("ha1_Amazon.csv", delimiter=';')
2. Download complete historical price data using yfinance
3. Calculate 2-day percentage changes for all historical dates: for each sequence of 3 consecutive trading days (Day 1, Day 2, Day 3), compute the return as Close_Day3 / Close_Day1 - 1. (Assume Day 2 may correspond to the earnings announcement.)
4. Identify positive earnings surprises (where "actual EPS > estimated EPS"). Both fields should be present in the file. You should obtain 36 data points for use in the descriptive analysis (median) later.
5. Calculate 2-day percentage changes following positive earnings surprises. Show your answer in % (closest number to the 2nd digit): return \* 100.0
6. (Optional) Compare the median 2-day percentage change for positive surprises vs. all historical dates. Do you see the difference?
   Context: Earnings announcements, especially when they exceed analyst expectations, can significantly impact stock prices in the short term.

Reference: Yahoo Finance earnings calendar - https://finance.yahoo.com/calendar/earnings?symbol=AMZN

Additional: Is there a correlation between the magnitude of the earnings surprise and the stock price reaction? Does the market react differently to earnings surprises during bull vs. bear markets?)

```python
from datetime import datetime

```

```python
# 1. Load earnings data
earnings = pd.read_csv("ha1_Amazon.csv", delimiter=';')
earnings['test_date'] = pd.to_datetime(earnings['Earnings Date'], format='mixed')
earnings['Date'] = pd.to_datetime(earnings['test_date'].dt.strftime('%Y-%m-%d'))


# 2. Download historical price data
amzn = yf.download("AMZN",interval='1d',period='max')
amzn = amzn[['Close']].reset_index()
amzn['Date'] = pd.to_datetime(amzn['Date'])
amzn.columns = amzn.columns.droplevel(1)

```

```python
#sanity check
len(earnings), len(amzn)
```

    (117, 7060)

```python
merged_data = pd.merge(amzn, earnings, on='Date', how='left')
merged_data['2_day_return'] = 0.0

date_match_df = amzn['Date'].isin(earnings['Date']).reindex(amzn.index, fill_value=False)

merged_data['cleaned_actual_eps'] = pd.to_numeric(merged_data['Reported EPS'].str.replace('$', '', regex=False), errors='coerce')
merged_data['cleaned_estimate_eps'] = pd.to_numeric(merged_data['EPS Estimate'].str.replace('$', '', regex=False), errors='coerce')
merged_data['cleaned_actual_eps'].fillna(0, inplace=True)
merged_data['cleaned_estimate_eps'].fillna(0, inplace=True)

#calculate the 2-day return around the earnings date
for i in range(len(date_match_df)-1):
    if date_match_df[i] == True:
        close_day = merged_data['Close'][i]
        close_day_after_2 = merged_data['Close'][i+2] if i + 2 < len(merged_data) else None
        earning_day_rate = close_day_after_2 / close_day - 1 if close_day is not None else None
        merged_data.loc[i,'2_day_return'] = earning_day_rate

    i+= 1
```

```python
#sanity check
len(merged_data)
```

    7060

```python

positive_earning_surpises = merged_data[merged_data['cleaned_actual_eps'] > merged_data['cleaned_estimate_eps']]
# Calculate percentiles
percentiles = positive_earning_surpises['2_day_return'].quantile([0.25, 0.5, 0.75]).astype(float)
print("positive_earning_surpises percentiles:")
print(percentiles*100)

# Show the median duration
print(f"\nMedian positive_earning_surpises: {percentiles.loc[0.5]*100}")
```

```python
all_amzn_data = merged_data.copy()
```

```python
#calculate the 2-day return around the earnings date for all_amzn_data
for i in range(len(date_match_df)-1):

    close_day = all_amzn_data['Close'][i]
    close_day_after_2 = all_amzn_data['Close'][i+2] if i + 2 < len(all_amzn_data) else 0
    earning_day_rate = close_day_after_2 / close_day - 1 if close_day is not None else None
    all_amzn_data.loc[i,'2_day_return'] = earning_day_rate

    i+= 1

# Calculate percentiles
percentiles = all_amzn_data['2_day_return'].quantile([0.25, 0.5, 0.75]).astype(float)
print("positive_earning_surpises percentiles:")
print(percentiles*100)

# Show the median duration
print(f"\nMedian positive_earning_surpises: {percentiles.loc[0.5]*100}")
```

    positive_earning_surpises percentiles:
    0.25   -1.802157
    0.50    0.165817
    0.75    2.151773
    Name: 2_day_return, dtype: float64

    Median positive_earning_surpises: 0.16581674487468057

```python

```
