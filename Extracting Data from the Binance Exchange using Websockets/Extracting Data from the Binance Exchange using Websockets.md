# Extracting Data from the Binance Exchange by using Websockets

The current global cryptocurrency market is worth more than 1.5 trillion dolars. Cryptocurrencies have emerged from literally an asset with no profitable value such as the famous story where [a man has traded his 10,000 BTC for pizza](https://bitcointalk.org/index.php?topic=137.0) to an asset worth more than U$ 30,000 as of this date. It is no wonder that we can see a future marked by the usage of cryptocurrencies instead of the old fiat.

This tutorial is about how one can extract data from the Binance API available at [Binance API](https://www.binance.com/en/my/settings/api-management). This example uses the documentation available at [python-binance](https://python-binance.readthedocs.io/en/latest/index.html) and a few other tricks so we can accomplish our goals!

## Importing the Client and Setting API Key and API Secret
To access the Binance API, it is needed first to initialize the Client and then authenticate within the Binance API. This tutorial assumes you have already set the keys in a keys.ipynb file as below, which is located inside the same directory of the current Jupyter notebook file.
![image.png](attachment:fac5000e-42cf-47c1-ac67-09e8554ea32c.png)


```python
from binance.client import Client
# Get keys for usage with Binance API
%run keys.ipynb

# Initializes Client inside the client variable
client = Client(API_KEY,API_SECRET)
```

## Constants

**TRADING_PAIR** corresponds to the pair from which the Binance API will extract the data. In this case, the author has opted to select the pair of the ADA coin which is expressed in terms of the BNB coin.

**STARTING_DATE** is defined as the starting date for the samples for which the data will be gathered and **ENDING_DATE**, correspondingly, the ending date for these samples.


```python
# Escolhe o par
TRADING_PAIR = 'ADABNB'
# Data de Inicio da Captura dos Dados
STARTING_DATE = "22 May, 2021"
# Data Final da Captura dos Dados
END_DATE = "23 May, 2021"
```

## Process Message Function
The Process Message function defines how the contents returned by the calls to the API will be processed. In our case, if there are any errors, the socket will stop connecting to the Binance API and close the Binance Socket Manager. Else, it will proceed with the operations.


```python
def process_message(msg):
    if msg['e'] == 'error':
        bm.stop_socket(conn_key)
        bm.close()
    else:
        pass
```

## Binance Websockets
[Websockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) allow to open a two-way interactive communication session between the Jupyter environment and a server. With a Websockets API, Jupyter can send messages to a server and receive event-driven responses without having to poll the Binance API again for a reply.

First, we import the [Binance Websockets API](https://python-binance.readthedocs.io/en/latest/websockets.html) and initialize the [Binance Socket Manager](https://python-binance.readthedocs.io/en/latest/websockets.html#binancesocketmanager-websocket-usage) with an user timeout of 60 seconds. Then, we start a trading socket on the trading pair previously defined using the **Process Message Function**.


```python
# Importing Binance Websockets API
from binance.client import *
from binance.websockets import BinanceSocketManager
from twisted.internet import reactor

# Initialize Binance Socket Manager
bm = BinanceSocketManager(client, user_timeout=60)
list_messages = []
conn_key = bm.start_trade_socket(TRADING_PAIR, process_message)
bm.start()
```

## Candlestick Data

In this example, the author has opted to approach only the values for the [Kline/Candlesticks Endpoint](https://python-binance.readthedocs.io/en/latest/market_data.html#id6), albeit the API has an [extensive amount of other endpoints for Market Data](https://python-binance.readthedocs.io/en/latest/market_data.html). The **TRADING_PAIR** ADA/BNB is used, as stated above, and the interval we have opted to choose was for 5M - or 5 minute - candles.

You can see the available intervals inside the [Constants](https://python-binance.readthedocs.io/en/latest/constants.html) for the Python wrapper for Binance API.


```python
candles = client.get_klines(symbol=TRADING_PAIR, interval=Client.KLINE_INTERVAL_5MINUTE)
```

## Initializing the Data


Now the data is initialized by assigning the values for the indexes - or header columns - to an array and initialize the data from the candle data saved inside the *candles* variable into a Pandas DataFrame. Then, we'll show the first 5 columns of the DataFrame.


```python
import pandas as pd

indexes = ['Open Time', 'Open', 'High','Low', 'Close', 'Volume', 'Close Time', 'QAV', 'No. Trades', 'Taker BBAV', 'Taker BQAV', 'Ignore']
data = pd.DataFrame(columns=indexes,data=candles)
data.head()
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
      <th>Open Time</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Close Time</th>
      <th>QAV</th>
      <th>No. Trades</th>
      <th>Taker BBAV</th>
      <th>Taker BQAV</th>
      <th>Ignore</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1621758600000</td>
      <td>0.00505500</td>
      <td>0.00506300</td>
      <td>0.00501400</td>
      <td>0.00504600</td>
      <td>81838.00000000</td>
      <td>1621758899999</td>
      <td>412.45858000</td>
      <td>263</td>
      <td>29382.00000000</td>
      <td>148.02874200</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1621758900000</td>
      <td>0.00504200</td>
      <td>0.00506600</td>
      <td>0.00500400</td>
      <td>0.00500400</td>
      <td>120175.00000000</td>
      <td>1621759199999</td>
      <td>605.99748800</td>
      <td>348</td>
      <td>53294.00000000</td>
      <td>268.38967800</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1621759200000</td>
      <td>0.00501100</td>
      <td>0.00502200</td>
      <td>0.00498300</td>
      <td>0.00500000</td>
      <td>90252.00000000</td>
      <td>1621759499999</td>
      <td>451.05157900</td>
      <td>268</td>
      <td>21405.00000000</td>
      <td>106.94323100</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1621759500000</td>
      <td>0.00500400</td>
      <td>0.00502500</td>
      <td>0.00498300</td>
      <td>0.00498300</td>
      <td>162699.00000000</td>
      <td>1621759799999</td>
      <td>814.71476800</td>
      <td>370</td>
      <td>84077.00000000</td>
      <td>421.09383800</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1621759800000</td>
      <td>0.00499100</td>
      <td>0.00499500</td>
      <td>0.00493600</td>
      <td>0.00494000</td>
      <td>167842.00000000</td>
      <td>1621760099999</td>
      <td>833.68666400</td>
      <td>512</td>
      <td>53889.00000000</td>
      <td>267.76022000</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



## Data Preprocessing
Fixing the wrong timestamp for the **Open Time** columns and setting this column as the index for the DataFrame is crucial for proper visualization of the DataFrame. Then, filtering out the columns so our DataFrame will only contain the **Open Time, Open, High, Low, Close and Volume** - ie. Open Time + OHLCV. 


```python
from datetime import datetime
data['Open Time'] = data['Open Time'].apply(lambda x: datetime.fromtimestamp(int(x)/1000))
data.set_index(['Open Time'], inplace=True)
data = data.filter(['Open Time', 'Open', 'High', 'Low', 'Close', 'Volume'])
```

Then, again, the first 5 columns are shown with the data and columns fixed.


```python
data.head()
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
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
    </tr>
    <tr>
      <th>Open Time</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2021-05-23 05:30:00</th>
      <td>0.00505500</td>
      <td>0.00506300</td>
      <td>0.00501400</td>
      <td>0.00504600</td>
      <td>81838.00000000</td>
    </tr>
    <tr>
      <th>2021-05-23 05:35:00</th>
      <td>0.00504200</td>
      <td>0.00506600</td>
      <td>0.00500400</td>
      <td>0.00500400</td>
      <td>120175.00000000</td>
    </tr>
    <tr>
      <th>2021-05-23 05:40:00</th>
      <td>0.00501100</td>
      <td>0.00502200</td>
      <td>0.00498300</td>
      <td>0.00500000</td>
      <td>90252.00000000</td>
    </tr>
    <tr>
      <th>2021-05-23 05:45:00</th>
      <td>0.00500400</td>
      <td>0.00502500</td>
      <td>0.00498300</td>
      <td>0.00498300</td>
      <td>162699.00000000</td>
    </tr>
    <tr>
      <th>2021-05-23 05:50:00</th>
      <td>0.00499100</td>
      <td>0.00499500</td>
      <td>0.00493600</td>
      <td>0.00494000</td>
      <td>167842.00000000</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
