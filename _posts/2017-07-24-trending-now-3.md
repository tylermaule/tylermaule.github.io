---
layout: post
title: "Trending Now (Pt. 3)"
date: 2017-07-24
---

Thus far, we've used simple moving averages and their intersections to develop a series of signals -- flags that propose the purchase or sale of a company's stock. Eventually, we'll build and backtest a trading algorithm, but there's plenty more information to ferret out of stock data. Let's set the stage once more.


```python
import pandas as pd
import numpy as np
from pandas_datareader import data, wb
from datetime import date
import matplotlib.pyplot as plt
from datetime import date
%matplotlib inline
```


```python
# update our makeStocksDict to include simple moving averages (SMAs)
def makeStocksDictSignal(tickers, start, end):
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
        value['Regime'] = np.where(value['60d Avg'] < value['180d Avg'], -1, 1)
        value['Signal'] = value["Regime"] - value["Regime"].shift(1)
        stocksDict[ticker] = value
    return stocksDict
```

We should start by adding volume to our candlestick chart. Volume, the number of shares exchanged, often indicates the importance of a move in the markets. Therefore, we should measure trends in volume to judge the significances of the SMAs. If volume is increasing, a 'buy' or 'sell' signal should be weighted more heavily.


```python
# plotly w/ volume
```

Ok. It's time to start building a simple trading algorithm, adapted from the wonderful work of [Curtis Miller](https://ntguardian.wordpress.com/2016/09/26/introduction-stock-market-data-python-2/). In the future, we'll strengthen the algorithm, but we've got to start somewhere. We'll use a moving average crossover strategy, and buy more (with arbitrary weight) if volume is on the upswing.


```python
def signals(stockDict):
    """
    Given a dictionary containing tickers as keys and stock-information dataframes as values,
    creates and returns a dataframe of dates with a signal change (based on a moving average
    crossover strategy)
    """
    trades = pd.DataFrame({"Date":[],
                            "Ticker": [],
                            "Close": [],
                            "Volume": [],
                            "Regime": [],
                            "Signal": []
                            })
    signalList = []

    #add signal entries to dataframe
    for ticker, data in stockDict.items():
        data['Ticker'] = ticker
        data['Date'] = data.index
        signals = data.loc[abs(data["Signal"]) > 0]
        signals = signals[["Date","Ticker","Close","Volume","Regime","Signal"]]
        signalList.append(signals)

    trades = pd.concat(signalList)  
    trades.sort_index()
    return trades.sort_index()

```

Let's see what kind of signals we receive from these SMA crossovers. To gauge the performance of our algorithm, we'll use a concept known as *backtesting*. That is, we will apply our algorithms to stock movements in prior years.


