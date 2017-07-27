---
layout: post
title: "Trending Now (Pt. 28)"
date: 2017-07-17
---

So, we've checked out pandas and looked at stocks' ROI (return on investment) graphically. But there's a lot more to dig into with our stock-toting dataframes, and it would be a mess to use a line chart on every occassion. First let's catch up by using the customary imports:


```python
#use the typical import statements
import pandas as pd
import numpy as np
from pandas_datareader import data, wb
from datetime import date
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns
sns.set();
```

This time, we'll write a function that takes an array of tickers and creates a dictionary of stock value dataframes.


```python
def makeStockDict(tickers, start, end):
    """
    Takes an array of string ticker symbols representing different
    stocks, a datetime object representing a start date, and a
    datetime object representing an end date. Returns a dictionary,
    keys being stock tickers and values being pandas DataFrames
    with indexed by Date with fields Open, High, Low, Close, and Volume.
    """
    stocksDict = {}
    for ticker in tickers:
        value = data.DataReader(ticker,'google',start,end)
        stocksDict[ticker] = value
    return stocksDict
```


```python
businessIntelligence = makeStockDict(['CRM','ORCL','SAP'],date(2017,1,1),date.today())
businessIntelligence['CRM'].tail()
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
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-07-20</th>
      <td>89.71</td>
      <td>90.63</td>
      <td>89.42</td>
      <td>90.54</td>
      <td>3481042</td>
    </tr>
    <tr>
      <th>2017-07-21</th>
      <td>90.34</td>
      <td>90.63</td>
      <td>89.54</td>
      <td>89.69</td>
      <td>2928673</td>
    </tr>
    <tr>
      <th>2017-07-24</th>
      <td>89.65</td>
      <td>89.81</td>
      <td>88.96</td>
      <td>89.67</td>
      <td>2393543</td>
    </tr>
    <tr>
      <th>2017-07-25</th>
      <td>89.73</td>
      <td>90.82</td>
      <td>89.24</td>
      <td>90.63</td>
      <td>3345983</td>
    </tr>
    <tr>
      <th>2017-07-26</th>
      <td>90.63</td>
      <td>91.20</td>
      <td>90.15</td>
      <td>91.07</td>
      <td>3638134</td>
    </tr>
  </tbody>
</table>
</div>




```python
#view first five entries of Salesforce stock data
(businessIntelligence['CRM']).head()
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
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-01-03</th>
      <td>69.29</td>
      <td>70.86</td>
      <td>69.00</td>
      <td>70.54</td>
      <td>8112161</td>
    </tr>
    <tr>
      <th>2017-01-04</th>
      <td>71.08</td>
      <td>73.06</td>
      <td>70.76</td>
      <td>72.80</td>
      <td>9289476</td>
    </tr>
    <tr>
      <th>2017-01-05</th>
      <td>72.81</td>
      <td>73.66</td>
      <td>72.53</td>
      <td>72.79</td>
      <td>4695604</td>
    </tr>
    <tr>
      <th>2017-01-06</th>
      <td>72.89</td>
      <td>74.13</td>
      <td>72.55</td>
      <td>73.80</td>
      <td>4466130</td>
    </tr>
    <tr>
      <th>2017-01-09</th>
      <td>74.05</td>
      <td>74.44</td>
      <td>73.51</td>
      <td>73.96</td>
      <td>3933113</td>
    </tr>
  </tbody>
</table>
</div>



Instead of using a tangle of line charts, we'll go with a candlestick plot. Invented by Homma Munehisa to study the futures market of rice, these financial charts can clearly communicate all the information within our DataFrame. Check out this explanatory diagram courtesy of Probe-meteo.com:

<img src="https://upload.wikimedia.org/wikipedia/commons/e/ea/Candlestick_chart_scheme_03-en.svg" alt="Drawing" align="center" style="width: 600px;"/>


With the goal of efficiently creating candlesitck charts, it's time to dive into another Python library: Plotly. Although libraries like matplotlib and Bokeh can create custom candlestick charts with ease, I'll be using Plotly for its out-of-the-box interactivity and pleasing aesthetics. As is often the case, you can download Plotly using pip.


```python
import plotly.plotly as py
import plotly.graph_objs as go
from plotly import tools

#set plotly credentials -- create a plotly account to learn more
tools.set_credentials_file(username='XXX', api_key='XXX')

from math import *
from plotly.graph_objs import *
```


```python
def simpleCandlestickChart(ticker, stock):
    """
    Given a ticker symbol and an accompanying dataframe of
    stock information, creates a candlestick chart with
    simple moving averages
    """
    # create a candlestick chart using Plotly's defaults
    candleticker = go.Candlestick(x=ticker.index,
                           open=ticker.Open,
                           high=ticker.High,
                           low=ticker.Low,
                           close=ticker.Close,
                           name = 'Price: ',
                           increasing=dict(line=dict(color= 'black')),
                           decreasing=dict(line=dict(color= 'red')),
                           opacity=0.8)

    #label and format the chart
    layoutCandle = Layout(
        title= stock + ' Stock Prices',
    )

    #assign the data to our overall figure
    data=[candleticker]
    figCandle = Figure(data=data, layout=layoutCandle)

    #generate the figure
    py.iplot(data)
```



