## 실습  
### Geographical data 시각화
1. 서울시 1년치 아파트 매매 값 하나의 dataframe으로 통합하기
2. 서울시 구 기준으로 아파트 평균 매매 값 구하기
3. 서울시 지도 위에 아파트 평균 매매 값 기준으로 heatmap그리기


```python
import pandas as pd
import numpy as np
import os
```


```python
df_list=[]
for roots, dirs, files in os.walk(r"C:\LSJ\2.데이터분석\realestate_seoul"):
    for file in files:
        df_list.append(pd.read_csv(roots +"/"+ file, header=[15], 
                                   encoding="euc-kr", low_memory=False))
```


```python
df = pd.concat(df_list)
df.head(5)
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
      <th>시군구</th>
      <th>번지</th>
      <th>본번</th>
      <th>부번</th>
      <th>단지명</th>
      <th>전용면적(㎡)</th>
      <th>계약년월</th>
      <th>계약일</th>
      <th>거래금액(만원)</th>
      <th>층</th>
      <th>건축년도</th>
      <th>도로명</th>
      <th>해제사유발생일</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강원도 강릉시 견소동</td>
      <td>202</td>
      <td>0202</td>
      <td>0.0</td>
      <td>송정한신</td>
      <td>43.380</td>
      <td>202001</td>
      <td>3</td>
      <td>12,000</td>
      <td>12</td>
      <td>1997</td>
      <td>경강로2539번길 8</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강원도 강릉시 견소동</td>
      <td>202</td>
      <td>0202</td>
      <td>0.0</td>
      <td>송정한신</td>
      <td>59.800</td>
      <td>202001</td>
      <td>15</td>
      <td>10,000</td>
      <td>3</td>
      <td>1997</td>
      <td>경강로2539번길 8</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강원도 강릉시 견소동</td>
      <td>202</td>
      <td>0202</td>
      <td>0.0</td>
      <td>송정한신</td>
      <td>84.945</td>
      <td>202001</td>
      <td>18</td>
      <td>13,500</td>
      <td>11</td>
      <td>1997</td>
      <td>경강로2539번길 8</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강원도 강릉시 견소동</td>
      <td>202</td>
      <td>0202</td>
      <td>0.0</td>
      <td>송정한신</td>
      <td>59.800</td>
      <td>202001</td>
      <td>18</td>
      <td>10,500</td>
      <td>12</td>
      <td>1997</td>
      <td>경강로2539번길 8</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강원도 강릉시 견소동</td>
      <td>202</td>
      <td>0202</td>
      <td>0.0</td>
      <td>송정한신</td>
      <td>116.175</td>
      <td>202001</td>
      <td>21</td>
      <td>19,000</td>
      <td>5</td>
      <td>1997</td>
      <td>경강로2539번길 8</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.shape
```




    (852001, 13)




```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 852001 entries, 0 to 86446
    Data columns (total 13 columns):
     #   Column    Non-Null Count   Dtype  
    ---  ------    --------------   -----  
     0   시군구       852001 non-null  object 
     1   번지        851975 non-null  object 
     2   본번        851990 non-null  object 
     3   부번        851990 non-null  float64
     4   단지명       852001 non-null  object 
     5   전용면적(㎡)   852001 non-null  float64
     6   계약년월      852001 non-null  int64  
     7   계약일       852001 non-null  int64  
     8   거래금액(만원)  852001 non-null  object 
     9   층         852001 non-null  int64  
     10  건축년도      852001 non-null  int64  
     11  도로명       852001 non-null  object 
     12  해제사유발생일   38511 non-null   float64
    dtypes: float64(3), int64(4), object(6)
    memory usage: 91.0+ MB
    


```python
# 거래금액의 천단위 comma 제거 및 거리금액기준으로 데이터 sorting
df['거래금액(만원)'] = df['거래금액(만원)'].apply(lambda x : int(x.replace(",", "")))
df.sort_values(by = ['거래금액(만원)'], ascending=False, inplace=True)
```


```python
# 서울 데이터만 추출
seoul_df = df[df.시군구.str.contains('서울')].copy()
seoul_df.shape
```




    (83532, 13)




```python
# 시군구 컬럼에서 구에 해당하는 부분만 분리해서 새로운 컬럼 생성
seoul_df['구'] = seoul_df['시군구'].apply(lambda x : x.split()[1])
seoul_df.head(3)
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
      <th>시군구</th>
      <th>번지</th>
      <th>본번</th>
      <th>부번</th>
      <th>단지명</th>
      <th>전용면적(㎡)</th>
      <th>계약년월</th>
      <th>계약일</th>
      <th>거래금액(만원)</th>
      <th>층</th>
      <th>건축년도</th>
      <th>도로명</th>
      <th>해제사유발생일</th>
      <th>구</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>40134</th>
      <td>서울특별시 용산구 한남동</td>
      <td>810</td>
      <td>0810</td>
      <td>0.0</td>
      <td>한남더힐</td>
      <td>243.642</td>
      <td>202009</td>
      <td>4</td>
      <td>775000</td>
      <td>1</td>
      <td>2011</td>
      <td>독서당로 111</td>
      <td>NaN</td>
      <td>용산구</td>
    </tr>
    <tr>
      <th>72082</th>
      <td>서울특별시 용산구 한남동</td>
      <td>810</td>
      <td>0810</td>
      <td>0.0</td>
      <td>한남더힐</td>
      <td>241.052</td>
      <td>202011</td>
      <td>9</td>
      <td>760000</td>
      <td>-1</td>
      <td>2011</td>
      <td>독서당로 111</td>
      <td>NaN</td>
      <td>용산구</td>
    </tr>
    <tr>
      <th>40135</th>
      <td>서울특별시 용산구 한남동</td>
      <td>810</td>
      <td>0810</td>
      <td>0.0</td>
      <td>한남더힐</td>
      <td>240.230</td>
      <td>202009</td>
      <td>21</td>
      <td>730000</td>
      <td>3</td>
      <td>2011</td>
      <td>독서당로 111</td>
      <td>NaN</td>
      <td>용산구</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 구별로 구분하여 basic statistics 값을 확인 
