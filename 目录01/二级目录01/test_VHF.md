

```python
import os
import sys
sys.path.append(os.path.abspath(os.path.join(os.getcwd(), '..')))

import pandas as pd
pd.core.common.is_list_like = pd.api.types.is_list_like
try:
    import empyrical as emp
except:
    emp = None
import tushare as ts
import time
import random
# from concurrent.futures import ProcessPoolExecutor

from common.log import *
from common.config import Config
from spider.spider_nasdaq import Spider_nasdaq
from spider.spider_coinmarketcap import Spider_coinmarketcap

from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = 'all'

from pandas_highcharts.core import serialize
from pandas_highcharts.display import display_charts

from IPython.core.display import display, HTML
# display(HTML("<style>.container { width:70% !important; }</style>"))

CONF = Config('../conf/secret.yaml').data[0]
ts_token = CONF['TUSHARE']['TOKEN']
ts.set_token(ts_token)
pro = ts.pro_api()

CONF = Config().data[0]
MONGODB = CONF['MONGODB']
NASDAQ = CONF['NASDAQ']
CRYPTOCURRENCY = CONF['CRYPTOCURRENCY']
CRYPTOCURRENCY = list(CRYPTOCURRENCY.keys())
NASDAQ100 = CONF['NASDAQ100']

TEST_LIST = ['399300']

pd
```



<script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
<script src="https://code.highcharts.com/stock/highstock.js"></script>
<script src="https://code.highcharts.com/stock/modules/exporting.js"></script>
<script src="https://code.highcharts.com/stock/modules/export-data.js"></script>






    <module 'pandas' from 'd:\\python3\\lib\\site-packages\\pandas\\__init__.py'>




```python
%%time

stock_df_dict = {}

for symbol in TEST_LIST:
    stock_data_file = '../database/market/%s.csv' % symbol
    try:
        stock_df = pd.read_csv(stock_data_file)
    except:
        print(symbol)
        continue

    # 筛选字段
#     stock_df = stock_df.loc[:, ['date', 'open', 'close']]
    
    # 特殊处理，用当天收盘价做判定和交易
#     stock_df['open'] = stock_df['close']

    # 去掉Nasdaq行情首行的当天行情
    if symbol in NASDAQ100:
        stock_df = stock_df.drop([0])

    # 抛弃空值异常值
    stock_df.dropna(axis=0, how='any', inplace=True)

    # 格式化日期
    # 445 ms ± 17.5 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
    stock_df = stock_df.assign(date=pd.to_datetime(stock_df['date']))  # need .index.to_period('D')

    # 用日期作索引，日期升序排序
    # 95.1 µs ± 1.58 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)
    if symbol in NASDAQ100 or symbol in CRYPTOCURRENCY:
        stock_df = stock_df[::-1]
    stock_df.set_index(['date'], inplace=True)
    stock_df.index = stock_df.index.to_period('D')

    # 计算每天涨跌幅
#     stock_df['o_pct_chg'] = stock_df.open.pct_change(1)
#     stock_df['c_o_pct_chg'] = (stock_df.open - stock_df.close.shift(1)) / stock_df.close.shift(1)
#     stock_df['N_chg'] = stock_df.open.pct_change(N)
    # 用昨天收盘价做判定
#     stock_df['N_chg'] = (stock_df.close.shift(1) - stock_df.close.shift(N)) / stock_df.close.shift(N)
#     stock_df['N_sht'] = stock_df.open.shift(N)
#     stock_df['N_chn'] = stock_df.open.shift(N) - stock_df.open
#     stock_df['y_close'] = stock_df.close.shift(1)
    
    # MA均线指标
#     stock_df['MA%d' % M] = stock_df['open'].rolling(M).mean()
#     stock_df['MA%d' % M] = stock_df['close'].rolling(M).mean().shift(1)
    
    # 减少数据
    stock_df.dropna(how='any', inplace=True)
    
    stock_df_dict[symbol] = stock_df
```

    Wall time: 96.4 ms
    


```python
for symbol in TEST_LIST:
    symbol
    stock_df_dict[symbol].head(2)
    stock_df_dict[symbol].tail(2)
```




    '399300'






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
      <th>open</th>
      <th>close</th>
      <th>high</th>
      <th>low</th>
      <th>volume</th>
      <th>code</th>
    </tr>
    <tr>
      <th>date</th>
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
      <th>2005-04-08</th>
      <td>984.66</td>
      <td>1003.45</td>
      <td>1003.70</td>
      <td>979.53</td>
      <td>14762500.0</td>
      <td>sz399300</td>
    </tr>
    <tr>
      <th>2005-04-11</th>
      <td>1003.88</td>
      <td>995.42</td>
      <td>1008.73</td>
      <td>992.77</td>
      <td>15936100.0</td>
      <td>sz399300</td>
    </tr>
  </tbody>
</table>
</div>






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
      <th>open</th>
      <th>close</th>
      <th>high</th>
      <th>low</th>
      <th>volume</th>
      <th>code</th>
    </tr>
    <tr>
      <th>date</th>
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
      <th>2019-11-13</th>
      <td>3905.28</td>
      <td>3899.98</td>
      <td>3908.42</td>
      <td>3882.43</td>
      <td>76749078.0</td>
      <td>sz399300</td>
    </tr>
    <tr>
      <th>2019-11-14</th>
      <td>3905.93</td>
      <td>3905.86</td>
      <td>3916.95</td>
      <td>3895.14</td>
      <td>87008446.0</td>
      <td>sz399300</td>
    </tr>
  </tbody>
</table>
</div>




```python
'''
Select the number of periods (n) to include in the indicator. 
This should be based on the length of the cycle that you are analyzing. 
The most popular is 28 days (for intermediate cycles).
Determine the highest closing price (HCP) in n periods.
Determine the lowest closing price (LCP) in n periods.
Calculate the range of closing prices in n periods:
           HCP - LCP
Next, calculate the movement in closing price for each period:
           Closing price [today] - Closing price [yesterday]
Add up all price movements for n periods, disregarding whether they are up or down:
           Sum of absolute values of ( Close [today] - Close [yesterday] ) for n periods
Divide Step 4 by Step 6:
           VHF = (HCP - LCP) / (Sum of absolute values for n periods)
'''
```




    '\nSelect the number of periods (n) to include in the indicator. \nThis should be based on the length of the cycle that you are analyzing. \nThe most popular is 28 days (for intermediate cycles).\nDetermine the highest closing price (HCP) in n periods.\nDetermine the lowest closing price (LCP) in n periods.\nCalculate the range of closing prices in n periods:\n           HCP - LCP\nNext, calculate the movement in closing price for each period:\n           Closing price [today] - Closing price [yesterday]\nAdd up all price movements for n periods, disregarding whether they are up or down:\n           Sum of absolute values of ( Close [today] - Close [yesterday] ) for n periods\nDivide Step 4 by Step 6:\n           VHF = (HCP - LCP) / (Sum of absolute values for n periods)\n'




```python
df = stock_df_dict['399300'].copy()
N = 120
df['CLOSE_MA'] = df['close'].rolling(N).mean()
# df['close'] = df['CLOSE_MA']
df['HCP'] = df['close'].rolling(N).max()
df['LCP'] = df['close'].rolling(N).min()
df['DIFFCP'] = df['close'] - df['close'].shift()
df['DIFFCP'] = df['DIFFCP'].apply(abs)
df['SUMDIFFCP'] = df['DIFFCP'].rolling(N).sum()
df['VHF'] = (df['HCP'] - df['LCP']) / df['SUMDIFFCP']
df['VHF_MA'] = df['VHF'].rolling(N).mean()
df
o_df = df.copy()
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
      <th>open</th>
      <th>close</th>
      <th>high</th>
      <th>low</th>
      <th>volume</th>
      <th>code</th>
      <th>CLOSE_MA</th>
      <th>HCP</th>
      <th>LCP</th>
      <th>DIFFCP</th>
      <th>SUMDIFFCP</th>
      <th>VHF</th>
      <th>VHF_MA</th>
    </tr>
    <tr>
      <th>date</th>
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
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2005-04-08</th>
      <td>984.66</td>
      <td>1003.45</td>
      <td>1003.70</td>
      <td>979.53</td>
      <td>14762500.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-11</th>
      <td>1003.88</td>
      <td>995.42</td>
      <td>1008.73</td>
      <td>992.77</td>
      <td>15936100.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>8.03</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-12</th>
      <td>993.71</td>
      <td>978.70</td>
      <td>993.71</td>
      <td>978.20</td>
      <td>10226200.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>16.72</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-13</th>
      <td>987.95</td>
      <td>1000.90</td>
      <td>1006.50</td>
      <td>987.95</td>
      <td>16071700.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>22.20</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-14</th>
      <td>1004.64</td>
      <td>986.97</td>
      <td>1006.42</td>
      <td>985.58</td>
      <td>12945700.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>13.93</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-15</th>
      <td>982.61</td>
      <td>974.08</td>
      <td>982.61</td>
      <td>971.93</td>
      <td>10409000.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12.89</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-18</th>
      <td>970.91</td>
      <td>963.77</td>
      <td>970.91</td>
      <td>958.65</td>
      <td>8598400.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10.31</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-19</th>
      <td>962.92</td>
      <td>965.89</td>
      <td>968.87</td>
      <td>957.91</td>
      <td>9212620.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.12</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-20</th>
      <td>964.15</td>
      <td>950.87</td>
      <td>964.15</td>
      <td>946.20</td>
      <td>8850700.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>15.02</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-21</th>
      <td>948.86</td>
      <td>943.98</td>
      <td>955.55</td>
      <td>938.59</td>
      <td>9946150.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.89</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-22</th>
      <td>942.91</td>
      <td>939.10</td>
      <td>947.91</td>
      <td>934.96</td>
      <td>10691900.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4.88</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-25</th>
      <td>935.99</td>
      <td>930.07</td>
      <td>935.99</td>
      <td>920.16</td>
      <td>11470500.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9.03</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-26</th>
      <td>928.43</td>
      <td>937.08</td>
      <td>939.70</td>
      <td>924.66</td>
      <td>11690500.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>7.01</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-27</th>
      <td>938.57</td>
      <td>926.60</td>
      <td>938.91</td>
      <td>925.90</td>
      <td>10780600.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10.48</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-28</th>
      <td>923.53</td>
      <td>942.07</td>
      <td>945.50</td>
      <td>914.83</td>
      <td>14343500.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>15.47</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-04-29</th>
      <td>940.81</td>
      <td>932.40</td>
      <td>942.45</td>
      <td>929.81</td>
      <td>11235400.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9.67</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-09</th>
      <td>934.65</td>
      <td>909.17</td>
      <td>937.39</td>
      <td>909.17</td>
      <td>8529110.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>23.23</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-10</th>
      <td>905.54</td>
      <td>913.08</td>
      <td>913.39</td>
      <td>892.31</td>
      <td>10494300.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.91</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-11</th>
      <td>911.84</td>
      <td>901.85</td>
      <td>917.22</td>
      <td>900.44</td>
      <td>9042050.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>11.23</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-12</th>
      <td>899.97</td>
      <td>885.82</td>
      <td>900.06</td>
      <td>883.51</td>
      <td>10228700.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>16.03</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-13</th>
      <td>883.51</td>
      <td>887.54</td>
      <td>898.51</td>
      <td>875.58</td>
      <td>11244900.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.72</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-16</th>
      <td>885.39</td>
      <td>875.27</td>
      <td>885.39</td>
      <td>869.33</td>
      <td>8224289.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12.27</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-17</th>
      <td>873.08</td>
      <td>881.46</td>
      <td>888.28</td>
      <td>868.21</td>
      <td>8772270.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.19</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-18</th>
      <td>881.14</td>
      <td>883.20</td>
      <td>890.40</td>
      <td>871.82</td>
      <td>7878610.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.74</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-19</th>
      <td>882.84</td>
      <td>884.17</td>
      <td>888.02</td>
      <td>871.29</td>
      <td>8149579.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.97</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-20</th>
      <td>883.51</td>
      <td>882.76</td>
      <td>891.02</td>
      <td>879.18</td>
      <td>7226679.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.41</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-23</th>
      <td>880.28</td>
      <td>863.34</td>
      <td>880.28</td>
      <td>862.10</td>
      <td>7279800.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>19.42</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-24</th>
      <td>861.20</td>
      <td>868.46</td>
      <td>871.77</td>
      <td>855.59</td>
      <td>9206929.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5.12</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-25</th>
      <td>867.66</td>
      <td>868.45</td>
      <td>876.30</td>
      <td>861.66</td>
      <td>7238560.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.01</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2005-05-26</th>
      <td>867.76</td>
      <td>857.33</td>
      <td>872.84</td>
      <td>854.96</td>
      <td>6622789.0</td>
      <td>sz399300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>11.12</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2019-09-27</th>
      <td>3843.46</td>
      <td>3852.65</td>
      <td>3859.50</td>
      <td>3832.25</td>
      <td>74003947.0</td>
      <td>sz399300</td>
      <td>3805.445500</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>11.51</td>
      <td>4077.60</td>
      <td>0.136338</td>
      <td>0.200318</td>
    </tr>
    <tr>
      <th>2019-09-30</th>
      <td>3842.07</td>
      <td>3814.53</td>
      <td>3857.23</td>
      <td>3813.55</td>
      <td>65931313.0</td>
      <td>sz399300</td>
      <td>3803.423000</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>38.12</td>
      <td>4110.72</td>
      <td>0.135239</td>
      <td>0.199494</td>
    </tr>
    <tr>
      <th>2019-10-08</th>
      <td>3818.59</td>
      <td>3837.68</td>
      <td>3861.64</td>
      <td>3818.59</td>
      <td>76657860.0</td>
      <td>sz399300</td>
      <td>3801.441750</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>23.15</td>
      <td>4115.67</td>
      <td>0.135076</td>
      <td>0.198651</td>
    </tr>
    <tr>
      <th>2019-10-09</th>
      <td>3822.61</td>
      <td>3843.24</td>
      <td>3845.34</td>
      <td>3805.07</td>
      <td>75854512.0</td>
      <td>sz399300</td>
      <td>3799.420000</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>5.56</td>
      <td>4110.81</td>
      <td>0.135236</td>
      <td>0.197726</td>
    </tr>
    <tr>
      <th>2019-10-10</th>
      <td>3838.49</td>
      <td>3874.64</td>
      <td>3877.14</td>
      <td>3829.43</td>
      <td>79222694.0</td>
      <td>sz399300</td>
      <td>3798.395500</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>31.40</td>
      <td>4053.94</td>
      <td>0.137133</td>
      <td>0.196836</td>
    </tr>
    <tr>
      <th>2019-10-11</th>
      <td>3885.52</td>
      <td>3911.73</td>
      <td>3922.37</td>
      <td>3868.84</td>
      <td>93412577.0</td>
      <td>sz399300</td>
      <td>3797.754750</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>37.09</td>
      <td>4082.07</td>
      <td>0.136188</td>
      <td>0.195921</td>
    </tr>
    <tr>
      <th>2019-10-14</th>
      <td>3944.86</td>
      <td>3953.24</td>
      <td>3983.81</td>
      <td>3934.73</td>
      <td>122393611.0</td>
      <td>sz399300</td>
      <td>3797.569083</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>41.51</td>
      <td>4110.48</td>
      <td>0.135247</td>
      <td>0.194994</td>
    </tr>
    <tr>
      <th>2019-10-15</th>
      <td>3953.16</td>
      <td>3936.25</td>
      <td>3953.16</td>
      <td>3928.15</td>
      <td>85273949.0</td>
      <td>sz399300</td>
      <td>3796.322917</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>16.99</td>
      <td>4017.20</td>
      <td>0.138387</td>
      <td>0.194134</td>
    </tr>
    <tr>
      <th>2019-10-16</th>
      <td>3939.28</td>
      <td>3922.69</td>
      <td>3964.05</td>
      <td>3919.02</td>
      <td>83229707.0</td>
      <td>sz399300</td>
      <td>3794.951667</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>13.56</td>
      <td>4029.31</td>
      <td>0.137972</td>
      <td>0.193236</td>
    </tr>
    <tr>
      <th>2019-10-17</th>
      <td>3929.39</td>
      <td>3925.22</td>
      <td>3935.81</td>
      <td>3913.92</td>
      <td>64863745.0</td>
      <td>sz399300</td>
      <td>3793.727833</td>
      <td>4120.61</td>
      <td>3564.68</td>
      <td>2.53</td>
      <td>4016.68</td>
      <td>0.138405</td>
      <td>0.192307</td>
    </tr>
    <tr>
      <th>2019-10-18</th>
      <td>3935.42</td>
      <td>3869.38</td>
      <td>3940.54</td>
      <td>3864.97</td>
      <td>82321140.0</td>
      <td>sz399300</td>
      <td>3791.634250</td>
      <td>4030.09</td>
      <td>3564.68</td>
      <td>55.84</td>
      <td>4023.99</td>
      <td>0.115659</td>
      <td>0.191084</td>
    </tr>
    <tr>
      <th>2019-10-21</th>
      <td>3865.34</td>
      <td>3880.84</td>
      <td>3884.74</td>
      <td>3856.04</td>
      <td>74182918.0</td>
      <td>sz399300</td>
      <td>3790.427833</td>
      <td>4030.09</td>
      <td>3564.68</td>
      <td>11.46</td>
      <td>3940.45</td>
      <td>0.118111</td>
      <td>0.189886</td>
    </tr>
    <tr>
      <th>2019-10-22</th>
      <td>3896.31</td>
      <td>3895.88</td>
      <td>3897.44</td>
      <td>3870.55</td>
      <td>68918566.0</td>
      <td>sz399300</td>
      <td>3789.401750</td>
      <td>4030.09</td>
      <td>3564.68</td>
      <td>15.04</td>
      <td>3948.89</td>
      <td>0.117858</td>
      <td>0.188687</td>
    </tr>
    <tr>
      <th>2019-10-23</th>
      <td>3892.68</td>
      <td>3871.08</td>
      <td>3901.42</td>
      <td>3862.20</td>
      <td>70198625.0</td>
      <td>sz399300</td>
      <td>3788.076667</td>
      <td>3972.95</td>
      <td>3564.68</td>
      <td>24.80</td>
      <td>3962.61</td>
      <td>0.103031</td>
      <td>0.187367</td>
    </tr>
    <tr>
      <th>2019-10-24</th>
      <td>3877.92</td>
      <td>3870.67</td>
      <td>3890.20</td>
      <td>3852.58</td>
      <td>71334575.0</td>
      <td>sz399300</td>
      <td>3787.483750</td>
      <td>3972.95</td>
      <td>3564.68</td>
      <td>0.41</td>
      <td>3874.75</td>
      <td>0.105367</td>
      <td>0.186099</td>
    </tr>
    <tr>
      <th>2019-10-25</th>
      <td>3871.77</td>
      <td>3896.79</td>
      <td>3899.72</td>
      <td>3849.03</td>
      <td>77453620.0</td>
      <td>sz399300</td>
      <td>3787.546417</td>
      <td>3972.95</td>
      <td>3564.68</td>
      <td>26.12</td>
      <td>3848.32</td>
      <td>0.106090</td>
      <td>0.184815</td>
    </tr>
    <tr>
      <th>2019-10-28</th>
      <td>3904.98</td>
      <td>3926.58</td>
      <td>3927.80</td>
      <td>3897.82</td>
      <td>97406020.0</td>
      <td>sz399300</td>
      <td>3787.765167</td>
      <td>3972.95</td>
      <td>3564.68</td>
      <td>29.79</td>
      <td>3867.05</td>
      <td>0.105577</td>
      <td>0.183517</td>
    </tr>
    <tr>
      <th>2019-10-29</th>
      <td>3929.71</td>
      <td>3910.23</td>
      <td>3932.37</td>
      <td>3910.23</td>
      <td>82257075.0</td>
      <td>sz399300</td>
      <td>3787.740333</td>
      <td>3972.95</td>
      <td>3564.68</td>
      <td>16.35</td>
      <td>3870.52</td>
      <td>0.105482</td>
      <td>0.182202</td>
    </tr>
    <tr>
      <th>2019-10-30</th>
      <td>3904.72</td>
      <td>3891.23</td>
      <td>3908.10</td>
      <td>3883.45</td>
      <td>77872923.0</td>
      <td>sz399300</td>
      <td>3789.462083</td>
      <td>3972.95</td>
      <td>3564.68</td>
      <td>19.00</td>
      <td>3660.93</td>
      <td>0.111521</td>
      <td>0.181036</td>
    </tr>
    <tr>
      <th>2019-10-31</th>
      <td>3906.86</td>
      <td>3886.75</td>
      <td>3907.15</td>
      <td>3878.88</td>
      <td>86714539.0</td>
      <td>sz399300</td>
      <td>3790.846083</td>
      <td>3972.95</td>
      <td>3564.68</td>
      <td>4.48</td>
      <td>3629.36</td>
      <td>0.112491</td>
      <td>0.179842</td>
    </tr>
    <tr>
      <th>2019-11-01</th>
      <td>3883.81</td>
      <td>3952.39</td>
      <td>3956.08</td>
      <td>3876.42</td>
      <td>100689612.0</td>
      <td>sz399300</td>
      <td>3793.220500</td>
      <td>3972.95</td>
      <td>3564.68</td>
      <td>65.64</td>
      <td>3641.79</td>
      <td>0.112107</td>
      <td>0.178657</td>
    </tr>
    <tr>
      <th>2019-11-04</th>
      <td>3964.01</td>
      <td>3978.12</td>
      <td>3985.36</td>
      <td>3964.01</td>
      <td>104193855.0</td>
      <td>sz399300</td>
      <td>3796.374000</td>
      <td>3978.12</td>
      <td>3564.68</td>
      <td>25.73</td>
      <td>3599.76</td>
      <td>0.114852</td>
      <td>0.177517</td>
    </tr>
    <tr>
      <th>2019-11-05</th>
      <td>3982.12</td>
      <td>4002.81</td>
      <td>4030.64</td>
      <td>3970.87</td>
      <td>112911880.0</td>
      <td>sz399300</td>
      <td>3798.643667</td>
      <td>4002.81</td>
      <td>3564.68</td>
      <td>24.69</td>
      <td>3493.70</td>
      <td>0.125406</td>
      <td>0.176514</td>
    </tr>
    <tr>
      <th>2019-11-06</th>
      <td>4006.33</td>
      <td>3984.88</td>
      <td>4007.50</td>
      <td>3973.78</td>
      <td>97312952.0</td>
      <td>sz399300</td>
      <td>3801.278250</td>
      <td>4002.81</td>
      <td>3564.68</td>
      <td>17.93</td>
      <td>3449.91</td>
      <td>0.126998</td>
      <td>0.175546</td>
    </tr>
    <tr>
      <th>2019-11-07</th>
      <td>3984.55</td>
      <td>3991.88</td>
      <td>4005.38</td>
      <td>3975.55</td>
      <td>87254898.0</td>
      <td>sz399300</td>
      <td>3804.167667</td>
      <td>4002.81</td>
      <td>3564.68</td>
      <td>7.00</td>
      <td>3433.33</td>
      <td>0.127611</td>
      <td>0.174575</td>
    </tr>
    <tr>
      <th>2019-11-08</th>
      <td>4017.67</td>
      <td>3973.01</td>
      <td>4022.04</td>
      <td>3971.57</td>
      <td>97106948.0</td>
      <td>sz399300</td>
      <td>3806.217000</td>
      <td>4002.81</td>
      <td>3564.68</td>
      <td>18.87</td>
      <td>3370.26</td>
      <td>0.129999</td>
      <td>0.173642</td>
    </tr>
    <tr>
      <th>2019-11-11</th>
      <td>3948.62</td>
      <td>3902.98</td>
      <td>3948.62</td>
      <td>3897.55</td>
      <td>94155621.0</td>
      <td>sz399300</td>
      <td>3807.542167</td>
      <td>4002.81</td>
      <td>3564.68</td>
      <td>70.03</td>
      <td>3423.42</td>
      <td>0.127980</td>
      <td>0.172686</td>
    </tr>
    <tr>
      <th>2019-11-12</th>
      <td>3905.96</td>
      <td>3903.69</td>
      <td>3914.88</td>
      <td>3877.74</td>
      <td>76543069.0</td>
      <td>sz399300</td>
      <td>3809.666583</td>
      <td>4002.81</td>
      <td>3564.68</td>
      <td>0.71</td>
      <td>3328.93</td>
      <td>0.131613</td>
      <td>0.171787</td>
    </tr>
    <tr>
      <th>2019-11-13</th>
      <td>3905.28</td>
      <td>3899.98</td>
      <td>3908.42</td>
      <td>3882.43</td>
      <td>76749078.0</td>
      <td>sz399300</td>
      <td>3812.018167</td>
      <td>4002.81</td>
      <td>3564.68</td>
      <td>3.71</td>
      <td>3301.67</td>
      <td>0.132700</td>
      <td>0.170894</td>
    </tr>
    <tr>
      <th>2019-11-14</th>
      <td>3905.93</td>
      <td>3905.86</td>
      <td>3916.95</td>
      <td>3895.14</td>
      <td>87008446.0</td>
      <td>sz399300</td>
      <td>3814.010500</td>
      <td>4002.81</td>
      <td>3564.68</td>
      <td>5.88</td>
      <td>3258.56</td>
      <td>0.134455</td>
      <td>0.170029</td>
    </tr>
  </tbody>
</table>
<p>3551 rows × 13 columns</p>
</div>




```python
%matplotlib inline

import matplotlib
import matplotlib.pyplot as plt
plt.rcParams['figure.figsize'] = [15, 5]

df = o_df.copy()
# df = o_df.loc[:, ['close', 'VHF_MA20']].copy()
# df.drop(columns=['PROPERTY'], inplace=True)
df.columns
df = df.dropna(how='any', inplace=False)

# df['close'] = (df['close'] - df.iloc[0]['close']) / df.iloc[0]['close']
# ax = df.plot(kind='line', title='VHF_N%d' % N, linewidth=1, grid=True)
# ax = df.plot(kind='line', y='CLOSE_MA', secondary_y=False, title='VHF_N%d' % N, linewidth=1, grid=True)
# ax = df.plot(kind='line', y='VHF_MA', secondary_y=True, title='VHF_N%d' % N, linewidth=0.5, grid=True, ax=ax)