```python
simpleCandlestickChart(businessIntelligence['SAP'], 'SAP')
```
<div markdown="0">
{% include simplestCandle.html %}
</div>

Almost too easy. So let's dig deeper. With so much movement in the market (even with market volatility down), we need a method of cutting through all the noise and finding underlying trends. One simple way to accomplish this is with simple moving averages, formulas that take the mean of the last *n* data points. Here, we'll use 20-day, 60-day, and 180-day simple moving averages of stock closing prices -- we can find the optimal timeframes later on.


```python
# update our makeStocksDict to include simple moving averages (SMAs)
def makeStocksDict(tickers, start, end):
    """
    Takes an array of string ticker symbols representing different
    stocks, a datetime object representing a start date, and a
    datetime object representing an end date. Returns a dictionary,
    keys being stock tickers and values being pandas DataFrames
    with indexed by Date with fields Open, High, Low, Close, and Volume
    alongside calculate fields 20d Avg, 60d Avg, 180d Avg
    """
    stocksDict = {}
    for ticker in tickers:
        value = data.DataReader(ticker,'google',start,end)
        value['20d Avg'] = value['Close'].rolling(20).mean()
        value['60d Avg'] = value['Close'].rolling(60).mean()
        value['180d Avg'] = value['Close'].rolling(180).mean()
        stocksDict[ticker] = value
    return stocksDict
```

All together now...


```python
def candlestickChart(ticker, stock):
    """
    Given a ticker symbol and an accompanying dataframe of
    stock information, creates a candlestick chart with
    simple moving averages
    """

    #create a candlestick chart using plotly's defaulta
    candleticker = go.Candlestick(x=ticker.index,
                           open=ticker.Open,
                           high=ticker.High,
                           low=ticker.Low,
                           close=ticker.Close,
                           name = 'Price: ',
                           increasing=dict(line=dict(color= 'black')),
                           decreasing=dict(line=dict(color= 'red')),
                           opacity=0.8)

    avg20 = go.Scatter(x = ticker.index,
                   y = ticker['20d Avg'],
                   name = '20d Avg')

    avg60 = go.Scatter(x = ticker.index,
                   y = ticker['60d Avg'],
                   name = '60d Avg')

    avg180 = go.Scatter(x = ticker.index,
                   y = ticker['180d Avg'],
                   name = '180 Avg')


    layoutCandle = Layout(
        title= stock + ' Stock Prices',
    )

    #compile data, match with layout, and plot figure
    data=[candleticker, avg20, avg60, avg180]
    figCandle = Figure(data=data, layout=layoutCandle)
    py.plot(data)
```


```python
BP = makeStocksDict(['BP'],date(2015,1,1),date.today())
candlestickChart(BP['BP'],'BP')
```

<div markdown="0">
{% include simpleCandle.html %}
</div>

Ok, visualizations are wonderful on their own, but what actionable information can we glean from these SMAs?

I'm glad you asked. The relationship between SMAs of different timespans can be described as *regimes* At the point where a short-term SMA lies above a longer-term SMA, we're seeing a regime of "1". If a short-term SMA rests below a SMA with a greater timespan, we use '-1'.

We call SMAs' points of intersection *signals* -- a 'buy' signal occurs when a short-term SMA overtakes a long term SMA, and 'sell' occurs when the short-term crosses downwards. As their names imply, 'buy' occurs at the sign of an upward trend, and 'sell' when SMAs suggest a downward trend.

Take a look at SAP's signals from the beginning of 2017 until today.


```python
# update our makeStocksDict to include regimes and signals
def makeStocksSignals(tickers, start, end):
    """
    Takes an array of string ticker symbols representing different
    stocks, a datetime object representing a start date, and a
    datetime object representing an end date. Returns a dictionary,
    keys being stock tickers and values being pandas DataFrames
    with indexed by Date with fields Open, High, Low, Close, and Volume
    alongside calculate fields 20d Avg, 60d Avg, 180d Avg. Includes
    regimes and signals.
    """
    stocksDict = {}
    for ticker in tickers:
        value = data.DataReader(ticker,'google',start,end)
        value['20d Avg'] = value['Close'].rolling(20).mean()
        value['60d Avg'] = value['Close'].rolling(60).mean()
        value['180d Avg'] = value['Close'].rolling(180).mean()
        value['Regime'] = np.where(value['20d Avg'] < value['60d Avg'], -1, 1)
        value['Signal'] = value["Regime"] - value["Regime"].shift(1)
        stocksDict[ticker] = value
    return stocksDict
```


