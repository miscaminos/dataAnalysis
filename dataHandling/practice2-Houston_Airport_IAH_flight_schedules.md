```python
import pandas as pd
import numpy as np
from datetime import datetime, time
```

# 문제 1

### target_a.csv, target_b.csv 파일을 read_csv() 함수로 읽고 해당 컬럼의 값을 
### target_a는 date, AAAAA라는 컬럼으로 
### target_b는 date, BBBBB라는 컬럼으로 이름을 변경하세요


```python
df_ta = pd.read_csv('./data/target_a.csv')
df_ta.rename(columns = {'0':'date'}, inplace = True)
df_ta.rename(columns = {'1':'AAAAA'}, inplace = True)
```


```python
df_ta.head()
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
      <th>date</th>
      <th>AAAAA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7-May-15 21:45:38</td>
      <td>225.006247</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7-May-15 21:46:37</td>
      <td>228.073931</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7-May-15 21:47:37</td>
      <td>227.022569</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7-May-15 21:48:37</td>
      <td>226.136637</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7-May-15 21:49:37</td>
      <td>226.864333</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tb = pd.read_csv('./data/target_b.csv')
df_tb.rename(columns = {'0':'date'}, inplace = True)
df_tb.rename(columns = {'1':'BBBBB'}, inplace = True)
```


```python
df_tb.head()
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
      <th>date</th>
      <th>BBBBB</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7-May-15 21:53:27</td>
      <td>41.419933</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7-May-15 21:54:27</td>
      <td>41.849564</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7-May-15 21:55:26</td>
      <td>43.259666</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7-May-15 21:56:26</td>
      <td>42.472425</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7-May-15 21:57:26</td>
      <td>42.664728</td>
    </tr>
  </tbody>
</table>
</div>



**또는 df_ta, df_tb의 열 이름을 변경하는 더 간결한 방법은 다음과 같다:**

    target_a.columns = ['date', 'AAAAAA']
    
    target_b.columns = ['date', 'BBBBBB']

## 문제 2

### 날짜 변환 예제를 사용하여 두 dataframe의 date 형식을 datetime 객체로 변환하세요


```python
df_ta['date']=pd.to_datetime(df_ta['date'])
```


```python
df_ta.head()
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
      <th>date</th>
      <th>AAAAA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2015-05-07 21:45:38</td>
      <td>225.006247</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2015-05-07 21:46:37</td>
      <td>228.073931</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015-05-07 21:47:37</td>
      <td>227.022569</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015-05-07 21:48:37</td>
      <td>226.136637</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2015-05-07 21:49:37</td>
      <td>226.864333</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tb['date']=pd.to_datetime(df_tb['date'])
df_tb.head()
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
      <th>date</th>
      <th>BBBBB</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2015-05-07 21:53:27</td>
      <td>41.419933</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2015-05-07 21:54:27</td>
      <td>41.849564</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015-05-07 21:55:26</td>
      <td>43.259666</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015-05-07 21:56:26</td>
      <td>42.472425</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2015-05-07 21:57:26</td>
      <td>42.664728</td>
    </tr>
  </tbody>
</table>
</div>



**또는 datetime을 사용하여 formatting을 할수도 있다:**


```python
datetime.strptime("7-May-15 21:53:27", "%d-%b-%y %H:%M:%S")
```




    datetime.datetime(2015, 5, 7, 21, 53, 27)




```python
target_b['date'] = target_b['date'].apply(lambda x :  datetime.strptime(x, "%d-%b-%y %H:%M:%S"))
```


```python
target_a['date'] = target_a['date'].apply(lambda x :  datetime.strptime(x, "%d-%b-%y %H:%M:%S"))
```

## 문제 3

### 각 dataframe의 date 컬럼을 index로 이동시키세요 


