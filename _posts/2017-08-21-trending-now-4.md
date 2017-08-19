---
layout: post
title: "Trending Now (Pt. 4): Imperfect Ratios"
date: 2017-08-21
categories: [Finance & Economics]
---

With wildly varying market capitalizations, dividend payments, and trading volumes, it's often difficult to objectively compare securities. Out of this difficulty, mathemeticians and financial practictioners have developed an array of ratios to judge stocks and portfolios -- commonly known as performance indicators. Here we'll take a look at a few risk-related indicators a high level, and implement them in Python when applicable.

**Sharpe Ratio**

In 1966, William F. Sharpe introduced a "reward-to-variability ratio" that gained prominence as the Sharpe ratio. He based his work on Markowitz's mean-variance paradigm: the success of a portfolio can be predicted by time-dependent mean and standard deviation return alone.

Although the ratio comes in predictive and historical flavors, it assumes that past results have bearings on the future of a portfolio. It also relies on the existence of linear risk and a normal distribution of expected returns, limiting its application. Finally, the ratio must be applied to a zero-investment strategy. This refers to a strategy of decreasing exposure to a certain security by buying and selling/shorting the same security (net zero value).

Here we'll use the ex-post (historical) Sharpe ratio. We begin by calculating the differential return in a given period *t* by subtracting the return on a benchmark fund (risk free rate, S&P returns, etc) from the return on our fund.

$$ D_t = R_{F_t} - R_{B_t} $$

Next we'll take the average value of D<sub>t</sub> over period *t*.


$$ {\bar{D}} = \frac{1}{T} \sum_{t=1}^{T}  D_t\ $$

To get the Sharpe ratio, we'll divide the average of D<sub>t</sub> over *t* by its standard deviation over *t*.

$$ {S_h} = \frac{\bar{D}}{\sigma_d} $$

Next, we'll look at the Sharpe Ratio in Python by calculating it for an algorithmically-generated portfolio, saved as a csv.


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from pandas_datareader import data, wb
from datetime import date
%matplotlib inline
```


```python
def generateAvgRet(stockDataFrame):
    """
    Given a dataframe of stock information, calculate and add Average Daily Return and Volatility
    """
    stockDataFrame['Daily Ret'] = 100 * ((stockDataFrame['Close']-stockDataFrame['Close'].shift(1))/stockDataFrame['Close'].shift(1))
    stockDataFrame['Avg Daily Ret'] = (stockDataFrame['Daily Ret']).rolling(252).mean()
    stockDataFrame['Portfolio Std'] = (stockDataFrame['Daily Ret']).rolling(252).std()
    return stockDataFrame

def generateSharpe(stockDataFrame):
    """
    Given a dataframe of stock information, calculate and add the Sharpe ratio
    """
    stockDataFrame['Sharpe'] = np.sqrt(252) * stockDataFrame['Avg Daily Ret'] / stockDataFrame['Portfolio Std']
    return stockDataFrame
```


```python
testPortfolio = data.DataReader('CRM','google',date(2015,1,1), date(2017,1,1))
generateAvgRet(testPortfolio)
generateSharpe(testPortfolio).tail()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Daily Ret</th>
      <th>Avg Daily Ret</th>
      <th>Portfolio Std</th>
      <th>Sharpe</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-12-23</th>
      <td>69.69</td>
      <td>70.01</td>
      <td>69.31</td>
      <td>69.87</td>
      <td>2274299</td>
      <td>-0.042918</td>
      <td>-0.020349</td>
      <td>2.113983</td>
      <td>-0.152806</td>
    </tr>
    <tr>
      <th>2016-12-27</th>
      <td>69.84</td>
      <td>70.43</td>
      <td>69.64</td>
      <td>69.85</td>
      <td>2939715</td>
      <td>-0.028625</td>
      <td>-0.022349</td>
      <td>2.113751</td>
      <td>-0.167844</td>
    </tr>
    <tr>
      <th>2016-12-28</th>
      <td>69.85</td>
      <td>70.06</td>
      <td>68.85</td>
      <td>69.09</td>
      <td>4313702</td>
      <td>-1.088046</td>
      <td>-0.032959</td>
      <td>2.112357</td>
      <td>-0.247689</td>
    </tr>
    <tr>
      <th>2016-12-29</th>
      <td>68.85</td>
      <td>69.82</td>
      <td>68.77</td>
      <td>69.15</td>
      <td>3229058</td>
      <td>0.086843</td>
      <td>-0.031116</td>
      <td>2.112258</td>
      <td>-0.233849</td>
    </tr>
    <tr>
      <th>2016-12-30</th>
      <td>69.36</td>
      <td>69.38</td>
      <td>68.23</td>
      <td>68.46</td>
      <td>4523271</td>
      <td>-0.997831</td>
      <td>-0.031365</td>
      <td>2.112368</td>
      <td>-0.235709</td>
    </tr>
  </tbody>