seoul_gu = seoul_df.groupby(['구'], as_index=False)['거래금액(만원)'].agg(['mean',
                                                    'max', 'min', 'std'])

# 구 컬럼이 index가 되지않도록 reset_index
seoul_gu.reset_index(inplace=True)

seoul_gu
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
      <th>구</th>
      <th>mean</th>
      <th>max</th>
      <th>min</th>
      <th>std</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강남구</td>
      <td>180793.334296</td>
      <td>670000</td>
      <td>15000</td>
      <td>95857.318288</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강동구</td>
      <td>83895.275214</td>
      <td>249000</td>
      <td>8000</td>
      <td>36894.115216</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강북구</td>
      <td>55995.924039</td>
      <td>128000</td>
      <td>7500</td>
      <td>18462.817813</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강서구</td>
      <td>67769.541724</td>
      <td>162000</td>
      <td>7700</td>
      <td>26646.292703</td>
    </tr>
    <tr>
      <th>4</th>
      <td>관악구</td>
      <td>61873.218365</td>
      <td>136000</td>
      <td>9000</td>
      <td>20990.492035</td>
    </tr>
    <tr>
      <th>5</th>
      <td>광진구</td>
      <td>106792.921875</td>
      <td>269500</td>
      <td>8500</td>
      <td>42803.658600</td>
    </tr>
    <tr>
      <th>6</th>
      <td>구로구</td>
      <td>54513.885652</td>
      <td>170000</td>
      <td>6000</td>
      <td>25607.255537</td>
    </tr>
    <tr>
      <th>7</th>
      <td>금천구</td>
      <td>46959.113811</td>
      <td>125000</td>
      <td>8000</td>
      <td>22214.725826</td>
    </tr>
    <tr>
      <th>8</th>
      <td>노원구</td>
      <td>52225.097713</td>
      <td>157000</td>
      <td>8000</td>
      <td>19728.118687</td>
    </tr>
    <tr>
      <th>9</th>
      <td>도봉구</td>
      <td>45738.368514</td>
      <td>140000</td>
      <td>6500</td>
      <td>18206.688270</td>
    </tr>
    <tr>
      <th>10</th>
      <td>동대문구</td>
      <td>71150.548959</td>
      <td>165000</td>
      <td>8500</td>
      <td>28144.818152</td>
    </tr>
    <tr>
      <th>11</th>
      <td>동작구</td>
      <td>97796.615986</td>
      <td>290000</td>
      <td>12400</td>
      <td>31696.478721</td>
    </tr>
    <tr>
      <th>12</th>
      <td>마포구</td>
      <td>101331.594440</td>
      <td>240000</td>
      <td>13300</td>
      <td>36852.202912</td>
    </tr>
    <tr>
      <th>13</th>
      <td>서대문구</td>
      <td>78501.623668</td>
      <td>176000</td>
      <td>11000</td>
      <td>31778.589972</td>
    </tr>
    <tr>
      <th>14</th>
      <td>서초구</td>
      <td>175100.116950</td>
      <td>550000</td>
      <td>15000</td>
      <td>87090.356300</td>
    </tr>
    <tr>
      <th>15</th>
      <td>성동구</td>
      <td>114389.284577</td>
      <td>670000</td>
      <td>12050</td>
      <td>47467.009148</td>
    </tr>
    <tr>
      <th>16</th>
      <td>성북구</td>
      <td>68732.441661</td>
      <td>149000</td>
      <td>11500</td>
      <td>20816.994439</td>
    </tr>
    <tr>
      <th>17</th>
      <td>송파구</td>
      <td>130284.833333</td>
      <td>410000</td>
      <td>18000</td>
      <td>55118.970308</td>
    </tr>
    <tr>
      <th>18</th>
      <td>양천구</td>
      <td>88919.382276</td>
      <td>294500</td>
      <td>11450</td>
      <td>48943.192604</td>
    </tr>
    <tr>
      <th>19</th>
      <td>영등포구</td>
      <td>89860.834458</td>
      <td>324500</td>
      <td>9000</td>
      <td>42438.479596</td>
    </tr>
    <tr>
      <th>20</th>
      <td>용산구</td>
      <td>152238.629849</td>
      <td>775000</td>
      <td>15000</td>
      <td>89971.568454</td>
    </tr>
    <tr>
      <th>21</th>
      <td>은평구</td>
      <td>62468.976364</td>
      <td>153000</td>
      <td>8510</td>
      <td>24038.781279</td>
    </tr>
    <tr>
      <th>22</th>
      <td>종로구</td>
      <td>82773.158520</td>
      <td>235000</td>
      <td>8186</td>
      <td>48041.267772</td>
    </tr>
    <tr>
      <th>23</th>
      <td>중구</td>
      <td>92208.706981</td>
      <td>304000</td>
      <td>8300</td>
      <td>36787.271691</td>
    </tr>
    <tr>
      <th>24</th>
      <td>중랑구</td>
      <td>50977.233298</td>
      <td>160000</td>
      <td>8300</td>
      <td>19246.727155</td>
    </tr>
  </tbody>