```python
df_ta = df_ta.set_index('date')
df_tb = df_tb.set_index('date')

df_ta.head()
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
      <th>AAAAA</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-05-07 21:45:38</th>
      <td>225.006247</td>
    </tr>
    <tr>
      <th>2015-05-07 21:46:37</th>
      <td>228.073931</td>
    </tr>
    <tr>
      <th>2015-05-07 21:47:37</th>
      <td>227.022569</td>
    </tr>
    <tr>
      <th>2015-05-07 21:48:37</th>
      <td>226.136637</td>
    </tr>
    <tr>
      <th>2015-05-07 21:49:37</th>
      <td>226.864333</td>
    </tr>
  </tbody>
</table>
</div>



**또는 inplace parameter를 설정한다:**


```python
target_a.set_index('date', inplace=True)
target_b.set_index('date', inplace=True)
```

===================================================================================================

# 데이터 설명
* A data frame with 227,496 rows and 21 columns.
* This dataset contains all flights departing from Houston airports IAH (George Bush Interconti-
nental) and HOU (Houston Hobby). The data comes from the Research and Innovation Technol-
ogy Administration at the Bureau of Transporation statistics: http://www.transtats.bts.gov/
DatabaseInfo.asp?DB_ID=120&Link=0

### Details
#### Variables
- Year, Month, DayofMonth: date of departure
- DayOfWeek: day of week of departure (useful for removing weekend effects)
- DepTime, ArrTime: departure and arrival times (in local time, hhmm)
- UniqueCarrier: unique abbreviation for a carrier
- FlightNum: flight number
- TailNum: airplane tail number
- ActualElapsedTime: elapsed time of flight, in minutes
- AirTime: flight time, in minutes
- ArrDelay, DepDelay: arrival and departure delays, in minutes
- Origin, Dest origin and destination airport codes
- Distance: distance of flight, in miles
- TaxiIn, TaxiOut: taxi in and out times in minutes
- Cancelled: cancelled indicator: 1 = Yes, 0 = No
- CancellationCode: reason for cancellation: A = carrier, B = weather, C = national air system,
D = security
- Diverted: diverted indicator: 1 = Yes, 0 = No

## 문제 4


```python
hflight = pd.read_csv("./data/hflight.csv")
```


```python
print (hflight.shape)
```

    (227496, 21)
    

### 출발 공항에 대해서 도착 공항별로 평균 출발 지연시간 평균 도착지연시간을 구해서 아래와 같이 DataFrame 을 만들어주세요


![image.png](attachment:image.png)


```python
print (hflight.columns)
```

    Index(['Year', 'Month', 'DayofMonth', 'DayOfWeek', 'DepTime', 'ArrTime',
           'UniqueCarrier', 'FlightNum', 'TailNum', 'ActualElapsedTime', 'AirTime',
           'ArrDelay', 'DepDelay', 'Origin', 'Dest', 'Distance', 'TaxiIn',
           'TaxiOut', 'Cancelled', 'CancellationCode', 'Diverted'],
          dtype='object')
    


```python
delay = pd.DataFrame({'Origin':hflight['Origin'], 'Dest':hflight['Dest'], 
                      'DepDelay':hflight['DepDelay'],'ArrDelay':hflight['ArrDelay']})
```


```python
delay.head()
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
      <th>Origin</th>
      <th>Dest</th>
      <th>DepDelay</th>
      <th>ArrDelay</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>IAH</td>
      <td>DFW</td>
      <td>0.0</td>
      <td>-10.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>IAH</td>
      <td>DFW</td>
      <td>1.0</td>
      <td>-9.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>IAH</td>
      <td>DFW</td>
      <td>-8.0</td>
      <td>-8.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>IAH</td>
      <td>DFW</td>
      <td>3.0</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>IAH</td>
      <td>DFW</td>
      <td>5.0</td>
      <td>-3.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_delay = delay.groupby(['Origin','Dest']).agg({'DepDelay':['mean'],'ArrDelay':['mean']}).reset_index()
```


```python
df_delay.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>Origin</th>
      <th>Dest</th>
      <th>DepDelay</th>
      <th>ArrDelay</th>
    </tr>
    <tr>
      <th></th>
      <th></th>
      <th></th>
      <th>mean</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>HOU</td>
      <td>ABQ</td>
      <td>11.581854</td>
      <td>6.000987</td>
    </tr>
    <tr>
      <th>1</th>
      <td>HOU</td>
      <td>ATL</td>
      <td>9.129112</td>
      <td>6.810014</td>
    </tr>
    <tr>
      <th>2</th>
      <td>HOU</td>
      <td>AUS</td>
      <td>12.188736</td>
      <td>9.274145</td>
    </tr>
    <tr>
      <th>3</th>
      <td>HOU</td>
      <td>BHM</td>
      <td>15.014599</td>
      <td>6.672540</td>
    </tr>
    <tr>
      <th>4</th>
      <td>HOU</td>
      <td>BKG</td>
      <td>-3.201835</td>
      <td>-16.233645</td>
    </tr>
  </tbody>