```python
stockDict = makeStocksDictSignal(['AAPL','IBM','MSFT','HPQ','YHOO','SNY','TWTR','AMZN','GOOGL'],date(2010,1,1),date.today())
signalRecords = signals(stockDict)
signalRecords.tail()
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
      <th>Date</th>
      <th>Ticker</th>
      <th>Close</th>
      <th>Volume</th>
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-08-29</th>
      <td>2016-08-29</td>
      <td>TWTR</td>
      <td>18.47</td>
      <td>11417408</td>
      <td>1</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>2017-01-17</th>
      <td>2017-01-17</td>
      <td>SNY</td>
      <td>40.96</td>
      <td>2536087</td>
      <td>1</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>2017-02-01</th>
      <td>2017-02-01</td>
      <td>TWTR</td>
      <td>17.24</td>
      <td>21208131</td>
      <td>-1</td>
      <td>-2.0</td>
    </tr>
    <tr>
      <th>2017-06-01</th>
      <td>2017-06-01</td>
      <td>IBM</td>
      <td>152.67</td>
      <td>2918126</td>
      <td>-1</td>
      <td>-2.0</td>
    </tr>
    <tr>
      <th>2017-06-29</th>
      <td>2017-06-29</td>
      <td>TWTR</td>
      <td>17.65</td>
      <td>18796467</td>
      <td>1</td>
      <td>2.0</td>
    </tr>
  </tbody>
</table>
</div>



Great! But for all this buying and selling, what kind of return will we get on our investment? It's time to create and test a virtual trading systems with real prices.


```python
def backtest1(stockDict, tradeSignals, initialValue, batchSize):
    """
    Given a dictionary containing tickers as keys and stock-information dataframes as values,
    a dataframe containing moving average crossover signals, an initial amount of cash in the
    portfolio, and a default minimum batch size of stocks, simulates the buying of stocks and
    the returns of a portfolio. Return a dataframe with results from each trade.
    """

    cash = initialValue

    #create a dictionary to store number of stocks invested in each company
    sharesDict = {}
    for stock in tradeSignals.Ticker.unique():
        sharesDict[stock] = 0

    #creates an empty dataframe to store results
    results = pd.DataFrame({"Value":[],
                           "Ticker":[],
                            "Price":[],
                           "Signal":[],
                            "Action":[],
                           "Shares":[],
                           "Return":[],
                            "Cash":[]
                           })

    #iterates through dataframe of each trading signal
    for day, entry in tradeSignals.iterrows():
        if day in tradeSignals.index.tolist():
            if (((entry["Signal"] == -2) and (cash > entry["Close"] * batchSize)) or
                ((entry["Signal"] == 2) and (sharesDict[entry["Ticker"]] > 0))):
                if (entry["Signal"] == -2):
                    #if signal is "buy", buy the batch size of shares
                    sharesDict[entry['Ticker']] -= entry["Signal"]/2
                    amt = (entry['Close'] * batchSize * entry["Signal"]/2)
                    cash += amt

                    port_value = 0

                    #calculate current portfolio value
                    for stock, value in stockDict.items():
                        shares = sharesDict[stock] * batchSize
                        closeSeries = value['Close'].sort_index()
                        try:
                            close = closeSeries.loc[day]
                        except:
                            close = 0

                        port_value += shares * close

                    #add portfolio status and trade action to results dataframe    
                    entryAction = pd.DataFrame({
                            "Cash": cash,
                            "Value": cash + port_value,
                            "Ticker": entry['Ticker'],
                            "Price": entry['Close'],
                            "Signal": entry['Signal'],
                            "Action": "Buy" if (entry["Signal"]==-2) else "Sell",
                            "Shares": sharesDict[entry['Ticker']] * batchSize,
                            "Port": port_value,
                            "Return": (cash + port_value)/initialValue,
                    }, index=[day])

                    results = pd.concat([results,entryAction],axis=0)

    return results[["Ticker","Value","Action","Shares","Price","Port","Cash","Return"]]

```


```python
simpleCalc = backtest1(stockDict,signalRecords,1000000,100)
simpleCalc.tail()
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
      <th>Ticker</th>
      <th>Value</th>
      <th>Action</th>
      <th>Shares</th>
      <th>Price</th>
      <th>Port</th>
      <th>Cash</th>
      <th>Return</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-03-28</th>
      <td>AMZN</td>
      <td>1278756.0</td>
      <td>Buy</td>
      <td>400.0</td>
      <td>579.87</td>
      <td>794330.0</td>
      <td>484426.0</td>
      <td>1.278756</td>
    </tr>
    <tr>
      <th>2016-06-17</th>
      <td>MSFT</td>
      <td>1304819.0</td>
      <td>Buy</td>
      <td>500.0</td>
      <td>50.13</td>
      <td>825406.0</td>
      <td>479413.0</td>
      <td>1.304819</td>
    </tr>
    <tr>
      <th>2016-06-22</th>
      <td>GOOGL</td>
      <td>1312011.0</td>
      <td>Buy</td>
      <td>600.0</td>
      <td>710.47</td>
      <td>903645.0</td>
      <td>408366.0</td>
      <td>1.312011</td>
    </tr>
    <tr>
      <th>2017-02-01</th>
      <td>TWTR</td>
      <td>1454238.0</td>
      <td>Buy</td>
      <td>400.0</td>
      <td>17.24</td>
      <td>1047596.0</td>
      <td>406642.0</td>
      <td>1.454238</td>
    </tr>
    <tr>
      <th>2017-06-01</th>
      <td>IBM</td>
      <td>1630097.0</td>
      <td>Buy</td>
      <td>700.0</td>
      <td>152.67</td>
      <td>1238722.0</td>
      <td>391375.0</td>
      <td>1.630097</td>
    </tr>
  </tbody>