df.reset_index(drop=False, inplace=True)
df['date'] = df['date'].apply(lambda x: x.to_timestamp().to_datetime64())
df.set_index(keys=['date'], inplace=True)
show_columns = ['close', 'VHF']
df = df.loc[:, show_columns]
display_charts(df, secondary_y=['VHF'], chart_type='stock', kind='line', figsize=(900, 600), logy=False)
```




    Index(['open', 'close', 'high', 'low', 'volume', 'code', 'CLOSE_MA', 'HCP',
           'LCP', 'DIFFCP', 'SUMDIFFCP', 'VHF', 'VHF_MA'],
          dtype='object')




<div id="chart_idhTYXlpLI"</div>
    <script type="text/javascript">new Highcharts.StockChart({"chart":{"renderTo":"chart_idhTYXlpLI","width":900,"height":600,"type":"line"},"legend":{"enabled":true},"series":[{"name":"VHF (right)","yAxis":1,"data":[[1144195200000,0.2933383161],[1144281600000,0.296618834],[1144368000000,0.3002227931],[1144627200000,0.3171714719],[1144713600000,0.3230836332],[1144800000000,0.3233735524],[1144886400000,0.3189374731],[1144972800000,0.3124099392],[1145232000000,0.3133132079],[1145318400000,0.3201380146],[1145404800000,0.3260504591],[1145491200000,0.3262922494],[1145577600000,0.334090935],[1145836800000,0.3363652044],[1145923200000,0.343345423],[1146009600000,0.3454732368],[1146096000000,0.3485572441],[1146182400000,0.3643045891],[1147046400000,0.3978972374],[1147132800000,0.4234104821],[1147219200000,0.4357625765],[1147305600000,0.4324886935],[1147392000000,0.4456360872],[1147651200000,0.4773060029],[1147737600000,0.4688339704],[1147824000000,0.4741682057],[1147910400000,0.4730365717],[1147996800000,0.4718139638],[1148256000000,0.4751675371],[1148342400000,0.4543614492],[1148428800000,0.4507953768],[1148515200000,0.4584500134],[1148601600000,0.4497502165],[1148860800000,0.4416285083],[1148947200000,0.4437672562],[1149033600000,0.4407345151],[1149120000000,0.4479690618],[1149206400000,0.444730961],[1149465600000,0.4434427901],[1149552000000,0.4429007945],[1149638400000,0.4163857821],[1149724800000,0.41555175],[1149811200000,0.403977495],[1150070400000,0.3990380629],[1150156800000,0.4010338503],[1150243200000,0.3867958486],[1150329600000,0.3893684792],[1150416000000,0.379887525],[1150675200000,0.3700136547],[1150761600000,0.3715655866],[1150848000000,0.3662730293],[1150934400000,0.3671317124],[1151020800000,0.364940809],[1151280000000,0.3598068634],[1151366400000,0.3567171717],[1151452800000,0.3527363544],[1151539200000,0.3439768502],[1151625600000,0.345285335],[1151884800000,0.3505837095],[1151971200000,0.3469178297],[1152057600000,0.3452569554],[1152144000000,0.3288583691],[1152230400000,0.3188341675],[1152489600000,0.3207876911],[1152576000000,0.3217887045],[1152662400000,0.3228277769],[1152748800000,0.3076330044],[1152835200000,0.306324889],[1153094400000,0.3053302549],[1153180800000,0.3060919563],[1153267200000,0.3003029106],[1153353600000,0.2864987207],[1153440000000,0.2830895693],[1153699200000,0.2827186652],[1153785600000,0.2782847733],[1153872000000,0.2763534259],[1153958400000,0.2706330102],[1154044800000,0.2699918871],[1154304000000,0.2659476531],[1154390400000,0.2644295558],[1154476800000,0.263785669],[1154563200000,0.2654893802],[1154649600000,0.2626249235],[1154908800000,0.259823241],[1154995200000,0.2564009319],[1155081600000,0.2567237315],[1155168000000,0.2569409701],[1155254400000,0.2563187795],[1155513600000,0.2518709131],[1155600000000,0.2514203172],[1155686400000,0.2488960953],[1155772800000,0.2476425765],[1155859200000,0.2482796078],[1156118400000,0.248118193],[1156204800000,0.2467143501],[1156291200000,0.2471834668],[1156377600000,0.2488439842],[1156464000000,0.2488395186],[1156723200000,0.244792157],[1156809600000,0.2476528966],[1156896000000,0.24782847],[1156982400000,0.2452447798],[1157068800000,0.2373912119],[1157328000000,0.2362501763],[1157414400000,0.2326186102],[1157500800000,0.2333033102],[1157587200000,0.2310387445],[1157673600000,0.2230648612],[1157932800000,0.2221682431],[1158019200000,0.2203458052],[1158105600000,0.2200454387],[1158192000000,0.2201429038],[1158278400000,0.213167699],[1158537600000,0.2097097874],[1158624000000,0.2100141084],[1158710400000,0.2111277708],[1158796800000,0.2080572673],[1158883200000,0.1966949299],[1159142400000,0.1926515787],[1159228800000,0.1894788172],[1159315200000,0.1891636579],[1159401600000,0.1877912663],[1159488000000,0.186059238],[1160352000000,0.1930464025],[1160438400000,0.19416998],[1160524800000,0.1947106932],[1160611200000,0.1821315271],[1160697600000,0.1809154787],[1160956800000,0.1762736863],[1161043200000,0.1747696072],[1161129600000,0.1733530007],[1161216000000,0.1702613593],[1161302400000,0.1720964317],[1161561600000,0.1696481898],[1161648000000,0.1592904027],[1161734400000,0.1636459567],[1161820800000,0.1584785523],[1161907200000,0.1327386671],[1162166400000,0.1324552088],[1162252800000,0.1384205192],[1162339200000,0.1469714588],[1162425600000,0.1480220098],[1162512000000,0.1559646026],[1162771200000,0.1712043243],[1162857600000,0.1775302622],[1162944000000,0.1760807559],[1163030400000,0.1788759632],[1163116800000,0.1804056893],[1163376000000,0.1781910006],[1163462400000,0.1822994682],[1163548800000,0.1848550483],[1163635200000,0.1847528085],[1163721600000,0.200348556],[1163980800000,0.2193169595],[1164067200000,0.2297574864],[1164153600000,0.2369450073],[1164240000000,0.2472807604],[1164326400000,0.2499545515],[1164585600000,0.258835633],[1164672000000,0.2582464361],[1164758400000,0.2768325221],[1164844800000,0.2986003679],[1164931200000,0.3108564677],[1165190400000,0.3327256317],[1165276800000,0.3381854851],[1165363200000,0.3381012531],[1165449600000,0.3376627222],[1165536000000,0.3314767118],[1165795200000,0.3200406415],[1165881600000,0.3231154067],[1165968000000,0.3243684785],[1166054400000,0.3367204907],[1166140800000,0.3495125594],[1166400000000,0.3708997942],[1166486400000,0.3727894793],[1166572800000,0.3780177217],[1166659200000,0.3789547031],[1166745600000,0.3765154158],[1167004800000,0.3744808334],[1167091200000,0.376147513],[1167177600000,0.3936622239],[1167264000000,0.398357807],[1167350400000,0.4173137043],[1167868800000,0.4253250521],[1167955200000,0.4283890114],[1168214400000,0.4449685689],[1168300800000,0.4796491056],[1168387200000,0.496177223],[1168473600000,0.4939871509],[1168560000000,0.4809304754],[1168819200000,0.4784240319],[1168905600000,0.4954349313],[1168992000000,0.4881354614],[1169078400000,0.4868586056],[1169164800000,0.4917137966],[1169424000000,0.5118323956],[1169510400000,0.5184020348],[1169596800000,0.5268201233],[1169683200000,0.5192041399],[1169769600000,0.5095635629],[1170028800000,0.513905403],[1170115200000,0.5097056652],[1170201600000,0.4847357785],[1170288000000,0.4783550795],[1170374400000,0.4668010871],[1170633600000,0.4627267229],[1170720000000,0.4588873223],[1170806400000,0.4511777665],[1170892800000,0.4427192728],[1170979200000,0.443056397],[1171238400000,0.4327409406],[1171324800000,0.4291516956],[1171411200000,0.4234162736],[1171497600000,0.4336728456],[1171584000000,0.436990893],[1172448000000,0.440375879],[1172534400000,0.4084841508],[1172620800000,0.3923915161],[1172707200000,0.3879460285],[1172793600000,0.3846258176],[1173052800000,0.3816038798],[1173139200000,0.3773899535],[1173225600000,0.3697220852],[1173312000000,0.3678437417],[1173398400000,0.3665923365],[1173657600000,0.3666810223],[1173744000000,0.3650954589],[1173830400000,0.3597407681],[1173916800000,0.3558537606],[1174003200000,0.352879028],[1174262400000,0.3487513307],[1174348800000,0.342661993],[1174435200000,0.3421591537],[1174521600000,0.3434751719],[1174608000000,0.344538838],[1174867200000,0.3523958195],[1174953600000,0.3564151834],[1175040000000,0.3597212161],[1175126400000,0.3586550436],[1175212800000,0.3564747599],[1175472000000,0.3606712961],[1175558400000,0.3640429053],[1175644800000,0.3678512257],[1175731200000,0.375942407],[1175817600000,0.3801420578],[1176076800000,0.3900805316],[1176163200000,0.3972378616],[1176249600000,0.4032896906],[1176336000000,0.4121197568],[1176422400000,0.4118182877],[1176681600000,0.4240667749],[1176768000000,0.4278675655],[1176854400000,0.4306616932],[1176940800000,0.4122750991],[1177027200000,0.4026953573],[1177286400000,0.4178699074],[1177372800000,0.4203747048],[1177459200000,0.4207442527],[1177545600000,0.4230450084],[1177632000000,0.4202638052],[1177891200000,0.4273097659],[1178582400000,0.4419094728],[1178668800000,0.4443703439],[1178755200000,0.4486835174],[1178841600000,0.4474612628],[1179100800000,0.4481953245],[1179187200000,0.4391976899],[1179273600000,0.4328845361],[1179360000000,0.4337431279],[1179446400000,0.4275437244],[1179705600000,0.4364559356],[1179792000000,0.4352977338],[1179878400000,0.4390620525],[1179964800000,0.4364594764],[1180051200000,0.4391121129],[1180310400000,0.4470484026],[1180396800000,0.4571821209],[1180483200000,0.433889099],[1180569600000,0.4319387003],[1180656000000,0.4196363258],[1180915200000,0.3943563624],[1181001600000,0.389604561],[1181088000000,0.3878766732],[1181174400000,0.3834451392],[1181260800000,0.3821282248],[1181520000000,0.3774797601],[1181606400000,0.3717365186],[1181692800000,0.3589033571],[1181779200000,0.3589050324],[1181865600000,0.3581617706],[1182124800000,0.355407962],[1182211200000,0.3548671197],[1182297600000,0.3473991046],[1182384000000,0.3478496964],[1182470400000,0.3407835524],[1182729600000,0.3331392555],[1182816000000,0.3320576117],[1182902400000,0.3215757316],[1182988800000,0.3155051059],[1183075200000,0.3059206542],[1183334400000,0.3075023945],[1183420800000,0.2963626425],[1183507200000,0.2917974314],[1183593600000,0.2841920981],[1183680000000,0.2706491425],[1183939200000,0.2635070742],[1184025600000,0.2642723597],[1184112000000,0.2656781069],[1184198400000,0.2655861416],[1184284800000,0.2548419586],[1184544000000,0.2545616675],[1184630400000,0.253741704],[1184716800000,0.2546228467],[1184803200000,0.2548714644],[1184889600000,0.2520864056],[1185148800000,0.2492442284],[1185235200000,0.2496270462],[1185321600000,0.2478815863],[1185408000000,0.2549889664],[1185494400000,0.2572979506],[1185753600000,0.2690072456],[1185840000000,0.2744588593],[1185926400000,0.2743374257],[1186012800000,0.2697437338],[1186099200000,0.2844494109],[1186358400000,0.2891444596],[1186444800000,0.2859463969],[1186531200000,0.2825188645],[1186617600000,0.2865422516],[1186704000000,0.2780424603],[1186963200000,0.2808243389],[1187049600000,0.2817703927],[1187136000000,0.2842963445],[1187222400000,0.2844161855],[1187308800000,0.2814332389],[1187568000000,0.2840702612],[1187654400000,0.2980858916],[1187740800000,0.3078035234],[1187827200000,0.317113612],[1187913600000,0.3250462621],[1188172800000,0.3230700231],[1188259200000,0.3172454308],[1188345600000,0.31589514],[1188432000000,0.3147258016],[1188518400000,0.3185800708],[1188777600000,0.3284627096],[1188864000000,0.3271360338],[1188950400000,0.3278557544],[1189036800000,0.3278328448],[1189123200000,0.3185892197],[1189382400000,0.3160527015],[1189468800000,0.3042255635],[1189555200000,0.3015960527],[1189641600000,0.2964749654],[1189728000000,0.2898893456],[1189987200000,0.2949228375],[1190073600000,0.2948562685],[1190160000000,0.2934569891],[1190246400000,0.2915269185],[1190332800000,0.2834262822],[1190592000000,0.2816431427],[1190678400000,0.2784644048],[1190764800000,0.2728684509],[1190851200000,0.2690485515],[1190937600000,0.2655809348],[1191801600000,0.2684297055],[1191888000000,0.2672342731],[1191974400000,0.2660637682],[1192060800000,0.2733120251],[1192147200000,0.2728648097],[1192406400000,0.2793555209],[1192492800000,0.2843488725],[1192579200000,0.2833979065],[1192665600000,0.2674485058],[1192752000000,0.2564073534],[1193011200000,0.2549699215],[1193097600000,0.2532258956],[1193184000000,0.2497397982],[1193270400000,0.2444404495],[1193356800000,0.239362461],[1193616000000,0.2387498625],[1193702400000,0.2397080668],[1193788800000,0.2378471035],[1193875200000,0.2364190721],[1193961600000,0.2338392105],[1194220800000,0.2319862246],[1194307200000,0.2339828323],[1194393600000,0.2354398256],[1194480000000,0.231327033],[1194566400000,0.2301751389],[1194825600000,0.2300081959],[1194912000000,0.2300090904],[1194998400000,0.2269598052],[1195084800000,0.2259716963],[1195171200000,0.2258002323],[1195430400000,0.2274083501],[1195516800000,0.2278628414],[1195603200000,0.2325684502],[1195689600000,0.2284472478],[1195776000000,0.2293432835],[1196035200000,0.2321427686],[1196121600000,0.2329347815],[1196208000000,0.2324846112],[1196294400000,0.230910661],[1196380800000,0.2293468629],[1196640000000,0.2306748128],[1196726400000,0.2317649623],[1196812800000,0.2305186808],[1196899200000,0.23136985],[1196985600000,0.2303056193],[1197244800000,0.231124159],[1197331200000,0.231558529],[1197417600000,0.2323124108],[1197504000000,0.2288269363],[1197590400000,0.2300079036],[1197849600000,0.2312235153],[1197936000000,0.2317557796],[1198022400000,0.2316614703],[1198108800000,0.2337691143],[1198195200000,0.2344667389],[1198454400000,0.2321667247],[1198540800000,0.2336712926],[1198627200000,0.2346186204],[1198713600000,0.2208163567],[1198800000000,0.2240763609],[1199232000000,0.2255651472],[1199318400000,0.2257696233],[1199404800000,0.2246670557],[1199664000000,0.2236091573],[1199750400000,0.2230292548],[1199836800000,0.2144461874],[1199923200000,0.2133936413],[1200009600000,0.2131941012],[1200268800000,0.1955690772],[1200355200000,0.178976726],[1200441600000,0.1783857795],[1200528000000,0.1662601764],[1200614400000,0.1634282524],[1200873600000,0.1597908556],[1200960000000,0.1537833801],[1201046400000,0.1520433654],[1201132800000,0.1520165629],[1201219200000,0.1396604946],[1201478400000,0.1215863789],[1201564800000,0.1204163377],[1201651200000,0.1210476953],[1201737600000,0.1208309338],[1201824000000,0.1255865552],[1202083200000,0.1224183548],[1202169600000,0.1226751579],[1202860800000,0.1215299859],[1202947200000,0.121645626],[1203033600000,0.1209270524],[1203292800000,0.1206936887],[1203379200000,0.1205331954],[1203465600000,0.1221898424],[1203552000000,0.1228174835],[1203638400000,0.1217315073],[1203897600000,0.1254469455],[1203984000000,0.1267463258],[1204070400000,0.1255928367],[1204156800000,0.1254876265],[1204243200000,0.1258059956],[1204502400000,0.1252645959],[1204588800000,0.124531294],[1204675200000,0.1254483193],[1204761600000,0.1254775661],[1204848000000,0.1247828143],[1205107200000,0.1307814718],[1205193600000,0.1320677839],[1205280000000,0.142568442],[1205366400000,0.1546390138],[1205452800000,0.1589771566],[1205712000000,0.1760445949],[1205798400000,0.1918616],[1205884800000,0.1914569391],[1205971200000,0.1898930956],[1206057600000,0.1902618695],[1206316800000,0.1884785701],[1206403200000,0.188111814],[1206489600000,0.188736792],[1206576000000,0.1882937066],[1206662400000,0.1870420139],[1206921600000,0.186044804],[1207008000000,0.1996104102],[1207094400000,0.2033067171],[1207180800000,0.2018981313],[1207526400000,0.1987066943],[1207612800000,0.1992008771],[1207699200000,0.1961795349],[1207785600000,0.196471274],[1207872000000,0.1924279686],[1208131200000,0.18260479],[1208217600000,0.1850895127],[1208304000000,0.187273622],[1208390400000,0.1968208274],[1208476800000,0.2055933194],[1208736000000,0.2067498406],[1208822400000,0.2107367375],[1208908800000,0.209013091],[1208995200000,0.2054028687],[1209081600000,0.2064204352],[1209340800000,0.206741688],[1209427200000,0.2073596297],[1209513600000,0.2064929112],[1209945600000,0.206769444],[1210032000000,0.2067324955],[1210118400000,0.2040535628],[1210204800000,0.2066761441],[1210291200000,0.2067980752],[1210550400000,0.2074294493],[1210636800000,0.2071814541],[1210723200000,0.2086296217],[1210809600000,0.2092868208],[1210896000000,0.2103853549],[1211155200000,0.2102272291],[1211241600000,0.2079512639],[1211328000000,0.2079430163],[1211414400000,0.2106698971],[1211500800000,0.2115243477],[1211760000000,0.2104432082],[1211846400000,0.2117442753],[1211932800000,0.2110618143],[1212019200000,0.2128476835],[1212105600000,0.2142206389],[1212364800000,0.2146079471],[1212451200000,0.2154489244],[1212537600000,0.2167670507],[1212624000000,0.2162027711],[1212710400000,0.2171104394],[1213056000000,0.2188070803],[1213142400000,0.2233905546],[1213228800000,0.2283260477],[1213315200000,0.2392339341],[1213574400000,0.2429737429],[1213660800000,0.2527896654],[1213747200000,0.2501367965],[1213833600000,0.2539244976],[1213920000000,0.2542367347],[1214179200000,0.2543444828],[1214265600000,0.2552947684],[1214352000000,0.2529389487],[1214438400000,0.2537383065],[1214524800000,0.2523878783],[1214784000000,0.2524968658],[1214870400000,0.2578480882],[1214956800000,0.2586324939],[1215043200000,0.258645946],[1215129600000,0.2598463576],[1215388800000,0.2573689806],[1215475200000,0.2588322127],[1215561600000,0.2576253769],[1215648000000,0.2573106906],[1215734400000,0.2545828479],[1215993600000,0.2386492887],[1216080000000,0.2322315647],[1216166400000,0.2328830783],[1216252800000,0.2102253107],[1216339200000,0.2074112821],[1216598400000,0.2129105931],[1216684800000,0.2170861791],[1216771200000,0.2176956152],[1216857600000,0.2116604647],[1216944000000,0.2177610667],[1217203200000,0.2179357848],[1217289600000,0.217858302],[1217376000000,0.2192776226],[1217462400000,0.2186436449],[1217548800000,0.2259308802],[1217808000000,0.2250692923],[1217894400000,0.2258502449],[1217980800000,0.2268553611],[1218067200000,0.2283204233],[1218153600000,0.2380963582],[1218412800000,0.2397285454],[1218499200000,0.2401021674],[1218585600000,0.2324197974],[1218672000000,0.2365284214],[1218758400000,0.240857397],[1219017600000,0.2508630144],[1219104000000,0.253148854],[1219190400000,0.2489073492],[1219276800000,0.2479986145],[1219363200000,0.239263977],[1219622400000,0.2420776607],[1219708800000,0.2414224157],[1219795200000,0.2361784417],[1219881600000,0.2188909681],[1219968000000,0.2219579592],[1220227200000,0.2071042489],[1220313600000,0.2003398412],[1220400000000,0.2016723135],[1220486400000,0.1916293076],[1220572800000,0.2009064854],[1220832000000,0.2102715069],[1220918400000,0.2128765802],[1221004800000,0.2154666083],[1221091200000,0.2206772337],[1221177600000,0.2250592814],[1221523200000,0.2324171437],[1221609600000,0.2388052997],[1221696000000,0.2461851835],[1221782400000,0.2459642566],[1222041600000,0.2457719697],[1222128000000,0.2492766235],[1222214400000,0.2498389192],[1222300800000,0.2503613785],[1222387200000,0.2555450383],[1223251200000,0.2534542365],[1223337600000,0.2588208366],[1223424000000,0.2584191823],[1223510400000,0.2584634053],[1223596800000,0.2634786714],[1223856000000,0.2624681453],[1223942400000,0.2636990315],[1224028800000,0.266534456],[1224115200000,0.2765064534],[1224201600000,0.2762531876],[1224460800000,0.2750719718],[1224547200000,0.2799696339],[1224633600000,0.289871464],[1224720000000,0.2908943235],[1224806400000,0.2968109057],[1225065600000,0.3101732421],[1225152000000,0.3155137343],[1225238400000,0.3116239701],[1225324800000,0.3072018953],[1225411200000,0.3136601226],[1225670400000,0.3178329373],[1225756800000,0.3222584249],[1225843200000,0.3206013272],[1225929600000,0.3211094867],[1226016000000,0.321545917],[1226275200000,0.315680933],[1226361600000,0.3123092727],[1226448000000,0.2944759375],[1226534400000,0.2998422388],[1226620800000,0.2899872241],[1226880000000,0.2862038903],[1226966400000,0.2818288255],[1227052800000,0.2819292861],[1227139200000,0.2817838548],[1227225600000,0.2782315786],[1227484800000,0.2787077786],[1227571200000,0.2797657219],[1227657600000,0.2783312595],[1227744000000,0.2683426782],[1227830400000,0.2644647353],[1228089600000,0.261305644],[1228176000000,0.2221612456],[1228262400000,0.2189659291],[1228348800000,0.2120116711],[1228435200000,0.2026472964],[1228694400000,0.2033497592],[1228780800000,0.2025390189],[1228867200000,0.2041489905],[1228953600000,0.2071527012],[1229040000000,0.2113250369],[1229299200000,0.2133378235],[1229385600000,0.2146684512],[1229472000000,0.2165113361],[1229558400000,0.2190388228],[1229644800000,0.2191897041],[1229904000000,0.223797871],[1229990400000,0.2211438072],[1230076800000,0.2233363705],[1230163200000,0.2227965962],[1230249600000,0.2246851279],[1230508800000,0.2251014471],[1230595200000,0.2295556379],[1230681600000,0.2296833301],[1231113600000,0.2252219929],[1231200000000,0.2245310485],[1231286400000,0.2248531357],[1231372800000,0.2240758697],[1231459200000,0.2275716473],[1231718400000,0.2316754758],[1231804800000,0.2310025401],[1231891200000,0.2317299735],[1231977600000,0.2355816227],[1232064000000,0.2343938602],[1232323200000,0.2343600768],[1232409600000,0.2348084934],[1232496000000,0.2362516326],[1232582400000,0.2264177354],[1232668800000,0.2243759943],[1233532800000,0.2164717933],[1233619200000,0.2175323511],[1233705600000,0.2045982309],[1233792000000,0.1972452416],[1233878400000,0.1966693514],[1234137600000,0.1950148669],[1234224000000,0.1711148478],[1234310400000,0.1643600312],[1234396800000,0.1680788049],[1234483200000,0.1659808711],[1234742400000,0.1640948479],[1234828800000,0.1618705293],[1234915200000,0.1588773728],[1235001600000,0.1620528098],[1235088000000,0.1617389284],[1235347200000,0.15232812],[1235433600000,0.1517831321],[1235520000000,0.1528017344],[1235606400000,0.1497933921],[1235692800000,0.1503146829],[1235952000000,0.1498305073],[1236038400000,0.1495097205],[1236124800000,0.1472093397],[1236211200000,0.1488557856],[1236297600000,0.1489998411],[1236556800000,0.1478226674],[1236643200000,0.1469620148],[1236729600000,0.1481970183],[1236816000000,0.1495844925],[1236902400000,0.1496475306],[1237161600000,0.1487894332],[1237248000000,0.1485314875],[1237334400000,0.1484118233],[1237420800000,0.1491356463],[1237507200000,0.1509919084],[1237766400000,0.1502738095],[1237852800000,0.1548681517],[1237939200000,0.1573221174],[1238025600000,0.1608007292],[1238112000000,0.1642960599],[1238371200000,0.1665016628],[1238457600000,0.1680931816],[1238544000000,0.1783546864],[1238630400000,0.1837463222],[1238716800000,0.1864062408],[1239062400000,0.1872919532],[1239148800000,0.1869503647],[1239235200000,0.1884427697],[1239321600000,0.1911074425],[1239580800000,0.2015303424],[1239667200000,0.20850301],[1239753600000,0.2106080459],[1239840000000,0.2133188617],[1239926400000,0.2124163348],[1240185600000,0.216153626],[1240272000000,0.2148305085],[1240358400000,0.2128832711],[1240444800000,0.2175864827],[1240531200000,0.2189310006],[1240790400000,0.2183996845],[1240876800000,0.2199207812],[1240963200000,0.2175794685],[1241049600000,0.2172538038],[1241395200000,0.2113686184],[1241481600000,0.2160782449],[1241568000000,0.2182287171],[1241654400000,0.1989043031],[1241740800000,0.2076328015],[1242000000000,0.2016073047],[1242086400000,0.1966432004],[1242172800000,0.2035883603],[1242259200000,0.2056122986],[1242345600000,0.2073259555],[1242604800000,0.2132252383],[1242691200000,0.2228043412],[1242777600000,0.2224896574],[1242864000000,0.2200402906],[1242950400000,0.2235898243],[1243209600000,0.2231656978],[1243296000000,0.2220142326],[1243382400000,0.2213906754],[1243814400000,0.222545862],[1243900800000,0.2253176321],[1243987200000,0.2377265637],[1244073600000,0.2443790012],[1244160000000,0.2452134985],[1244419200000,0.2463343554],[1244505600000,0.2516182441],[1244592000000,0.2594469536],[1244678400000,0.2610408935],[1244764800000,0.2607336506],[1245024000000,0.2622542492],[1245110400000,0.2628236038],[1245196800000,0.2657483648],[1245283200000,0.2737518742],[1245369600000,0.280041176],[1245628800000,0.2808864254],[1245715200000,0.2832737857],[1245801600000,0.2955984973],[1245888000000,0.2975608419],[1245974400000,0.2997139375],[1246233600000,0.3084770326],[1246320000000,0.3080473338],[1246406400000,0.3175488337],[1246492800000,0.3124086326],[1246579200000,0.3238293978],[1246838400000,0.3353706161],[1246924800000,0.3336799517],[1247011200000,0.3360301372],[1247097600000,0.3398215172],[1247184000000,0.3402951973],[1247443200000,0.3232252661],[1247529600000,0.3347625116],[1247616000000,0.3326443309],[1247702400000,0.3314875056],[1247788800000,0.3338362083],[1248048000000,0.3452204192],[1248134400000,0.3392396907],[1248220800000,0.3394097241],[1248307200000,0.3414237854],[1248393600000,0.3342906043],[1248652800000,0.3420557398],[1248739200000,0.3480212088],[1248825600000,0.3348986387],[1248912000000,0.3355944105],[1248998400000,0.3328004121],[1249257600000,0.3376769864],[1249344000000,0.3379521398],[1249430400000,0.3356874034],[1249516800000,0.3358859913],[1249603200000,0.3328414448],[1249862400000,0.3373704797],[1249948800000,0.3442555772],[1250035200000,0.3347095426],[1250121600000,0.3348790478],[1250208000000,0.3328347167],[1250467200000,0.3265272936],[1250553600000,0.3246396814],[1250640000000,0.3218878415],[1250726400000,0.3166043677],[1250812800000,0.3144827481],[1251072000000,0.3027384985],[1251158400000,0.3040881577],[1251244800000,0.3016113221],[1251331200000,0.3017405545],[1251417600000,0.2997363868],[1251676800000,0.2899413925],[1251763200000,0.2903123371],[1251849600000,0.2880624933],[1251936000000,0.2739494368],[1252022400000,0.2601358732],[1252281600000,0.2608037615],[1252368000000,0.2498100493],[1252454400000,0.2509670826],[1252540800000,0.2458505948],[1252627200000,0.2451694176],[1252886400000,0.2433234005],[1252972800000,0.2312967283],[1253059200000,0.232698594],[1253145600000,0.230942983],[1253232000000,0.2267003218],[1253491200000,0.2272672765],[1253577600000,0.2258097381],[1253664000000,0.2241538607],[1253750400000,0.2235805232],[1253836800000,0.2229724661],[1254096000000,0.2176198734],[1254182400000,0.2190408386],[1254268800000,0.2207624967],[1255046400000,0.2170783874],[1255305600000,0.2173847745],[1255392000000,0.2160291037],[1255478400000,0.2149804807],[1255564800000,0.2158614273],[1255651200000,0.2178890216],[1255910400000,0.2158500874],[1255996800000,0.2177225403],[1256083200000,0.2180576242],[1256169600000,0.2180094787],[1256256000000,0.2168776458],[1256515200000,0.2021773684],[1256601600000,0.1987417518],[1256688000000,0.1832348319],[1256774400000,0.1825856969],[1256860800000,0.1819441793],[1257120000000,0.1796635905],[1257206400000,0.1784460893],[1257292800000,0.1785559532],[1257379200000,0.1801676638],[1257465600000,0.1815325843],[1257724800000,0.1819246405],[1257811200000,0.1823414441],[1257897600000,0.182198565],[1257984000000,0.1825141939],[1258070400000,0.1828512786],[1258329600000,0.1803832555],[1258416000000,0.182240565],[1258502400000,0.1824726916],[1258588800000,0.1824673881],[1258675200000,0.1762861707],[1258934400000,0.1643318929],[1259020800000,0.1638029625],[1259107200000,0.161732066],[1259193600000,0.1598521707],[1259280000000,0.157512619],[1259539200000,0.1545929002],[1259625600000,0.1536004007],[1259712000000,0.1530005709],[1259798400000,0.1535550409],[1259884800000,0.1529499407],[1260144000000,0.1536870038],[1260230400000,0.1540604384],[1260316800000,0.152475115],[1260403200000,0.1531244749],[1260489600000,0.1542258323],[1260748800000,0.1538498647],[1260835200000,0.1531884643],[1260921600000,0.1526682964],[1261008000000,0.1516101327],[1261094400000,0.1495811615],[1261353600000,0.1497127048],[1261440000000,0.1487923336],[1261526400000,0.1483898685],[1261612800000,0.147681878],[1261699200000,0.1483786612],[1261958400000,0.1481748319],[1262044800000,0.1487578403],[1262131200000,0.1481981508],[1262217600000,0.1480988669],[1262563200000,0.1482102722],[1262649600000,0.1476201277],[1262736000000,0.147973347],[1262822400000,0.1485210531],[1262908800000,0.1492266576],[1263168000000,0.1493585805],[1263254400000,0.1485663939],[1263340800000,0.1475814177],[1263427200000,0.1476449583],[1263513600000,0.1488716063],[1263772800000,0.1495021128],[1263859200000,0.1497119786],[1263945600000,0.1488647267],[1264032000000,0.1488259633],[1264118400000,0.1524962827],[1264377600000,0.1534153336],[1264464000000,0.1537841599],[1264550400000,0.1539177618],[1264636800000,0.1463768095],[1264723200000,0.1357287213],[1264982400000,0.136298163],[1265068800000,0.1385830832],[1265155200000,0.1369100351],[1265241600000,0.1369185968],[1265328000000,0.1390275104],[1265587200000,0.139994631],[1265673600000,0.1418613928],[1265760000000,0.1457855095],[1265846400000,0.1464371432],[1265932800000,0.1497487936],[1266796800000,0.1528270433],[1266883200000,0.1534910258],[1266969600000,0.1529236974],[1267056000000,0.1549720571],[1267142400000,0.1564945636],[1267401600000,0.1557217666],[1267488000000,0.1585550045],[1267574400000,0.1619051805],[1267660800000,0.1505216012],[1267747200000,0.1357646248],[1268006400000,0.1394379562],[1268092800000,0.1395818037],[1268179200000,0.139615685],[1268265600000,0.1414236124],[1268352000000,0.1408638696],[1268611200000,0.1403374595],[1268697600000,0.1419052637],[1268784000000,0.1414769836],[1268870400000,0.1415714443],[1268956800000,0.1421180142],[1269216000000,0.1436631483],[1269302400000,0.1465004082],[1269388800000,0.1467396285],[1269475200000,0.147661632],[1269561600000,0.1484539194],[1269820800000,0.1464864514],[1269907200000,0.1469193943],[1269993600000,0.1489466346],[1270080000000,0.1406202385],[1270166400000,0.1110552842],[1270512000000,0.1148907661],[1270598400000,0.1147480358],[1270684800000,0.1148891746],[1270771200000,0.1147863012],[1271030400000,0.1143930949],[1271116800000,0.1134451464],[1271203200000,0.1153211208],[1271289600000,0.1163216571],[1271376000000,0.115568021],[1271635200000,0.1116667076],[1271721600000,0.1131715974],[1271808000000,0.1116660633],[1271894400000,0.1132391676],[1271980800000,0.1133031086],[1272240000000,0.1149155283],[1272326400000,0.1223110131],[1272412800000,0.1275716285],[1272499200000,0.13601491],[1272585600000,0.1363619035],[1272931200000,0.1443983091],[1273017600000,0.1444492195],[1273104000000,0.1668785245],[1273190400000,0.177994044],[1273449600000,0.1774810312],[1273536000000,0.1830359516],[1273622400000,0.1830860756],[1273708800000,0.1846025884],[1273795200000,0.1839837413],[1274054400000,0.195984629],[1274140800000,0.1942560964],[1274227200000,0.1944095909],[1274313600000,0.1943400569],[1274400000000,0.1973450163],[1274659200000,0.1964055275],[1274745600000,0.1998945354],[1274832000000,0.2042859957],[1274918400000,0.2079976177],[1275004800000,0.2098251651],[1275264000000,0.2079718208],[1275350400000,0.206957555],[1275436800000,0.2087414431],[1275523200000,0.1991125639],[1275609600000,0.1982407882],[1275868800000,0.2032597679],[1275955200000,0.204135499],[1276041600000,0.2005504371],[1276128000000,0.1943608683],[1276214400000,0.1934823545],[1276732800000,0.1937640219],[1276819200000,0.195234469],[1277078400000,0.1954522353],[1277164800000,0.1955022203],[1277251200000,0.1983976533],[1277337600000,0.1997594593],[1277424000000,0.203436877],[1277683200000,0.2032043943],[1277769600000,0.2235837879],[1277856000000,0.2299136202],[1277942400000,0.239323406],[1278028800000,0.2372229705],[1278288000000,0.241313013],[1278374400000,0.2351413082],[1278460800000,0.2337810448],[1278547200000,0.2372999819],[1278633600000,0.2338515292],[1278892800000,0.2324282527],[1278979200000,0.226753418],[1279065600000,0.2318000855],[1279152000000,0.23200114],[1279238400000,0.2323204392],[1279497600000,0.2297505288],[1279584000000,0.2045013355],[1279670400000,0.2096169446],[1279756800000,0.2081666434],[1279843200000,0.2096529173],[1280102400000,0.2106782137],[1280188800000,0.2141918808],[1280275200000,0.2129794819],[1280361600000,0.212722897],[1280448000000,0.2123858807],[1280707200000,0.2125697177],[1280793600000,0.2103466008],[1280880000000,0.214065582],[1280966400000,0.2133445474],[1281052800000,0.214317661],[1281312000000,0.2133566632],[1281398400000,0.2099405861],[1281484800000,0.2112816633],[1281571200000,0.2099113106],[1281657600000,0.2095202278],[1281916800000,0.2071385698],[1282003200000,0.2078452028],[1282089600000,0.2098440179],[1282176000000,0.2112819129],[1282262400000,0.2089534416],[1282521600000,0.2109731352],[1282608000000,0.2108650722],[1282694400000,0.2086420309],[1282780800000,0.2125105078],[1282867200000,0.2125720423],[1283126400000,0.2110384645],[1283212800000,0.2114482909],[1283299200000,0.2117842922],[1283385600000,0.210074253],[1283472000000,0.2121844851],[1283731200000,0.2119665864],[1283817600000,0.2125896807],[1283904000000,0.216082056],[1283990400000,0.2135765125],[1284076800000,0.21457018],[1284336000000,0.2135135284],[1284422400000,0.2147349719],[1284508800000,0.2121538255],[1284595200000,0.2117785238],[1284681600000,0.2139168605],[1284940800000,0.2176841021],[1285027200000,0.2177232198],[1285545600000,0.216343725],[1285632000000,0.2174983526],[1285718400000,0.2175418734],[1285804800000,0.2141723419],[1286496000000,0.2095922195],[1286755200000,0.2072825874],[1286841600000,0.2069516803],[1286928000000,0.2061396021],[1287014400000,0.2077581689],[1287100800000,0.2012867378],[1287360000000,0.1921764576],[1287446400000,0.1949598507],[1287532800000,0.2071863169],[1287619200000,0.2062924763],[1287705600000,0.2091865622],[1287964800000,0.2254924708],[1288051200000,0.2253353056],[1288137600000,0.2230178201],[1288224000000,0.2260170449],[1288310400000,0.2256719511],[1288569600000,0.2227641723],[1288656000000,0.2226214351],[1288742400000,0.2229091972],[1288828800000,0.2207212425],[1288915200000,0.2350698682],[1289174400000,0.2434218151],[1289260800000,0.2432503735],[1289347200000,0.245123908],[1289433600000,0.245505285],[1289520000000,0.2370733039],[1289779200000,0.2368185721],[1289865600000,0.2372630001],[1289952000000,0.2367151658],[1290038400000,0.234756392],[1290124800000,0.2350423393],[1290384000000,0.2369659873],[1290470400000,0.2391506642],[1290556800000,0.2385560494],[1290643200000,0.2360658309],[1290729600000,0.2369944075],[1290988800000,0.2372336458],[1291075200000,0.2385432992],[1291161600000,0.2400900215],[1291248000000,0.23976596],[1291334400000,0.2407825979],[1291593600000,0.2408180457],[1291680000000,0.2415812127],[1291766400000,0.2401779006],[1291852800000,0.2420851935],[1291939200000,0.2417084357],[1292198400000,0.2367343243],[1292284800000,0.2371466584],[1292371200000,0.2384885767],[1292457600000,0.2421938556],[1292544000000,0.2421213656],[1292803200000,0.2409076357],[1292889600000,0.237040632],[1292976000000,0.2364304808],[1293062400000,0.2360069673],[1293148800000,0.241501203],[1293408000000,0.239614598],[1293494400000,0.2385818288],[1293580800000,0.2380410274],[1293667200000,0.2276588829],[1293753600000,0.2238552934],[1294099200000,0.2216512318],[1294185600000,0.2136356517],[1294272000000,0.2163703804],[1294358400000,0.2174752797],[1294617600000,0.2166211061],[1294704000000,0.2167235184],[1294790400000,0.2162887709],[1294876800000,0.2013231431],[1294963200000,0.1882911363],[1295222400000,0.1844967797],[1295308800000,0.1767692739],[1295395200000,0.1726532802],[1295481600000,0.1686326747],[1295568000000,0.1678511156],[1295827200000,0.162654011],[1295913600000,0.1645736912],[1296000000000,0.1636092264],[1296086400000,0.1621980847],[1296172800000,0.1635797411],[1296432000000,0.163993285],[1296518400000,0.1643569935],[1297209600000,0.1639607948],[1297296000000,0.1633690075],[1297382400000,0.1635057087],[1297641600000,0.1630520517],[1297728000000,0.1636566594],[1297814400000,0.1579336624],[1297900800000,0.1592207616],[1297987200000,0.1603814073],[1298246400000,0.1594674887],[1298332800000,0.1563208576],[1298419200000,0.1565682612],[1298505600000,0.1580014982],[1298592000000,0.1578260513],[1298851200000,0.1568969344],[1298937600000,0.1572709039],[1299024000000,0.1571109607],[1299110400000,0.1566495461],[1299196800000,0.1569130302],[1299456000000,0.1551011529],[1299542400000,0.1556609028],[1299628800000,0.1569176112],[1299715200000,0.1549199283],[1299801600000,0.1556792838],[1300060800000,0.1554188863],[1300147200000,0.1534791672],[1300233600000,0.1538261712],[1300320000000,0.1523168847],[1300406400000,0.1526890038],[1300665600000,0.1524927326],[1300752000000,0.153699284],[1300838400000,0.1541546798],[1300924800000,0.1538156739],[1301011200000,0.15108137],[1301270400000,0.147413331],[1301356800000,0.1478974183],[1301443200000,0.1486223273],[1301529600000,0.1344280558],[1301616000000,0.134085483],[1302048000000,0.1361908888],[1302134400000,0.1385205878],[1302220800000,0.1388640343],[1302480000000,0.1396564002],[1302566400000,0.1396532025],[1302652800000,0.1415419889],[1302739200000,0.1416422437],[1302825600000,0.1437836079],[1303084800000,0.144489034],[1303171200000,0.1430843652],[1303257600000,0.1432160686],[1303344000000,0.1459861702],[1303430400000,0.1459013099],[1303689600000,0.1463157188],[1303776000000,0.1458807209],[1303862400000,0.145749994],[1303948800000,0.1473450258],[1304035200000,0.1465988218],[1304380800000,0.1474629421],[1304467200000,0.1466848484],[1304553600000,0.1480158422],[1304640000000,0.1428400596],[1304899200000,0.13998623],[1304985600000,0.1400335788],[1305072000000,0.1062930134],[1305158400000,0.111050467],[1305244800000,0.1109529491],[1305504000000,0.1144008067],[1305590400000,0.1159328019],[1305676800000,0.1165783457],[1305763200000,0.116960611],[1305849600000,0.1171487268],[1306108800000,0.1160862088],[1306195200000,0.1181954051],[1306281600000,0.1185367935],[1306368000000,0.1190797409],[1306454400000,0.1187446571],[1306713600000,0.120202362],[1306800000000,0.1186854717],[1306886400000,0.1192219132],[1306972800000,0.1177525452],[1307059200000,0.1170139808],[1307404800000,0.1175490941],[1307491200000,0.1183215798],[1307577600000,0.1180556691],[1307664000000,0.1189816632],[1307923200000,0.1219102593],[1308009600000,0.1207284154],[1308096000000,0.1204395385],[1308182400000,0.1253001883],[1308268800000,0.1315685576],[1308528000000,0.1374213558],[1308614400000,0.1388290029],[1308700800000,0.1401428124],[1308787200000,0.1392708263],[1308873600000,0.1375677628],[1309132800000,0.1396636587],[1309219200000,0.1416348997],[1309305600000,0.1406467493],[1309392000000,0.1390087382],[1309478400000,0.1413204463],[1309737600000,0.1408875009],[1309824000000,0.1414285958],[1309910400000,0.1417200948],[1309996800000,0.1415163626],[1310083200000,0.1435982611],[1310342400000,0.1441269841],[1310428800000,0.1425204909],[1310515200000,0.1405747637],[1310601600000,0.1421797288],[1310688000000,0.1465543672],[1310947200000,0.1464252999],[1311033600000,0.1481586343],[1311120000000,0.1525500184],[1311206400000,0.1528464433],[1311292800000,0.1538102163],[1311552000000,0.1499086309],[1311638400000,0.1512909627],[1311724800000,0.152486381],[1311811200000,0.15207682],[1311897600000,0.1535256895],[1312156800000,0.1532951378],[1312243200000,0.1540070075],[1312329600000,0.1570080884],[1312416000000,0.1575734332],[1312502400000,0.1593559472],[1312761600000,0.1794504093],[1312848000000,0.1809426935],[1312934400000,0.1796321786],[1313020800000,0.1791440205],[1313107200000,0.1812552711],[1313366400000,0.184248099],[1313452800000,0.1837129638],[1313539200000,0.1839836552],[1313625600000,0.1813815152],[1313712000000,0.1822592544],[1313971200000,0.1864832296],[1314057600000,0.1846509518],[1314144000000,0.1852611626],[1314230400000,0.1827053452],[1314316800000,0.1862103327],[1314576000000,0.1835949689],[1314662400000,0.1830480908],[1314748800000,0.1861187672],[1314835200000,0.1873291784],[1314921600000,0.1864393485],[1315180800000,0.1970310942],[1315267200000,0.2049920055],[1315353600000,0.2046886586],[1315440000000,0.2044048838],[1315526400000,0.204635068],[1315872000000,0.2046220888],[1315958400000,0.2065114068],[1316044800000,0.2071355701],[1316131200000,0.2096798582],[1316390400000,0.2192889159],[1316476800000,0.2208281534],[1316563200000,0.2153856677],[1316649600000,0.2119245006],[1316736000000,0.217126592],[1316995200000,0.2337640399],[1317081600000,0.2327909466],[1317168000000,0.2330094711],[1317254400000,0.2396829668],[1317340800000,0.2417611986],[1318204800000,0.2469172275],[1318291200000,0.24951099],[1318377600000,0.2429502455],[1318464000000,0.2290966997],[1318550400000,0.232917941],[1318809600000,0.2319916828],[1318896000000,0.223115473],[1318982400000,0.2086064593],[1319068800000,0.2116781579],[1319155200000,0.2099115879],[1319414400000,0.206987388],[1319500800000,0.2069240479],[1319587200000,0.207211826],[1319673600000,0.1908561525],[1319760000000,0.1925968371],[1320019200000,0.1919780576],[1320105600000,0.1921209397],[1320192000000,0.1900557498],[1320278400000,0.1888570768],[1320364800000,0.1865423642],[1320624000000,0.1874270993],[1320710400000,0.1884309653],[1320796800000,0.1886386651],[1320883200000,0.1866046523],[1320969600000,0.1845278422],[1321228800000,0.1825501343],[1321315200000,0.1823057638],[1321401600000,0.1836025734],[1321488000000,0.1833391395],[1321574400000,0.1822822188],[1321833600000,0.182752664],[1321920000000,0.1835553598],[1322006400000,0.1826553722],[1322092800000,0.1849477032],[1322179200000,0.1840520435],[1322438400000,0.1865460695],[1322524800000,0.186288097],[1322611200000,0.1825029241],[1322697600000,0.1794598375],[1322784000000,0.1810403384],[1323043200000,0.1796753166],[1323129600000,0.1800378624],[1323216000000,0.1816875901],[1323302400000,0.1831455704],[1323388800000,0.1857639301],[1323648000000,0.1933979775],[1323734400000,0.2027692108],[1323820800000,0.2144322845],[1323907200000,0.2254420804],[1323993600000,0.2262591188],[1324252800000,0.2305731992],[1324339200000,0.2306884446],[1324425600000,0.2327043655],[1324512000000,0.2354333478],[1324598400000,0.237278766],[1324857600000,0.2370356425],[1324944000000,0.2492972194],[1325030400000,0.2491178098],[1325116800000,0.2495222231],[1325203200000,0.2478445293],[1325635200000,0.2468017802],[1325721600000,0.252075744],[1325808000000,0.2552525129],[1326067200000,0.2530830112],[1326153600000,0.2479826863],[1326240000000,0.2462532811],[1326326400000,0.2386661769],[1326412800000,0.2367060122],[1326672000000,0.2268857947],[1326758400000,0.2216435671],[1326844800000,0.2009574958],[1326931200000,0.2039903933],[1327017600000,0.2024914796],[1327881600000,0.1960043061],[1327968000000,0.1959372625],[1328054400000,0.1945053817],[1328140800000,0.1870003172],[1328227200000,0.1870765326],[1328486400000,0.1870632287],[1328572800000,0.1735154259],[1328659200000,0.1731688092],[1328745600000,0.1780981651],[1328832000000,0.1780932207],[1329091200000,0.1793023451],[1329177600000,0.1809680177],[1329264000000,0.1800001684],[1329350400000,0.1775039959],[1329436800000,0.1784774588],[1329696000000,0.178881977],[1329782400000,0.180426156],[1329868800000,0.1799893289],[1329955200000,0.1810848641],[1330041600000,0.1811606756],[1330300800000,0.1812878676],[1330387200000,0.1852369009],[1330473600000,0.1695931553],[1330560000000,0.1702031791],[1330646400000,0.1684180999],[1330905600000,0.1642100184],[1330992000000,0.1538528851],[1331078400000,0.1471729157],[1331164800000,0.1483560427],[1331251200000,0.1480070426],[1331510400000,0.1476226433],[1331596800000,0.1474606842],[1331683200000,0.1444061661],[1331769600000,0.1448820281],[1331856000000,0.1438229299],[1332115200000,0.1437213789],[1332201600000,0.1420448345],[1332288000000,0.1441721368],[1332374400000,0.1444470728],[1332460800000,0.1444513274],[1332720000000,0.148086477],[1332806400000,0.1484432586],[1332892800000,0.1478273002],[1332979200000,0.14761149],[1333065600000,0.1483081923],[1333584000000,0.1467218198],[1333670400000,0.1467147528],[1333929600000,0.1466966465],[1334016000000,0.1458384454],[1334102400000,0.1499910749],[1334188800000,0.1485029648],[1334275200000,0.1484491366],[1334534400000,0.1487554369],[1334620800000,0.150714204],[1334707200000,0.1484690349],[1334793600000,0.1511723488],[1334880000000,0.1503269587],[1335139200000,0.1525856212],[1335225600000,0.154893498],[1335312000000,0.1551449709],[1335398400000,0.1551612708],[1335484800000,0.157478076],[1335916800000,0.1552893194],[1336003200000,0.1550023694],[1336089600000,0.1560196049],[1336348800000,0.1560201044],[1336435200000,0.1526738646],[1336521600000,0.1515004686],[1336608000000,0.1518994624],[1336694400000,0.1516128929],[1336953600000,0.1531155678],[1337040000000,0.1532517604],[1337126400000,0.1520802278],[1337212800000,0.1417892708],[1337299200000,0.1433829262],[1337558400000,0.1431434548],[1337644800000,0.1438539657],[1337731200000,0.1435050621],[1337817600000,0.1425135123],[1337904000000,0.1426659814],[1338163200000,0.140994394],[1338249600000,0.140223524],[1338336000000,0.1399904853],[1338422400000,0.1411106884],[1338508800000,0.1451043434],[1338768000000,0.144539846],[1338854400000,0.1457983748],[1338940800000,0.1474869183],[1339027200000,0.1469663774],[1339113600000,0.1466753065],[1339372800000,0.1451940789],[1339459200000,0.145359521],[1339545600000,0.1446596946],[1339632000000,0.1453729259],[1339718400000,0.1469849748],[1339977600000,0.148713802],[1340064000000,0.1494206181],[1340150400000,0.1494059506],[1340236800000,0.1477515415],[1340582400000,0.1468807923],[1340668800000,0.1469111025],[1340755200000,0.1474071254],[1340841600000,0.1475051548],[1340928000000,0.1472487807],[1341187200000,0.14721293],[1341273600000,0.1472104751],[1341360000000,0.1487273315],[1341446400000,0.1493533739],[1341532800000,0.1435792983],[1341792000000,0.1233206742],[1341878400000,0.126191106],[1341964800000,0.1288083849],[1342051200000,0.1282813206],[1342137600000,0.128234463],[1342396800000,0.127793651],[1342483200000,0.1105203718],[1342569600000,0.1151133036],[1342656000000,0.1163096862],[1342742400000,0.1176474923],[1343001600000,0.1299465241],[1343088000000,0.1315381772],[1343174400000,0.1329215997],[1343260800000,0.1387695905],[1343347200000,0.1417242369],[1343606400000,0.1465685936],[1343692800000,0.147607505],[1343779200000,0.1487853155],[1343865600000,0.1515101411],[1343952000000,0.1504517148],[1344211200000,0.1488526442],[1344297600000,0.1487594361],[1344384000000,0.1492758458],[1344470400000,0.1496002084],[1344556800000,0.1496927266],[1344816000000,0.1470165788],[1344902400000,0.1469340694],[1344988800000,0.1472279198],[1345075200000,0.1531280651],[1345161600000,0.155664046],[1345420800000,0.1620409864],[1345507200000,0.1618291805],[1345593600000,0.1634640328],[1345680000000,0.1648497107],[1345766400000,0.1709074602],[1346025600000,0.1891972315],[1346112000000,0.1897118943],[1346198400000,0.1962733162],[1346284800000,0.1987488177],[1346371200000,0.203398501],[1346630400000,0.2038034084],[1346716800000,0.2028529092],[1346803200000,0.2064489897],[1346889600000,0.2113368617],[1346976000000,0.2046720071],[1347235200000,0.2070018506],[1347321600000,0.2063182217],[1347408000000,0.2094369991],[1347494400000,0.207900959],[1347580800000,0.2068183361],[1347840000000,0.2046914215],[1347926400000,0.2030088745],[1348012800000,0.2027942455],[1348099200000,0.2061045788],[1348185600000,0.2084651646],[1348444800000,0.208076144],[1348531200000,0.2125303423],[1348617600000,0.2154326927],[1348704000000,0.2118232084],[1348790400000,0.2104221948],[1349654400000,0.2085438443],[1349740800000,0.2085675147],[1349827200000,0.2090625552],[1349913600000,0.2078248763],[1350000000000,0.2102984645],[1350259200000,0.214390029],[1350345600000,0.2144374784],[1350432000000,0.2168935089],[1350518400000,0.215622724],[1350604800000,0.2154100508],[1350864000000,0.2164600461],[1350950400000,0.2143693304],[1351036800000,0.2144581901],[1351123200000,0.2180374218],[1351209600000,0.2149330666],[1351468800000,0.2160046696],[1351555200000,0.2123110694],[1351641600000,0.1909236338],[1351728000000,0.1914637459],[1351814400000,0.1882301614],[1352073600000,0.1894084745],[1352160000000,0.1903151497],[1352246400000,0.190092321],[1352332800000,0.1901411899],[1352419200000,0.1928802053],[1352678400000,0.1952261843],[1352764800000,0.1931096219],[1352851200000,0.1955096064],[1352937600000,0.1939762296],[1353024000000,0.1975910752],[1353283200000,0.200192679],[1353369600000,0.2071898462],[1353456000000,0.2040617426],[1353542400000,0.1993569406],[1353628800000,0.1989409535],[1353888000000,0.1757274669],[1353974400000,0.1855769189],[1354060800000,0.1930633024],[1354147200000,0.1978032717],[1354233600000,0.1970697552],[1354492800000,0.1988699994],[1354579200000,0.1998214822],[1354665600000,0.1950095779],[1354752000000,0.1979665222],[1354838400000,0.1960813616],[1355097600000,0.1947346289],[1355184000000,0.1854683409],[1355270400000,0.1840006966],[1355356800000,0.1659254993],[1355443200000,0.1452918739],[1355702400000,0.1479408822],[1355788800000,0.1479517131],[1355875200000,0.1482368954],[1355961600000,0.1487071521],[1356048000000,0.1501223644],[1356307200000,0.1497768335],[1356393600000,0.1459488521],[1356480000000,0.1456321563],[1356566400000,0.146897766],[1356652800000,0.1503138287],[1356912000000,0.1686191634],[1357257600000,0.1697576758],[1357516800000,0.1750086042],[1357603200000,0.1759421352],[1357689600000,0.1759892546],[1357776000000,0.1794238476],[1357862400000,0.1769802237],[1358121600000,0.1869641846],[1358208000000,0.1935651828],[1358294400000,0.1941121842],[1358380800000,0.1947876378],[1358467200000,0.1923169571],[1358726400000,0.1982913882],[1358812800000,0.1981810215],[1358899200000,0.1974840986],[1358985600000,0.1966032534],[1359072000000,0.195973191],[1359331200000,0.2075496218],[1359417600000,0.2167068598],[1359504000000,0.2221251786],[1359590400000,0.2247110023],[1359676800000,0.2409090046],[1359936000000,0.2423486411],[1360022400000,0.2511499784],[1360108800000,0.2534743995],[1360195200000,0.2565800103],[1360281600000,0.2559135329],[1361145600000,0.2550474923],[1361232000000,0.2512127695],[1361318400000,0.2501913036],[1361404800000,0.2428668181],[1361491200000,0.2426865476],[1361750400000,0.2435505603],[1361836800000,0.2408453908],[1361923200000,0.2407958295],[1362009600000,0.2381163182],[1362096000000,0.238603558],[1362355200000,0.2304009451],[1362441600000,0.2246846956],[1362528000000,0.2231146197],[1362614400000,0.2225770605],[1362700800000,0.2234277751],[1362960000000,0.222679607],[1363046400000,0.2212892031],[1363132800000,0.2266460972],[1363219200000,0.2268550011],[1363305600000,0.2275655242],[1363564800000,0.2253207575],[1363651200000,0.225244666],[1363737600000,0.2201890287],[1363824000000,0.2240356581],[1363910400000,0.2255623079],[1364169600000,0.2260048387],[1364256000000,0.2269460833],[1364342400000,0.2265321718],[1364428800000,0.221435401],[1364515200000,0.2215199753],[1364774400000,0.2232527781],[1364860800000,0.2278305899],[1364947200000,0.2308706759],[1365379200000,0.231818324],[1365465600000,0.2345063515],[1365552000000,0.2344931602],[1365638400000,0.2356663593],[1365724800000,0.2345253165],[1365984000000,0.2332442772],[1366070400000,0.2316669503],[1366156800000,0.2317893222],[1366243200000,0.2341408447],[1366329600000,0.2288884161],[1366588800000,0.2293575142],[1366675200000,0.2253519699],[1366761600000,0.2222129678],[1366848000000,0.2213898312],[1366934400000,0.2230750707],[1367452800000,0.2238033722],[1367539200000,0.2208949193],[1367798400000,0.2195765106],[1367884800000,0.2224397369],[1367971200000,0.2221456191],[1368057600000,0.2213993846],[1368144000000,0.2211512638],[1368403200000,0.2207589323],[1368489600000,0.2210999473],[1368576000000,0.2204349938],[1368662400000,0.2179250091],[1368748800000,0.2179299934],[1369008000000,0.2174404882],[1369094400000,0.2191731757],[1369180800000,0.2201279864],[1369267200000,0.2177613665],[1369353600000,0.2174582112],[1369612800000,0.2194370877],[1369699200000,0.217475228],[1369785600000,0.2184245687],[1369872000000,0.2190587231],[1369958400000,0.2188481263],[1370217600000,0.220124354],[1370304000000,0.2184324372],[1370390400000,0.2197892364],[1370476800000,0.2122032425],[1370563200000,0.1871514866],[1371081600000,0.1866746699],[1371168000000,0.1732299325],[1371427200000,0.1749481586],[1371513600000,0.1754882536],[1371600000000,0.1751746161],[1371686400000,0.1712355163],[1371772800000,0.1482318554],[1372032000000,0.1934351756],[1372118400000,0.1956035505],[1372204800000,0.1955120814],[1372291200000,0.1967230941],[1372377600000,0.1950889969],[1372636800000,0.1950976598],[1372723200000,0.1951323194],[1372809600000,0.1982153848],[1372896000000,0.1976466203],[1372982400000,0.1981662135],[1373241600000,0.1964090135],[1373328000000,0.1990756623],[1373414400000,0.1952878201],[1373500800000,0.1898011578],[1373587200000,0.187449336],[1373846400000,0.1856877632],[1373932800000,0.1853458926],[1374019200000,0.1860370864],[1374105600000,0.1892999809],[1374192000000,0.1871846527],[1374451200000,0.1875876792],[1374537600000,0.1853648834],[1374624000000,0.1868281734],[1374710400000,0.1870548666],[1374796800000,0.1870736403],[1375056000000,0.1849652232],[1375142400000,0.1855953171],[1375228800000,0.1860134573],[1375315200000,0.1875934002],[1375401600000,0.1888669518],[1375660800000,0.1878156469],[1375747200000,0.1870457655],[1375833600000,0.1895484857],[1375920000000,0.1895993169],[1376006400000,0.1904458185],[1376265600000,0.1855753081],[1376352000000,0.186123093],[1376438400000,0.1757867626],[1376524800000,0.1655116383],[1376611200000,0.1672701232],[1376870400000,0.1577233971],[1376956800000,0.1613541929],[1377043200000,0.161867289],[1377129600000,0.1620566356],[1377216000000,0.1631070494],[1377475200000,0.1619936415],[1377561600000,0.1643873731],[1377648000000,0.157931079],[1377734400000,0.1639276181],[1377820800000,0.1680180149],[1378080000000,0.167216425],[1378166400000,0.1670177717],[1378252800000,0.1675239966],[1378339200000,0.1678495941],[1378425600000,0.1690653895],[1378684800000,0.165892579],[1378771200000,0.1643423181],[1378857600000,0.1642083961],[1378944000000,0.1649257422],[1379030400000,0.1651538435],[1379289600000,0.1694688374],[1379376000000,0.1667660923],[1379462400000,0.1666586258],[1379894400000,0.1646965874],[1379980800000,0.1652396149],[1380067200000,0.1648801975],[1380153600000,0.1671389864],[1380240000000,0.1667752937],[1380499200000,0.1660777264],[1381190400000,0.1646096978],[1381276800000,0.1641108822],[1381363200000,0.1633895511],[1381449600000,0.1621808329],[1381708800000,0.1621857278],[1381795200000,0.1623169144],[1381881600000,0.1606785698],[1381968000000,0.161604753],[1382054400000,0.1621492946],[1382313600000,0.159784056],[1382400000000,0.1587846711],[1382486400000,0.1609823646],[1382572800000,0.1601921166],[1382659200000,0.1628487246],[1382918400000,0.1652695429],[1383004800000,0.1664985454],[1383091200000,0.1656516527],[1383177600000,0.1638878723],[1383264000000,0.1656862712],[1383523200000,0.1673234544],[1383609600000,0.1673605127],[1383696000000,0.1663622322],[1383782400000,0.1664761931],[1383868800000,0.1653633501],[1384128000000,0.1654838732],[1384214400000,0.1662415697],[1384300800000,0.164081929],[1384387200000,0.1657351021],[1384473600000,0.1653446932],[1384732800000,0.1619879887],[1384819200000,0.1613664194],[1384905600000,0.1608560006],[1384992000000,0.1619505596],[1385078400000,0.1620781066],[1385337600000,0.1617003919],[1385424000000,0.1634795472],[1385510400000,0.1593182934],[1385596800000,0.1490926121],[1385683200000,0.1491804662],[1385942400000,0.1359560031],[1386028800000,0.1348202453],[1386115200000,0.1226667914],[1386201600000,0.1168627187],[1386288000000,0.1179668814],[1386547200000,0.1213865345],[1386633600000,0.1220007389],[1386720000000,0.1208260442],[1386806400000,0.12134023],[1386892800000,0.1219642606],[1387152000000,0.1237300293],[1387238400000,0.123401514],[1387324800000,0.1301325647],[1387411200000,0.1292096937],[1387497600000,0.126781679],[1387756800000,0.1261266864],[1387843200000,0.1278216673],[1387929600000,0.1276238983],[1388016000000,0.1261705328],[1388102400000,0.1252538389],[1388361600000,0.1259000949],[1388448000000,0.124729588],[1388620800000,0.1272701099],[1388707200000,0.1210095752],[1388966400000,0.1214195713],[1389052800000,0.126131508],[1389139200000,0.12844865],[1389225600000,0.1290607675],[1389312000000,0.1287189842],[1389571200000,0.1299216919],[1389657600000,0.1308623358],[1389744000000,0.1335479296],[1389830400000,0.1340236196],[1389916800000,0.1356858379],[1390176000000,0.1400109887],[1390262400000,0.1394421009],[1390348800000,0.1370512332],[1390435200000,0.139067858],[1390521600000,0.1390469055],[1390780800000,0.1375830003],[1390867200000,0.1403199494],[1390953600000,0.1399731097],[1391040000000,0.1403032295],[1391731200000,0.1406082717],[1391990400000,0.1382161867],[1392076800000,0.1374268535],[1392163200000,0.1376229244],[1392249600000,0.1407473662],[1392336000000,0.1401845755],[1392595200000,0.139834968],[1392681600000,0.1397371156],[1392768000000,0.1392368417],[1392854400000,0.1395823199],[1392940800000,0.1393436602],[1393200000000,0.1367816829],[1393286400000,0.1371644954],[1393372800000,0.1377862746],[1393459200000,0.141480915],[1393545600000,0.1403792445],[1393804800000,0.1404584826],[1393891200000,0.1406625691],[1393977600000,0.1397783958],[1394064000000,0.1396005768],[1394150400000,0.141212344],[1394409600000,0.1594648564],[1394496000000,0.1593463869],[1394582400000,0.1600042181],[1394668800000,0.1636232197],[1394755200000,0.1647278795],[1395014400000,0.1639237341],[1395100800000,0.1577380832],[1395187200000,0.1535798304],[1395273600000,0.1551699326],[1395360000000,0.1538928138],[1395619200000,0.153130301],[1395705600000,0.1554402278],[1395792000000,0.1570127254],[1395878400000,0.1569822461],[1395964800000,0.1596192199],[1396224000000,0.1599388543],[1396310400000,0.1597584917],[1396396800000,0.1607608777],[1396483200000,0.1604983192],[1396569600000,0.1607508914],[1396915200000,0.1599124986],[1397001600000,0.1600925491],[1397088000000,0.1581282233],[1397174400000,0.1609515201],[1397433600000,0.1613516008],[1397520000000,0.1595982189],[1397606400000,0.1624162646],[1397692800000,0.1636280862],[1397779200000,0.165517506],[1398038400000,0.1641699514],[1398124800000,0.1657408317],[1398211200000,0.1657245566],[1398297600000,0.16591014],[1398384000000,0.1668213816],[1398643200000,0.1668852135],[1398729600000,0.1660101872],[1398816000000,0.1663174673],[1399248000000,0.1663980315],[1399334400000,0.1685175216],[1399420800000,0.1680076869],[1399507200000,0.1702761389],[1399593600000,0.1707517706],[1399852800000,0.169112938],[1399939200000,0.1726235741],[1400025600000,0.1736972816],[1400112000000,0.1751029633],[1400198400000,0.1813446328],[1400457600000,0.1801604025],[1400544000000,0.1811745048],[1400630400000,0.1807291182],[1400716800000,0.1813192203],[1400803200000,0.1806265181],[1401062400000,0.1800935334],[1401148800000,0.1816407895],[1401235200000,0.181894435],[1401321600000,0.1807425826],[1401408000000,0.1823430212],[1401753600000,0.1838500668],[1401840000000,0.1814836501],[1401926400000,0.1731316339],[1402012800000,0.1731340885],[1402272000000,0.173214311],[1402358400000,0.1522703372],[1402444800000,0.1538659389],[1402531200000,0.1519271898],[1402617600000,0.1323014184],[1402876800000,0.1286676665],[1402963200000,0.1280191749],[1403049600000,0.1157916091],[1403136000000,0.1142102642],[1403222400000,0.1166465905],[1403481600000,0.1168619494],[1403568000000,0.116466614],[1403654400000,0.116769315],[1403740800000,0.1181353799],[1403827200000,0.1202969562],[1404086400000,0.1196550061],[1404172800000,0.1174269111],[1404259200000,0.1123630344],[1404345600000,0.1136061404],[1404432000000,0.1165921144],[1404691200000,0.1164851231],[1404777600000,0.1164688197],[1404864000000,0.1157446282],[1404950400000,0.1164350196],[1405036800000,0.116798794],[1405296000000,0.1165213693],[1405382400000,0.1165630804],[1405468800000,0.1164899546],[1405555200000,0.1176827991],[1405641600000,0.1180184581],[1405900800000,0.1192252587],[1405987200000,0.1211526433],[1406073600000,0.1215971944],[1406160000000,0.1199489624],[1406246400000,0.1203550442],[1406505600000,0.1229955407],[1406592000000,0.1269036851],[1406678400000,0.1279648149],[1406764800000,0.136548226],[1406851200000,0.1390139975],[1407110400000,0.1501742885],[1407196800000,0.1501289866],[1407283200000,0.1505651736],[1407369600000,0.1490306425],[1407456000000,0.1499916859],[1407715200000,0.1496022182],[1407801600000,0.1510046926],[1407888000000,0.1525505243],[1407974400000,0.1525666505],[1408060800000,0.1546170864],[1408320000000,0.1582346137],[1408406400000,0.1586948155],[1408492800000,0.158753293],[1408579200000,0.15989298],[1408665600000,0.1599177835],[1408924800000,0.1584778658],[1409011200000,0.1586102304],[1409097600000,0.1591480493],[1409184000000,0.158185186],[1409270400000,0.1620536717],[1409529600000,0.1614951661],[1409616000000,0.1651875589],[1409702400000,0.177906379],[1409788800000,0.1875241833],[1409875200000,0.1999459143],[1410220800000,0.2000419641],[1410307200000,0.2005369202],[1410393600000,0.1875308697],[1410480000000,0.1937197059],[1410739200000,0.1955999953],[1410825600000,0.1904360217],[1410912000000,0.1894447972],[1410998400000,0.1903091128],[1411084800000,0.1889306071],[1411344000000,0.1846885192],[1411430400000,0.1843085121],[1411516800000,0.181823129],[1411603200000,0.1829010609],[1411689600000,0.1849492126],[1411948800000,0.1892709454],[1412035200000,0.1900474765],[1412726400000,0.2064520529],[1412812800000,0.2084287443],[1412899200000,0.2068876055],[1413158400000,0.2101241315],[1413244800000,0.2094857371],[1413331200000,0.2083458762],[1413417600000,0.2061041252],[1413504000000,0.2101855353],[1413763200000,0.209772054],[1413849600000,0.2075783469],[1413936000000,0.2062628978],[1414022400000,0.2062559391],[1414108800000,0.2095108634],[1414368000000,0.209704897],[1414454400000,0.2041461446],[1414540800000,0.2005149257],[1414627200000,0.1987021148],[1414713600000,0.2107683896],[1414972800000,0.2127610594],[1415059200000,0.2132037067],[1415145600000,0.2174456973],[1415232000000,0.2177526123],[1415318400000,0.2175812042],[1415577600000,0.2416511587],[1415664000000,0.2409726827],[1415750400000,0.2552566168],[1415836800000,0.2474744801],[1415923200000,0.2499625652],[1416182400000,0.2487718682],[1416268800000,0.2476987403],[1416355200000,0.2481418955],[1416441600000,0.2492773565],[1416528000000,0.2460894517],[1416787200000,0.2677399844],[1416873600000,0.2812997241],[1416960000000,0.2955604696],[1417046400000,0.3096560912],[1417132800000,0.3312333026],[1417392000000,0.33737298],[1417478400000,0.3694229432],[1417564800000,0.3866575902],[1417651200000,0.4230572796],[1417737600000,0.4294973317],[1417996800000,0.4635987088],[1418083200000,0.4399832755],[1418169600000,0.4246411573],[1418256000000,0.4200299177],[1418342400000,0.421251665],[1418601600000,0.4190159296],[1418688000000,0.4246455027],[1418774400000,0.4380148476],[1418860800000,0.434132377],[1418947200000,0.4388105641],[1419206400000,0.4412305935],[1419292800000,0.4328832599],[1419379200000,0.4192573735],[1419465600000,0.4058370919],[1419552000000,0.4090802061],[1419811200000,0.4110512636],[1419897600000,0.41174702],[1419984000000,0.4259960306],[1420416000000,0.448571249],[1420502400000,0.4477478153],[1420588800000,0.4460311651],[1420675200000,0.4380436064],[1420761600000,0.4368440229],[1421020800000,0.4331493],[1421107200000,0.4327714748],[1421193600000,0.431568309],[1421280000000,0.4118775741],[1421366400000,0.4098839764],[1421625600000,0.3699531109],[1421712000000,0.363609123],[1421798400000,0.3387421512],[1421884800000,0.3426382821],[1421971200000,0.3429336888],[1422230400000,0.3405758948],[1422316800000,0.3401568411],[1422403200000,0.3376777071],[1422489600000,0.3379089111],[1422576000000,0.3344200334],[1422835200000,0.328298233],[1422921600000,0.3244824756],[1423008000000,0.3219727539],[1423094400000,0.321926082],[1423180800000,0.3183702433],[1423440000000,0.3159808018],[1423526400000,0.3131537697],[1423612800000,0.3129706268],[1423699200000,0.313351127],[1423785600000,0.3113923163],[1424044800000,0.3098701927],[1424131200000,0.3090838661],[1424822400000,0.306773367],[1424908800000,0.3022462455],[1424995200000,0.3030863397],[1425254400000,0.3013780734],[1425340800000,0.2902117178],[1425427200000,0.2866866176],[1425513600000,0.2825834478],[1425600000000,0.2834201775],[1425859200000,0.2811173608],[1425945600000,0.2811322377],[1426032000000,0.2823150114],[1426118400000,0.2783609593],[1426204800000,0.2776317595],[1426464000000,0.2861815769],[1426550400000,0.2948889512],[1426636800000,0.3080365335],[1426723200000,0.3107660072],[1426809600000,0.3178603762],[1427068800000,0.3294827267],[1427155200000,0.3307438241],[1427241600000,0.3316772421],[1427328000000,0.332429845],[1427414400000,0.3338621534],[1427673600000,0.349700913],[1427760000000,0.3471064203],[1427846400000,0.3499306148],[1427932800000,0.3502673957],[1428019200000,0.3580832111],[1428364800000,0.3695602897],[1428451200000,0.3750391686],[1428537600000,0.3734532072],[1428624000000,0.3774704992],[1428883200000,0.3877198617],[1428969600000,0.3911267611],[1429056000000,0.3871026275],[1429142400000,0.3923890373],[1429228800000,0.4029820502],[1429488000000,0.3986922138],[1429574400000,0.3975089559],[1429660800000,0.4104526963],[1429747200000,0.4038153483],[1429833600000,0.3984438222],[1430092800000,0.4020837811],[1430179200000,0.3931159019],[1430265600000,0.3935803169],[1430352000000,0.3922270502],[1430697600000,0.3897583283],[1430784000000,0.378174708],[1430870400000,0.3756550304],[1430956800000,0.3652395905],[1431043200000,0.3637923659],[1431302400000,0.3566487281],[1431388800000,0.3554661081],[1431475200000,0.3546659752],[1431561600000,0.3537641261],[1431648000000,0.349984046],[1431907200000,0.349088108],[1431993600000,0.3411223659],[1432080000000,0.3329775193],[1432166400000,0.3261866705],[1432252800000,0.3349862577],[1432512000000,0.3456376425],[1432598400000,0.3523451914],[1432684800000,0.3452095869],[1432771200000,0.3296668426],[1432857600000,0.3154200878],[1433116800000,0.3038457139],[1433203200000,0.2835926584],[1433289600000,0.2878654029],[1433376000000,0.2871821553],[1433462400000,0.2947146376],[1433721600000,0.3022060482],[1433808000000,0.3055387825],[1433894400000,0.3053980244],[1433980800000,0.3023345984],[1434067200000,0.3002798649],[1434326400000,0.2991048098],[1434412800000,0.2949848573],[1434499200000,0.2925729655],[1434585600000,0.2858365933],[1434672000000,0.2753758704],[1435017600000,0.272566349],[1435104000000,0.2620530029],[1435190400000,0.2597660575],[1435276800000,0.2514495119],[1435536000000,0.2473359044],[1435622400000,0.2392380778],[1435708800000,0.2352724905],[1435795200000,0.2342683425],[1435881600000,0.228458329],[1436140800000,0.2256827707],[1436227200000,0.2260323724],[1436313600000,0.2198865521],[1436400000000,0.215216431],[1436486400000,0.2105879416],[1436745600000,0.2085733174],[1436832000000,0.2086168009],[1436918400000,0.206206822],[1437004800000,0.2115359107],[1437091200000,0.209085825],[1437350400000,0.2122061716],[1437436800000,0.2125004164],[1437523200000,0.2123958352],[1437609600000,0.2111350839],[1437696000000,0.2102333409],[1437955200000,0.2037708991],[1438041600000,0.2045033556],[1438128000000,0.2030412472],[1438214400000,0.2023460818],[1438300800000,0.2040083509],[1438560000000,0.2044812315],[1438646400000,0.2027730015],[1438732800000,0.1989171571],[1438819200000,0.1928422492],[1438905600000,0.1898850875],[1439164800000,0.1862517861],[1439251200000,0.1834641529],[1439337600000,0.1821994952],[1439424000000,0.1816755895],[1439510400000,0.1820445513],[1439769600000,0.1827410327],[1439856000000,0.1798509195],[1439942400000,0.1789211851],[1440028800000,0.1772926805],[1440115200000,0.1759857277],[1440374400000,0.1898495159],[1440460800000,0.2073453156],[1440547200000,0.2089030986],[1440633600000,0.2066644],[1440720000000,0.2044949071],[1440979200000,0.2041325249],[1441065600000,0.2052796422],[1441152000000,0.2056617358],[1441584000000,0.2051663938],[1441670400000,0.2045879961],[1441756800000,0.2050140855],[1441843200000,0.2043764222],[1441929600000,0.2051410837],[1442188800000,0.205384137],[1442275200000,0.203092368],[1442361600000,0.2009123672],[1442448000000,0.1998317611],[1442534400000,0.1999592362],[1442793600000,0.2009864312],[1442880000000,0.2010940687],[1442966400000,0.2010367634],[1443052800000,0.2006713006],[1443139200000,0.200545283],[1443398400000,0.2019141453],[1443484800000,0.2014225534],[1443571200000,0.2015892932],[1444262400000,0.2013931061],[1444348800000,0.2019698493],[1444608000000,0.2003971687],[1444694400000,0.2013507798],[1444780800000,0.203003113],[1444867200000,0.2030365744],[1444953600000,0.2035152685],[1445212800000,0.2052579236],[1445299200000,0.2066632993],[1445385600000,0.2047841553],[1445472000000,0.2045497079],[1445558400000,0.2056017981],[1445817600000,0.2064717652],[1445904000000,0.2070014111],[1445990400000,0.2062034825],[1446076800000,0.20674295],[1446163200000,0.2102936447],[1446422400000,0.2100164815],[1446508800000,0.2114042766],[1446595200000,0.209979165],[1446681600000,0.2110202676],[1446768000000,0.2104383024],[1447027200000,0.2100964632],[1447113600000,0.2102966841],[1447200000000,0.2118833652],[1447286400000,0.2119605297],[1447372800000,0.2140457465],[1447632000000,0.2141601468],[1447718400000,0.2157542911],[1447804800000,0.21711327],[1447891200000,0.2189258803],[1447977600000,0.2209796131],[1448236800000,0.2209049659],[1448323200000,0.2284230191],[1448409600000,0.2279555616],[1448496000000,0.2328150798],[1448582400000,0.2301282279],[1448841600000,0.230329953],[1448928000000,0.2306160698],[1449014400000,0.2287774577],[1449100800000,0.2291048915],[1449187200000,0.2283026795],[1449446400000,0.2282649951],[1449532800000,0.226873123],[1449619200000,0.2160101852],[1449705600000,0.2099975255],[1449792000000,0.2129889451],[1450051200000,0.1914314801],[1450137600000,0.1900185055],[1450224000000,0.1957252517],[1450310400000,0.1973623155],[1450396800000,0.1804608067],[1450656000000,0.1566598655],[1450742400000,0.1630040838],[1450828800000,0.1655072964],[1450915200000,0.1443880276],[1451001600000,0.1477999865],[1451260800000,0.1484168202],[1451347200000,0.151872623],[1451433600000,0.1539590622],[1451520000000,0.1546657779],[1451865600000,0.1547511529],[1451952000000,0.1593032434],[1452038400000,0.1624269647],[1452124800000,0.1594633186],[1452211200000,0.1601403345],[1452470400000,0.1596453484],[1452556800000,0.1597981914],[1452643200000,0.1617883989],[1452729600000,0.1605884959],[1452816000000,0.1585632599],[1453075200000,0.1584986409],[1453161600000,0.148880668],[1453248000000,0.1374453423],[1453334400000,0.1423331904],[1453420800000,0.1418662772],[1453680000000,0.1438685944],[1453766400000,0.1539087729],[1453852800000,0.1550907354],[1453939200000,0.1639724341],[1454025600000,0.1645554556],[1454284800000,0.1653563304],[1454371200000,0.1647976388],[1454457600000,0.1661865811],[1454544000000,0.1685256979],[1454630400000,0.168449632],[1455494400000,0.1692288338],[1455580800000,0.1685069069],[1455667200000,0.167943508],[1455753600000,0.1415361161],[1455840000000,0.1465663984],[1456099200000,0.145095336],[1456185600000,0.14708094],[1456272000000,0.1503611419],[1456358400000,0.1531346367],[1456444800000,0.1579417868],[1456704000000,0.1566521699],[1456790400000,0.1597524467],[1456876800000,0.160152862],[1456963200000,0.1605842739],[1457049600000,0.1598063521],[1457308800000,0.1596268087],[1457395200000,0.162479352],[1457481600000,0.1637215659],[1457568000000,0.1638925871],[1457654400000,0.164859551],[1457913600000,0.1638812971],[1458000000000,0.1653907653],[1458086400000,0.1684866393],[1458172800000,0.1719661706],[1458259200000,0.1726770625],[1458518400000,0.1708540783],[1458604800000,0.1718104957],[1458691200000,0.1724035068],[1458777600000,0.173038083],[1458864000000,0.1732138805],[1459123200000,0.1739490483],[1459209600000,0.1732561251],[1459296000000,0.1727598821],[1459382400000,0.1734123742],[1459468800000,0.1760905714],[1459814400000,0.1761224044],[1459900800000,0.1792269363],[1459987200000,0.1778067082],[1460073600000,0.1782829955],[1460332800000,0.1794190021],[1460419200000,0.1805463142],[1460505600000,0.1791924019],[1460592000000,0.1801095478],[1460678400000,0.1833647614],[1460937600000,0.1836122598],[1461024000000,0.1848351251],[1461110400000,0.1835335277],[1461196800000,0.1829816691],[1461283200000,0.1847550073],[1461542400000,0.1846056398],[1461628800000,0.1840622267],[1461715200000,0.1855606709],[1461801600000,0.1857335055],[1461888000000,0.1912626133],[1462233600000,0.1920035586],[1462320000000,0.1950539131],[1462406400000,0.1966482316],[1462492800000,0.193799742],[1462752000000,0.1914663043],[1462838400000,0.192722669],[1462924800000,0.1940158669],[1463011200000,0.1944052223],[1463097600000,0.194056352],[1463356800000,0.1948859512],[1463443200000,0.1967639744],[1463529600000,0.1961076381],[1463616000000,0.1966928483],[1463702400000,0.1961211731],[1463961600000,0.1968275774],[1464048000000,0.1967711655],[1464134400000,0.2045669972],[1464220800000,0.2047487901],[1464307200000,0.2057196982],[1464566400000,0.2110700513],[1464652800000,0.2078320229],[1464739200000,0.2105136437],[1464825600000,0.2106640746],[1464912000000,0.2125183856],[1465171200000,0.2126239563],[1465257600000,0.21311653],[1465344000000,0.2132035873],[1465776000000,0.21345316],[1465862400000,0.2137783453],[1465948800000,0.2123829316],[1466035200000,0.214554477],[1466121600000,0.2143894542],[1466380800000,0.2187796741],[1466467200000,0.2167705972],[1466553600000,0.2099610125],[1466640000000,0.2108758145],[1466726400000,0.1939315275],[1466985600000,0.1967443136],[1467072000000,0.1975284561],[1467158400000,0.1896403008],[1467244800000,0.1493308897],[1467331200000,0.1583003565],[1467590400000,0.1567763107],[1467676800000,0.1176108189],[1467763200000,0.1244113857],[1467849600000,0.1049588565],[1467936000000,0.1090662429],[1468195200000,0.1094114536],[1468281600000,0.1091279999],[1468368000000,0.1125782467],[1468454400000,0.1155095789],[1468540800000,0.1158688884],[1468800000000,0.1183660297],[1468886400000,0.1195182616],[1468972800000,0.1223260508],[1469059200000,0.1229287941],[1469145600000,0.1225100995],[1469404800000,0.12925],[1469491200000,0.1281483871],[1469577600000,0.1219871814],[1469664000000,0.125360636],[1469750400000,0.1264488278],[1470009600000,0.1277703537],[1470096000000,0.1277884776],[1470182400000,0.1290766277],[1470268800000,0.1296209849],[1470355200000,0.1301728788],[1470614400000,0.1327843777],[1470700800000,0.1329363388],[1470787200000,0.1327613309],[1470873600000,0.1324204791],[1470960000000,0.1364171991],[1471219200000,0.1651187946],[1471305600000,0.165383962],[1471392000000,0.1758448871],[1471478400000,0.1770954898],[1471564800000,0.1627399881],[1471824000000,0.1349250279],[1471910400000,0.1406958761],[1471996800000,0.1404423007],[1472083200000,0.1412026379],[1472169600000,0.1416798],[1472428800000,0.1417928535],[1472515200000,0.1434807872],[1472601600000,0.1438773319],[1472688000000,0.1271438572],[1472774400000,0.1288537763],[1473033600000,0.1290288953],[1473120000000,0.1286469341],[1473206400000,0.1302702155],[1473292800000,0.1326750991],[1473379200000,0.1356896342],[1473638400000,0.1339605322],[1473724800000,0.1343931217],[1473811200000,0.1361806342],[1474243200000,0.1357149819],[1474329600000,0.1369591856],[1474416000000,0.1383815254],[1474502400000,0.1416893848],[1474588800000,0.1408752909],[1474848000000,0.1378774001],[1474934400000,0.1391472707],[1475020800000,0.138979456],[1475107200000,0.1410167689],[1475193600000,0.1418951337],[1476057600000,0.1421232367],[1476144000000,0.1420604051],[1476230400000,0.1442918392],[1476316800000,0.1450313767],[1476403200000,0.1450578165],[1476662400000,0.1460588711],[1476748800000,0.1439450391],[1476835200000,0.1472546454],[1476921600000,0.1484365284],[1477008000000,0.1487775126],[1477267200000,0.1470123746],[1477353600000,0.1481200775],[1477440000000,0.1481588211],[1477526400000,0.1479122591],[1477612800000,0.1477984362],[1477872000000,0.151341829],[1477958400000,0.1500716252],[1478044800000,0.148653758],[1478131200000,0.1521625301],[1478217600000,0.1559856985],[1478476800000,0.1560643703],[1478563200000,0.1560039025],[1478649600000,0.1552256692],[1478736000000,0.1536300907],[1478822400000,0.1641004249],[1479081600000,0.1697822237],[1479168000000,0.171161529],[1479254400000,0.171577083],[1479340800000,0.1751925818],[1479427200000,0.1743772242],[1479686400000,0.1764956001],[1479772800000,0.1856115174],[1479859200000,0.188404936],[1479945600000,0.1919738218],[1480032000000,0.2041415207],[1480291200000,0.2190803803],[1480377600000,0.2304668145],[1480464000000,0.22840543],[1480550400000,0.2283592205],[1480636800000,0.2257173893],[1480896000000,0.2199629499],[1480982400000,0.2202349408],[1481068800000,0.2239952367],[1481155200000,0.2238679955],[1481241600000,0.2256102918],[1481500800000,0.2192413641],[1481587200000,0.2203852269],[1481673600000,0.2180430294],[1481760000000,0.214952571],[1481846400000,0.2170603336],[1482105600000,0.2170207466],[1482192000000,0.1995206119],[1482278400000,0.1936095829],[1482364800000,0.1879415707],[1482451200000,0.1856885408],[1482710400000,0.1845393703],[1482796800000,0.1739358342],[1482883200000,0.1767872024],[1482969600000,0.1766705499],[1483056000000,0.1764377062],[1483401600000,0.1744320837],[1483488000000,0.1737754522],[1483574400000,0.1745983918],[1483660800000,0.1785926222],[1483920000000,0.1780569353],[1484006400000,0.1780961425],[1484092800000,0.1762134733],[1484179200000,0.1760041708],[1484265600000,0.1769265825],[1484524800000,0.1777504086],[1484611200000,0.1784046836],[1484697600000,0.1795889479],[1484784000000,0.1792282976],[1484870400000,0.180318808],[1485129600000,0.1839202979],[1485216000000,0.1841410026],[1485302400000,0.1846454577],[1485388800000,0.1801141072],[1486080000000,0.1770252628],[1486339200000,0.1729671897],[1486425600000,0.1711710855],[1486512000000,0.1628884878],[1486598400000,0.1641346937],[1486684800000,0.164573458],[1486944000000,0.1638585368],[1487030400000,0.1646025304],[1487116800000,0.1683628621],[1487203200000,0.1752051836],[1487289600000,0.1748213806],[1487548800000,0.1709423746],[1487635200000,0.1706994638],[1487721600000,0.1701594196],[1487808000000,0.1711553832],[1487894400000,0.1715394567],[1488153600000,0.1702131861],[1488240000000,0.1714234571],[1488326400000,0.1711044166],[1488412800000,0.1692023793],[1488499200000,0.1689511367],[1488758400000,0.1687179763],[1488844800000,0.1702787601],[1488931200000,0.1708949231],[1489017600000,0.1695318647],[1489104000000,0.1713851094],[1489363200000,0.1689983432],[1489449600000,0.1689859178],[1489536000000,0.1702014218],[1489622400000,0.1734179062],[1489708800000,0.1705499985],[1489968000000,0.1720651208],[1490054400000,0.1727245855],[1490140800000,0.1718138733],[1490227200000,0.1715838509],[1490313600000,0.1713161269],[1490572800000,0.1716479216],[1490659200000,0.1703656118],[1490745600000,0.1717535672],[1490832000000,0.1632562663],[1490918400000,0.1582634475],[1491350400000,0.1429488807],[1491436800000,0.145147594],[1491523200000,0.1458316405],[1491782400000,0.1454055669],[1491868800000,0.1447058112],[1491955200000,0.1443646217],[1492041600000,0.135931119],[1492128000000,0.1370027321],[1492387200000,0.136899579],[1492473600000,0.1358668578],[1492560000000,0.135335757],[1492646400000,0.1370125643],[1492732800000,0.1366546005],[1492992000000,0.1350744911],[1493078400000,0.135032864],[1493164800000,0.1351257072],[1493251200000,0.1352836969],[1493337600000,0.1363743048],[1493683200000,0.1372517806],[1493769600000,0.1385523744],[1493856000000,0.1387084051],[1493942400000,0.137324414],[1494201600000,0.136677661],[1494288000000,0.1375066881],[1494374400000,0.1391336939],[1494460800000,0.1396907028],[1494547200000,0.1385538109],[1494806400000,0.1375958816],[1494892800000,0.1355595228],[1494979200000,0.1347571367],[1495065600000,0.1352487843],[1495152000000,0.1364857274],[1495411200000,0.1378840717],[1495497600000,0.1374176101],[1495584000000,0.1384131785],[1495670400000,0.13637013],[1495756800000,0.1369676285],[1496188800000,0.1381363378],[1496275200000,0.1396651548],[1496361600000,0.1218129416],[1496620800000,0.1168878154],[1496707200000,0.1191321841],[1496793600000,0.1259320497],[1496880000000,0.1396088914],[1496966400000,0.1469189811],[1497225600000,0.1486272228],[1497312000000,0.1583628804],[1497398400000,0.1546754376],[1497484800000,0.156339638],[1497571200000,0.1588261236],[1497830400000,0.156269224],[1497916800000,0.1571210038],[1498003200000,0.1583784492],[1498089600000,0.1620744286],[1498176000000,0.1771877333],[1498435200000,0.1999589639],[1498521600000,0.2044406842],[1498608000000,0.2019717102],[1498694400000,0.201092523],[1498780800000,0.1947384724],[1499040000000,0.1903426808],[1499126400000,0.1904726346],[1499212800000,0.1890988811],[1499299200000,0.1891088952],[1499385600000,0.1907198325],[1499644800000,0.1921555755],[1499731200000,0.1909748219],[1499817600000,0.1921855659],[1499904000000,0.1965795048],[1499990400000,0.2037170773],[1500249600000,0.195981834],[1500336000000,0.1947909827],[1500422400000,0.2034299532],[1500508800000,0.20751901],[1500595200000,0.208184665],[1500854400000,0.2075862243],[1500940800000,0.2051423113],[1501027200000,0.2048636257],[1501113600000,0.2054032399],[1501200000000,0.2068294356],[1501459200000,0.2060750387],[1501545600000,0.2146847803],[1501632000000,0.2155489354],[1501718400000,0.2134204088],[1501804800000,0.2130998173],[1502064000000,0.2134751633],[1502150400000,0.212955079],[1502236800000,0.2143179105],[1502323200000,0.2147540414],[1502409600000,0.2096470189],[1502668800000,0.2099134983],[1502755200000,0.2099185903],[1502841600000,0.2101530929],[1502928000000,0.2098045871],[1503014400000,0.2095140328],[1503273600000,0.2106677703],[1503360000000,0.2101847399],[1503446400000,0.2103727762],[1503532800000,0.2105672975],[1503619200000,0.2172201699],[1503878400000,0.2363139497],[1503964800000,0.2362376738],[1504051200000,0.2367904048],[1504137600000,0.2378588418],[1504224000000,0.2370215662],[1504483200000,0.2400922705],[1504569600000,0.2443379094],[1504656000000,0.2442632126],[1504742400000,0.2440669204],[1504828800000,0.2477720316],[1505088000000,0.2481817052],[1505174400000,0.2487117845],[1505260800000,0.2501035376],[1505347200000,0.2500168491],[1505433600000,0.2532204117],[1505692800000,0.2531858467],[1505779200000,0.2528419464],[1505865600000,0.2521422503],[1505952000000,0.2550910145],[1506038400000,0.2575195613],[1506297600000,0.2611321172],[1506384000000,0.2620769351],[1506470400000,0.2624729618],[1506556800000,0.2639375921],[1506643200000,0.2637003864],[1507507200000,0.2712662346],[1507593600000,0.2747324374],[1507680000000,0.2832640619],[1507766400000,0.287874011],[1507852800000,0.2932452542],[1508112000000,0.2946242316],[1508198400000,0.2969173131],[1508284800000,0.3046971166],[1508371200000,0.3081997205],[1508457600000,0.309033652],[1508716800000,0.3090746007],[1508803200000,0.312531419],[1508889600000,0.3196537688],[1508976000000,0.327403059],[1509062400000,0.3390462883],[1509321600000,0.3384576576],[1509408000000,0.3416413283],[1509494400000,0.3439839135],[1509580800000,0.3449845726],[1509667200000,0.3371986944],[1509926400000,0.321133823],[1510012800000,0.3302247163],[1510099200000,0.3314876375],[1510185600000,0.3426972257],[1510272000000,0.3577693796],[1510531200000,0.3622114192],[1510617600000,0.3544417084],[1510704000000,0.3449289708],[1510790400000,0.3418483804],[1510876800000,0.3177568292],[1511136000000,0.3315049523],[1511222400000,0.3557872735],[1511308800000,0.3609183484],[1511395200000,0.3413833128],[1511481600000,0.3428377798],[1511740800000,0.3265521104],[1511827200000,0.3176269728],[1511913600000,0.3232648929],[1512000000000,0.320264775],[1512086400000,0.3213144331],[1512345600000,0.3185791785],[1512432000000,0.3166677091],[1512518400000,0.3199005289],[1512604800000,0.3144717986],[1512691200000,0.2992101078],[1512950400000,0.2951631007],[1513036800000,0.2717385301],[1513123200000,0.2715812084],[1513209600000,0.2565911999],[1513296000000,0.2552169968],[1513555200000,0.2596649429],[1513641600000,0.254935363],[1513728000000,0.2574970334],[1513814400000,0.2559027924],[1513900800000,0.2546991851],[1514160000000,0.2550081213],[1514246400000,0.2454565453],[1514332800000,0.2431208229],[1514419200000,0.2403741787],[1514505600000,0.2396019161],[1514851200000,0.2343461596],[1514937600000,0.2336977098],[1515024000000,0.2331877133],[1515110400000,0.2349021275],[1515369600000,0.2344048576],[1515456000000,0.2353929165],[1515542400000,0.2339794902],[1515628800000,0.2398158251],[1515715200000,0.2396890181],[1515974400000,0.2415892209],[1516060800000,0.2525247517],[1516147200000,0.2539476746],[1516233600000,0.2583488229],[1516320000000,0.2633534066],[1516579200000,0.2796951657],[1516665600000,0.2947737259],[1516752000000,0.3007342786],[1516838400000,0.2988846267],[1516924800000,0.3009195281],[1517184000000,0.2938890208],[1517270400000,0.290814668],[1517356800000,0.2891859999],[1517443200000,0.2859750743],[1517529600000,0.2848549716],[1517788800000,0.2735883955],[1517875200000,0.2628741395],[1517961600000,0.2544216229],[1518048000000,0.2440333889],[1518134400000,0.2300708661],[1518393600000,0.2230657411],[1518480000000,0.2208694011],[1518566400000,0.2193888826],[1519257600000,0.2135278169],[1519344000000,0.1938112449],[1519603200000,0.1874637508],[1519689600000,0.1867002581],[1519776000000,0.1850593898],[1519862400000,0.1835624789],[1519948800000,0.1823676233],[1520208000000,0.182764371],[1520294400000,0.180835999],[1520380800000,0.1797858032],[1520467200000,0.1779211126],[1520553600000,0.1772766147],[1520812800000,0.1764607906],[1520899200000,0.1745186324],[1520985600000,0.1742194233],[1521072000000,0.1732623042],[1521158400000,0.171852377],[1521417600000,0.1710053176],[1521504000000,0.1714352496],[1521590400000,0.1711465111],[1521676800000,0.1696050849],[1521763200000,0.1642115778],[1522022400000,0.1630434783],[1522108800000,0.1615724863],[1522195200000,0.158414766],[1522281600000,0.155831564],[1522368000000,0.1518675046],[1522627200000,0.1508271271],[1522713600000,0.1517132795],[1522800000000,0.1517145367],[1523232000000,0.1521727084],[1523318400000,0.149522228],[1523404800000,0.1493953357],[1523491200000,0.1481002111],[1523577600000,0.1470250476],[1523836800000,0.1542470161],[1523923200000,0.1681221963],[1524009600000,0.1675406213],[1524096000000,0.1657387879],[1524182400000,0.1647851818],[1524441600000,0.1652978842],[1524528000000,0.1627583511],[1524614400000,0.1633221183],[1524700800000,0.1608251301],[1524787200000,0.1608900954],[1525219200000,0.1610246314],[1525305600000,0.1598668711],[1525392000000,0.1593120205],[1525651200000,0.1580788459],[1525737600000,0.1576463068],[1525824000000,0.1576145332],[1525910400000,0.1578648062],[1525996800000,0.1584808575],[1526256000000,0.1576901134],[1526342400000,0.158231362],[1526428800000,0.1580137943],[1526515200000,0.1581131461],[1526601600000,0.157214972],[1526860800000,0.1574002027],[1526947200000,0.1597068119],[1527033600000,0.1580628702],[1527120000000,0.161967811],[1527206400000,0.1616037137],[1527465600000,0.1631451069],[1527552000000,0.1621804129],[1527638400000,0.1652870428],[1527724800000,0.1640115555],[1527811200000,0.1630554176],[1528070400000,0.1624089903],[1528156800000,0.1617613867],[1528243200000,0.1624070117],[1528329600000,0.1639417649],[1528416000000,0.1631763721],[1528675200000,0.1658463759],[1528761600000,0.1661568683],[1528848000000,0.1660103663],[1528934400000,0.1663820907],[1529020800000,0.1674416735],[1529366400000,0.1871174081],[1529452800000,0.1887585108],[1529539200000,0.1938789412],[1529625600000,0.1948950469],[1529884800000,0.2011110195],[1529971200000,0.2074122378],[1530057600000,0.2215669655],[1530144000000,0.2315444443],[1530230400000,0.2282758132],[1530489600000,0.2270689141],[1530576000000,0.2300065119],[1530662400000,0.2391555651],[1530748800000,0.2439010192],[1530835200000,0.2431796179],[1531094400000,0.2391460196],[1531180800000,0.2402871195],[1531267200000,0.2380221968],[1531353600000,0.2342266796],[1531440000000,0.2346348817],[1531699200000,0.2335696271],[1531785600000,0.234118833],[1531872000000,0.2337160733],[1531958400000,0.2347805622],[1532044800000,0.2321491499],[1532304000000,0.2330966279],[1532390400000,0.2325816351],[1532476800000,0.2308444401],[1532563200000,0.2299914765],[1532649600000,0.2124962631],[1532908800000,0.2101147521],[1532995200000,0.2121847211],[1533081600000,0.2093847969],[1533168000000,0.2072225756],[1533254400000,0.211833347],[1533513600000,0.1917855625],[1533600000000,0.1883614494],[1533686400000,0.1902077726],[1533772800000,0.1883390279],[1533859200000,0.1953990655],[1534118400000,0.1969652959],[1534204800000,0.1982495383],[1534291200000,0.1959938706],[1534377600000,0.1992165641],[1534464000000,0.2079999074],[1534723200000,0.2084682614],[1534809600000,0.2084701972],[1534896000000,0.2092785884],[1534982400000,0.2099326757],[1535068800000,0.211282517],[1535328000000,0.2073984947],[1535414400000,0.2094328138],[1535500800000,0.2102324601],[1535587200000,0.2105020123],[1535673600000,0.211228846],[1535932800000,0.2041112443],[1536019200000,0.2038394859],[1536105600000,0.2015706197],[1536192000000,0.1967073188],[1536278400000,0.1978536767],[1536537600000,0.1964903826],[1536624000000,0.1937780042],[1536710400000,0.1892490674],[1536796800000,0.17052576],[1536883200000,0.1749789095],[1537142400000,0.1744763837],[1537228800000,0.1731987863],[1537315200000,0.1743264359],[1537401600000,0.1763716759],[1537488000000,0.1724108056],[1537833600000,0.1716427652],[1537920000000,0.1711241877],[1538006400000,0.1708847099],[1538092800000,0.169572637],[1538956800000,0.1667417735],[1539043200000,0.1638495383],[1539129600000,0.1650710047],[1539216000000,0.1776038718],[1539302400000,0.1782234906],[1539561600000,0.178858644],[1539648000000,0.183710181],[1539734400000,0.1848770638],[1539820800000,0.1965753333],[1539907200000,0.1929078605],[1540166400000,0.1904785619],[1540252800000,0.1875526333],[1540339200000,0.1903297902],[1540425600000,0.1901356244],[1540512000000,0.1895661168],[1540771200000,0.1868523062],[1540857600000,0.1862600624],[1540944000000,0.1868943847],[1541030400000,0.1877431595],[1541116800000,0.1835838618],[1541376000000,0.1833554266],[1541462400000,0.1833756835],[1541548800000,0.1839589302],[1541635200000,0.1835785304],[1541721600000,0.1830393487],[1541980800000,0.1826995402],[1542067200000,0.1829576183],[1542153600000,0.1792881067],[1542240000000,0.1677655147],[1542326400000,0.1671051561],[1542585600000,0.1667721664],[1542672000000,0.164529243],[1542758400000,0.1648231341],[1542844800000,0.1654082699],[1542931200000,0.1657399456],[1543190400000,0.1684154527],[1543276800000,0.1694053796],[1543363200000,0.1692371408],[1543449600000,0.1674278367],[1543536000000,0.1651452805],[1543795200000,0.1613086982],[1543881600000,0.16280838],[1543968000000,0.1622894448],[1544054400000,0.1537023264],[1544140800000,0.151785454],[1544400000000,0.1469656112],[1544486400000,0.1226333862],[1544572800000,0.1258053736],[1544659200000,0.1192738475],[1544745600000,0.118989503],[1545004800000,0.1135267844],[1545091200000,0.1139034157],[1545177600000,0.1137122613],[1545264000000,0.1148826628],[1545350400000,0.1180298371],[1545609600000,0.1200475574],[1545696000000,0.1249109249],[1545782400000,0.1278915587],[1545955200000,0.1289415617],[1546387200000,0.1355714121],[1546473600000,0.1371584213],[1546560000000,0.1378643728],[1546819200000,0.1375541859],[1546905600000,0.1392115401],[1546992000000,0.1405676758],[1547078400000,0.1407549891],[1547164800000,0.1407071511],[1547424000000,0.1405695978],[1547510400000,0.139232278],[1547596800000,0.139305224],[1547683200000,0.1408099304],[1547769600000,0.1400462227],[1548028800000,0.1403738202],[1548115200000,0.1297293984],[1548201600000,0.1274545288],[1548288000000,0.1265581682],[1548374400000,0.1259945529],[1548633600000,0.1100265404],[1548720000000,0.1095827708],[1548806400000,0.1108886154],[1548892800000,0.11147615],[1548979200000,0.1113776648],[1549843200000,0.1123419222],[1549929600000,0.1131596223],[1550016000000,0.1136024848],[1550102400000,0.1136673189],[1550188800000,0.1123536383],[1550448000000,0.1116137223],[1550534400000,0.1135856356],[1550620800000,0.1151273494],[1550707200000,0.1161676989],[1550793600000,0.131185961],[1551052800000,0.1744665996],[1551139200000,0.1734341615],[1551225600000,0.1736618336],[1551312000000,0.1735152357],[1551398400000,0.1781422923],[1551657600000,0.1866084589],[1551744000000,0.1911813592],[1551830400000,0.1985384691],[1551916800000,0.1975327693],[1552003200000,0.1915943601],[1552262400000,0.190343707],[1552348800000,0.1920137915],[1552435200000,0.1921850895],[1552521600000,0.1917286949],[1552608000000,0.1917728211],[1552867200000,0.188439638],[1552953600000,0.188615967],[1553040000000,0.1899518754],[1553126400000,0.1901175976],[1553212800000,0.1915166984],[1553472000000,0.1904286475],[1553558400000,0.1904556367],[1553644800000,0.1888095066],[1553731200000,0.1923044399],[1553817600000,0.1920529241],[1554076800000,0.2106919224],[1554163200000,0.2111773816],[1554249600000,0.2205506884],[1554336000000,0.2341793086],[1554681600000,0.2340399667],[1554768000000,0.2362952419],[1554854400000,0.2462167303],[1554940800000,0.2439847734],[1555027200000,0.2458748879],[1555286400000,0.2465443413],[1555372800000,0.2416032311],[1555459200000,0.2457377936],[1555545600000,0.2498625355],[1555632000000,0.2623601678],[1555891200000,0.2618750906],[1555977600000,0.2617665511],[1556064000000,0.2614722278],[1556150400000,0.257533691],[1556236800000,0.2600953273],[1556496000000,0.2614077664],[1556582400000,0.263234373],[1557100800000,0.2514713786],[1557187200000,0.2557658347],[1557273600000,0.2543138541],[1557360000000,0.2516520675],[1557446400000,0.2457913675],[1557705600000,0.2430733445],[1557792000000,0.2441903458],[1557878400000,0.2419290354],[1557964800000,0.2427099039],[1558051200000,0.2395527183],[1558310400000,0.2398738959],[1558396800000,0.2382083006],[1558483200000,0.2391710123],[1558569600000,0.2397052449],[1558656000000,0.2396123147],[1558915200000,0.2380709123],[1559001600000,0.2398440289],[1559088000000,0.2395403061],[1559174400000,0.238614029],[1559260800000,0.2401160931],[1559520000000,0.2420805476],[1559606400000,0.2421566286],[1559692800000,0.2466468769],[1559779200000,0.2453036024],[1560124800000,0.2437335774],[1560211200000,0.2417852996],[1560297600000,0.2403737766],[1560384000000,0.2419381511],[1560470400000,0.2411582071],[1560729600000,0.2416993423],[1560816000000,0.2435471398],[1560902400000,0.2438276749],[1560988800000,0.2384023861],[1561075200000,0.2397549698],[1561334400000,0.2412548584],[1561420800000,0.2404392843],[1561507200000,0.2420019808],[1561593600000,0.240406776],[1561680000000,0.2409927709],[1561939200000,0.2363131896],[1562025600000,0.2366644961],[1562112000000,0.2365419526],[1562198400000,0.2212924126],[1562284800000,0.2211770575],[1562544000000,0.2179431188],[1562630400000,0.2137313615],[1562716800000,0.2147948816],[1562803200000,0.2149821636],[1562889600000,0.2149198462],[1563148800000,0.20649783],[1563235200000,0.2083248182],[1563321600000,0.208258613],[1563408000000,0.2013542014],[1563494400000,0.2020682101],[1563753600000,0.201701644],[1563840000000,0.2031274949],[1563926400000,0.1983138283],[1564012800000,0.195768523],[1564099200000,0.196503459],[1564358400000,0.1963571645],[1564444800000,0.1961201525],[1564531200000,0.1889233805],[1564617600000,0.1795605218],[1564704000000,0.1670613342],[1564963200000,0.1617423731],[1565049600000,0.1595256117],[1565136000000,0.1612291945],[1565222400000,0.1598203756],[1565308800000,0.1399866386],[1565568000000,0.1411902274],[1565654400000,0.1397587878],[1565740800000,0.1396341375],[1565827200000,0.1236285851],[1565913600000,0.1159071292],[1566172800000,0.1191058226],[1566259200000,0.1201724995],[1566345600000,0.1201810726],[1566432000000,0.1201101869],[1566518400000,0.1215012097],[1566777600000,0.121221438],[1566864000000,0.1204563183],[1566950400000,0.1209203283],[1567036800000,0.1216306688],[1567123200000,0.1255269522],[1567382400000,0.1262005889],[1567468800000,0.1267793533],[1567555200000,0.1267438017],[1567641600000,0.1263514604],[1567728000000,0.1270247112],[1567987200000,0.129459462],[1568073600000,0.1295831838],[1568160000000,0.1287524723],[1568246400000,0.1275463733],[1568592000000,0.1272086989],[1568678400000,0.1279249842],[1568764800000,0.1286214029],[1568851200000,0.1294796638],[1568937600000,0.1295919438],[1569196800000,0.1326516977],[1569283200000,0.1356016128],[1569369600000,0.134699396],[1569456000000,0.1353892688],[1569542400000,0.1363375515],[1569801600000,0.1352390822],[1570492800000,0.1350764274],[1570579200000,0.1352361213],[1570665600000,0.137133258],[1570752000000,0.1361882574],[1571011200000,0.1352469785],[1571097600000,0.138387434],[1571184000000,0.1379715137],[1571270400000,0.1384053497],[1571356800000,0.1156588361],[1571616000000,0.1181108757],[1571702400000,0.1178584362],[1571788800000,0.1030305783],[1571875200000,0.1053667979],[1571961600000,0.1060904499],[1572220800000,0.1055766023],[1572307200000,0.1054819507],[1572393600000,0.1115208431],[1572480000000,0.1124909075],[1572566400000,0.1121069584],[1572825600000,0.1148521013],[1572912000000,0.1254057303],[1572998400000,0.1269975159],[1573084800000,0.1276108035],[1573171200000,0.1299988725],[1573430400000,0.1279802069],[1573516800000,0.1316128606],[1573603200000,0.1326995127],[1573689600000,0.1344550967]]},{"name":"close","yAxis":0,"data":[[1144195200000,1099.97],[1144281600000,1103.24],[1144368000000,1103.15],[1144627200000,1117.91],[1144713600000,1123.31],[1144800000000,1117.07],[1144886400000,1093.93],[1144972800000,1118.61],[1145232000000,1124.41],[1145318400000,1131.28],[1145404800000,1138.24],[1145491200000,1134.38],[1145577600000,1149.16],[1145836800000,1142.7],[1145923200000,1141.93],[1146009600000,1155.73],[1146096000000,1155.27],[1146182400000,1172.35],[1147046400000,1218.44],[1147132800000,1251.61],[1147219200000,1265.93],[1147305600000,1255.04],[1147392000000,1296.26],[1147651200000,1352.16],[1147737600000,1331.13],[1147824000000,1335.52],[1147910400000,1331.2],[1147996800000,1366.1],[1148256000000,1373.67],[1148342400000,1317.65],[1148428800000,1308.24],[1148515200000,1307.7],[1148601600000,1331.02],[1148860800000,1366.29],[1148947200000,1378.76],[1149033600000,1365.45],[1149120000000,1402.88],[1149206400000,1390.12],[1149465600000,1403.16],[1149552000000,1399.14],[1149638400000,1320.23],[1149724800000,1325.98],[1149811200000,1294.19],[1150070400000,1297.67],[1150156800000,1298.28],[1150243200000,1283.88],[1150329600000,1285.39],[1150416000000,1318.01],[1150675200000,1334.89],[1150761600000,1338.22],[1150848000000,1333.53],[1150934400000,1331.55],[1151020800000,1339.45],[1151280000000,1363.41],[1151366400000,1363.9],[1151452800000,1362.89],[1151539200000,1395.12],[1151625600000,1393.96],[1151884800000,1420.33],[1151971200000,1411.01],[1152057600000,1393.01],[1152144000000,1418.68],[1152230400000,1410.43],[1152489600000,1412.12],[1152576000000,1418.57],[1152662400000,1419.2],[1152748800000,1346.09],[1152835200000,1357.13],[1153094400000,1372.25],[1153180800000,1373.42],[1153267200000,1336.64],[1153353600000,1345.19],[1153440000000,1356.03],[1153699200000,1358.12],[1153785600000,1374.17],[1153872000000,1371.3],[1153958400000,1355.55],[1154044800000,1341.39],[1154304000000,1294.33],[1154390400000,1282.06],[1154476800000,1275.09],[1154563200000,1271.74],[1154649600000,1241.91],[1154908800000,1224.1],[1154995200000,1252.39],[1155081600000,1251.3],[1155168000000,1271.47],[1155254400000,1275.65],[1155513600000,1245.72],[1155600000000,1265.86],[1155686400000,1283.57],[1155772800000,1271.63],[1155859200000,1267.87],[1156118400000,1270.56],[1156204800000,1285.27],[1156291200000,1285.68],[1156377600000,1292.4],[1156464000000,1295.44],[1156723200000,1325.89],[1156809600000,1330.16],[1156896000000,1334.67],[1156982400000,1338.69],[1157068800000,1318.1],[1157328000000,1337.24],[1157414400000,1340.68],[1157500800000,1346.37],[1157587200000,1328.38],[1157673600000,1332.15],[1157932800000,1338.76],[1158019200000,1347.64],[1158105600000,1338.39],[1158192000000,1338.28],[1158278400000,1362.32],[1158537600000,1375.56],[1158624000000,1378.31],[1158710400000,1378.46],[1158796800000,1387.37],[1158883200000,1374.85],[1159142400000,1372.4],[1159228800000,1357.65],[1159315200000,1371.12],[1159401600000,1387.0],[1159488000000,1403.27],[1160352000000,1436.07],[1160438400000,1437.24],[1160524800000,1435.91],[1160611200000,1426.5],[1160697600000,1430.88],[1160956800000,1418.52],[1161043200000,1414.45],[1161129600000,1437.59],[1161216000000,1439.38],[1161302400000,1440.18],[1161561600000,1408.71],[1161648000000,1440.05],[1161734400000,1446.82],[1161820800000,1456.09],[1161907200000,1439.05],[1162166400000,1446.24],[1162252800000,1464.47],[1162339200000,1479.41],[1162425600000,1479.66],[1162512000000,1488.29],[1162771200000,1507.89],[1162857600000,1516.1],[1162944000000,1498.17],[1163030400000,1524.71],[1163116800000,1504.06],[1163376000000,1475.78],[1163462400000,1493.78],[1163548800000,1534.76],[1163635200000,1533.29],[1163721600000,1562.08],[1163980800000,1593.16],[1164067200000,1612.25],[1164153600000,1624.03],[1164240000000,1634.91],[1164326400000,1636.58],[1164585600000,1651.8],[1164672000000,1644.01],[1164758400000,1667.14],[1164844800000,1714.36],[1164931200000,1729.22],[1165190400000,1780.74],[1165276800000,1794.23],[1165363200000,1779.41],[1165449600000,1775.71],[1165536000000,1711.58],[1165795200000,1789.92],[1165881600000,1802.79],[1165968000000,1803.86],[1166054400000,1836.14],[1166140800000,1867.64],[1166400000000,1916.11],[1166486400000,1921.44],[1166572800000,1936.55],[1166659200000,1908.98],[1166745600000,1895.64],[1167004800000,1939.1],[1167091200000,1938.24],[1167177600000,1982.88],[1167264000000,1979.93],[1167350400000,2041.05],[1167868800000,2067.09],[1167955200000,2072.88],[1168214400000,2131.56],[1168300800000,2200.09],[1168387200000,2255.97],[1168473600000,2231.63],[1168560000000,2173.75],[1168819200000,2287.34],[1168905600000,2353.87],[1168992000000,2308.93],[1169078400000,2317.09],[1169164800000,2396.09],[1169424000000,2491.31],[1169510400000,2508.13],[1169596800000,2536.43],[1169683200000,2452.83],[1169769600000,2512.92],[1170028800000,2576.92],[1170115200000,2551.88],[1170201600000,2385.33],[1170288000000,2395.17],[1170374400000,2298.0],[1170633600000,2271.8],[1170720000000,2316.04],[1170806400000,2369.79],[1170892800000,2410.6],[1170979200000,2397.25],[1171238400000,2485.39],[1171324800000,2522.63],[1171411200000,2588.35],[1171497600000,2668.63],[1171584000000,2676.74],[1172448000000,2707.68],[1172534400000,2457.49],[1172620800000,2544.57],[1172707200000,2473.54],[1172793600000,2508.73],[1173052800000,2475.61],[1173139200000,2520.29],[1173225600000,2589.44],[1173312000000,2627.63],[1173398400000,2611.39],[1173657600000,2616.17],[1173744000000,2640.17],[1173830400000,2597.36],[1173916800000,2645.55],[1174003200000,2604.23],[1174262400000,2659.41],[1174348800000,2672.77],[1174435200000,2702.6],[1174521600000,2711.32],[1174608000000,2716.27],[1174867200000,2764.03],[1174953600000,2784.02],[1175040000000,2797.65],[1175126400000,2783.3],[1175212800000,2781.78],[1175472000000,2850.11],[1175558400000,2888.11],[1175644800000,2911.82],[1175731200000,2945.04],[1175817600000,2972.01],[1176076800000,3038.17],[1176163200000,3081.57],[1176249600000,3121.32],[1176336000000,3176.44],[1176422400000,3169.23],[1176681600000,3256.0],[1176768000000,3283.6],[1176854400000,3304.5],[1176940800000,3150.3],[1177027200000,3289.28],[1177286400000,3431.32],[1177372800000,3445.2],[1177459200000,3448.28],[1177545600000,3493.58],[1177632000000,3470.52],[1177891200000,3558.71],[1178582400000,3686.03],[1178668800000,3701.28],[1178755200000,3724.51],[1178841600000,3702.61],[1179100800000,3734.42],[1179187200000,3604.64],[1179273600000,3700.29],[1179360000000,3778.6],[1179446400000,3776.63],[1179705600000,3831.44],[1179792000000,3870.49],[1179878400000,3938.95],[1179964800000,3919.75],[1180051200000,3985.25],[1180310400000,4072.58],[1180396800000,4168.29],[1180483200000,3886.46],[1180569600000,3927.95],[1180656000000,3803.95],[1180915200000,3511.43],[1181001600000,3634.63],[1181088000000,3677.58],[1181174400000,3802.3],[1181260800000,3837.87],[1181520000000,3931.86],[1181606400000,4036.11],[1181692800000,4118.27],[1181779200000,4075.82],[1181865600000,4099.38],[1182124800000,4227.57],[1182211200000,4253.0],[1182297600000,4157.6],[1182384000000,4197.28],[1182470400000,4051.43],[1182729600000,3877.59],[1182816000000,3928.21],[1182902400000,4040.48],[1182988800000,3858.52],[1183075200000,3764.08],[1183334400000,3757.66],[1183420800000,3832.23],[1183507200000,3743.58],[1183593600000,3537.44],[1183680000000,3710.28],[1183939200000,3821.3],[1184025600000,3775.62],[1184112000000,3789.87],[1184198400000,3816.92],[1184284800000,3820.12],[1184544000000,3697.97],[1184630400000,3789.65],[1184716800000,3807.57],[1184803200000,3807.0],[1184889600000,3971.88],[1185148800000,4156.72],[1185235200000,4161.35],[1185321600000,4255.46],[1185408000000,4303.19],[1185494400000,4307.14],[1185753600000,4410.3],[1185840000000,4460.56],[1185926400000,4290.48],[1186012800000,4436.19],[1186099200000,4598.38],[1186358400000,4703.98],[1186444800000,4724.55],[1186531200000,4668.09],[1186617600000,4777.29],[1186704000000,4726.68],[1186963200000,4721.19],[1187049600000,4795.57],[1187136000000,4798.75],[1187222400000,4721.94],[1187308800000,4626.58],[1187568000000,4885.43],[1187654400000,4972.71],[1187740800000,5051.69],[1187827200000,5135.93],[1187913600000,5217.58],[1188172800000,5243.15],[1188259200000,5251.77],[1188345600000,5171.82],[1188432000000,5241.23],[1188518400000,5296.81],[1188777600000,5419.17],[1188864000000,5360.33],[1188950400000,5363.25],[1189036800000,5412.04],[1189123200000,5294.79],[1189382400000,5377.22],[1189468800000,5124.09],[1189555200000,5202.86],[1189641600000,5349.97],[1189728000000,5397.28],[1189987200000,5498.91],[1190073600000,5476.84],[1190160000000,5419.27],[1190246400000,5494.92],[1190332800000,5468.1],[1190592000000,5513.9],[1190678400000,5454.62],[1190764800000,5361.02],[1190851200000,5427.66],[1190937600000,5580.81],[1191801600000,5653.14],[1191888000000,5675.93],[1191974400000,5685.76],[1192060800000,5760.08],[1192147200000,5737.22],[1192406400000,5821.45],[1192492800000,5877.2],[1192579200000,5824.12],[1192665600000,5615.75],[1192752000000,5614.06],[1193011200000,5472.68],[1193097600000,5540.09],[1193184000000,5588.01],[1193270400000,5333.79],[1193356800000,5394.81],[1193616000000,5508.36],[1193702400000,5596.07],[1193788800000,5688.54],[1193875200000,5605.23],[1193961600000,5472.93],[1194220800000,5360.31],[1194307200000,5317.55],[1194393600000,5350.63],[1194480000000,5093.67],[1194566400000,5040.52],[1194825600000,4978.25],[1194912000000,4939.24],[1194998400000,5145.89],[1195084800000,5081.11],[1195171200000,5007.66],[1195430400000,4994.42],[1195516800000,5069.38],[1195603200000,4997.62],[1195689600000,4772.62],[1195776000000,4856.16],[1196035200000,4800.08],[1196121600000,4711.15],[1196208000000,4648.75],[1196294400000,4842.07],[1196380800000,4737.41],[1196640000000,4772.67],[1196726400000,4829.21],[1196812800000,4965.95],[1196899200000,4971.06],[1196985600000,5041.35],[1197244800000,5133.56],[1197331200000,5140.0],[1197417600000,5077.39],[1197504000000,4884.3],[1197590400000,4977.65],[1197849600000,4857.29],[1197936000000,4829.91],[1198022400000,4946.29],[1198108800000,5037.19],[1198195200000,5101.85],[1198454400000,5207.13],[1198540800000,5216.81],[1198627200000,5265.03],[1198713600000,5367.53],[1198800000000,5338.27],[1199232000000,5385.1],[1199318400000,5422.03],[1199404800000,5483.65],[1199664000000,5556.59],[1199750400000,5528.05],[1199836800000,5613.76],[1199923200000,5672.15],[1200009600000,5699.15],[1200268800000,5731.76],[1200355200000,5696.45],[1200441600000,5505.72],[1200528000000,5365.62],[1200614400000,5414.47],[1200873600000,5145.73],[1200960000000,4753.87],[1201046400000,4975.11],[1201132800000,5027.21],[1201219200000,5077.43],[1201478400000,4731.88],[1201564800000,4762.08],[1201651200000,4710.65],[1201737600000,4620.4],[1201824000000,4571.94],[1202083200000,4950.12],[1202169600000,4921.83],[1202860800000,4816.08],[1202947200000,4880.25],[1203033600000,4813.31],[1203292800000,4910.99],[1203379200000,5020.75],[1203465600000,4908.72],[1203552000000,4876.03],[1203638400000,4702.24],[1203897600000,4519.78],[1203984000000,4515.53],[1204070400000,4639.77],[1204156800000,4622.06],[1204243200000,4674.55],[1204502400000,4790.74],[1204588800000,4671.15],[1204675200000,4628.72],[1204761600000,4685.03],[1204848000000,4621.69],[1205107200000,4431.59],[1205193600000,4441.18],[1205280000000,4309.65],[1205366400000,4198.96],[1205452800000,4157.87],[1205712000000,3965.28],[1205798400000,3763.95],[1205884800000,3888.86],[1205971200000,4001.83],[1206057600000,4037.83],[1206316800000,3857.09],[1206403200000,3905.77],[1206489600000,3914.37],[1206576000000,3748.92],[1206662400000,3918.16],[1206921600000,3790.53],[1207008000000,3582.85],[1207094400000,3547.98],[1207180800000,3650.7],[1207526400000,3845.82],[1207612800000,3891.06],[1207699200000,3688.12],[1207785600000,3754.72],[1207872000000,3783.73],[1208131200000,3536.33],[1208217600000,3583.3],[1208304000000,3494.02],[1208390400000,3386.63],[1208476800000,3272.5],[1208736000000,3267.55],[1208822400000,3296.28],[1208908800000,3453.73],[1208995200000,3774.5],[1209081600000,3803.07],[1209340800000,3729.15],[1209427200000,3776.94],[1209513600000,3959.12],[1209945600000,4055.78],[1210032000000,4010.89],[1210118400000,3821.32],[1210204800000,3925.04],[1210291200000,3878.92],[1210550400000,3904.92],[1210636800000,3851.69],[1210723200000,3975.78],[1210809600000,3948.09],[1210896000000,3936.12],[1211155200000,3914.07],[1211241600000,3710.82],[1211328000000,3783.05],[1211414400000,3711.44],[1211500800000,3675.15],[1211760000000,3559.22],[1211846400000,3576.2],[1211932800000,3676.23],[1212019200000,3580.87],[1212105600000,3611.33],[1212364800000,3625.83],[1212451200000,3614.11],[1212537600000,3546.92],[1212624000000,3512.14],[1212710400000,3489.5],[1213056000000,3206.56],[1213142400000,3140.3],[1213228800000,3084.63],[1213315200000,2979.12],[1213574400000,2952.24],[1213660800000,2842.68],[1213747200000,2991.27],[1213833600000,2773.08],[1213920000000,2849.67],[1214179200000,2789.94],[1214265600000,2851.92],[1214352000000,2969.54],[1214438400000,2980.91],[1214524800000,2816.02],[1214784000000,2791.82],[1214870400000,2698.35],[1214956800000,2699.6],[1215043200000,2760.61],[1215129600000,2741.85],[1215388800000,2882.76],[1215475200000,2901.84],[1215561600000,3015.13],[1215648000000,2973.73],[1215734400000,2953.5],[1215993600000,2975.87],[1216080000000,2852.98],[1216166400000,2745.6],[1216252800000,2718.07],[1216339200000,2815.46],[1216598400000,2911.05],[1216684800000,2904.74],[1216771200000,2883.32],[1216857600000,2977.36],[1216944000000,2939.2],[1217203200000,2960.85],[1217289600000,2905.63],[1217376000000,2884.38],[1217462400000,2805.21],[1217548800000,2840.79],[1217808000000,2773.15],[1217894400000,2703.08],[1217980800000,2721.69],[1218067200000,2720.44],[1218153600000,2591.46],[1218412800000,2456.81],[1218499200000,2444.16],[1218585600000,2444.67],[1218672000000,2443.51],[1218758400000,2447.61],[1219017600000,2313.4],[1219104000000,2348.47],[1219190400000,2532.94],[1219276800000,2443.98],[1219363200000,2404.93],[1219622400000,2400.55],[1219708800000,2331.53],[1219795200000,2325.29],[1219881600000,2335.86],[1219968000000,2391.64],[1220227200000,2309.17],[1220313600000,2285.41],[1220400000000,2245.96],[1220486400000,2251.15],[1220572800000,2183.43],[1220832000000,2126.52],[1220918400000,2139.15],[1221004800000,2143.18],[1221091200000,2072.13],[1221177600000,2077.85],[1221523200000,2000.65],[1221609600000,1929.14],[1221696000000,1895.99],[1221782400000,2073.11],[1222041600000,2207.61],[1222128000000,2123.48],[1222214400000,2138.85],[1222300800000,2223.53],[1222387200000,2243.66],[1223251200000,2128.7],[1223337600000,2102.45],[1223424000000,2022.88],[1223510400000,1995.3],[1223596800000,1906.96],[1223856000000,1985.49],[1223942400000,1934.62],[1224028800000,1914.36],[1224115200000,1820.9],[1224201600000,1833.26],[1224460800000,1896.73],[1224547200000,1881.41],[1224633600000,1833.32],[1224720000000,1834.78],[1224806400000,1781.6],[1225065600000,1654.67],[1225152000000,1705.82],[1225238400000,1658.22],[1225324800000,1697.66],[1225411200000,1663.66],[1225670400000,1653.54],[1225756800000,1627.76],[1225843200000,1691.42],[1225929600000,1649.78],[1226016000000,1677.83],[1226275200000,1801.67],[1226361600000,1781.36],[1226448000000,1801.82],[1226534400000,1874.08],[1226620800000,1943.65],[1226880000000,1987.22],[1226966400000,1839.82],[1227052800000,1953.16],[1227139200000,1932.43],[1227225600000,1920.73],[1227484800000,1837.64],[1227571200000,1834.29],[1227657600000,1843.49],[1227744000000,1870.47],[1227830400000,1829.92],[1228089600000,1864.2],[1228176000000,1868.63],[1228262400000,1952.67],[1228348800000,1982.93],[1228435200000,2013.18],[1228694400000,2095.04],[1228780800000,2040.85],[1228867200000,2096.39],[1228953600000,2046.34],[1229040000000,1960.38],[1229299200000,1975.03],[1229385600000,1994.45],[1229472000000,2001.42],[1229558400000,2045.1],[1229644800000,2052.11],[1229904000000,2017.55],[1229990400000,1918.95],[1230076800000,1887.07],[1230163200000,1870.77],[1230249600000,1862.1],[1230508800000,1854.76],[1230595200000,1833.44],[1230681600000,1817.72],[1231113600000,1882.96],[1231200000000,1942.8],[1231286400000,1931.18],[1231372800000,1887.99],[1231459200000,1918.36],[1231718400000,1920.69],[1231804800000,1876.19],[1231891200000,1955.24],[1231977600000,1954.87],[1232064000000,1990.21],[1232323200000,2012.46],[1232409600000,2025.19],[1232496000000,2021.71],[1232582400000,2044.55],[1232668800000,2032.68],[1233532800000,2057.06],[1233619200000,2108.91],[1233705600000,2166.41],[1233792000000,2150.97],[1233878400000,2237.28],[1234137600000,2296.67],[1234224000000,2326.75],[1234310400000,2331.14],[1234396800000,2318.34],[1234483200000,2399.06],[1234742400000,2462.25],[1234828800000,2385.29],[1234915200000,2275.84],[1235001600000,2298.41],[1235088000000,2344.32],[1235347200000,2410.48],[1235433600000,2301.85],[1235520000000,2304.25],[1235606400000,2190.19],[1235692800000,2140.49],[1235952000000,2164.67],[1236038400000,2142.15],[1236124800000,2285.15],[1236211200000,2304.92],[1236297600000,2286.58],[1236556800000,2202.53],[1236643200000,2240.78],[1236729600000,2220.38],[1236816000000,2215.7],[1236902400000,2205.42],[1237161600000,2241.61],[1237248000000,2322.4],[1237334400000,2332.65],[1237420800000,2382.56],[1237507200000,2379.84],[1237766400000,2439.4],[1237852800000,2451.78],[1237939200000,2401.33],[1238025600000,2479.79],[1238112000000,2498.93],[1238371200000,2484.49],[1238457600000,2507.79],[1238544000000,2548.22],[1238630400000,2576.4],[1238716800000,2570.5],[1239062400000,2576.95],[1239148800000,2479.35],[1239235200000,2517.67],[1239321600000,2595.53],[1239580800000,2656.52],[1239667200000,2676.87],[1239753600000,2686.99],[1239840000000,2687.11],[1239926400000,2650.69],[1240185600000,2707.67],[1240272000000,2675.44],[1240358400000,2576.28],[1240444800000,2593.56],[1240531200000,2572.89],[1240790400000,2513.29],[1240876800000,2518.53],[1240963200000,2605.37],[1241049600000,2622.93],[1241395200000,2714.3],[1241481600000,2727.01],[1241568000000,2764.98],[1241654400000,2767.08],[1241740800000,2789.22],[1242000000000,2725.32],[1242086400000,2788.56],[1242172800000,2814.0],[1242259200000,2792.6],[1242345600000,2796.12],[1242604800000,2810.57],[1242691200000,2840.08],[1242777600000,2812.86],[1242864000000,2750.01],[1242950400000,2740.68],[1243209600000,2752.72],[1243296000000,2719.76],[1243382400000,2759.71],[1243814400000,2858.34],[1243900800000,2865.1],[1243987200000,2939.39],[1244073600000,2953.75],[1244160000000,2939.31],[1244419200000,2948.48],[1244505600000,2960.56],[1244592000000,2989.59],[1244678400000,2961.63],[1244764800000,2906.29],[1245024000000,2966.19],[1245110400000,2961.22],[1245196800000,3010.59],[1245283200000,3057.43],[1245369600000,3080.0],[1245628800000,3082.56],[1245715200000,3083.9],[1245801600000,3120.73],[1245888000000,3117.92],[1245974400000,3128.42],[1246233600000,3179.97],[1246320000000,3166.47],[1246406400000,3237.9],[1246492800000,3282.36],[1246579200000,3327.14],[1246838400000,3374.75],[1246924800000,3340.49],[1247011200000,3352.27],[1247097600000,3396.3],[1247184000000,3398.31],[1247443200000,3361.01],[1247529600000,3454.75],[1247616000000,3493.3],[1247702400000,3501.24],[1247788800000,3519.81],[1248048000000,3591.12],[1248134400000,3539.83],[1248220800000,3606.92],[1248307200000,3651.97],[1248393600000,3667.56],[1248652800000,3743.63],[1248739200000,3755.82],[1248825600000,3558.51],[1248912000000,3634.82],[1248998400000,3734.62],[1249257600000,3787.03],[1249344000000,3786.61],[1249430400000,3740.94],[1249516800000,3663.12],[1249603200000,3555.09],[1249862400000,3544.54],[1249948800000,3556.38],[1250035200000,3397.4],[1250121600000,3440.82],[1250208000000,3344.46],[1250467200000,3140.27],[1250553600000,3171.99],[1250640000000,3014.57],[1250726400000,3144.39],[1250812800000,3203.62],[1251072000000,3229.6],[1251158400000,3109.83],[1251244800000,3172.39],[1251331200000,3156.3],[1251417600000,3046.78],[1251676800000,2830.27],[1251763200000,2843.7],[1251849600000,2890.93],[1251936000000,3051.96],[1252022400000,3077.14],[1252281600000,3104.21],[1252368000000,3170.97],[1252454400000,3194.91],[1252540800000,3162.91],[1252627200000,3238.13],[1252886400000,3293.39],[1252972800000,3302.64],[1253059200000,3258.24],[1253145600000,3320.1],[1253232000000,3199.69],[1253491200000,3208.6],[1253577600000,3131.03],[1253664000000,3060.07],[1253750400000,3080.93],[1253836800000,3058.53],[1254096000000,2972.64],[1254182400000,2972.29],[1254268800000,3004.8],[1255046400000,3163.71],[1255305600000,3151.63],[1255392000000,3198.52],[1255478400000,3227.4],[1255564800000,3239.64],[1255651200000,3241.71],[1255910400000,3329.16],[1255996800000,3377.57],[1256083200000,3369.28],[1256169600000,3347.32],[1256256000000,3413.25],[1256515200000,3414.24],[1256601600000,3314.72],[1256688000000,3329.33],[1256774400000,3247.05],[1256860800000,3280.37],[1257120000000,3392.8],[1257206400000,3435.43],[1257292800000,3453.89],[1257379200000,3464.32],[1257465600000,3483.02],[1257724800000,3495.79],[1257811200000,3503.78],[1257897600000,3495.67],[1257984000000,3499.99],[1258070400000,3518.72],[1258329600000,3625.8],[1258416000000,3628.35],[1258502400000,3630.23],[1258588800000,3642.44],[1258675200000,3631.01],[1258934400000,3665.51],[1259020800000,3548.08],[1259107200000,3629.63],[1259193600000,3485.77],[1259280000000,3382.51],[1259539200000,3511.67],[1259625600000,3560.83],[1259712000000,3597.33],[1259798400000,3590.88],[1259884800000,3643.49],[1260144000000,3668.83],[1260230400000,3624.02],[1260316800000,3554.48],[1260403200000,3577.24],[1260489600000,3575.02],[1260748800000,3612.75],[1260835200000,3583.34],[1260921600000,3560.72],[1261008000000,3480.15],[1261094400000,3391.74],[1261353600000,3396.62],[1261440000000,3305.54],[1261526400000,3336.48],[1261612800000,3438.82],[1261699200000,3424.783],[1261958400000,3478.433],[1262044800000,3500.737],[1262131200000,3559.287],[1262217600000,3575.395],[1262563200000,3536.221],[1262649600000,3564.038],[1262736000000,3542.209],[1262822400000,3472.313],[1262908800000,3480.403],[1263168000000,3482.68],[1263254400000,3535.407],[1263340800000,3421.116],[1263427200000,3469.616],[1263513600000,3483.312],[1263772800000,3501.258],[1263859200000,3507.877],[1263945600000,3395.435],[1264032000000,3409.299],[1264118400000,3366.717],[1264377600000,3327.992],[1264464000000,3243.149],[1264550400000,3198.868],[1264636800000,3207.379],[1264723200000,3204.912],[1264982400000,3152.904],[1265068800000,3146.313],[1265155200000,3230.806],[1265241600000,3219.349],[1265328000000,3153.272],[1265587200000,3151.52],[1265673600000,3169.058],[1265760000000,3214.138],[1265846400000,3220.262],[1265932800000,3251.044],[1266796800000,3234.015],[1266883200000,3198.521],[1266969600000,3244.769],[1267056000000,3292.06],[1267142400000,3282.143],[1267401600000,3324.825],[1267488000000,3311.53],[1267574400000,3335.654],[1267660800000,3250.573],[1267747200000,3260.271],[1268006400000,3286.144],[1268092800000,3306.176],[1268179200000,3280.317],[1268265600000,3277.335],[1268352000000,3233.824],[1268611200000,3183.276],[1268697600000,3203.66],[1268784000000,3273.779],[1268870400000,3267.814],[1268956800000,3293.292],[1269216000000,3302.439],[1269302400000,3275.928],[1269388800000,3277.087],[1269475200000,3229.156],[1269561600000,3274.941],[1269820800000,3358.819],[1269907200000,3367.207],[1269993600000,3345.844],[1270080000000,3391.905],[1270166400000,3407.281],[1270512000000,3405.442],[1270598400000,3387.705],[1270684800000,3346.409],[1270771200000,3379.365],[1271030400000,3351.478],[1271116800000,3391.716],[1271203200000,3404.24],[1271289600000,3394.803],[1271376000000,3357.22],[1271635200000,3177.299],[1271721600000,3173.591],[1271808000000,3236.83],[1271894400000,3202.314],[1271980800000,3190.308],[1272240000000,3172.736],[1272326400000,3108.961],[1272412800000,3097.8],[1272499200000,3060.768],[1272585600000,3067.852],[1272931200000,3019.449],[1273017600000,3036.564],[1273104000000,2897.549],[1273190400000,2836.781],[1273449600000,2858.403],[1273536000000,2801.028],[1273622400000,2818.46],[1273708800000,2886.602],[1273795200000,2868.24],[1274054400000,2714.7],[1274140800000,2770.23],[1274227200000,2762.678],[1274313600000,2726.422],[1274400000000,2769.094],[1274659200000,2873.771],[1274745600000,2814.703],[1274832000000,2814.05],[1274918400000,2859.866],[1275004800000,2850.66],[1275264000000,2773.637],[1275350400000,2744.703],[1275436800000,2757.914],[1275523200000,2736.58],[1275609600000,2744.623],[1275868800000,2696.27],[1275955200000,2699.687],[1276041600000,2782.163],[1276128000000,2750.22],[1276214400000,2759.29],[1276732800000,2743.275],[1276819200000,2696.877],[1277078400000,2780.27],[1277164800000,2784.0],[1277251200000,2758.546],[1277337600000,2759.278],[1277424000000,2736.491],[1277683200000,2717.51],[1277769600000,2592.046],[1277856000000,2562.554],[1277942400000,2526.48],[1278028800000,2533.52],[1278288000000,2513.122],[1278374400000,2562.41],[1278460800000,2580.608],[1278547200000,2575.557],[1278633600000,2647.174],[1278892800000,2676.22],[1278979200000,2634.829],[1279065600000,2653.609],[1279152000000,2608.828],[1279238400000,2616.631],[1279497600000,2682.472],[1279584000000,2741.628],[1279670400000,2747.123],[1279756800000,2781.079],[1279843200000,2793.21],[1280102400000,2811.179],[1280188800000,2795.959],[1280275200000,2864.004],[1280361600000,2877.579],[1280448000000,2868.442],[1280707200000,2916.809],[1280793600000,2865.761],[1280880000000,2876.403],[1280966400000,2850.829],[1281052800000,2897.876],[1281312000000,2918.42],[1281398400000,2832.689],[1281484800000,2850.735],[1281571200000,2816.983],[1281657600000,2855.716],[1281916800000,2921.814],[1282003200000,2942.632],[1282089600000,2937.362],[1282176000000,2955.654],[1282262400000,2898.577],[1282521600000,2896.861],[1282608000000,2912.328],[1282694400000,2843.023],[1282780800000,2850.09],[1282867200000,2858.57],[1283126400000,2915.01],[1283212800000,2903.19],[1283299200000,2884.04],[1283385600000,2921.39],[1283472000000,2920.21],[1283731200000,2975.09],[1283817600000,2983.11],[1283904000000,2980.97],[1283990400000,2926.46],[1284076800000,2932.55],[1284336000000,2962.32],[1284422400000,2965.01],[1284508800000,2913.19],[1284595200000,2857.79],[1284681600000,2861.37],[1284940800000,2849.83],[1285027200000,2857.48],[1285545600000,2905.03],[1285632000000,2880.91],[1285718400000,2874.81],[1285804800000,2935.57],[1286496000000,3044.23],[1286755200000,3132.9],[1286841600000,3172.73],[1286928000000,3217.58],[1287014400000,3224.14],[1287100800000,3327.68],[1287360000000,3306.16],[1287446400000,3375.67],[1287532800000,3396.88],[1287619200000,3374.69],[1287705600000,3378.66],[1287964800000,3481.08],[1288051200000,3466.08],[1288137600000,3403.87],[1288224000000,3397.69],[1288310400000,3379.98],[1288569600000,3473.0],[1288656000000,3463.13],[1288742400000,3420.34],[1288828800000,3480.5],[1288915200000,3520.8],[1289174400000,3548.57],[1289260800000,3523.95],[1289347200000,3499.11],[1289433600000,3509.98],[1289520000000,3291.83],[1289779200000,3314.89],[1289865600000,3169.54],[1289952000000,3103.91],[1290038400000,3147.96],[1290124800000,3178.85],[1290384000000,3171.94],[1290470400000,3107.18],[1290556800000,3177.04],[1290643200000,3223.48],[1290729600000,3194.85],[1290988800000,3190.05],[1291075200000,3136.99],[1291161600000,3136.02],[1291248000000,3155.06],[1291334400000,3158.16],[1291593600000,3165.57],[1291680000000,3200.34],[1291766400000,3171.88],[1291852800000,3123.37],[1291939200000,3161.98],[1292198400000,3261.06],[1292284800000,3269.47],[1292371200000,3247.64],[1292457600000,3230.67],[1292544000000,3225.66],[1292803200000,3178.66],[1292889600000,3249.51],[1292976000000,3215.45],[1293062400000,3188.61],[1293148800000,3162.96],[1293408000000,3099.71],[1293494400000,3044.93],[1293580800000,3061.83],[1293667200000,3064.1],[1293753600000,3128.26],[1294099200000,3189.68],[1294185600000,3175.66],[1294272000000,3159.64],[1294358400000,3166.62],[1294617600000,3108.19],[1294704000000,3124.92],[1294790400000,3142.34],[1294876800000,3141.28],[1294963200000,3091.86],[1295222400000,2974.35],[1295308800000,2977.65],[1295395200000,3044.85],[1295481600000,2944.71],[1295568000000,2983.46],[1295827200000,2954.23],[1295913600000,2938.65],[1296000000000,2978.43],[1296086400000,3026.47],[1296172800000,3036.74],[1296432000000,3076.51],[1296518400000,3077.28],[1297209600000,3040.95],[1297296000000,3104.16],[1297382400000,3120.96],[1297641600000,3219.14],[1297728000000,3217.67],[1297814400000,3248.53],[1297900800000,3245.91],[1297987200000,3211.88],[1298246400000,3257.91],[1298332800000,3163.58],[1298419200000,3174.74],[1298505600000,3190.94],[1298592000000,3197.62],[1298851200000,3239.56],[1298937600000,3254.89],[1299024000000,3243.3],[1299110400000,3221.72],[1299196800000,3270.67],[1299456000000,3334.51],[1299542400000,3337.46],[1299628800000,3338.86],[1299715200000,3280.26],[1299801600000,3247.38],[1300060800000,3262.92],[1300147200000,3203.96],[1300233600000,3248.2],[1300320000000,3197.1],[1300406400000,3215.69],[1300665600000,3207.11],[1300752000000,3222.96],[1300838400000,3264.93],[1300924800000,3251.36],[1301011200000,3294.48],[1301270400000,3290.57],[1301356800000,3257.98],[1301443200000,3256.08],[1301529600000,3223.29],[1301616000000,3272.73],[1302048000000,3311.07],[1302134400000,3324.42],[1302220800000,3353.36],[1302480000000,3333.43],[1302566400000,3326.77],[1302652800000,3372.03],[1302739200000,3353.56],[1302825600000,3358.94],[1303084800000,3359.44],[1303171200000,3295.81],[1303257600000,3295.76],[1303344000000,3317.37],[1303430400000,3299.94],[1303689600000,3249.57],[1303776000000,3230.96],[1303862400000,3209.5],[1303948800000,3161.78],[1304035200000,3192.72],[1304380800000,3211.13],[1304467200000,3129.03],[1304553600000,3126.12],[1304640000000,3121.4],[1304899200000,3129.76],[1304985600000,3153.22],[1305072000000,3145.08],[1305158400000,3101.6],[1305244800000,3128.09],[1305504000000,3100.46],[1305590400000,3116.03],[1305676800000,3139.38],[1305763200000,3120.64],[1305849600000,3121.6],[1306108800000,3022.98],[1306195200000,3026.22],[1306281600000,2990.34],[1306368000000,2978.38],[1306454400000,2963.31],[1306713600000,2954.51],[1306800000000,3001.56],[1306886400000,3004.17],[1306972800000,2955.71],[1307059200000,2986.35],[1307404800000,3004.26],[1307491200000,3008.65],[1307577600000,2951.89],[1307664000000,2961.93],[1307923200000,2950.35],[1308009600000,2993.56],[1308096000000,2963.12],[1308182400000,2917.58],[1308268800000,2892.16],[1308528000000,2874.9],[1308614400000,2909.07],[1308700800000,2908.58],[1308787200000,2957.63],[1308873600000,3027.47],[1309132800000,3036.49],[1309219200000,3041.73],[1309305600000,3000.17],[1309392000000,3044.09],[1309478400000,3049.75],[1309737600000,3121.98],[1309824000000,3122.5],[1309910400000,3113.71],[1309996800000,3101.68],[1310083200000,3109.18],[1310342400000,3113.21],[1310428800000,3056.91],[1310515200000,3106.25],[1310601600000,3115.75],[1310688000000,3128.89],[1310947200000,3122.6],[1311033600000,3095.12],[1311120000000,3091.57],[1311206400000,3059.14],[1311292800000,3067.99],[1311552000000,2968.29],[1311638400000,2977.77],[1311724800000,3000.05],[1311811200000,2981.0],[1311897600000,2972.08],[1312156800000,2977.72],[1312243200000,2956.38],[1312329600000,2954.87],[1312416000000,2960.31],[1312502400000,2897.42],[1312761600000,2793.9],[1312848000000,2798.19],[1312934400000,2824.12],[1313020800000,2866.92],[1313107200000,2875.36],[1313366400000,2917.88],[1313452800000,2897.58],[1313539200000,2886.01],[1313625600000,2834.25],[1313712000000,2807.66],[1313971200000,2777.79],[1314057600000,2821.0],[1314144000000,2810.02],[1314230400000,2903.84],[1314316800000,2901.22],[1314576000000,2852.81],[1314662400000,2841.74],[1314748800000,2846.78],[1314835200000,2834.53],[1314921600000,2803.85],[1315180800000,2743.82],[1315267200000,2723.3],[1315353600000,2779.09],[1315440000000,2756.1],[1315526400000,2751.09],[1315872000000,2720.28],[1315958400000,2733.11],[1316044800000,2729.05],[1316131200000,2733.99],[1316390400000,2679.27],[1316476800000,2689.84],[1316563200000,2771.01],[1316649600000,2685.69],[1316736000000,2669.48],[1316995200000,2610.92],[1317081600000,2637.88],[1317168000000,2610.59],[1317254400000,2588.19],[1317340800000,2581.35],[1318204800000,2557.08],[1318291200000,2551.99],[1318377600000,2644.76],[1318464000000,2662.6],[1318550400000,2653.78],[1318809600000,2666.95],[1318896000000,2592.21],[1318982400000,2583.08],[1319068800000,2520.53],[1319155200000,2507.88],[1319414400000,2576.67],[1319500800000,2625.43],[1319587200000,2651.65],[1319673600000,2657.48],[1319760000000,2709.02],[1320019200000,2695.31],[1320105600000,2697.53],[1320192000000,2742.39],[1320278400000,2744.3],[1320364800000,2763.75],[1320624000000,2736.25],[1320710400000,2727.71],[1320796800000,2751.65],[1320883200000,2699.59],[1320969600000,2695.0],[1321228800000,2750.2],[1321315200000,2744.68],[1321401600000,2670.12],[1321488000000,2662.02],[1321574400000,2606.5],[1321833600000,2609.69],[1321920000000,2609.48],[1322006400000,2584.01],[1322092800000,2588.92],[1322179200000,2569.97],[1322438400000,2573.32],[1322524800000,2608.57],[1322611200000,2521.52],[1322697600000,2583.61],[1322784000000,2557.06],[1323043200000,2520.96],[1323129600000,2516.34],[1323216000000,2528.23],[1323302400000,2525.0],[1323388800000,2503.46],[1323648000000,2477.69],[1323734400000,2442.5],[1323820800000,2401.57],[1323907200000,2353.5],[1323993600000,2390.13],[1324252800000,2384.41],[1324339200000,2377.07],[1324425600000,2339.11],[1324512000000,2341.33],[1324598400000,2359.16],[1324857600000,2335.7],[1324944000000,2305.03],[1325030400000,2307.93],[1325116800000,2311.36],[1325203200000,2345.74],[1325635200000,2298.75],[1325721600000,2276.39],[1325808000000,2290.6],[1326067200000,2368.57],[1326153600000,2447.35],[1326240000000,2435.61],[1326326400000,2435.22],[1326412800000,2394.33],[1326672000000,2345.65],[1326758400000,2460.6],[1326844800000,2422.19],[1326931200000,2468.35],[1327017600000,2504.09],[1327881600000,2460.72],[1327968000000,2464.26],[1328054400000,2428.99],[1328140800000,2486.24],[1328227200000,2506.09],[1328486400000,2504.32],[1328572800000,2457.95],[1328659200000,2528.24],[1328745600000,2529.23],[1328832000000,2533.62],[1329091200000,2531.98],[1329177600000,2522.11],[1329264000000,2549.61],[1329350400000,2536.07],[1329436800000,2537.09],[1329696000000,2540.71],[1329782400000,2562.45],[1329868800000,2597.48],[1329955200000,2606.26],[1330041600000,2648.02],[1330300800000,2656.57],[1330387200000,2662.46],[1330473600000,2634.14],[1330560000000,2633.34],[1330646400000,2679.93],[1330905600000,2662.7],[1330992000000,2621.05],[1331078400000,2603.0],[1331164800000,2635.79],[1331251200000,2664.3],[1331510400000,2654.4],[1331596800000,2681.07],[1331683200000,2605.11],[1331769600000,2585.55],[1331856000000,2623.52],[1332115200000,2630.01],[1332201600000,2584.45],[1332288000000,2587.79],[1332374400000,2583.75],[1332460800000,2552.94],[1332720000000,2555.44],[1332806400000,2547.14],[1332892800000,2474.9],[1332979200000,2443.12],[1333065600000,2454.9],[1333584000000,2512.83],[1333670400000,2519.83],[1333929600000,2495.15],[1334016000000,2519.79],[1334102400000,2520.04],[1334188800000,2570.44],[1334275200000,2580.45],[1334534400000,2574.04],[1334620800000,2541.88],[1334707200000,2599.91],[1334793600000,2596.06],[1334880000000,2626.84],[1335139200000,2606.04],[1335225600000,2604.87],[1335312000000,2625.99],[1335398400000,2631.49],[1335484800000,2626.16],[1335916800000,2683.49],[1336003200000,2691.52],[1336089600000,2715.88],[1336348800000,2717.78],[1336435200000,2709.12],[1336521600000,2657.51],[1336608000000,2657.21],[1336694400000,2636.92],[1336953600000,2615.53],[1337040000000,2617.37],[1337126400000,2574.65],[1337212800000,2613.94],[1337299200000,2573.98],[1337558400000,2587.23],[1337644800000,2627.52],[1337731200000,2616.87],[1337817600000,2595.26],[1337904000000,2573.1],[1338163200000,2614.69],[1338249600000,2650.85],[1338336000000,2642.26],[1338422400000,2632.04],[1338508800000,2633.0],[1338768000000,2559.03],[1338854400000,2558.84],[1338940800000,2557.4],[1339027200000,2542.18],[1339113600000,2524.33],[1339372800000,2558.26],[1339459200000,2540.18],[1339545600000,2580.64],[1339632000000,2560.42],[1339718400000,2568.05],[1339977600000,2581.21],[1340064000000,2558.62],[1340150400000,2552.61],[1340236800000,2512.19],[1340582400000,2456.52],[1340668800000,2454.92],[1340755200000,2447.2],[1340841600000,2425.73],[1340928000000,2461.61],[1341187200000,2465.24],[1341273600000,2468.72],[1341360000000,2464.92],[1341446400000,2430.37],[1341532800000,2472.61],[1341792000000,2416.04],[1341878400000,2406.71],[1341964800000,2425.57],[1342051200000,2449.18],[1342137600000,2450.63],[1342396800000,2399.73],[1342483200000,2414.2],[1342569600000,2414.33],[1342656000000,2424.32],[1342742400000,2398.46],[1343001600000,2365.43],[1343088000000,2375.99],[1343174400000,2360.08],[1343260800000,2347.49],[1343347200000,2349.11],[1343606400000,2335.79],[1343692800000,2332.92],[1343779200000,2358.65],[1343865600000,2334.88],[1343952000000,2353.74],[1344211200000,2385.61],[1344297600000,2388.87],[1344384000000,2389.79],[1344470400000,2411.7],[1344556800000,2399.75],[1344816000000,2351.93],[1344902400000,2357.02],[1344988800000,2331.61],[1345075200000,2319.67],[1345161600000,2313.48],[1345420800000,2301.79],[1345507200000,2313.7],[1345593600000,2295.59],[1345680000000,2302.2],[1345766400000,2275.68],[1346025600000,2228.2],[1346112000000,2238.41],[1346198400000,2214.81],[1346284800000,2211.37],[1346371200000,2204.87],[1346630400000,2228.37],[1346716800000,2204.41],[1346803200000,2199.88],[1346889600000,2217.82],[1346976000000,2317.18],[1347235200000,2326.67],[1347321600000,2311.89],[1347408000000,2320.07],[1347494400000,2298.46],[1347580800000,2315.54],[1347840000000,2258.71],[1347926400000,2235.24],[1348012800000,2246.24],[1348099200000,2195.95],[1348185600000,2199.06],[1348444800000,2215.52],[1348531200000,2210.15],[1348617600000,2184.89],[1348704000000,2251.72],[1348790400000,2293.11],[1349654400000,2270.05],[1349740800000,2320.16],[1349827200000,2324.12],[1349913600000,2302.53],[1350000000000,2304.53],[1350259200000,2294.86],[1350345600000,2298.16],[1350432000000,2300.8],[1350518400000,2336.08],[1350604800000,2332.47],[1350864000000,2341.59],[1350950400000,2312.08],[1351036800000,2307.78],[1351123200000,2291.24],[1351209600000,2247.91],[1351468800000,2235.85],[1351555200000,2239.88],[1351641600000,2254.82],[1351728000000,2297.88],[1351814400000,2306.77],[1352073600000,2301.88],[1352160000000,2292.21],[1352246400000,2287.5],[1352332800000,2245.41],[1352419200000,2240.92],[1352678400000,2251.85],[1352764800000,2212.44],[1352851200000,2223.11],[1352937600000,2193.62],[1353024000000,2177.24],[1353283200000,2174.99],[1353369600000,2164.88],[1353456000000,2194.9],[1353542400000,2177.55],[1353628800000,2192.68],[1353888000000,2175.6],[1353974400000,2150.64],[1354060800000,2129.16],[1354147200000,2115.68],[1354233600000,2139.66],[1354492800000,2108.85],[1354579200000,2131.47],[1354665600000,2207.88],[1354752000000,2203.6],[1354838400000,2246.76],[1355097600000,2271.05],[1355184000000,2258.5],[1355270400000,2267.77],[1355356800000,2242.64],[1355443200000,2355.86],[1355702400000,2366.7],[1355788800000,2368.12],[1355875200000,2371.11],[1355961600000,2384.82],[1356048000000,2372.0],[1356307200000,2381.22],[1356393600000,2448.4],[1356480000000,2457.62],[1356566400000,2444.59],[1356652800000,2480.05],[1356912000000,2522.95],[1357257600000,2524.41],[1357516800000,2535.99],[1357603200000,2525.33],[1357689600000,2526.13],[1357776000000,2530.57],[1357862400000,2483.23],[1358121600000,2577.73],[1358208000000,2595.86],[1358294400000,2577.09],[1358380800000,2552.76],[1358467200000,2595.44],[1358726400000,2610.9],[1358812800000,2596.9],[1358899200000,2607.46],[1358985600000,2582.75],[1359072000000,2571.67],[1359331200000,2651.86],[1359417600000,2675.87],[1359504000000,2688.71],[1359590400000,2686.88],[1359676800000,2743.32],[1359936000000,2748.03],[1360022400000,2771.68],[1360108800000,2775.84],[1360195200000,2759.87],[1360281600000,2771.73],[1361145600000,2737.47],[1361232000000,2685.61],[1361318400000,2702.64],[1361404800000,2610.55],[1361491200000,2596.6],[1361750400000,2604.96],[1361836800000,2567.59],[1361923200000,2594.68],[1362009600000,2673.33],[1362096000000,2668.84],[1362355200000,2545.72],[1362441600000,2622.81],[1362528000000,2650.2],[1362614400000,2619.48],[1362700800000,2606.93],[1362960000000,2592.37],[1363046400000,2555.61],[1363132800000,2527.49],[1363219200000,2534.27],[1363305600000,2539.87],[1363564800000,2502.49],[1363651200000,2525.1],[1363737600000,2610.17],[1363824000000,2614.99],[1363910400000,2618.31],[1364169600000,2613.1],[1364256000000,2575.05],[1364342400000,2583.53],[1364428800000,2499.3],[1364515200000,2495.08],[1364774400000,2493.19],[1364860800000,2486.39],[1364947200000,2483.55],[1365379200000,2472.3],[1365465600000,2489.43],[1365552000000,2485.31],[1365638400000,2477.88],[1365724800000,2462.11],[1365984000000,2436.82],[1366070400000,2459.59],[1366156800000,2458.47],[1366243200000,2464.85],[1366329600000,2533.83],[1366588800000,2530.67],[1366675200000,2449.47],[1366761600000,2495.58],[1366848000000,2467.88],[1366934400000,2447.31],[1367452800000,2449.64],[1367539200000,2492.91],[1367798400000,2525.98],[1367884800000,2529.94],[1367971200000,2542.8],[1368057600000,2527.79],[1368144000000,2540.84],[1368403200000,2530.77],[1368489600000,2493.34],[1368576000000,2506.93],[1368662400000,2552.71],[1368748800000,2592.05],[1369008000000,2609.61],[1369094400000,2614.85],[1369180800000,2618.03],[1369267200000,2582.85],[1369353600000,2597.23],[1369612800000,2599.59],[1369699200000,2644.36],[1369785600000,2642.56],[1369872000000,2634.32],[1369958400000,2606.43],[1370217600000,2602.62],[1370304000000,2565.67],[1370390400000,2560.54],[1370476800000,2527.84],[1370563200000,2484.16],[1371081600000,2399.94],[1371168000000,2416.77],[1371427200000,2403.84],[1371513600000,2418.75],[1371600000000,2400.76],[1371686400000,2321.47],[1371772800000,2317.39],[1372032000000,2171.21],[1372118400000,2165.42],[1372204800000,2168.3],[1372291200000,2160.74],[1372377600000,2200.64],[1372636800000,2213.32],[1372723200000,2221.98],[1372809600000,2203.83],[1372896000000,2221.98],[1372982400000,2226.85],[1373241600000,2163.62],[1373328000000,2162.67],[1373414400000,2224.06],[1373500800000,2326.69],[1373587200000,2275.37],[1373846400000,2307.3],[1373932800000,2317.85],[1374019200000,2282.84],[1374105600000,2245.33],[1374192000000,2190.48],[1374451200000,2202.19],[1374537600000,2265.84],[1374624000000,2249.15],[1374710400000,2237.68],[1374796800000,2224.01],[1375056000000,2175.97],[1375142400000,2189.39],[1375228800000,2193.02],[1375315200000,2245.36],[1375401600000,2247.26],[1375660800000,2278.33],[1375747200000,2293.64],[1375833600000,2280.62],[1375920000000,2276.78],[1376006400000,2286.01],[1376265600000,2352.79],[1376352000000,2359.07],[1376438400000,2349.08],[1376524800000,2321.58],[1376611200000,2304.14],[1376870400000,2331.43],[1376956800000,2312.47],[1377043200000,2308.59],[1377129600000,2303.93],[1377216000000,2286.93],[1377475200000,2335.62],[1377561600000,2340.88],[1377648000000,2328.06],[1377734400000,2318.31],[1377820800000,2313.91],[1378080000000,2320.34],[1378166400000,2354.5],[1378252800000,2350.7],[1378339200000,2341.74],[1378425600000,2357.78],[1378684800000,2440.61],[1378771200000,2474.89],[1378857600000,2482.89],[1378944000000,2507.46],[1379030400000,2488.9],[1379289600000,2478.39],[1379376000000,2427.32],[1379462400000,2432.51],[1379894400000,2472.29],[1379980800000,2443.89],[1380067200000,2429.03],[1380153600000,2384.44],[1380240000000,2394.97],[1380499200000,2409.04],[1381190400000,2441.81],[1381276800000,2453.58],[1381363200000,2429.32],[1381449600000,2468.51],[1381708800000,2472.54],[1381795200000,2467.52],[1381881600000,2421.37],[1381968000000,2413.33],[1382054400000,2426.05],[1382313600000,2471.32],[1382400000000,2445.89],[1382486400000,2418.49],[1382572800000,2400.51],[1382659200000,2368.56],[1382918400000,2365.95],[1383004800000,2372.05],[1383091200000,2407.47],[1383177600000,2373.72],[1383264000000,2384.96],[1383523200000,2380.45],[1383609600000,2383.77],[1383696000000,2353.57],[1383782400000,2340.55],[1383868800000,2307.95],[1384128000000,2315.89],[1384214400000,2340.0],[1384300800000,2288.12],[1384387200000,2304.5],[1384473600000,2350.73],[1384732800000,2428.9],[1384819200000,2412.16],[1384905600000,2424.85],[1384992000000,2409.99],[1385078400000,2397.96],[1385337600000,2388.63],[1385424000000,2387.42],[1385510400000,2414.48],[1385596800000,2439.53],[1385683200000,2438.94],[1385942400000,2418.79],[1386028800000,2442.78],[1386115200000,2475.14],[1386201600000,2468.2],[1386288000000,2452.29],[1386547200000,2450.87],[1386633600000,2453.32],[1386720000000,2412.76],[1386806400000,2410.01],[1386892800000,2406.64],[1387152000000,2367.92],[1387238400000,2356.38],[1387324800000,2357.23],[1387411200000,2332.41],[1387497600000,2278.14],[1387756800000,2284.6],[1387843200000,2288.25],[1387929600000,2305.11],[1388016000000,2265.33],[1388102400000,2303.48],[1388361600000,2299.46],[1388448000000,2330.03],[1388620800000,2321.98],[1388707200000,2290.78],[1388966400000,2238.64],[1389052800000,2238.0],[1389139200000,2241.91],[1389225600000,2222.22],[1389312000000,2204.85],[1389571200000,2193.68],[1389657600000,2212.85],[1389744000000,2208.94],[1389830400000,2211.84],[1389916800000,2178.49],[1390176000000,2165.99],[1390262400000,2187.41],[1390348800000,2243.8],[1390435200000,2231.89],[1390521600000,2245.68],[1390780800000,2215.92],[1390867200000,2219.85],[1390953600000,2227.78],[1391040000000,2202.45],[1391731200000,2212.48],[1391990400000,2267.53],[1392076800000,2285.56],[1392163200000,2291.25],[1392249600000,2279.55],[1392336000000,2295.57],[1392595200000,2311.65],[1392681600000,2282.44],[1392768000000,2308.66],[1392854400000,2287.44],[1392940800000,2264.29],[1393200000000,2214.51],[1393286400000,2157.91],[1393372800000,2163.41],[1393459200000,2154.11],[1393545600000,2178.97],[1393804800000,2190.37],[1393891200000,2184.27],[1393977600000,2163.98],[1394064000000,2173.63],[1394150400000,2168.36],[1394409600000,2097.79],[1394496000000,2108.66],[1394582400000,2114.13],[1394668800000,2140.33],[1394755200000,2122.84],[1395014400000,2143.04],[1395100800000,2138.13],[1395187200000,2120.87],[1395273600000,2086.97],[1395360000000,2158.8],[1395619200000,2176.55],[1395705600000,2174.44],[1395792000000,2171.05],[1395878400000,2155.71],[1395964800000,2151.97],[1396224000000,2146.3],[1396310400000,2163.11],[1396396800000,2180.73],[1396483200000,2165.01],[1396569600000,2185.47],[1396915200000,2237.32],[1397001600000,2238.62],[1397088000000,2273.76],[1397174400000,2270.67],[1397433600000,2268.61],[1397520000000,2229.46],[1397606400000,2232.53],[1397692800000,2224.8],[1397779200000,2224.48],[1398038400000,2187.25],[1398124800000,2196.79],[1398211200000,2193.95],[1398297600000,2190.47],[1398384000000,2167.83],[1398643200000,2134.97],[1398729600000,2158.47],[1398816000000,2158.66],[1399248000000,2156.47],[1399334400000,2157.33],[1399420800000,2137.32],[1399507200000,2135.5],[1399593600000,2133.91],[1399852800000,2180.05],[1399939200000,2174.85],[1400025600000,2172.37],[1400112000000,2144.08],[1400198400000,2145.95],[1400457600000,2115.14],[1400544000000,2115.77],[1400630400000,2135.91],[1400716800000,2130.87],[1400803200000,2148.41],[1401062400000,2155.98],[1401148800000,2147.28],[1401235200000,2169.35],[1401321600000,2155.16],[1401408000000,2156.46],[1401753600000,2149.92],[1401840000000,2128.27],[1401926400000,2150.6],[1402012800000,2134.72],[1402272000000,2134.28],[1402358400000,2161.27],[1402444800000,2160.77],[1402531200000,2153.41],[1402617600000,2176.24],[1402876800000,2191.85],[1402963200000,2169.67],[1403049600000,2160.24],[1403136000000,2126.91],[1403222400000,2136.73],[1403481600000,2134.11],[1403568000000,2144.82],[1403654400000,2133.37],[1403740800000,2149.08],[1403827200000,2150.26],[1404086400000,2165.12],[1404172800000,2164.56],[1404259200000,2170.87],[1404345600000,2180.19],[1404432000000,2178.7],[1404691200000,2176.29],[1404777600000,2180.47],[1404864000000,2148.71],[1404950400000,2142.85],[1405036800000,2148.01],[1405296000000,2171.76],[1405382400000,2174.98],[1405468800000,2170.87],[1405555200000,2157.07],[1405641600000,2164.14],[1405900800000,2166.29],[1405987200000,2192.7],[1406073600000,2197.83],[1406160000000,2237.01],[1406246400000,2260.45],[1406505600000,2323.9],[1406592000000,2331.37],[1406678400000,2322.01],[1406764800000,2350.25],[1406851200000,2329.4],[1407110400000,2375.62],[1407196800000,2369.35],[1407283200000,2363.22],[1407369600000,2327.46],[1407456000000,2331.13],[1407715200000,2365.35],[1407801600000,2357.05],[1407888000000,2358.9],[1407974400000,2335.95],[1408060800000,2360.64],[1408320000000,2374.56],[1408406400000,2374.77],[1408492800000,2366.14],[1408579200000,2354.24],[1408665600000,2365.36],[1408924800000,2342.86],[1409011200000,2324.09],[1409097600000,2327.59],[1409184000000,2311.28],[1409270400000,2338.29],[1409529600000,2355.32],[1409616000000,2386.46],[1409702400000,2408.84],[1409788800000,2426.22],[1409875200000,2449.26],[1410220800000,2445.22],[1410307200000,2432.43],[1410393600000,2423.45],[1410480000000,2438.36],[1410739200000,2437.19],[1410825600000,2388.76],[1410912000000,2401.33],[1410998400000,2408.66],[1411084800000,2425.21],[1411344000000,2378.92],[1411430400000,2399.46],[1411516800000,2441.86],[1411603200000,2436.97],[1411689600000,2437.2],[1411948800000,2447.8],[1412035200000,2450.99],[1412726400000,2478.38],[1412812800000,2481.96],[1412899200000,2466.79],[1413158400000,2454.95],[1413244800000,2446.56],[1413331200000,2463.87],[1413417600000,2444.4],[1413504000000,2441.73],[1413763200000,2454.71],[1413849600000,2433.39],[1413936000000,2418.64],[1414022400000,2395.94],[1414108800000,2390.71],[1414368000000,2368.83],[1414454400000,2416.65],[1414540800000,2451.38],[1414627200000,2468.93],[1414713600000,2508.32],[1414972800000,2512.55],[1415059200000,2513.17],[1415145600000,2503.45],[1415232000000,2506.07],[1415318400000,2502.15],[1415577600000,2565.73],[1415664000000,2558.61],[1415750400000,2594.32],[1415836800000,2579.75],[1415923200000,2581.09],[1416182400000,2567.1],[1416268800000,2541.42],[1416355200000,2537.22],[1416441600000,2537.1],[1416528000000,2583.46],[1416787200000,2649.26],[1416873600000,2685.56],[1416960000000,2723.02],[1417046400000,2754.49],[1417132800000,2808.82],[1417392000000,2819.81],[1417478400000,2923.94],[1417564800000,2967.55],[1417651200000,3104.35],[1417737600000,3124.89],[1417996800000,3252.88],[1418083200000,3106.91],[1418169600000,3221.55],[1418256000000,3183.01],[1418342400000,3193.23],[1418601600000,3217.23],[1418688000000,3303.4],[1418774400000,3360.6],[1418860800000,3345.93],[1418947200000,3383.17],[1419206400000,3394.48],[1419292800000,3324.92],[1419379200000,3230.39],[1419465600000,3335.42],[1419552000000,3445.84],[1419811200000,3455.46],[1419897600000,3457.55],[1419984000000,3533.71],[1420416000000,3641.54],[1420502400000,3641.06],[1420588800000,3643.79],[1420675200000,3559.26],[1420761600000,3546.72],[1421020800000,3513.58],[1421107200000,3514.04],[1421193600000,3502.42],[1421280000000,3604.12],[1421366400000,3635.15],[1421625600000,3355.16],[1421712000000,3396.22],[1421798400000,3548.89],[1421884800000,3567.61],[1421971200000,3571.73],[1422230400000,3607.99],[1422316800000,3574.93],[1422403200000,3525.32],[1422489600000,3481.8],[1422576000000,3434.39],[1422835200000,3353.96],[1422921600000,3437.45],[1423008000000,3401.77],[1423094400000,3366.95],[1423180800000,3312.42],[1423440000000,3345.92],[1423526400000,3406.94],[1423612800000,3434.12],[1423699200000,3442.87],[1423785600000,3469.83],[1424044800000,3499.48],[1424131200000,3522.32],[1424822400000,3478.73],[1424908800000,3566.29],[1424995200000,3572.84],[1425254400000,3601.26],[1425340800000,3507.9],[1425427200000,3530.82],[1425513600000,3496.34],[1425600000000,3478.52],[1425859200000,3537.75],[1425945600000,3520.61],[1426032000000,3524.65],[1426118400000,3592.84],[1426204800000,3617.66],[1426464000000,3705.67],[1426550400000,3757.12],[1426636800000,3846.05],[1426723200000,3839.74],[1426809600000,3892.57],[1427068800000,3972.06],[1427155200000,3973.05],[1427241600000,3940.41],[1427328000000,3950.0],[1427414400000,3971.697],[1427673600000,4088.18],[1427760000000,4051.2],[1427846400000,4123.9],[1427932800000,4124.78],[1428019200000,4170.54],[1428364800000,4260.04],[1428451200000,4295.8],[1428537600000,4262.14],[1428624000000,4344.42],[1428883200000,4421.07],[1428969600000,4438.18],[1429056000000,4380.51],[1429142400000,4513.55],[1429228800000,4596.14],[1429488000000,4521.92],[1429574400000,4619.16],[1429660800000,4739.81],[1429747200000,4740.89],[1429833600000,4702.64],[1430092800000,4807.59],[1430179200000,4741.86],[1430265600000,4774.33],[1430352000000,4749.89],[1430697600000,4787.74],[1430784000000,4596.84],[1430870400000,4553.33],[1430956800000,4470.09],[1431043200000,4558.4],[1431302400000,4690.53],[1431388800000,4747.42],[1431475200000,4718.44],[1431561600000,4700.78],[1431648000000,4617.47],[1431907200000,4575.14],[1431993600000,4731.22],[1432080000000,4754.92],[1432166400000,4840.98],[1432252800000,4951.33],[1432512000000,5099.5],[1432598400000,5198.92],[1432684800000,5181.43],[1432771200000,4834.01],[1432857600000,4840.83],[1433116800000,5076.18],[1433203200000,5161.87],[1433289600000,5143.59],[1433376000000,5181.42],[1433462400000,5230.55],[1433721600000,5353.75],[1433808000000,5317.46],[1433894400000,5309.11],[1433980800000,5306.59],[1434067200000,5335.12],[1434326400000,5221.17],[1434412800000,5064.82],[1434499200000,5138.83],[1434585600000,4930.55],[1434672000000,4637.05],[1435017600000,4786.09],[1435104000000,4880.13],[1435190400000,4706.52],[1435276800000,4336.19],[1435536000000,4191.55],[1435622400000,4473.0],[1435708800000,4253.02],[1435795200000,4108.0],[1435881600000,3885.92],[1436140800000,3998.54],[1436227200000,3928.0],[1436313600000,3663.04],[1436400000000,3897.63],[1436486400000,4106.56],[1436745600000,4211.81],[1436832000000,4112.15],[1436918400000,3966.76],[1437004800000,3997.36],[1437091200000,4151.5],[1437350400000,4160.61],[1437436800000,4166.01],[1437523200000,4157.16],[1437609600000,4250.81],[1437696000000,4176.28],[1437955200000,3818.73],[1438041600000,3811.09],[1438128000000,3930.38],[1438214400000,3815.41],[1438300800000,3816.7],[1438560000000,3829.24],[1438646400000,3948.16],[1438732800000,3866.9],[1438819200000,3831.85],[1438905600000,3906.95],[1439164800000,4084.36],[1439251200000,4066.67],[1439337600000,4016.13],[1439424000000,4075.46],[1439510400000,4073.54],[1439769600000,4077.87],[1439856000000,3825.41],[1439942400000,3886.14],[1440028800000,3761.45],[1440115200000,3589.54],[1440374400000,3275.53],[1440460800000,3042.93],[1440547200000,3025.69],[1440633600000,3205.64],[1440720000000,3342.29],[1440979200000,3366.54],[1441065600000,3362.08],[1441152000000,3365.83],[1441584000000,3250.49],[1441670400000,3334.02],[1441756800000,3399.3],[1441843200000,3357.56],[1441929600000,3347.19],[1442188800000,3281.13],[1442275200000,3152.23],[1442361600000,3309.25],[1442448000000,3237.0],[1442534400000,3251.27],[1442793600000,3308.25],[1442880000000,3339.03],[1442966400000,3263.03],[1443052800000,3285.0],[1443139200000,3231.95],[1443398400000,3242.75],[1443484800000,3178.85],[1443571200000,3202.95],[1444262400000,3296.48],[1444348800000,3340.12],[1444608000000,3447.69],[1444694400000,3445.04],[1444780800000,3406.11],[1444867200000,3486.81],[1444953600000,3534.06],[1445212800000,3534.18],[1445299200000,3577.7],[1445385600000,3473.25],[1445472000000,3524.53],[1445558400000,3571.24],[1445817600000,3589.26],[1445904000000,3592.88],[1445990400000,3524.92],[1446076800000,3533.31],[1446163200000,3534.08],[1446422400000,3475.96],[1446508800000,3465.49],[1446595200000,3628.54],[1446681600000,3705.97],[1446768000000,3793.37],[1447027200000,3840.35],[1447113600000,3833.24],[1447200000000,3833.65],[1447286400000,3795.32],[1447372800000,3746.24],[1447632000000,3764.13],[1447718400000,3758.39],[1447804800000,3715.58],[1447891200000,3774.97],[1447977600000,3774.38],[1448236800000,3753.33],[1448323200000,3753.89],[1448409600000,3781.61],[1448496000000,3759.43],[1448582400000,3556.99],[1448841600000,3566.41],[1448928000000,3591.7],[1449014400000,3721.96],[1449100800000,3749.3],[1449187200000,3677.59],[1449446400000,3687.61],[1449532800000,3623.02],[1449619200000,3635.94],[1449705600000,3623.08],[1449792000000,3608.06],[1450051200000,3711.32],[1450137600000,3694.39],[1450224000000,3685.44],[1450310400000,3755.89],[1450396800000,3767.91],[1450656000000,3865.97],[1450742400000,3876.73],[1450828800000,3866.38],[1450915200000,3829.4],[1451001600000,3838.2],[1451260800000,3727.63],[1451347200000,3761.88],[1451433600000,3765.18],[1451520000000,3731.0],[1451865600000,3470.41],[1451952000000,3478.78],[1452038400000,3539.81],[1452124800000,3294.38],[1452211200000,3361.56],[1452470400000,3192.45],[1452556800000,3215.71],[1452643200000,3155.88],[1452729600000,3221.57],[1452816000000,3118.73],[1453075200000,3130.73],[1453161600000,3223.12],[1453248000000,3174.38],[1453334400000,3081.34],[1453420800000,3113.46],[1453680000000,3128.89],[1453766400000,2940.51],[1453852800000,2930.35],[1453939200000,2853.76],[1454025600000,2946.09],[1454284800000,2901.05],[1454371200000,2961.33],[1454457600000,2948.64],[1454544000000,2984.76],[1454630400000,2963.79],[1455494400000,2946.71],[1455580800000,3037.03],[1455667200000,3063.32],[1455753600000,3053.7],[1455840000000,3051.58],[1456099200000,3118.87],[1456185600000,3089.36],[1456272000000,3109.54],[1456358400000,2918.75],[1456444800000,2948.03],[1456704000000,2877.47],[1456790400000,2930.69],[1456876800000,3051.33],[1456963200000,3058.42],[1457049600000,3093.89],[1457308800000,3104.84],[1457395200000,3107.67],[1457481600000,3071.91],[1457568000000,3013.15],[1457654400000,3018.28],[1457913600000,3065.69],[1458000000000,3074.78],[1458086400000,3090.03],[1458172800000,3124.2],[1458259200000,3171.96],[1458518400000,3249.44],[1458604800000,3225.79],[1458691200000,3236.09],[1458777600000,3181.85],[1458864000000,3197.82],[1459123200000,3169.73],[1459209600000,3135.41],[1459296000000,3216.27],[1459382400000,3218.09],[1459468800000,3221.9],[1459814400000,3264.49],[1459900800000,3257.53],[1459987200000,3209.29],[1460073600000,3185.73],[1460332800000,3230.1],[1460419200000,3218.45],[1460505600000,3261.38],[1460592000000,3275.83],[1460678400000,3272.21],[1460937600000,3228.45],[1461024000000,3238.3],[1461110400000,3181.03],[1461196800000,3160.6],[1461283200000,3174.9],[1461542400000,3162.03],[1461628800000,3179.16],[1461715200000,3165.92],[1461801600000,3160.58],[1461888000000,3156.75],[1462233600000,3213.54],[1462320000000,3209.46],[1462406400000,3213.92],[1462492800000,3130.35],[1462752000000,3065.61],[1462838400000,3069.11],[1462924800000,3082.81],[1463011200000,3090.14],[1463097600000,3074.94],[1463356800000,3095.31],[1463443200000,3086.02],[1463529600000,3068.03],[1463616000000,3062.5],[1463702400000,3078.22],[1463961600000,3087.22],[1464048000000,3063.55],[1464134400000,3059.23],[1464220800000,3064.21],[1464307200000,3062.5],[1464566400000,3066.71],[1464652800000,3169.56],[1464739200000,3160.55],[1464825600000,3167.1],[1464912000000,3189.32],[1465171200000,3178.79],[1465257600000,3177.05],[1465344000000,3163.99],[1465776000000,3066.34],[1465862400000,3075.98],[1465948800000,3116.37],[1466035200000,3094.67],[1466121600000,3110.36],[1466380800000,3112.67],[1466467200000,3106.32],[1466553600000,3133.96],[1466640000000,3117.32],[1466726400000,3077.16],[1466985600000,3120.54],[1467072000000,3136.4],[1467158400000,3151.39],[1467244800000,3153.92],[1467331200000,3154.2],[1467590400000,3204.7],[1467676800000,3207.38],[1467763200000,3216.8],[1467849600000,3209.95],[1467936000000,3192.28],[1468195200000,3203.33],[1468281600000,3273.18],[1468368000000,3282.87],[1468454400000,3276.76],[1468540800000,3276.28],[1468800000000,3262.02],[1468886400000,3248.23],[1468972800000,3237.6],[1469059200000,3252.52],[1469145600000,3225.16],[1469404800000,3230.89],[1469491200000,3269.59],[1469577600000,3218.24],[1469664000000,3221.14],[1469750400000,3203.93],[1470009600000,3176.81],[1470096000000,3189.05],[1470182400000,3193.51],[1470268800000,3201.29],[1470355200000,3205.11],[1470614400000,3234.18],[1470700800000,3256.98],[1470787200000,3243.34],[1470873600000,3233.36],[1470960000000,3294.23],[1471219200000,3393.42],[1471305600000,3378.25],[1471392000000,3373.05],[1471478400000,3364.49],[1471564800000,3365.02],[1471824000000,3336.79],[1471910400000,3341.83],[1471996800000,3329.86],[1472083200000,3308.97],[1472169600000,3307.09],[1472428800000,3307.78],[1472515200000,3311.99],[1472601600000,3327.79],[1472688000000,3301.58],[1472774400000,3314.11],[1473033600000,3319.68],[1473120000000,3342.62],[1473206400000,3340.82],[1473292800000,3339.56],[1473379200000,3318.04],[1473638400000,3262.6],[1473724800000,3260.33],[1473811200000,3238.73],[1474243200000,3263.12],[1474329600000,3257.4],[1474416000000,3266.64],[1474502400000,3291.12],[1474588800000,3275.67],[1474848000000,3220.28],[1474934400000,3240.75],[1475020800000,3230.89],[1475107200000,3244.39],[1475193600000,3253.28],[1476057600000,3293.87],[1476144000000,3306.56],[1476230400000,3300.01],[1476316800000,3302.65],[1476403200000,3305.85],[1476662400000,3277.88],[1476748800000,3321.33],[1476835200000,3316.24],[1476921600000,3318.6],[1477008000000,3327.74],[1477267200000,3367.58],[1477353600000,3367.45],[1477440000000,3354.8],[1477526400000,3345.7],[1477612800000,3340.13],[1477872000000,3336.28],[1477958400000,3359.05],[1478044800000,3333.35],[1478131200000,3365.08],[1478217600000,3354.17],[1478476800000,3356.59],[1478563200000,3371.12],[1478649600000,3353.05],[1478736000000,3390.61],[1478822400000,3417.22],[1479081600000,3430.25],[1479168000000,3429.87],[1479254400000,3429.59],[1479340800000,3436.53],[1479427200000,3417.46],[1479686400000,3441.11],[1479772800000,3468.36],[1479859200000,3474.73],[1479945600000,3488.74],[1480032000000,3521.3],[1480291200000,3535.08],[1480377600000,3564.04],[1480464000000,3538.0],[1480550400000,3565.04],[1480636800000,3528.95],[1480896000000,3469.41],[1480982400000,3459.15],[1481068800000,3475.75],[1481155200000,3470.14],[1481241600000,3493.7],[1481500800000,3409.18],[1481587200000,3405.04],[1481673600000,3378.95],[1481760000000,3340.43],[1481846400000,3346.03],[1482105600000,3328.98],[1482192000000,3309.06],[1482278400000,3338.54],[1482364800000,3335.67],[1482451200000,3307.6],[1482710400000,3322.4],[1482796800000,3316.39],[1482883200000,3301.89],[1482969600000,3297.76],[1483056000000,3310.08],[1483401600000,3342.23],[1483488000000,3368.31],[1483574400000,3367.79],[1483660800000,3347.67],[1483920000000,3363.9],[1484006400000,3358.27],[1484092800000,3334.5],[1484179200000,3317.62],[1484265600000,3319.91],[1484524800000,3319.45],[1484611200000,3326.36],[1484697600000,3339.37],[1484784000000,3329.29],[1484870400000,3354.89],[1485129600000,3364.08],[1485216000000,3364.45],[1485302400000,3375.9],[1485388800000,3387.96],[1486080000000,3364.49],[1486339200000,3373.21],[1486425600000,3365.68],[1486512000000,3383.29],[1486598400000,3396.29],[1486684800000,3413.49],[1486944000000,3436.27],[1487030400000,3435.8],[1487116800000,3421.71],[1487203200000,3440.93],[1487289600000,3421.44],[1487548800000,3471.39],[1487635200000,3482.82],[1487721600000,3489.76],[1487808000000,3473.32],[1487894400000,3473.85],[1488153600000,3446.22],[1488240000000,3452.81],[1488326400000,3458.44],[1488412800000,3435.1],[1488499200000,3427.86],[1488758400000,3446.48],[1488844800000,3453.96],[1488931200000,3448.73],[1489017600000,3426.94],[1489104000000,3427.89],[1489363200000,3458.1],[1489449600000,3456.69],[1489536000000,3463.64],[1489622400000,3481.51],[1489708800000,3445.81],[1489968000000,3449.61],[1490054400000,3466.35],[1490140800000,3450.05],[1490227200000,3461.98],[1490313600000,3489.6],[1490572800000,3478.04],[1490659200000,3469.81],[1490745600000,3465.19],[1490832000000,3436.76],[1490918400000,3456.05],[1491350400000,3503.89],[1491436800000,3514.05],[1491523200000,3517.46],[1491782400000,3505.14],[1491868800000,3517.33],[1491955200000,3509.44],[1492041600000,3514.57],[1492128000000,3486.5],[1492387200000,3479.94],[1492473600000,3462.74],[1492560000000,3445.88],[1492646400000,3461.55],[1492732800000,3466.79],[1492992000000,3431.26],[1493078400000,3440.97],[1493164800000,3445.18],[1493251200000,3446.72],[1493337600000,3439.75],[1493683200000,3426.58],[1493769600000,3413.13],[1493856000000,3404.39],[1493942400000,3382.55],[1494201600000,3358.81],[1494288000000,3352.53],[1494374400000,3337.7],[1494460800000,3356.65],[1494547200000,3385.38],[1494806400000,3399.19],[1494892800000,3428.65],[1494979200000,3409.97],[1495065600000,3398.11],[1495152000000,3403.85],[1495411200000,3411.24],[1495497600000,3424.19],[1495584000000,3424.17],[1495670400000,3485.66],[1495756800000,3480.43],[1496188800000,3492.88],[1496275200000,3497.74],[1496361600000,3486.51],[1496620800000,3468.75],[1496707200000,3492.88],[1496793600000,3533.87],[1496880000000,3560.98],[1496966400000,3576.17],[1497225600000,3574.39],[1497312000000,3582.27],[1497398400000,3535.3],[1497484800000,3528.79],[1497571200000,3518.76],[1497830400000,3553.67],[1497916800000,3546.49],[1498003200000,3587.96],[1498089600000,3590.34],[1498176000000,3622.88],[1498435200000,3668.09],[1498521600000,3674.72],[1498608000000,3646.17],[1498694400000,3668.83],[1498780800000,3666.8],[1499040000000,3650.85],[1499126400000,3619.98],[1499212800000,3659.68],[1499299200000,3660.1],[1499385600000,3655.93],[1499644800000,3653.69],[1499731200000,3670.81],[1499817600000,3658.82],[1499904000000,3686.92],[1499990400000,3703.09],[1500249600000,3663.56],[1500336000000,3667.18],[1500422400000,3729.75],[1500508800000,3747.88],[1500595200000,3728.6],[1500854400000,3743.47],[1500940800000,3719.56],[1501027200000,3705.39],[1501113600000,3712.19],[1501200000000,3721.89],[1501459200000,3737.87],[1501545600000,3770.38],[1501632000000,3760.85],[1501718400000,3727.83],[1501804800000,3707.58],[1502064000000,3726.79],[1502150400000,3732.21],[1502236800000,3731.04],[1502323200000,3715.92],[1502409600000,3647.35],[1502668800000,3694.68],[1502755200000,3706.06],[1502841600000,3701.42],[1502928000000,3721.28],[1503014400000,3724.67],[1503273600000,3740.99],[1503360000000,3752.3],[1503446400000,3756.09],[1503532800000,3734.65],[1503619200000,3795.75],[1503878400000,3842.71],[1503964800000,3834.54],[1504051200000,3834.3],[1504137600000,3822.09],[1504224000000,3830.54],[1504483200000,3845.62],[1504569600000,3857.05],[1504656000000,3849.45],[1504742400000,3829.87],[1504828800000,3825.99],[1505088000000,3825.65],[1505174400000,3837.93],[1505260800000,3842.61],[1505347200000,3829.96],[1505433600000,3831.3],[1505692800000,3843.14],[1505779200000,3832.12],[1505865600000,3842.44],[1505952000000,3837.82],[1506038400000,3837.73],[1506297600000,3817.79],[1506384000000,3820.78],[1506470400000,3821.2],[1506556800000,3822.54],[1506643200000,3836.5],[1507507200000,3882.21],[1507593600000,3889.86],[1507680000000,3902.69],[1507766400000,3912.95],[1507852800000,3921.0],[1508112000000,3913.45],[1508198400000,3913.07],[1508284800000,3944.16],[1508371200000,3931.25],[1508457600000,3926.85],[1508716800000,3930.8],[1508803200000,3959.4],[1508889600000,3976.95],[1508976000000,3993.58],[1509062400000,4021.97],[1509321600000,4009.72],[1509408000000,4006.72],[1509494400000,3996.62],[1509580800000,3997.13],[1509667200000,3992.7],[1509926400000,4020.89],[1510012800000,4054.25],[1510099200000,4048.01],[1510185600000,4075.9],[1510272000000,4111.91],[1510531200000,4128.07],[1510617600000,4099.35],[1510704000000,4073.67],[1510790400000,4105.01],[1510876800000,4120.85],[1511136000000,4143.83],[1511222400000,4217.7],[1511308800000,4227.57],[1511395200000,4102.4],[1511481600000,4104.2],[1511740800000,4049.95],[1511827200000,4055.82],[1511913600000,4053.75],[1512000000000,4006.1],[1512086400000,3998.14],[1512345600000,4018.86],[1512432000000,4040.17],[1512518400000,4015.82],[1512604800000,3971.06],[1512691200000,4003.38],[1512950400000,4069.5],[1513036800000,4016.02],[1513123200000,4050.09],[1513209600000,4026.15],[1513296000000,3980.86],[1513555200000,3985.29],[1513641600000,4035.33],[1513728000000,4030.49],[1513814400000,4067.85],[1513900800000,4054.6],[1514160000000,4041.54],[1514246400000,4053.62],[1514332800000,3991.21],[1514419200000,4018.9],[1514505600000,4030.85],[1514851200000,4087.4],[1514937600000,4111.39],[1515024000000,4128.81],[1515110400000,4138.75],[1515369600000,4160.16],[1515456000000,4189.3],[1515542400000,4207.81],[1515628800000,4205.59],[1515715200000,4225.0],[1515974400000,4225.24],[1516060800000,4258.47],[1516147200000,4248.12],[1516233600000,4271.42],[1516320000000,4285.4],[1516579200000,4336.6],[1516665600000,4382.61],[1516752000000,4389.89],[1516838400000,4365.08],[1516924800000,4381.3],[1517184000000,4302.02],[1517270400000,4256.1],[1517356800000,4275.9],[1517443200000,4245.9],[1517529600000,4271.23],[1517788800000,4274.15],[1517875200000,4148.89],[1517961600000,4050.5],[1518048000000,4012.05],[1518134400000,3840.65],[1518393600000,3890.1],[1518480000000,3935.63],[1518566400000,3966.96],[1519257600000,4052.73],[1519344000000,4071.09],[1519603200000,4118.42],[1519689600000,4058.98],[1519776000000,4023.64],[1519862400000,4049.09],[1519948800000,4016.46],[1520208000000,4018.1],[1520294400000,4066.56],[1520380800000,4036.65],[1520467200000,4077.6],[1520553600000,4108.87],[1520812800000,4127.67],[1520899200000,4091.25],[1520985600000,4073.34],[1521072000000,4096.16],[1521158400000,4056.42],[1521417600000,4074.25],[1521504000000,4077.7],[1521590400000,4061.05],[1521676800000,4020.35],[1521763200000,3904.94],[1522022400000,3879.89],[1522108800000,3913.27],[1522195200000,3842.72],[1522281600000,3894.05],[1522368000000,3898.5],[1522627200000,3886.92],[1522713600000,3862.48],[1522800000000,3854.86],[1523232000000,3852.93],[1523318400000,3927.17],[1523404800000,3938.34],[1523491200000,3898.64],[1523577600000,3871.14],[1523836800000,3808.86],[1523923200000,3748.64],[1524009600000,3766.28],[1524096000000,3811.84],[1524182400000,3760.85],[1524441600000,3766.33],[1524528000000,3843.49],[1524614400000,3828.7],[1524700800000,3755.49],[1524787200000,3756.88],[1525219200000,3763.65],[1525305600000,3793.0],[1525392000000,3774.6],[1525651200000,3834.19],[1525737600000,3878.68],[1525824000000,3871.62],[1525910400000,3893.06],[1525996800000,3872.84],[1526256000000,3909.29],[1526342400000,3924.1],[1526428800000,3892.84],[1526515200000,3864.05],[1526601600000,3903.06],[1526860800000,3921.24],[1526947200000,3906.21],[1527033600000,3854.58],[1527120000000,3827.22],[1527206400000,3816.5],[1527465600000,3833.26],[1527552000000,3804.01],[1527638400000,3723.37],[1527724800000,3802.38],[1527811200000,3770.59],[1528070400000,3807.58],[1528156800000,3845.32],[1528243200000,3837.35],[1528329600000,3831.01],[1528416000000,3779.62],[1528675200000,3779.98],[1528761600000,3825.95],[1528848000000,3788.34],[1528934400000,3773.37],[1529020800000,3753.43],[1529366400000,3621.12],[1529452800000,3635.44],[1529539200000,3592.97],[1529625600000,3608.9],[1529884800000,3560.48],[1529971200000,3531.11],[1530057600000,3459.26],[1530144000000,3423.53],[1530230400000,3510.98],[1530489600000,3407.96],[1530576000000,3409.28],[1530662400000,3363.75],[1530748800000,3342.44],[1530835200000,3365.12],[1531094400000,3459.18],[1531180800000,3467.52],[1531267200000,3407.53],[1531353600000,3481.06],[1531440000000,3492.69],[1531699200000,3472.09],[1531785600000,3449.38],[1531872000000,3431.32],[1531958400000,3428.34],[1532044800000,3492.89],[1532304000000,3525.75],[1532390400000,3581.71],[1532476800000,3577.75],[1532563200000,3536.25],[1532649600000,3521.23],[1532908800000,3515.08],[1532995200000,3517.66],[1533081600000,3447.39],[1533168000000,3370.96],[1533254400000,3315.28],[1533513600000,3273.27],[1533600000000,3368.87],[1533686400000,3314.51],[1533772800000,3397.53],[1533859200000,3405.02],[1534118400000,3390.34],[1534204800000,3372.91],[1534291200000,3291.98],[1534377600000,3276.73],[1534464000000,3229.62],[1534723200000,3267.25],[1534809600000,3326.65],[1534896000000,3307.95],[1534982400000,3320.03],[1535068800000,3325.33],[1535328000000,3406.57],[1535414400000,3400.17],[1535500800000,3386.57],[1535587200000,3351.09],[1535673600000,3334.5],[1535932800000,3321.82],[1536019200000,3363.9],[1536105600000,3298.14],[1536192000000,3262.88],[1536278400000,3277.64],[1536537600000,3230.07],[1536624000000,3224.21],[1536710400000,3202.02],[1536796800000,3236.57],[1536883200000,3242.09],[1537142400000,3204.92],[1537228800000,3269.43],[1537315200000,3312.48],[1537401600000,3310.13],[1537488000000,3410.49],[1537833600000,3379.8],[1537920000000,3417.24],[1538006400000,3403.59],[1538092800000,3438.86],[1538956800000,3290.9],[1539043200000,3288.69],[1539129600000,3281.6],[1539216000000,3124.11],[1539302400000,3170.73],[1539561600000,3126.45],[1539648000000,3100.97],[1539734400000,3118.25],[1539820800000,3044.39],[1539907200000,3134.95],[1540166400000,3270.27],[1540252800000,3183.43],[1540339200000,3188.2],[1540425600000,3194.31],[1540512000000,3173.64],[1540771200000,3076.89],[1540857600000,3110.26],[1540944000000,3153.82],[1541030400000,3177.03],[1541116800000,3290.25],[1541376000000,3262.84],[1541462400000,3243.15],[1541548800000,3221.91],[1541635200000,3212.77],[1541721600000,3167.44],[1541980800000,3205.14],[1542067200000,3237.38],[1542153600000,3204.94],[1542240000000,3242.37],[1542326400000,3257.67],[1542585600000,3294.6],[1542672000000,3218.41],[1542758400000,3226.49],[1542844800000,3214.43],[1542931200000,3143.48],[1543190400000,3141.24],[1543276800000,3137.24],[1543363200000,3178.93],[1543449600000,3137.65],[1543536000000,3172.69],[1543795200000,3260.95],[1543881600000,3267.71],[1543968000000,3252.0],[1544054400000,3181.67],[1544140800000,3181.56],[1544400000000,3144.76],[1544486400000,3159.82],[1544572800000,3170.61],[1544659200000,3219.69],[1544745600000,3165.91],[1545004800000,3161.2],[1545091200000,3128.43],[1545177600000,3091.13],[1545264000000,3067.42],[1545350400000,3029.4],[1545609600000,3038.2],[1545696000000,3017.28],[1545782400000,3002.03],[1545955200000,3010.65],[1546387200000,2969.54],[1546473600000,2964.84],[1546560000000,3035.87],[1546819200000,3054.3],[1546905600000,3047.7],[1546992000000,3078.48],[1547078400000,3072.69],[1547164800000,3094.78],[1547424000000,3067.78],[1547510400000,3127.99],[1547596800000,3128.65],[1547683200000,3111.42],[1547769600000,3168.17],[1548028800000,3185.64],[1548115200000,3143.32],[1548201600000,3141.05],[1548288000000,3158.78],[1548374400000,3184.47],[1548633600000,3183.78],[1548720000000,3193.97],[1548806400000,3168.48],[1548892800000,3201.63],[1548979200000,3247.4],[1549843200000,3306.47],[1549929600000,3330.34],[1550016000000,3397.03],[1550102400000,3402.14],[1550188800000,3338.7],[1550448000000,3445.74],[1550534400000,3439.61],[1550620800000,3451.93],[1550707200000,3442.71],[1550793600000,3520.12],[1551052800000,3729.48],[1551139200000,3684.69],[1551225600000,3678.39],[1551312000000,3669.37],[1551398400000,3749.71],[1551657600000,3794.1],[1551744000000,3816.01],[1551830400000,3848.09],[1551916800000,3808.85],[1552003200000,3657.58],[1552262400000,3729.95],[1552348800000,3755.35],[1552435200000,3724.19],[1552521600000,3698.49],[1552608000000,3745.0],[1552867200000,3851.75],[1552953600000,3833.96],[1553040000000,3835.44],[1553126400000,3836.89],[1553212800000,3833.8],[1553472000000,3742.83],[1553558400000,3700.44],[1553644800000,3743.39],[1553731200000,3728.4],[1553817600000,3872.34],[1554076800000,3973.93],[1554163200000,3971.29],[1554249600000,4022.16],[1554336000000,4062.23],[1554681600000,4057.23],[1554768000000,4075.43],[1554854400000,4085.85],[1554940800000,3997.58],[1555027200000,3988.62],[1555286400000,3975.52],[1555372800000,4085.79],[1555459200000,4087.24],[1555545600000,4072.08],[1555632000000,4120.61],[1555891200000,4025.61],[1555977600000,4019.01],[1556064000000,4030.09],[1556150400000,3941.82],[1556236800000,3889.27],[1556496000000,3900.33],[1556582400000,3913.21],[1557100800000,3684.62],[1557187200000,3720.67],[1557273600000,3667.46],[1557360000000,3599.7],[1557446400000,3730.45],[1557705600000,3668.73],[1557792000000,3645.15],[1557878400000,3727.09],[1557964800000,3743.96],[1558051200000,3648.76],[1558310400000,3617.79],[1558396800000,3666.78],[1558483200000,3649.38],[1558569600000,3583.96],[1558656000000,3593.91],[1558915200000,3637.2],[1559001600000,3672.26],[1559088000000,3663.91],[1559174400000,3641.18],[1559260800000,3629.79],[1559520000000,3632.01],[1559606400000,3598.47],[1559692800000,3597.1],[1559779200000,3564.68],[1560124800000,3610.74],[1560211200000,3719.28],[1560297600000,3691.1],[1560384000000,3685.39],[1560470400000,3654.88],[1560729600000,3654.82],[1560816000000,3667.62],[1560902400000,3715.94],[1560988800000,3828.52],[1561075200000,3833.94],[1561334400000,3841.27],[1561420800000,3801.31],[1561507200000,3794.33],[1561593600000,3834.82],[1561680000000,3825.59],[1561939200000,3935.81],[1562025600000,3937.17],[1562112000000,3893.53],[1562198400000,3873.1],[1562284800000,3893.2],[1562544000000,3802.79],[1562630400000,3793.13],[1562716800000,3786.74],[1562803200000,3785.22],[1562889600000,3808.73],[1563148800000,3824.19],[1563235200000,3806.84],[1563321600000,3804.64],[1563408000000,3768.4],[1563494400000,3807.96],[1563753600000,3781.68],[1563840000000,3789.91],[1563926400000,3819.83],[1564012800000,3851.07],[1564099200000,3858.57],[1564358400000,3854.27],[1564444800000,3870.32],[1564531200000,3835.36],[1564617600000,3803.47],[1564704000000,3747.44],[1564963200000,3675.69],[1565049600000,3636.33],[1565136000000,3621.43],[1565222400000,3669.29],[1565308800000,3633.53],[1565568000000,3699.1],[1565654400000,3665.75],[1565740800000,3682.4],[1565827200000,3694.0],[1565913600000,3710.54],[1566172800000,3791.09],[1566259200000,3787.73],[1566345600000,3781.76],[1566432000000,3793.51],[1566518400000,3820.86],[1566777600000,3765.91],[1566864000000,3816.95],[1566950400000,3802.58],[1567036800000,3790.19],[1567123200000,3799.59],[1567382400000,3848.32],[1567468800000,3853.61],[1567555200000,3886.0],[1567641600000,3925.32],[1567728000000,3948.51],[1567987200000,3972.95],[1568073600000,3959.26],[1568160000000,3930.1],[1568246400000,3972.38],[1568592000000,3957.72],[1568678400000,3891.22],[1568764800000,3910.08],[1568851200000,3924.38],[1568937600000,3935.65],[1569196800000,3890.66],[1569283200000,3901.08],[1569369600000,3870.98],[1569456000000,3841.14],[1569542400000,3852.65],[1569801600000,3814.53],[1570492800000,3837.68],[1570579200000,3843.24],[1570665600000,3874.64],[1570752000000,3911.73],[1571011200000,3953.24],[1571097600000,3936.25],[1571184000000,3922.69],[1571270400000,3925.22],[1571356800000,3869.38],[1571616000000,3880.84],[1571702400000,3895.88],[1571788800000,3871.08],[1571875200000,3870.67],[1571961600000,3896.79],[1572220800000,3926.58],[1572307200000,3910.23],[1572393600000,3891.23],[1572480000000,3886.75],[1572566400000,3952.39],[1572825600000,3978.12],[1572912000000,4002.81],[1572998400000,3984.88],[1573084800000,3991.88],[1573171200000,3973.01],[1573430400000,3902.98],[1573516800000,3903.69],[1573603200000,3899.98],[1573689600000,3905.86]]}],"xAxis":{"title":{"text":"date"},"type":"datetime"},"yAxis":[{},{"opposite":true}]});</script>