</table>
</div>



**또는 아래와 같이 한줄로! 동일한 결과를 얻을 수 있다:**


```python
hflight.groupby(['Origin', 'Dest'])[['DepDelay', 'ArrDelay']].mean()
```

## 문제 5
### 목적지 공항에 대해 연착 건수를 구하고, 
### 연착 건수가 2000회 이상인 공항에 대한 데이터만 추출 
### 참고할 컬럼명 ->  Dest :목적지 공항 ArrDelay (연착은 5분이상)


```python
df_delay_5over = hflight.loc[(hflight.ArrDelay >= 5)]

df_delay_5over.head()
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
      <th>Year</th>
      <th>Month</th>
      <th>DayofMonth</th>
      <th>DayOfWeek</th>
      <th>DepTime</th>
      <th>ArrTime</th>
      <th>UniqueCarrier</th>
      <th>FlightNum</th>
      <th>TailNum</th>
      <th>ActualElapsedTime</th>
      <th>...</th>
      <th>ArrDelay</th>
      <th>DepDelay</th>
      <th>Origin</th>
      <th>Dest</th>
      <th>Distance</th>
      <th>TaxiIn</th>
      <th>TaxiOut</th>
      <th>Cancelled</th>
      <th>CancellationCode</th>
      <th>Diverted</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>8</th>
      <td>2011</td>
      <td>1</td>
      <td>9</td>
      <td>7</td>
      <td>1443.0</td>
      <td>1554.0</td>
      <td>AA</td>
      <td>428</td>
      <td>N476AA</td>
      <td>71.0</td>
      <td>...</td>
      <td>44.0</td>
      <td>43.0</td>
      <td>IAH</td>
      <td>DFW</td>
      <td>224</td>
      <td>8.0</td>
      <td>22.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2011</td>
      <td>1</td>
      <td>10</td>
      <td>1</td>
      <td>1443.0</td>
      <td>1553.0</td>
      <td>AA</td>
      <td>428</td>
      <td>N504AA</td>
      <td>70.0</td>
      <td>...</td>
      <td>43.0</td>
      <td>43.0</td>
      <td>IAH</td>
      <td>DFW</td>
      <td>224</td>
      <td>6.0</td>
      <td>19.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2011</td>
      <td>1</td>
      <td>11</td>
      <td>2</td>
      <td>1429.0</td>
      <td>1539.0</td>
      <td>AA</td>
      <td>428</td>
      <td>N565AA</td>
      <td>70.0</td>
      <td>...</td>
      <td>29.0</td>
      <td>29.0</td>
      <td>IAH</td>
      <td>DFW</td>
      <td>224</td>
      <td>8.0</td>
      <td>20.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2011</td>
      <td>1</td>
      <td>12</td>
      <td>3</td>
      <td>1419.0</td>
      <td>1515.0</td>
      <td>AA</td>
      <td>428</td>
      <td>N577AA</td>
      <td>56.0</td>
      <td>...</td>
      <td>5.0</td>
      <td>19.0</td>
      <td>IAH</td>
      <td>DFW</td>
      <td>224</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2011</td>
      <td>1</td>
      <td>17</td>
      <td>1</td>
      <td>1530.0</td>
      <td>1634.0</td>
      <td>AA</td>
      <td>428</td>
      <td>N518AA</td>
      <td>64.0</td>
      <td>...</td>
      <td>84.0</td>
      <td>90.0</td>
      <td>IAH</td>
      <td>DFW</td>
      <td>224</td>
      <td>8.0</td>
      <td>8.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>




```python
result = df_delay_5over.groupby(['Dest']).count()
result = result.loc[result.ArrDelay >= 2000]
```


```python
result.reset_index(inplace=True)
result = result.rename(columns = {'index':'Dest'})
```


```python
df = df_delay_5over.merge(result['Dest'])
```


```python
df.Dest.unique()
```




    array(['LAX', 'DEN', 'MSY', 'ORD', 'ATL', 'DAL'], dtype=object)




```python
hflight2 = hflight[hflight.Dest.isin(df['Dest'])]
```


```python
hflight2.head()
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
      <th>Year</th>
      <th>Month</th>
      <th>DayofMonth</th>
      <th>DayOfWeek</th>
      <th>DepTime</th>
      <th>ArrTime</th>
      <th>UniqueCarrier</th>
      <th>FlightNum</th>
      <th>TailNum</th>
      <th>ActualElapsedTime</th>
      <th>...</th>
      <th>ArrDelay</th>
      <th>DepDelay</th>
      <th>Origin</th>
      <th>Dest</th>
      <th>Distance</th>
      <th>TaxiIn</th>
      <th>TaxiOut</th>
      <th>Cancelled</th>
      <th>CancellationCode</th>
      <th>Diverted</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>356</th>
      <td>2011</td>
      <td>1</td>
      <td>31</td>
      <td>1</td>
      <td>1825.0</td>
      <td>1925.0</td>
      <td>CO</td>
      <td>5</td>
      <td>N17245</td>
      <td>60.0</td>
      <td>...</td>
      <td>-9.0</td>
      <td>0.0</td>
      <td>IAH</td>
      <td>MSY</td>
      <td>305</td>
      <td>3.0</td>
      <td>15.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>358</th>
      <td>2011</td>
      <td>1</td>
      <td>31</td>
      <td>1</td>
      <td>1522.0</td>
      <td>1632.0</td>
      <td>CO</td>
      <td>33</td>
      <td>N16647</td>
      <td>70.0</td>
      <td>...</td>
      <td>-2.0</td>
      <td>-3.0</td>
      <td>IAH</td>
      <td>MSY</td>
      <td>305</td>
      <td>7.0</td>
      <td>21.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>360</th>
      <td>2011</td>
      <td>1</td>
      <td>31</td>
      <td>1</td>
      <td>1916.0</td>
      <td>2103.0</td>
      <td>CO</td>
      <td>47</td>
      <td>N76522</td>
      <td>227.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>6.0</td>
      <td>IAH</td>
      <td>LAX</td>
      <td>1379</td>
      <td>8.0</td>
      <td>20.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>361</th>
      <td>2011</td>
      <td>1</td>
      <td>31</td>
      <td>1</td>
      <td>747.0</td>
      <td>936.0</td>
      <td>CO</td>
      <td>52</td>
      <td>N67134</td>
      <td>229.0</td>
      <td>...</td>
      <td>5.0</td>
      <td>2.0</td>
      <td>IAH</td>
      <td>LAX</td>
      <td>1379</td>
      <td>11.0</td>
      <td>17.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>362</th>
      <td>2011</td>
      <td>1</td>
      <td>31</td>
      <td>1</td>
      <td>1803.0</td>
      <td>1927.0</td>
      <td>CO</td>
      <td>59</td>
      <td>N57870</td>
      <td>144.0</td>
      <td>...</td>
      <td>15.0</td>
      <td>28.0</td>
      <td>IAH</td>
      <td>DEN</td>
      <td>862</td>
      <td>12.0</td>
      <td>16.0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>