</table>
</div>



How did we do? Let's use a benchmark to put this backtesting's success in relative terms. Here we will chart our returns against those of the SPDR S&P 500 ETF Trust.


```python
spyder = data.DataReader('SPY','google',date(2010,1,1),date.today())
spyder['Returns'] = spyder['Close']/spyder['Close'].iloc[0]

plt.plot(simpleCalc.index, simpleCalc.Return, label = "Simple Calc Alg")
plt.plot(spyder.index, spyder.Returns, label="SPY")
plt.legend();
```


![png](output_13_0.png)


Hmm... we're falling short of the S&P 500's performance. Let's add some complexity. Instead of simply buying 100 shares at each "sell" signal, we will use the stocks' prices to buy the maximum amount of shares (using no more than 10% of our assets in each trade).


```python
def backtest2(stockDict, tradeSignals, initialValue, batchSize, port_risk):
    """
    Given a dictionary containing tickers as keys and stock-information dataframes as values,
    a dataframe containing moving average crossover signals, an initial amount of cash in the
    portfolio, and a default minimum batch size of stocks, simulates the buying of stocks and
    the returns of a portfolio. Return a dataframe with results from each trade. Uses port_risk
    to calculate the maximum percentage of the portfolio invested in one stock.
    """

    cash = initialValue

    #create a dictionary to store number of stocks invested in each company
    sharesDict = {}

    for stock in tradeSignals.Ticker.unique():
        sharesDict[stock] = 0

    results = pd.DataFrame({"Value":[],
                           "Ticker":[],
                            "Price":[],
                           "Signal":[],
                            "Action":[],
                           "Shares":[],
                           "Return":[],
                            "Cash":[],
                            "Port":[]
                           })

    #iterates through dataframe of each trading signal
    for day, entry in tradeSignals.iterrows():
        port_value = 0
        if day in tradeSignals.index.tolist():
            if (((entry["Signal"] == -2) and (cash > entry["Close"] * batchSize) and entry['Regime']==-1) or
                ((entry["Signal"] == 2) and (sharesDict[entry["Ticker"]] > 0) and (entry['Regime']==1))):
                if (entry["Signal"] == -2):
                    #if signal is "buy", buy the maximum number of shares given the port_risk
                    shareBuy = ((cash+port_value) * port_risk)  // (entry['Close'] * batchSize)
                    sharesDict[entry['Ticker']] += shareBuy
                    cash -= shareBuy * entry['Close'] * batchSize

                port_value = 0

                #calculate current portfolio value
                for stock, value in stockDict.items():
                    shares = sharesDict[stock] * batchSize
                    closeSeries = value['Close'].sort_index()
                    try:
                        close = closeSeries.loc[day]
                    except:
                        close = 0

                    port_value += shares * close

                entryAction = pd.DataFrame({
                    "Cash": cash,
                    "Value": cash + port_value,
                    "Ticker": entry['Ticker'],
                    "Price": entry['Close'],
                    "Signal": entry['Signal'],
                    "Action": "Buy" if (entry["Signal"]==-2) else "Sell",
                    "Shares": sharesDict[entry['Ticker']] * batchSize,
                    "Port": port_value,
                    "Return": (cash + port_value)/initialValue,
                }, index=[day])
                results = pd.concat([results,entryAction],axis=0)

    return results[["Ticker","Value","Action","Shares","Price","Port","Cash","Return"]]        
```


