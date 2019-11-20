# Demo

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
    # stock_df = stock_df.loc[:, ['date', 'open', 'close']]
    
    # 特殊处理，用当天收盘价做判定和交易
    # stock_df['open'] = stock_df['close']

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
    # stock_df['o_pct_chg'] = stock_df.open.pct_change(1)
    # stock_df['c_o_pct_chg'] = (stock_df.open - stock_df.close.shift(1)) / stock_df.close.shift(1)
    # stock_df['N_chg'] = stock_df.open.pct_change(N)
    # 用昨天收盘价做判定
    # stock_df['N_chg'] = (stock_df.close.shift(1) - stock_df.close.shift(N)) / stock_df.close.shift(N)
    # stock_df['N_sht'] = stock_df.open.shift(N)
    # stock_df['N_chn'] = stock_df.open.shift(N) - stock_df.open
    # stock_df['y_close'] = stock_df.close.shift(1)
    
    # MA均线指标
    # stock_df['MA%d' % M] = stock_df['open'].rolling(M).mean()
    # stock_df['MA%d' % M] = stock_df['close'].rolling(M).mean().shift(1)
    
    # 减少数据
    stock_df.dropna(how='any', inplace=True)
    
    stock_df_dict[symbol] = stock_df
```

> Wall time: 32.9 ms


```python
for symbol in TEST_LIST:
    symbol
    stock_df_dict[symbol].head(2)
    stock_df_dict[symbol].tail(2)
