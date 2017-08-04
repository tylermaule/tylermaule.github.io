---
layout: post
title: "Trending Now (Pt. 1): Return & Refresh"
date: 2017-07-10
categories: [Finance & Economics]
---



Those practiced in security valuation will be familiar with two warring methods: fundamental analysis and technical analysis. Each analysis attempts to determine a security's value, but use different views of the market to reach a conclusion. Fundamental analysis rests on an understanding of internal and external financial conditions. For example, an analyst could look at a company's SEC filing relative to its competitors, or react to changing company policies. Technical analysis, on the other hand, believes in a more efficient market, one which has already taken these external details into account. Instead, this technique looks directly at trends in a securities' price to predict how it will move in the future.

Especially as a student of finance, I believe that it's essential to approach valuation from both directions -- a full stack approach, if you will. But given the ease with which Python can parse stock prices, it seems natural to begin by diving into technical analysis. I highly recommend [installing Anaconda](https://www.continuum.io/downloads), which provides access to interactive Python notebooks and pletny of libraries for data analysis in Python.

Created by Wes Kinney in 2008, the pandas library provides tools for analyzing complex, time-indexed data. Named for the "panel data" used in econometric anaysis, pandas includes a core "DataFrame" data structure -- in which you can directly import stock data. given a stock ticker, date range, and source (I'd go with 'google').

Install pandas-datareader through the "Environments" tab of Anaconda Navigator, or using pip:
```
$ pip install pandas-datareader
```

```python
import pandas
from pandas_datareader import data, wb
from datetime import date

#set the timeframe -- from January 1st, 2017 to today
start = date(2017,1,1)
end = date.today()

#set the ticker to get stock data for Amazon
ticker = 'AMZN'

#get the stock data from google
source = 'google'

#load stock data into a dataframe
AMZN = data.DataReader(ticker, source, start, end)
```

And that's it! You can easily view the first few entries of a DataFrame using the .head() method, or use slicing by index for more specific results.


```python
#get the first 5 entries in the stock data
AMZN.head()
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
      <td>757.92</td>
      <td>758.76</td>
      <td>747.70</td>
      <td>753.67</td>
      <td>3521066</td>
    </tr>
    <tr>
      <th>2017-01-04</th>
      <td>758.39</td>
      <td>759.68</td>
      <td>754.20</td>
      <td>757.18</td>
      <td>2510526</td>
    </tr>
    <tr>
      <th>2017-01-05</th>
      <td>761.55</td>
      <td>782.40</td>
      <td>760.26</td>
      <td>780.45</td>
      <td>5830068</td>
    </tr>
    <tr>
      <th>2017-01-06</th>
      <td>782.36</td>
      <td>799.44</td>
      <td>778.48</td>
      <td>795.99</td>
      <td>5986234</td>
    </tr>
    <tr>
      <th>2017-01-09</th>
      <td>798.00</td>
      <td>801.77</td>
      <td>791.77</td>
      <td>796.92</td>
      <td>3446109</td>
    </tr>
  </tbody>
</table>
</div>




```python
#get opening stock prices from February 1st to February 28th, 2017
AMZN['Open'][30:58]
```




    Date
    2017-02-15    834.00
    2017-02-16    841.84
    2017-02-17    842.00
    2017-02-21    848.84
    2017-02-22    856.95
    2017-02-23    857.57
    2017-02-24    844.69
    2017-02-27    842.38
    2017-02-28    851.45
    2017-03-01    853.05
    2017-03-02    853.08
    2017-03-03    847.20
    2017-03-06    845.23
    2017-03-07    845.48
    2017-03-08    848.00
    2017-03-09    851.00
    2017-03-10    857.00
    2017-03-13    851.77
    2017-03-14    853.55
    2017-03-15    854.33
    2017-03-16    855.30
    2017-03-17    853.49
    2017-03-20    851.51
    2017-03-21    858.84
    2017-03-22    840.43
    2017-03-23    848.20
    2017-03-24    851.68
    2017-03-27    838.07
    Name: Open, dtype: float64



Obviously, sifting through thousands of such figures would be an impossible task. That's where matplotlib and seaborn come in. Based on MATLAB, matplotlib allows an object-oriented approach to data visualization. Seaborn improves the look and feel of matplotlib's graphs, styling and scaling for optimal presentation.


```python
import matplotlib.pyplot as plt
#display graphs as inline output
%matplotlib inline

#apply seaborn formatting
import seaborn as sns
sns.set();
```

With these tools in hand, we can easily graph closing stock prices over time, like so:


```python
plt.plot(AMZN[30:58]['Close'])
plt.show();
```


![png](http://i.imgur.com/dmbvQBd.png)


But when it comes to analyzing multiple stocks, we'd like to normalize the data for direct comparison. Ultimately, we're interested in the return to our stocks -- what percentage of our initial investment would we receive if we sold our shares each day?

Creating a new column in a pandas DataFrame is simple (dataframe1['column name']), and we can use a formula to calci;ate values in each row:


```python
AAPL = data.DataReader('AAPL', source, start, end) #get Apple stock data for the same time period

#calculate returns
AAPL['AAPL Return'] = (AAPL['Close'] - AAPL['Close'][0])/AAPL['Close'][0]
AMZN['AMZN Return'] = (AMZN['Close'] - AMZN['Close'][0])/AMZN['Close'][0]

AMZN['Return'].tail() #show the last few rows of the dataframe
```




    Date
    2017-07-11    0.319052
    2017-07-12    0.335478
    2017-07-13    0.327677
    2017-07-14    0.329242
    2017-07-17    0.340162
    Name: Return, dtype: float64




```python
plt.plot(AMZN['AMZN Return'])
plt.plot(AAPL['AAPL Return'])
plt.legend() #display a legend
plt.show();
```


![png](http://i.imgur.com/8W6k6Cs.png)


Wonderful! We're getting somewhere, slowly but surely. Stick around for more soon.

<br>
*These posts have been informed and inspired by the following wonderful resources*:
+ [Curtis Miller's Introduction to Stock Data with Python](https://ntguardian.wordpress.com/2016/09/19/introduction-stock-market-data-python-1/)
+ [Dr. Tucker Bulch's Introduction to Computational Investing](https://www.coursera.org/learn/computational-investing)