```python
shareCalc = backtest2(stockDict,signalRecords,1000000,100,0.1)
shareCalc.tail()
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
      <th>Ticker</th>
      <th>Value</th>
      <th>Action</th>
      <th>Shares</th>
      <th>Price</th>
      <th>Port</th>
      <th>Cash</th>
      <th>Return</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-08-29</th>
      <td>TWTR</td>
      <td>1793678.0</td>
      <td>Sell</td>
      <td>700.0</td>
      <td>18.47</td>
      <td>1738897.0</td>
      <td>54781.0</td>
      <td>1.793678</td>
    </tr>
    <tr>
      <th>2017-01-17</th>
      <td>SNY</td>
      <td>1862003.0</td>
      <td>Sell</td>
      <td>4900.0</td>
      <td>40.96</td>
      <td>1807222.0</td>
      <td>54781.0</td>
      <td>1.862003</td>
    </tr>
    <tr>
      <th>2017-02-01</th>
      <td>TWTR</td>
      <td>1900511.0</td>
      <td>Buy</td>
      <td>1000.0</td>
      <td>17.24</td>
      <td>1850902.0</td>
      <td>49609.0</td>
      <td>1.900511</td>
    </tr>
    <tr>
      <th>2017-06-01</th>
      <td>IBM</td>
      <td>2175192.0</td>
      <td>Buy</td>
      <td>900.0</td>
      <td>152.67</td>
      <td>2125583.0</td>
      <td>49609.0</td>
      <td>2.175192</td>
    </tr>
    <tr>
      <th>2017-06-29</th>
      <td>TWTR</td>
      <td>2162660.0</td>
      <td>Sell</td>
      <td>1000.0</td>
      <td>17.65</td>
      <td>2113051.0</td>
      <td>49609.0</td>
      <td>2.162660</td>
    </tr>
  </tbody>
</table>
</div>



The results are looking better already. Let's add a visual representation of how each strategy measures up.


```python
plt.plot(simpleCalc.index, simpleCalc.Return, label = "Simple Calc Alg")
plt.plot(shareCalc.index, shareCalc.Return, label = "Share Calc Alg")
plt.plot(spyder.index, spyder.Returns, label="SPY")
plt.legend();
```


![png](output_18_0.png)


As indicated earlier, it's time to incorporate volume -- it's one of the most popular technical indicators for good reason. While the crossover moving average strategy uses SMAs to find emerging trends in security prices, it may generate false positives. A surge in volume provides reassurance that the market really *is* moving in a certain direction -- there's a *breakout* occuring.

So, how will we incorporate volume? For now, we'll simple compute the SMAs of stocks' volumes, using them as a second signal. In the future, we'll use more sophisticated methods of adding volume to the mix. But let's get moving (averages).


```python
# update our makeStocksDict to include simple moving averages (SMAs)
def makeStocksDictSignal(tickers, start, end):
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
        value['Regime'] = np.where(value['60d Avg'] < value['180d Avg'], -1, 1)
        value['Signal'] = value["Regime"] - value["Regime"].shift(1)

        value['20d VMean'] = value['Volume'].rolling(20).mean()
        value['60d VMean'] = value['Volume'].rolling(60).mean()
        value['180d VMean'] = value['Volume'].rolling(180).mean()
        value['VRegime'] = np.where(value['Volume'] < value['180d VMean'], -1, 1)
        value['VSignal'] = value["VRegime"] - value["VRegime"].shift(1)
        stocksDict[ticker] = value
    return stocksDict