```python
energy = makeStocksSignals(['BP','XOM','CVX'],date(2015,1,1),date.today())
energy['BP'][200:210]
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
      <th>20d Avg</th>
      <th>60d Avg</th>
      <th>180d Avg</th>
      <th>Regime</th>
      <th>Signal</th>
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-10-19</th>
      <td>35.34</td>
      <td>35.39</td>
      <td>34.71</td>
      <td>34.94</td>
      <td>4918164</td>
      <td>33.0640</td>
      <td>33.559167</td>
      <td>38.504167</td>
      <td>-1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-10-20</th>
      <td>34.49</td>
      <td>35.08</td>
      <td>34.46</td>
      <td>34.97</td>
      <td>5405602</td>
      <td>33.2930</td>
      <td>33.541167</td>
      <td>38.470111</td>
      <td>-1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-10-21</th>
      <td>35.31</td>
      <td>35.47</td>
      <td>35.05</td>
      <td>35.14</td>
      <td>6497615</td>
      <td>33.5540</td>
      <td>33.505333</td>
      <td>38.440611</td>
      <td>1</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>2015-10-22</th>
      <td>35.63</td>
      <td>35.99</td>
      <td>35.56</td>
      <td>35.92</td>
      <td>8120264</td>
      <td>33.8420</td>
      <td>33.480833</td>
      <td>38.410278</td>
      <td>1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-10-23</th>
      <td>35.84</td>
      <td>36.00</td>
      <td>35.52</td>
      <td>35.72</td>
      <td>4749154</td>
      <td>34.1065</td>
      <td>33.448167</td>
      <td>38.380000</td>
      <td>1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-10-26</th>
      <td>35.60</td>
      <td>35.64</td>
      <td>35.05</td>
      <td>35.05</td>
      <td>5105227</td>
      <td>34.3900</td>
      <td>33.416167</td>
      <td>38.343556</td>
      <td>1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-10-27</th>
      <td>34.82</td>
      <td>35.19</td>
      <td>34.60</td>
      <td>34.82</td>
      <td>10123884</td>
      <td>34.6495</td>
      <td>33.388500</td>
      <td>38.307833</td>
      <td>1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-10-28</th>
      <td>35.09</td>
      <td>36.09</td>
      <td>35.07</td>
      <td>35.74</td>
      <td>9626609</td>
      <td>34.9085</td>
      <td>33.375167</td>
      <td>38.282222</td>
      <td>1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-10-29</th>
      <td>35.38</td>
      <td>36.01</td>
      <td>35.34</td>
      <td>35.71</td>
      <td>6965609</td>
      <td>35.1460</td>
      <td>33.372500</td>
      <td>38.250944</td>
      <td>1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2015-10-30</th>
      <td>35.62</td>
      <td>36.00</td>
      <td>35.33</td>
      <td>35.70</td>
      <td>5441985</td>
      <td>35.3050</td>
      <td>33.369500</td>
      <td>38.216333</td>
      <td>1</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
def candlestickChart(ticker, stock):
    """
    Given a ticker symbol and an accompanying dataframe of
    stock information, creates a candlestick chart with
    simple moving averages, regimes, and signals
    """
    candleticker = go.Candlestick(x=ticker.index,
                           open=ticker.Open,
                           high=ticker.High,
                           low=ticker.Low,
                           close=ticker.Close,
                           name = 'Price: ',
                           increasing=dict(line=dict(color= 'black')),
                           decreasing=dict(line=dict(color= 'red')),
                           opacity=0.8)

    avg20 = go.Scatter(x = ticker.index,
                   y = ticker['20d Avg'],
                   name = '20d Avg')

    avg60 = go.Scatter(x = ticker.index,
                   y = ticker['60d Avg'],
                   name = '60d Avg')

    avg180 = go.Scatter(x = ticker.index,
                   y = ticker['180d Avg'],
                   name = '180 Avg')

    regime = go.Scatter(x = ticker.index,
                       y = ticker['Regime'] + 30)

    #insert a vertical line at each signal change
    signal_change = []
    for index, row in ticker.iterrows():
        if (ticker['Signal'][index] != 0):
            signal_change.append({
                'type': 'line',
                'x0': index,
                'y0': 29,
                'x1': index,
                'y1': ticker['Close'][index],
                'line': {
                    'color': 'lime',
                },
            })

    layoutCandle = {
        'shapes': signal_change
    }

    data=[candleticker, avg20, avg60, avg180, regime]
    figCandle = Figure(data=data, layout=layoutCandle)
    py.plot(figCandle)
```


```python
candlestickChart(energy['BP'],'BP')
```
<div markdown="0">
{% include regimeCandle.html %}
</div>

Good stuff! 'Buy' and 'sell' do sound pretty straightforward. Next time, we'll take a look at some other indicators that complicate this seemingly simple picture.