```
> `399300`

|              | open     | close    | high     | low     | volume      | code     |
|--------------|----------|----------|----------|---------|-------------|----------|
| date         |          |          |          |         |             |          |
| 2005\-04\-08 | 984\.66  | 1003\.45 | 1003\.70 | 979\.53 | 14762500\.0 | sz399300 |
| 2005\-04\-11 | 1003\.88 | 995\.42  | 1008\.73 | 992\.77 | 15936100\.0 | sz399300 |


|              | open     | close    | high     | low      | volume      | code     |
|--------------|----------|----------|----------|----------|-------------|----------|
| date         |          |          |          |          |             |          |
| 2019\-11\-13 | 3905\.28 | 3899\.98 | 3908\.42 | 3882\.43 | 76749078\.0 | sz399300 |
| 2019\-11\-14 | 3905\.93 | 3905\.86 | 3916\.95 | 3895\.14 | 87008446\.0 | sz399300 |

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


|              | open     | close    | high     | low      | volume       | code     | CLOSE\_MA    | HCP      | LCP      | DIFFCP | SUMDIFFCP | VHF       | VHF\_MA   |
|--------------|----------|----------|----------|----------|--------------|----------|--------------|----------|----------|--------|-----------|-----------|-----------|
| date         |          |          |          |          |              |          |              |          |          |        |           |           |           |
| 2005\-04\-08 | 984\.66  | 1003\.45 | 1003\.70 | 979\.53  | 14762500\.0  | sz399300 | NaN          | NaN      | NaN      | NaN    | NaN       | NaN       | NaN       |
| 2005\-04\-11 | 1003\.88 | 995\.42  | 1008\.73 | 992\.77  | 15936100\.0  | sz399300 | NaN          | NaN      | NaN      | 8\.03  | NaN       | NaN       | NaN       |
| 2005\-04\-12 | 993\.71  | 978\.70  | 993\.71  | 978\.20  | 10226200\.0  | sz399300 | NaN          | NaN      | NaN      | 16\.72 | NaN       | NaN       | NaN       |
| 2005\-04\-13 | 987\.95  | 1000\.90 | 1006\.50 | 987\.95  | 16071700\.0  | sz399300 | NaN          | NaN      | NaN      | 22\.20 | NaN       | NaN       | NaN       |
| 2005\-04\-14 | 1004\.64 | 986\.97  | 1006\.42 | 985\.58  | 12945700\.0  | sz399300 | NaN          | NaN      | NaN      | 13\.93 | NaN       | NaN       | NaN       |
| 2005\-04\-15 | 982\.61  | 974\.08  | 982\.61  | 971\.93  | 10409000\.0  | sz399300 | NaN          | NaN      | NaN      | 12\.89 | NaN       | NaN       | NaN       |
| 2005\-04\-18 | 970\.91  | 963\.77  | 970\.91  | 958\.65  | 8598400\.0   | sz399300 | NaN          | NaN      | NaN      | 10\.31 | NaN       | NaN       | NaN       |
| 2005\-04\-19 | 962\.92  | 965\.89  | 968\.87  | 957\.91  | 9212620\.0   | sz399300 | NaN          | NaN      | NaN      | 2\.12  | NaN       | NaN       | NaN       |
| 2005\-04\-20 | 964\.15  | 950\.87  | 964\.15  | 946\.20  | 8850700\.0   | sz399300 | NaN          | NaN      | NaN      | 15\.02 | NaN       | NaN       | NaN       |
| 2005\-04\-21 | 948\.86  | 943\.98  | 955\.55  | 938\.59  | 9946150\.0   | sz399300 | NaN          | NaN      | NaN      | 6\.89  | NaN       | NaN       | NaN       |
| 2005\-04\-22 | 942\.91  | 939\.10  | 947\.91  | 934\.96  | 10691900\.0  | sz399300 | NaN          | NaN      | NaN      | 4\.88  | NaN       | NaN       | NaN       |
| 2005\-04\-25 | 935\.99  | 930\.07  | 935\.99  | 920\.16  | 11470500\.0  | sz399300 | NaN          | NaN      | NaN      | 9\.03  | NaN       | NaN       | NaN       |
| 2005\-04\-26 | 928\.43  | 937\.08  | 939\.70  | 924\.66  | 11690500\.0  | sz399300 | NaN          | NaN      | NaN      | 7\.01  | NaN       | NaN       | NaN       |
| 2005\-04\-27 | 938\.57  | 926\.60  | 938\.91  | 925\.90  | 10780600\.0  | sz399300 | NaN          | NaN      | NaN      | 10\.48 | NaN       | NaN       | NaN       |
| 2005\-04\-28 | 923\.53  | 942\.07  | 945\.50  | 914\.83  | 14343500\.0  | sz399300 | NaN          | NaN      | NaN      | 15\.47 | NaN       | NaN       | NaN       |
| 2005\-04\-29 | 940\.81  | 932\.40  | 942\.45  | 929\.81  | 11235400\.0  | sz399300 | NaN          | NaN      | NaN      | 9\.67  | NaN       | NaN       | NaN       |
| 2005\-05\-09 | 934\.65  | 909\.17  | 937\.39  | 909\.17  | 8529110\.0   | sz399300 | NaN          | NaN      | NaN      | 23\.23 | NaN       | NaN       | NaN       |
| 2005\-05\-10 | 905\.54  | 913\.08  | 913\.39  | 892\.31  | 10494300\.0  | sz399300 | NaN          | NaN      | NaN      | 3\.91  | NaN       | NaN       | NaN       |
| 2005\-05\-11 | 911\.84  | 901\.85  | 917\.22  | 900\.44  | 9042050\.0   | sz399300 | NaN          | NaN      | NaN      | 11\.23 | NaN       | NaN       | NaN       |
| 2005\-05\-12 | 899\.97  | 885\.82  | 900\.06  | 883\.51  | 10228700\.0  | sz399300 | NaN          | NaN      | NaN      | 16\.03 | NaN       | NaN       | NaN       |
| 2005\-05\-13 | 883\.51  | 887\.54  | 898\.51  | 875\.58  | 11244900\.0  | sz399300 | NaN          | NaN      | NaN      | 1\.72  | NaN       | NaN       | NaN       |
| 2005\-05\-16 | 885\.39  | 875\.27  | 885\.39  | 869\.33  | 8224289\.0   | sz399300 | NaN          | NaN      | NaN      | 12\.27 | NaN       | NaN       | NaN       |
| 2005\-05\-17 | 873\.08  | 881\.46  | 888\.28  | 868\.21  | 8772270\.0   | sz399300 | NaN          | NaN      | NaN      | 6\.19  | NaN       | NaN       | NaN       |
| 2005\-05\-18 | 881\.14  | 883\.20  | 890\.40  | 871\.82  | 7878610\.0   | sz399300 | NaN          | NaN      | NaN      | 1\.74  | NaN       | NaN       | NaN       |
| 2005\-05\-19 | 882\.84  | 884\.17  | 888\.02  | 871\.29  | 8149579\.0   | sz399300 | NaN          | NaN      | NaN      | 0\.97  | NaN       | NaN       | NaN       |
| 2005\-05\-20 | 883\.51  | 882\.76  | 891\.02  | 879\.18  | 7226679\.0   | sz399300 | NaN          | NaN      | NaN      | 1\.41  | NaN       | NaN       | NaN       |
| 2005\-05\-23 | 880\.28  | 863\.34  | 880\.28  | 862\.10  | 7279800\.0   | sz399300 | NaN          | NaN      | NaN      | 19\.42 | NaN       | NaN       | NaN       |
| 2005\-05\-24 | 861\.20  | 868\.46  | 871\.77  | 855\.59  | 9206929\.0   | sz399300 | NaN          | NaN      | NaN      | 5\.12  | NaN       | NaN       | NaN       |
| 2005\-05\-25 | 867\.66  | 868\.45  | 876\.30  | 861\.66  | 7238560\.0   | sz399300 | NaN          | NaN      | NaN      | 0\.01  | NaN       | NaN       | NaN       |
| 2005\-05\-26 | 867\.76  | 857\.33  | 872\.84  | 854\.96  | 6622789\.0   | sz399300 | NaN          | NaN      | NaN      | 11\.12 | NaN       | NaN       | NaN       |
| \.\.\.       | \.\.\.   | \.\.\.   | \.\.\.   | \.\.\.   | \.\.\.       | \.\.\.   | \.\.\.       | \.\.\.   | \.\.\.   | \.\.\. | \.\.\.    | \.\.\.    | \.\.\.    |
| 2019\-09\-27 | 3843\.46 | 3852\.65 | 3859\.50 | 3832\.25 | 74003947\.0  | sz399300 | 3805\.445500 | 4120\.61 | 3564\.68 | 11\.51 | 4077\.60  | 0\.136338 | 0\.200318 |
| 2019\-09\-30 | 3842\.07 | 3814\.53 | 3857\.23 | 3813\.55 | 65931313\.0  | sz399300 | 3803\.423000 | 4120\.61 | 3564\.68 | 38\.12 | 4110\.72  | 0\.135239 | 0\.199494 |
| 2019\-10\-08 | 3818\.59 | 3837\.68 | 3861\.64 | 3818\.59 | 76657860\.0  | sz399300 | 3801\.441750 | 4120\.61 | 3564\.68 | 23\.15 | 4115\.67  | 0\.135076 | 0\.198651 |
| 2019\-10\-09 | 3822\.61 | 3843\.24 | 3845\.34 | 3805\.07 | 75854512\.0  | sz399300 | 3799\.420000 | 4120\.61 | 3564\.68 | 5\.56  | 4110\.81  | 0\.135236 | 0\.197726 |
| 2019\-10\-10 | 3838\.49 | 3874\.64 | 3877\.14 | 3829\.43 | 79222694\.0  | sz399300 | 3798\.395500 | 4120\.61 | 3564\.68 | 31\.40 | 4053\.94  | 0\.137133 | 0\.196836 |
| 2019\-10\-11 | 3885\.52 | 3911\.73 | 3922\.37 | 3868\.84 | 93412577\.0  | sz399300 | 3797\.754750 | 4120\.61 | 3564\.68 | 37\.09 | 4082\.07  | 0\.136188 | 0\.195921 |
| 2019\-10\-14 | 3944\.86 | 3953\.24 | 3983\.81 | 3934\.73 | 122393611\.0 | sz399300 | 3797\.569083 | 4120\.61 | 3564\.68 | 41\.51 | 4110\.48  | 0\.135247 | 0\.194994 |
| 2019\-10\-15 | 3953\.16 | 3936\.25 | 3953\.16 | 3928\.15 | 85273949\.0  | sz399300 | 3796\.322917 | 4120\.61 | 3564\.68 | 16\.99 | 4017\.20  | 0\.138387 | 0\.194134 |
| 2019\-10\-16 | 3939\.28 | 3922\.69 | 3964\.05 | 3919\.02 | 83229707\.0  | sz399300 | 3794\.951667 | 4120\.61 | 3564\.68 | 13\.56 | 4029\.31  | 0\.137972 | 0\.193236 |
| 2019\-10\-17 | 3929\.39 | 3925\.22 | 3935\.81 | 3913\.92 | 64863745\.0  | sz399300 | 3793\.727833 | 4120\.61 | 3564\.68 | 2\.53  | 4016\.68  | 0\.138405 | 0\.192307 |
| 2019\-10\-18 | 3935\.42 | 3869\.38 | 3940\.54 | 3864\.97 | 82321140\.0  | sz399300 | 3791\.634250 | 4030\.09 | 3564\.68 | 55\.84 | 4023\.99  | 0\.115659 | 0\.191084 |
| 2019\-10\-21 | 3865\.34 | 3880\.84 | 3884\.74 | 3856\.04 | 74182918\.0  | sz399300 | 3790\.427833 | 4030\.09 | 3564\.68 | 11\.46 | 3940\.45  | 0\.118111 | 0\.189886 |
| 2019\-10\-22 | 3896\.31 | 3895\.88 | 3897\.44 | 3870\.55 | 68918566\.0  | sz399300 | 3789\.401750 | 4030\.09 | 3564\.68 | 15\.04 | 3948\.89  | 0\.117858 | 0\.188687 |
| 2019\-10\-23 | 3892\.68 | 3871\.08 | 3901\.42 | 3862\.20 | 70198625\.0  | sz399300 | 3788\.076667 | 3972\.95 | 3564\.68 | 24\.80 | 3962\.61  | 0\.103031 | 0\.187367 |
| 2019\-10\-24 | 3877\.92 | 3870\.67 | 3890\.20 | 3852\.58 | 71334575\.0  | sz399300 | 3787\.483750 | 3972\.95 | 3564\.68 | 0\.41  | 3874\.75  | 0\.105367 | 0\.186099 |
| 2019\-10\-25 | 3871\.77 | 3896\.79 | 3899\.72 | 3849\.03 | 77453620\.0  | sz399300 | 3787\.546417 | 3972\.95 | 3564\.68 | 26\.12 | 3848\.32  | 0\.106090 | 0\.184815 |
| 2019\-10\-28 | 3904\.98 | 3926\.58 | 3927\.80 | 3897\.82 | 97406020\.0  | sz399300 | 3787\.765167 | 3972\.95 | 3564\.68 | 29\.79 | 3867\.05  | 0\.105577 | 0\.183517 |
| 2019\-10\-29 | 3929\.71 | 3910\.23 | 3932\.37 | 3910\.23 | 82257075\.0  | sz399300 | 3787\.740333 | 3972\.95 | 3564\.68 | 16\.35 | 3870\.52  | 0\.105482 | 0\.182202 |
| 2019\-10\-30 | 3904\.72 | 3891\.23 | 3908\.10 | 3883\.45 | 77872923\.0  | sz399300 | 3789\.462083 | 3972\.95 | 3564\.68 | 19\.00 | 3660\.93  | 0\.111521 | 0\.181036 |
| 2019\-10\-31 | 3906\.86 | 3886\.75 | 3907\.15 | 3878\.88 | 86714539\.0  | sz399300 | 3790\.846083 | 3972\.95 | 3564\.68 | 4\.48  | 3629\.36  | 0\.112491 | 0\.179842 |
| 2019\-11\-01 | 3883\.81 | 3952\.39 | 3956\.08 | 3876\.42 | 100689612\.0 | sz399300 | 3793\.220500 | 3972\.95 | 3564\.68 | 65\.64 | 3641\.79  | 0\.112107 | 0\.178657 |
| 2019\-11\-04 | 3964\.01 | 3978\.12 | 3985\.36 | 3964\.01 | 104193855\.0 | sz399300 | 3796\.374000 | 3978\.12 | 3564\.68 | 25\.73 | 3599\.76  | 0\.114852 | 0\.177517 |
| 2019\-11\-05 | 3982\.12 | 4002\.81 | 4030\.64 | 3970\.87 | 112911880\.0 | sz399300 | 3798\.643667 | 4002\.81 | 3564\.68 | 24\.69 | 3493\.70  | 0\.125406 | 0\.176514 |
| 2019\-11\-06 | 4006\.33 | 3984\.88 | 4007\.50 | 3973\.78 | 97312952\.0  | sz399300 | 3801\.278250 | 4002\.81 | 3564\.68 | 17\.93 | 3449\.91  | 0\.126998 | 0\.175546 |
| 2019\-11\-07 | 3984\.55 | 3991\.88 | 4005\.38 | 3975\.55 | 87254898\.0  | sz399300 | 3804\.167667 | 4002\.81 | 3564\.68 | 7\.00  | 3433\.33  | 0\.127611 | 0\.174575 |
| 2019\-11\-08 | 4017\.67 | 3973\.01 | 4022\.04 | 3971\.57 | 97106948\.0  | sz399300 | 3806\.217000 | 4002\.81 | 3564\.68 | 18\.87 | 3370\.26  | 0\.129999 | 0\.173642 |
| 2019\-11\-11 | 3948\.62 | 3902\.98 | 3948\.62 | 3897\.55 | 94155621\.0  | sz399300 | 3807\.542167 | 4002\.81 | 3564\.68 | 70\.03 | 3423\.42  | 0\.127980 | 0\.172686 |
| 2019\-11\-12 | 3905\.96 | 3903\.69 | 3914\.88 | 3877\.74 | 76543069\.0  | sz399300 | 3809\.666583 | 4002\.81 | 3564\.68 | 0\.71  | 3328\.93  | 0\.131613 | 0\.171787 |
| 2019\-11\-13 | 3905\.28 | 3899\.98 | 3908\.42 | 3882\.43 | 76749078\.0  | sz399300 | 3812\.018167 | 4002\.81 | 3564\.68 | 3\.71  | 3301\.67  | 0\.132700 | 0\.170894 |
| 2019\-11\-14 | 3905\.93 | 3905\.86 | 3916\.95 | 3895\.14 | 87008446\.0  | sz399300 | 3814\.010500 | 4002\.81 | 3564\.68 | 5\.88  | 3258\.56  | 0\.134455 | 0\.170029 |

```python
%matplotlib inline