```


```python
def signals(stockDict):
    trades = pd.DataFrame({"Date":[],
                            "Ticker": [],
                            "Close": [],
                            "Volume": [],
                            "Regime": [],
                            "Signal": [],
                            "VRegime": [],
                            })
    signalList = []
    for ticker, data in stockDict.items():
        data['Ticker'] = ticker
        data['Date'] = data.index
        signals = data.loc[abs(data["Signal"]) > 0]
        signals = signals[["Date","Ticker","Close","Volume","Regime","Signal","VRegime"]]
        signalList.append(signals)

    trades = pd.concat(signalList)  
    trades.sort_index()
    return trades.sort_index()
```


```python
stockDict = makeStocksDictSignal(['AAPL','IBM','MSFT','HPQ','YHOO','SNY','TWTR','AMZN','GOOGL'],date(2010,1,1),date.today())
signalRecords = signals(stockDict)
signalRecords.head()
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
      <th>Date</th>
      <th>Ticker</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Regime</th>
      <th>Signal</th>
      <th>VRegime</th>
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010-09-20</th>
      <td>2010-09-20</td>
      <td>IBM</td>
      <td>131.79</td>
      <td>7214702</td>
      <td>-1</td>
      <td>-2.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2010-09-20</th>
      <td>2010-09-20</td>
      <td>HPQ</td>
      <td>39.39</td>
      <td>22196643</td>
      <td>-1</td>
      <td>-2.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2010-09-20</th>
      <td>2010-09-20</td>
      <td>SNY</td>
      <td>33.21</td>
      <td>1993360</td>
      <td>-1</td>
      <td>-2.0</td>
      <td>-1</td>
    </tr>
    <tr>
      <th>2010-09-21</th>
      <td>2010-09-21</td>
      <td>GOOGL</td>
      <td>256.99</td>
      <td>4467227</td>
      <td>-1</td>
      <td>-2.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2010-09-21</th>
      <td>2010-09-21</td>
      <td>AMZN</td>
      <td>150.73</td>
      <td>7546662</td>
      <td>-1</td>
      <td>-2.0</td>
      <td>-1</td>
    </tr>
  </tbody>
</table>
</div>




```python
def backtest3(stockDict, tradeSignals, initialValue, batchSize, port_risk):
    """
    Given a dictionary containing tickers as keys and stock-information dataframes as values,
    a dataframe containing moving average crossover signals, an initial amount of cash in the
    portfolio, and a default minimum batch size of stocks, simulates the buying of stocks and
    the returns of a portfolio. Return a dataframe with results from each trade.
    """

    cash = initialValue

    #create a dictionary to store number of stocks invested in each company
    sharesDict = {}

    for stock in tradeSignals.Ticker.unique():
        sharesDict[stock] = 0

    results = pd.DataFrame({"Value":[],
                           "Ticker":[],
                            "Price":[],
                           "Signal":[],
                            "VRegime": [],
                            "Action":[],
                           "Shares":[],
                           "Return":[],
                            "Cash":[],
                            "Port":[]
                           })

    #iterates through dataframe of each trading signal
    for day, entry in tradeSignals.iterrows():
        port_value = 0
        if day in tradeSignals.index.tolist():
            if (((entry["Signal"] == -2) and (cash > entry["Close"] * batchSize) and entry['Regime']==-1) or
                ((entry["Signal"] == 2) and (sharesDict[entry["Ticker"]] > 0) and (entry['Regime']==1))):
                if (entry["Signal"] == -2):
                    #if signal is "buy", buy the maximum number of shares given the port_risk and the volume regime
                    mult = {True:2, False:1}[entry['VRegime']==1]
                    shareBuy = ((cash+port_value) * port_risk * mult)  // (entry['Close'] * batchSize)
                    sharesDict[entry['Ticker']] += shareBuy
                    cash -= shareBuy * entry['Close'] * batchSize

                port_value = 0

                #calculate current portfolio value
                for stock, value in stockDict.items():
                    shares = sharesDict[stock] * batchSize
                    closeSeries = value['Close'].sort_index()
                    try:
                        close = closeSeries.loc[day]
                    except:
                        close = 0

                    port_value += shares * close

                #add portfolio status and trade action to results dataframe    
                entryAction = pd.DataFrame({
                    "Cash": cash,
                    "Value": cash + port_value,
                    "Ticker": entry['Ticker'],
                    "Price": entry['Close'],
                    "Signal": entry['Signal'],
                    "VRegime": entry['VRegime'],
                    "Action": "Buy" if (entry["Signal"]==-2) else "Sell",
                    "Shares": sharesDict[entry['Ticker']] * batchSize,
                    "Port": port_value,
                    "Return": (cash + port_value)/initialValue,
                }, index=[day])
                results = pd.concat([results,entryAction],axis=0)

    return results[["Ticker","Value","Action","Shares","Price","Port","Cash","Return"]]   
```