</table>
</div>



**Sortino Ratio**
While the Sharpe ratio treats positive and negative volatility equally, the Sortino ratio only accounts for negative volatility in its calculation. Dr. Frank Sortino saw no merit in punishing a portfolio for outperforming expectations, and in 1981 published a new ratio. His calculations used a target rate of return, and counted all "upside" deviations from this target as 0. The target can be self-selected (and is often 0%), but must be kept consistent.

Naturally, the Sortino Ratio appears similar to the Sharpe Ratio. But let's take a look.

First, the calculation of the "target downside deviation" -- deviation from the target return that represents underperformance.

$$ TDD = \frac{1}{N} \sum_{i=1}^{N} (\min(0,X_i - T)^2 $$

To find the ratio itself, find R, the average period return. Then subtract the target rate of return and divide by your target downside deviation.

$$ S = \frac{R - T}{TDD} $$

Nice! Now let's get back to Python and calculate the Sortino ratio for our portfolio.


```python
def generateSortino(stockDataFrame):
    """
    Given a dataframe of stock information, calculate and add the Sortino ratio
    """
    stockDataFrame['TDD'] = (np.minimum(stockDataFrame['Daily Ret'],0)).rolling(252).mean()
    stockDataFrame['Sortino'] = np.sqrt(252) * stockDataFrame['Avg Daily Ret']/stockDataFrame['TDD']
```


```python
generateSortino(testPortfolio)
testPortfolio.tail()
plt.plot(testPortfolio['Sharpe'])
plt.plot(testPortfolio['Sortino'])
plt.legend();
```


![png](http://i.imgur.com/4gARIIm.png)


Continuing the trend of portfolio analysis based on risk, we'll next take a look at the CALMAR ratio and the MAR ratio. These performance indicators take into account *maximum drawdown*, or the most a stock has fallen in a given period of time (usually the peak-to-trough percentage drop). Both come out to average rate of return divided by maximum drawdown in a given period. Although their formula is identical, the MAR takes into account stock movement since the portfolio's inception, while the CALMAR allows a more flexible timeframe.

$$MAR = \frac{R}{\max(DD)}$$ (since inception)

$$CALMAR = \frac{R}{\max(DD)}$$ (in a given timeframe)


```python
# python bits and accompanying mkdwn
def generateMAR(stockDataFrame):
    """
    Given a dataframe of stock information, calculate and add the MAR ratio
    """
    trough = np.argmax(np.maximum.accumulate(stockDataFrame['Close']) - stockDataFrame['Close'])
    peak = np.argmax(stockDataFrame['Close'][:trough])

    maxDD = stockDataFrame['Close'][peak] - stockDataFrame['Close'][trough]
    stockDataFrame['MAR'] = stockDataFrame['Avg Daily Ret']/maxDD

    plt.plot(stockDataFrame['Close'])
    plt.plot([trough, peak], [stockDataFrame['Close'][trough], stockDataFrame['Close'][peak]], ':', color='Purple', markersize=10)
    plt.plot([trough], [stockDataFrame['Close'][trough]], 'o', color='Red', markersize=10)
    plt.plot([peak], [stockDataFrame['Close'][peak]], 'o', color='Green', markersize=10)
    return stockDataFrame
```


```python
generateMAR(testPortfolio).tail()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Daily Ret</th>
      <th>Avg Daily Ret</th>
      <th>Portfolio Std</th>
      <th>Sharpe</th>
      <th>TDD</th>
      <th>Sortino</th>
      <th>MAR</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-12-23</th>
      <td>69.69</td>
      <td>70.01</td>
      <td>69.31</td>
      <td>69.87</td>
      <td>2274299</td>
      <td>-0.042918</td>
      <td>-0.020349</td>
      <td>2.113983</td>
      <td>-0.152806</td>
      <td>-0.675959</td>
      <td>0.477883</td>
      <td>-0.000724</td>
    </tr>
    <tr>
      <th>2016-12-27</th>
      <td>69.84</td>
      <td>70.43</td>
      <td>69.64</td>
      <td>69.85</td>
      <td>2939715</td>
      <td>-0.028625</td>
      <td>-0.022349</td>
      <td>2.113751</td>
      <td>-0.167844</td>
      <td>-0.676073</td>
      <td>0.524766</td>
      <td>-0.000796</td>
    </tr>
    <tr>
      <th>2016-12-28</th>
      <td>69.85</td>
      <td>70.06</td>
      <td>68.85</td>
      <td>69.09</td>
      <td>4313702</td>
      <td>-1.088046</td>
      <td>-0.032959</td>
      <td>2.112357</td>
      <td>-0.247689</td>
      <td>-0.680390</td>
      <td>0.768983</td>
      <td>-0.001173</td>
    </tr>
    <tr>
      <th>2016-12-29</th>
      <td>68.85</td>
      <td>69.82</td>
      <td>68.77</td>
      <td>69.15</td>
      <td>3229058</td>
      <td>0.086843</td>
      <td>-0.031116</td>
      <td>2.112258</td>
      <td>-0.233849</td>
      <td>-0.678892</td>
      <td>0.727581</td>
      <td>-0.001108</td>
    </tr>
    <tr>
      <th>2016-12-30</th>
      <td>69.36</td>
      <td>69.38</td>
      <td>68.23</td>
      <td>68.46</td>
      <td>4523271</td>
      <td>-0.997831</td>
      <td>-0.031365</td>
      <td>2.112368</td>
      <td>-0.235709</td>
      <td>-0.679141</td>
      <td>0.733137</td>
      <td>-0.001117</td>
    </tr>
  </tbody>
</table>
</div>




![png](http://i.imgur.com/GrxEf5P.png)


Like CALMAR and MAR, the Sterling Ratio uses a security's drawdown to gauge risk. But rather than pinpointing the maximum drawdown, this indicator uses the average drawdown over a period of time.

$$Sterling = \frac{R}{mean(DD)}$$ (in a given timeframe)

These performance indicators can be leveraged to judge and compare portfolios of securities -- in each case, the higher the ratio, the stronger the portfolio's performance.

*Works Referenced & Cited // More Reading*

[The Sharpe Ratio. William F. Sharpe Stanford University.](kantakji.com/media/4671/o121.doc)

[The Sortino Ratio: A Better Measure of Risk. Rollinger, Hoffman.](https://www.sunrisecapital.com/wp-content/uploads/2014/06/Futures_Mag_Sortino_0213.pdf)

[MAR and CALMAR ratios: Identical Twins, with Opposite Personalities](https://www.rcmalternatives.com/2013/08/mar-and-calmar-ratios-identical-twins-with-opposite-personalities/?__hstc=93618690.5dab998036779a1302754afa3b082187.1501000840889.1501000840889.1501000840889.1&__hssc=93618690.1.1501000840890&__hsfp=1695690550)