</table>
</div>



### 지도 그리기

지도를 그리기위해 서울시 행정구역 위치가 설정되어있는 json 파일을 활용한다. 

파일 출처: https://github.com/southkorea/southkorea-maps/blob/master/kostat/2013/json/skorea_municipalities_geo_simple.json


```python
import json
import folium
import warnings
warnings.simplefilter(action = "ignore", category = FutureWarning)
folder =r"C:\LSJ\2.데이터분석\seoul"
geo_path = folder+'./skorea_municipalities_geo_simple.json'
geo_str = json.load(open(geo_path, encoding='utf-8'))
```

geo_str 변수에 json 데이터를 로드해서 다음과 같이 각 구의 geographical coordinates 값을 모아놓은 feature collection을 얻는다.

  {'type': 'Feature',
  
   'id': '송파구',
   
   'properties': {'code': '11240',
   
   'name': '송파구',
    
   'name_eng': 'Songpa-gu',
    
   'base_year': '2013'},
    
   'geometry': {'type': 'Polygon',
   
   'coordinates': [[[127.0690698130372, 37.522279423505026],
      [127.10087519791962, 37.524841220167055],
      [127.1116764203608, 37.540669955324965],
      [127.12123165719615, 37.52528270089],
      [127.14672806823502, 37.51415680680291],
      [127.1634944215765, 37.497445406097484],
      [127.14206058413274, 37.47089819098501],
      [127.12440571080893, 37.46240445587048],
      [127.11117085201238, 37.485708381512445],
      [127.0719146000724, 37.50224013587669],
      [127.0690698130372, 37.522279423505026]]]}},


```python
map = folium.Map(location=[37.5502, 126.982], zoom_start=11, tiles='Stamen Toner')
```


### 서울시 구별 아파트매매 평균값의 heatmap을  map위에 그린다.

heatmap을 그리기위한 함수: **choropleth**

* choropleth 함수의 파라미터:
 - geo_data: json데이터에서 받아오는 구별 coordintes
 - data: 지도에 그리려는 데이터가있는 데이터프레임
 - columns: 지도에 그리려는 데이터 
 - fill color: heatmap의 색깔 scheme
 - key_on: geo_data에서 어떻게 feature를 접근할지 지정 (variable in the geo_data GeoJSON file to bind the data to) javascript object notation지정 형식
 
tips on using folium choropleth 자료:
1. plotly - Graphing Libraries
https://plotly.com/python/choropleth-maps/

2. folium map활용 예시
https://towardsdatascience.com/how-to-step-up-your-folium-choropleth-map-skills-17cf6de7c6fe



```python
map = folium.Map(location=[37.5502, 126.982], zoom_start=11, tiles='Stamen Toner')
map.choropleth(geo_data = geo_str,
               data = seoul_gu,
               columns = ['구', 'mean'],
               fill_color = 'PuRd', #PuRd, YlGnBu
               key_on = 'feature.id')
folium.LayerControl().add_to(map)
```



```python
# 얻은 결과를 html 형식으로 저장할 수 있다.
map.save("./2020_01_map_test.html")
```