```python
volCalc = backtest3(stockDict,signalRecords,1000000,1,0.1)
volCalc.tail()
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
      <th>Ticker</th>
      <th>Value</th>
      <th>Action</th>
      <th>Shares</th>
      <th>Price</th>
      <th>Port</th>
      <th>Cash</th>
      <th>Return</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-08-29</th>
      <td>TWTR</td>
      <td>1856572.97</td>
      <td>Sell</td>
      <td>101.0</td>
      <td>18.47</td>
      <td>1854481.07</td>
      <td>2091.90</td>
      <td>1.856573</td>
    </tr>
    <tr>
      <th>2017-01-17</th>
      <td>SNY</td>
      <td>1927353.63</td>
      <td>Sell</td>
      <td>3421.0</td>
      <td>40.96</td>
      <td>1925261.73</td>
      <td>2091.90</td>
      <td>1.927354</td>
    </tr>
    <tr>
      <th>2017-02-01</th>
      <td>TWTR</td>
      <td>1963528.76</td>
      <td>Buy</td>
      <td>113.0</td>
      <td>17.24</td>
      <td>1961643.74</td>
      <td>1885.02</td>
      <td>1.963529</td>
    </tr>
    <tr>
      <th>2017-06-01</th>
      <td>IBM</td>
      <td>2229454.67</td>
      <td>Buy</td>
      <td>1672.0</td>
      <td>152.67</td>
      <td>2227722.32</td>
      <td>1732.35</td>
      <td>2.229455</td>
    </tr>
    <tr>
      <th>2017-06-29</th>
      <td>TWTR</td>
      <td>2210915.82</td>
      <td>Sell</td>
      <td>113.0</td>
      <td>17.65</td>
      <td>2209183.47</td>
      <td>1732.35</td>
      <td>2.210916</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.plot(simpleCalc.index, simpleCalc.Return, label = "Simple Calc Alg")
plt.plot(shareCalc.index, shareCalc.Return, label = "Share Calc Alg")
plt.plot(volCalc.index, volCalc.Return, label="Volume Calc Avg")
plt.plot(spyder.index, spyder.Returns, label="SPY")
plt.legend();
```


![png](output_25_0.png)


So, by giving signals backed by volume more weight, we can eke out an extra 5% return. I'd argue that it's faily significant, but we'll refine and improve this algorithm even further in the future. For now -- let's see how our backtested algorithm performs.


```python
stockDict = makeStocksDictSignal(['AAPL','IBM','MSFT','HPQ','YHOO','SNY','TWTR','AMZN','GOOGL'],date(2017,7,21),date.today())
signalRecords = signals(stockDict)
volCalc = backtest3(stockDict,signalRecords,1000000,1,0.1)

spyder = data.DataReader('SPY','google',date(2017,7,21),date.today())
spyder['Returns'] = spyder['Close']/spyder['Close'].iloc[0]

plt.plot(volCalc.index, volCalc.Return, label="Volume Calc Avg")
plt.plot(spyder.index, spyder.Returns, label="SPY")
plt.legend();
```


![png](output_27_0.png)