import matplotlib
import matplotlib.pyplot as plt
plt.rcParams['figure.figsize'] = [15, 5]

df = o_df.copy()
df = df.iloc[3530:]
# df = o_df.loc[:, ['close', 'VHF_MA20']].copy()
# df.drop(columns=['PROPERTY'], inplace=True)
df.columns
df = df.dropna(how='any', inplace=False)

show_columns = ['close', 'VHF']
df = df.loc[:, show_columns]

# df['close'] = (df['close'] - df.iloc[0]['close']) / df.iloc[0]['close']
ax = df.plot(kind='line', y='close', title='VHF', linewidth=1, grid=True)
# ax = df.plot(kind='line', y='CLOSE_MA', secondary_y=False, title='VHF_N%d' % N, linewidth=1, grid=True)
ax = df.plot(kind='line', y='VHF', secondary_y=True, title='VHF', linewidth=0.5, grid=True, ax=ax)
```

```python
Index(['open', 'close', 'high', 'low', 'volume', 'code', 'CLOSE_MA', 'HCP', 'LCP', 'DIFFCP', 'SUMDIFFCP', 'VHF', 'VHF_MA'],dtype='object')
```

![png](https://bttun.github.io/r/a%E7%9B%AE%E5%BD%9502/output_5_1.png)


```python
df.reset_index(drop=False, inplace=True)
df['date'] = df['date'].apply(lambda x: x.to_timestamp().to_datetime64())
df.set_index(keys=['date'], inplace=True)
display_charts(df, secondary_y=['VHF'], chart_type='stock', kind='line', figsize=(900, 600), logy=False)
```

<iframe width="100%" height="548" src="//jsrun.pro/zeWKp/embedded/all/light" allowfullscreen="allowfullscreen" frameborder="0"></iframe>