**또는 [['ArrDelay']].count()와 같이 2D array형태로 바로 count값을 얻을 수 있다:**


```python
mask1 = hflight['ArrDelay'] >= 5

hflight2 = hflight[mask1].groupby('Dest', as_index=False)[['Year']].count()

condition = hflight2[hflight2['Year'] >= 2000]['Dest']

hflight3 = hflight[hflight.Dest.isin(condition)]
```

## 문제 6
### 위의 결과를 바탕으로 목적지 공항 별 결항 횟수, 회항 횟수
### 운항 횟수를 구하시오 (Cancelled, Diverted, Air)
##### 운항 횟수는 결항과 회항을 제외한 것 


```python
cancel_count=hflight2.groupby(['Dest'])[['Cancelled']].sum()
cancel_count
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
      <th>Cancelled</th>
    </tr>
    <tr>
      <th>Dest</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ATL</th>
      <td>141</td>
    </tr>
    <tr>
      <th>DAL</th>
      <td>442</td>
    </tr>
    <tr>
      <th>DEN</th>
      <td>28</td>
    </tr>
    <tr>
      <th>LAX</th>
      <td>33</td>
    </tr>
    <tr>
      <th>MSY</th>
      <td>40</td>
    </tr>
    <tr>
      <th>ORD</th>
      <td>99</td>
    </tr>
  </tbody>
</table>
</div>




```python
divert_count=hflight2.groupby(['Dest'])[['Diverted']].sum()
divert_count
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
      <th>Diverted</th>
    </tr>
    <tr>
      <th>Dest</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ATL</th>
      <td>28</td>
    </tr>
    <tr>
      <th>DAL</th>
      <td>27</td>
    </tr>
    <tr>
      <th>DEN</th>
      <td>24</td>
    </tr>
    <tr>
      <th>LAX</th>
      <td>14</td>
    </tr>
    <tr>
      <th>MSY</th>
      <td>3</td>
    </tr>
    <tr>
      <th>ORD</th>
      <td>15</td>
    </tr>
  </tbody>
</table>
</div>




```python
all_count=hflight2.groupby(['Dest'])[['Diverted']].count()
all_count
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
      <th>Diverted</th>
    </tr>
    <tr>
      <th>Dest</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ATL</th>
      <td>7886</td>
    </tr>
    <tr>
      <th>DAL</th>
      <td>9820</td>
    </tr>
    <tr>
      <th>DEN</th>
      <td>5920</td>
    </tr>
    <tr>
      <th>LAX</th>
      <td>6064</td>
    </tr>
    <tr>
      <th>MSY</th>
      <td>6823</td>
    </tr>
    <tr>
      <th>ORD</th>
      <td>5748</td>
    </tr>
  </tbody>
</table>
</div>




```python
result = pd.concat([cancel_count,divert_count,all_count], axis=1)
result
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
      <th>Cancelled</th>
      <th>Diverted</th>
      <th>Diverted</th>
    </tr>
    <tr>
      <th>Dest</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ATL</th>
      <td>141</td>
      <td>28</td>
      <td>7886</td>
    </tr>
    <tr>
      <th>DAL</th>
      <td>442</td>
      <td>27</td>
      <td>9820</td>
    </tr>
    <tr>
      <th>DEN</th>
      <td>28</td>
      <td>24</td>
      <td>5920</td>
    </tr>
    <tr>
      <th>LAX</th>
      <td>33</td>
      <td>14</td>
      <td>6064</td>
    </tr>
    <tr>
      <th>MSY</th>
      <td>40</td>
      <td>3</td>
      <td>6823</td>
    </tr>
    <tr>
      <th>ORD</th>
      <td>99</td>
      <td>15</td>
      <td>5748</td>
    </tr>
  </tbody>
</table>
</div>




```python
result['Air'] = result.apply(lambda x: x[2]-(x[1]+x[0]), axis=1)
```


```python
result.columns=['Cancelled','Diverted','all','Air']
result
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
      <th>Cancelled</th>
      <th>Diverted</th>
      <th>all</th>
      <th>Air</th>
    </tr>
    <tr>
      <th>Dest</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ATL</th>
      <td>141</td>
      <td>28</td>
      <td>7886</td>
      <td>7717</td>
    </tr>
    <tr>
      <th>DAL</th>
      <td>442</td>
      <td>27</td>
      <td>9820</td>
      <td>9351</td>
    </tr>
    <tr>
      <th>DEN</th>
      <td>28</td>
      <td>24</td>
      <td>5920</td>
      <td>5868</td>
    </tr>
    <tr>
      <th>LAX</th>
      <td>33</td>
      <td>14</td>
      <td>6064</td>
      <td>6017</td>
    </tr>
    <tr>
      <th>MSY</th>
      <td>40</td>
      <td>3</td>
      <td>6823</td>
      <td>6780</td>
    </tr>
    <tr>
      <th>ORD</th>
      <td>99</td>
      <td>15</td>
      <td>5748</td>
      <td>5634</td>
    </tr>
  </tbody>
</table>
</div>




```python
result.drop('all', axis=1)
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
      <th>Cancelled</th>
      <th>Diverted</th>
      <th>Air</th>
    </tr>
    <tr>
      <th>Dest</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ATL</th>
      <td>141</td>
      <td>28</td>
      <td>7717</td>
    </tr>
    <tr>
      <th>DAL</th>
      <td>442</td>
      <td>27</td>
      <td>9351</td>
    </tr>
    <tr>
      <th>DEN</th>
      <td>28</td>
      <td>24</td>
      <td>5868</td>
    </tr>
    <tr>
      <th>LAX</th>
      <td>33</td>
      <td>14</td>
      <td>6017</td>
    </tr>
    <tr>
      <th>MSY</th>
      <td>40</td>
      <td>3</td>
      <td>6780</td>
    </tr>
    <tr>
      <th>ORD</th>
      <td>99</td>
      <td>15</td>
      <td>5634</td>
    </tr>
  </tbody>
</table>
</div>


