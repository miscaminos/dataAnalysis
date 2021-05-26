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


```python
map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=%3C%21DOCTYPE%20html%3E%0A%3Chead%3E%20%20%20%20%0A%20%20%20%20%3Cmeta%20http-equiv%3D%22content-type%22%20content%3D%22text/html%3B%20charset%3DUTF-8%22%20/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%3Cscript%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20L_NO_TOUCH%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20L_DISABLE_3D%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%3C/script%3E%0A%20%20%20%20%0A%20%20%20%20%3Cstyle%3Ehtml%2C%20body%20%7Bwidth%3A%20100%25%3Bheight%3A%20100%25%3Bmargin%3A%200%3Bpadding%3A%200%3B%7D%3C/style%3E%0A%20%20%20%20%3Cstyle%3E%23map%20%7Bposition%3Aabsolute%3Btop%3A0%3Bbottom%3A0%3Bright%3A0%3Bleft%3A0%3B%7D%3C/style%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//code.jquery.com/jquery-1.12.4.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js%22%3E%3C/script%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css%22/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cmeta%20name%3D%22viewport%22%20content%3D%22width%3Ddevice-width%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20initial-scale%3D1.0%2C%20maximum-scale%3D1.0%2C%20user-scalable%3Dno%22%20/%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%23map_ec9c5d4f84324227a6b306f82f8be8e6%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20position%3A%20relative%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20width%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20height%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20left%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20top%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%0A%3C/head%3E%0A%3Cbody%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cdiv%20class%3D%22folium-map%22%20id%3D%22map_ec9c5d4f84324227a6b306f82f8be8e6%22%20%3E%3C/div%3E%0A%20%20%20%20%20%20%20%20%0A%3C/body%3E%0A%3Cscript%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20map_ec9c5d4f84324227a6b306f82f8be8e6%20%3D%20L.map%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22map_ec9c5d4f84324227a6b306f82f8be8e6%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20center%3A%20%5B37.5502%2C%20126.982%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20crs%3A%20L.CRS.EPSG3857%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoom%3A%2011%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoomControl%3A%20true%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20preferCanvas%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20tile_layer_016dc166510c41f5aa8628a229680e7d%20%3D%20L.tileLayer%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22https%3A//stamen-tiles-%7Bs%7D.a.ssl.fastly.net/toner/%7Bz%7D/%7Bx%7D/%7By%7D.png%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%22attribution%22%3A%20%22Map%20tiles%20by%20%5Cu003ca%20href%3D%5C%22http%3A//stamen.com%5C%22%5Cu003eStamen%20Design%5Cu003c/a%5Cu003e%2C%20under%20%5Cu003ca%20href%3D%5C%22http%3A//creativecommons.org/licenses/by/3.0%5C%22%5Cu003eCC%20BY%203.0%5Cu003c/a%5Cu003e.%20Data%20by%20%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//openstreetmap.org%5C%22%5Cu003eOpenStreetMap%5Cu003c/a%5Cu003e%2C%20under%20%5Cu003ca%20href%3D%5C%22http%3A//www.openstreetmap.org/copyright%5C%22%5Cu003eODbL%5Cu003c/a%5Cu003e.%22%2C%20%22detectRetina%22%3A%20false%2C%20%22maxNativeZoom%22%3A%2018%2C%20%22maxZoom%22%3A%2018%2C%20%22minZoom%22%3A%200%2C%20%22noWrap%22%3A%20false%2C%20%22opacity%22%3A%201%2C%20%22subdomains%22%3A%20%22abc%22%2C%20%22tms%22%3A%20false%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_ec9c5d4f84324227a6b306f82f8be8e6%29%3B%0A%20%20%20%20%20%20%20%20%0A%3C/script%3E onload="this.contentDocument.open();this.contentDocument.write(    decodeURIComponent(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



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




    <folium.map.LayerControl at 0x263ed875908>




```python
map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=%3C%21DOCTYPE%20html%3E%0A%3Chead%3E%20%20%20%20%0A%20%20%20%20%3Cmeta%20http-equiv%3D%22content-type%22%20content%3D%22text/html%3B%20charset%3DUTF-8%22%20/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%3Cscript%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20L_NO_TOUCH%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20L_DISABLE_3D%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%3C/script%3E%0A%20%20%20%20%0A%20%20%20%20%3Cstyle%3Ehtml%2C%20body%20%7Bwidth%3A%20100%25%3Bheight%3A%20100%25%3Bmargin%3A%200%3Bpadding%3A%200%3B%7D%3C/style%3E%0A%20%20%20%20%3Cstyle%3E%23map%20%7Bposition%3Aabsolute%3Btop%3A0%3Bbottom%3A0%3Bright%3A0%3Bleft%3A0%3B%7D%3C/style%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//code.jquery.com/jquery-1.12.4.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js%22%3E%3C/script%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css%22/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cmeta%20name%3D%22viewport%22%20content%3D%22width%3Ddevice-width%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20initial-scale%3D1.0%2C%20maximum-scale%3D1.0%2C%20user-scalable%3Dno%22%20/%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%23map_976d5813110e4e1aa4a466ca1f688fc5%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20position%3A%20relative%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20width%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20height%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20left%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20top%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js%22%3E%3C/script%3E%0A%3C/head%3E%0A%3Cbody%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cdiv%20class%3D%22folium-map%22%20id%3D%22map_976d5813110e4e1aa4a466ca1f688fc5%22%20%3E%3C/div%3E%0A%20%20%20%20%20%20%20%20%0A%3C/body%3E%0A%3Cscript%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20map_976d5813110e4e1aa4a466ca1f688fc5%20%3D%20L.map%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22map_976d5813110e4e1aa4a466ca1f688fc5%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20center%3A%20%5B37.5502%2C%20126.982%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20crs%3A%20L.CRS.EPSG3857%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoom%3A%2011%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoomControl%3A%20true%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20preferCanvas%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20tile_layer_19ca6284efbf4a6ca8c3dad93f630138%20%3D%20L.tileLayer%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22https%3A//stamen-tiles-%7Bs%7D.a.ssl.fastly.net/toner/%7Bz%7D/%7Bx%7D/%7By%7D.png%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%22attribution%22%3A%20%22Map%20tiles%20by%20%5Cu003ca%20href%3D%5C%22http%3A//stamen.com%5C%22%5Cu003eStamen%20Design%5Cu003c/a%5Cu003e%2C%20under%20%5Cu003ca%20href%3D%5C%22http%3A//creativecommons.org/licenses/by/3.0%5C%22%5Cu003eCC%20BY%203.0%5Cu003c/a%5Cu003e.%20Data%20by%20%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//openstreetmap.org%5C%22%5Cu003eOpenStreetMap%5Cu003c/a%5Cu003e%2C%20under%20%5Cu003ca%20href%3D%5C%22http%3A//www.openstreetmap.org/copyright%5C%22%5Cu003eODbL%5Cu003c/a%5Cu003e.%22%2C%20%22detectRetina%22%3A%20false%2C%20%22maxNativeZoom%22%3A%2018%2C%20%22maxZoom%22%3A%2018%2C%20%22minZoom%22%3A%200%2C%20%22noWrap%22%3A%20false%2C%20%22opacity%22%3A%201%2C%20%22subdomains%22%3A%20%22abc%22%2C%20%22tms%22%3A%20false%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_976d5813110e4e1aa4a466ca1f688fc5%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20choropleth_b9ecb66e73c94d77bd888ff2ba8649d1%20%3D%20L.featureGroup%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_976d5813110e4e1aa4a466ca1f688fc5%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20function%20geo_json_eccbe020563a454aadff403819c77fd6_styler%28feature%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20switch%28feature.id%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%22%5Cuac15%5Cub3d9%5Cuad6c%22%3A%20case%20%22%5Cuc601%5Cub4f1%5Cud3ec%5Cuad6c%22%3A%20case%20%22%5Cuc591%5Cucc9c%5Cuad6c%22%3A%20case%20%22%5Cuc11c%5Cub300%5Cubb38%5Cuad6c%22%3A%20case%20%22%5Cuc131%5Cubd81%5Cuad6c%22%3A%20case%20%22%5Cub3d9%5Cub300%5Cubb38%5Cuad6c%22%3A%20case%20%22%5Cuc885%5Cub85c%5Cuad6c%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23d4b9da%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%22%5Cuc1a1%5Cud30c%5Cuad6c%22%3A%20case%20%22%5Cuc131%5Cub3d9%5Cuad6c%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23df65b0%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%22%5Cuac15%5Cub0a8%5Cuad6c%22%3A%20case%20%22%5Cuc11c%5Cucd08%5Cuad6c%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23980043%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%22%5Cub3d9%5Cuc791%5Cuad6c%22%3A%20case%20%22%5Cub9c8%5Cud3ec%5Cuad6c%22%3A%20case%20%22%5Cuad11%5Cuc9c4%5Cuad6c%22%3A%20case%20%22%5Cuc911%5Cuad6c%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23c994c7%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%22%5Cuc6a9%5Cuc0b0%5Cuad6c%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23dd1c77%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23f1eef6%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_eccbe020563a454aadff403819c77fd6_onEachFeature%28feature%2C%20layer%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20layer.on%28%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%29%3B%0A%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20var%20geo_json_eccbe020563a454aadff403819c77fd6%20%3D%20L.geoJson%28null%2C%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20onEachFeature%3A%20geo_json_eccbe020563a454aadff403819c77fd6_onEachFeature%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20style%3A%20geo_json_eccbe020563a454aadff403819c77fd6_styler%2C%0A%20%20%20%20%20%20%20%20%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_eccbe020563a454aadff403819c77fd6_add%20%28data%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_eccbe020563a454aadff403819c77fd6%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addData%28data%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addTo%28choropleth_b9ecb66e73c94d77bd888ff2ba8649d1%29%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_eccbe020563a454aadff403819c77fd6_add%28%7B%22features%22%3A%20%5B%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.11519584981606%2C%2037.557533180704915%5D%2C%20%5B127.16683184366129%2C%2037.57672487388627%5D%2C%20%5B127.18408792330152%2C%2037.55814280369575%5D%2C%20%5B127.16530984307447%2C%2037.54221851258693%5D%2C%20%5B127.14672806823502%2C%2037.51415680680291%5D%2C%20%5B127.12123165719615%2C%2037.52528270089%5D%2C%20%5B127.1116764203608%2C%2037.540669955324965%5D%2C%20%5B127.11519584981606%2C%2037.557533180704915%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuac15%5Cub3d9%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211250%22%2C%20%22name%22%3A%20%22%5Cuac15%5Cub3d9%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Gangdong-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.0690698130372%2C%2037.522279423505026%5D%2C%20%5B127.10087519791962%2C%2037.524841220167055%5D%2C%20%5B127.1116764203608%2C%2037.540669955324965%5D%2C%20%5B127.12123165719615%2C%2037.52528270089%5D%2C%20%5B127.14672806823502%2C%2037.51415680680291%5D%2C%20%5B127.1634944215765%2C%2037.497445406097484%5D%2C%20%5B127.14206058413274%2C%2037.47089819098501%5D%2C%20%5B127.12440571080893%2C%2037.46240445587048%5D%2C%20%5B127.11117085201238%2C%2037.485708381512445%5D%2C%20%5B127.0719146000724%2C%2037.50224013587669%5D%2C%20%5B127.0690698130372%2C%2037.522279423505026%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc1a1%5Cud30c%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211240%22%2C%20%22name%22%3A%20%22%5Cuc1a1%5Cud30c%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Songpa-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.05867359288398%2C%2037.52629974922568%5D%2C%20%5B127.0690698130372%2C%2037.522279423505026%5D%2C%20%5B127.0719146000724%2C%2037.50224013587669%5D%2C%20%5B127.11117085201238%2C%2037.485708381512445%5D%2C%20%5B127.12440571080893%2C%2037.46240445587048%5D%2C%20%5B127.09842759318751%2C%2037.45862253857461%5D%2C%20%5B127.08640440578156%2C%2037.472697935184655%5D%2C%20%5B127.0559170481904%2C%2037.4659228914077%5D%2C%20%5B127.03621915098798%2C%2037.48175802427603%5D%2C%20%5B127.01397119667513%2C%2037.52503988289669%5D%2C%20%5B127.02302831890559%2C%2037.53231899582663%5D%2C%20%5B127.05867359288398%2C%2037.52629974922568%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuac15%5Cub0a8%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211230%22%2C%20%22name%22%3A%20%22%5Cuac15%5Cub0a8%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Gangnam-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.01397119667513%2C%2037.52503988289669%5D%2C%20%5B127.03621915098798%2C%2037.48175802427603%5D%2C%20%5B127.0559170481904%2C%2037.4659228914077%5D%2C%20%5B127.08640440578156%2C%2037.472697935184655%5D%2C%20%5B127.09842759318751%2C%2037.45862253857461%5D%2C%20%5B127.09046928565951%2C%2037.44296826114185%5D%2C%20%5B127.06778107605433%2C%2037.426197424057314%5D%2C%20%5B127.04957232987142%2C%2037.42805836845694%5D%2C%20%5B127.03881782597922%2C%2037.45382039851715%5D%2C%20%5B126.99072073195462%2C%2037.455326143310025%5D%2C%20%5B126.98367668291802%2C%2037.473856492692086%5D%2C%20%5B126.98223807916081%2C%2037.509314966770326%5D%2C%20%5B127.01397119667513%2C%2037.52503988289669%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc11c%5Cucd08%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211220%22%2C%20%22name%22%3A%20%22%5Cuc11c%5Cucd08%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Seocho-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.98367668291802%2C%2037.473856492692086%5D%2C%20%5B126.99072073195462%2C%2037.455326143310025%5D%2C%20%5B126.96520439085143%2C%2037.438249784006246%5D%2C%20%5B126.95000001010182%2C%2037.43613451165719%5D%2C%20%5B126.93084408056525%2C%2037.447382928333994%5D%2C%20%5B126.9167728146601%2C%2037.45490566423789%5D%2C%20%5B126.90156094129895%2C%2037.47753842789901%5D%2C%20%5B126.90531975801812%2C%2037.48218087575429%5D%2C%20%5B126.94922661389508%2C%2037.49125437495649%5D%2C%20%5B126.9725891850662%2C%2037.472561363278125%5D%2C%20%5B126.98367668291802%2C%2037.473856492692086%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuad00%5Cuc545%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211210%22%2C%20%22name%22%3A%20%22%5Cuad00%5Cuc545%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Gwanak-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.98223807916081%2C%2037.509314966770326%5D%2C%20%5B126.98367668291802%2C%2037.473856492692086%5D%2C%20%5B126.9725891850662%2C%2037.472561363278125%5D%2C%20%5B126.94922661389508%2C%2037.49125437495649%5D%2C%20%5B126.90531975801812%2C%2037.48218087575429%5D%2C%20%5B126.92177893174825%2C%2037.494889877415176%5D%2C%20%5B126.92810628828279%2C%2037.51329595732015%5D%2C%20%5B126.95249990298159%2C%2037.51722500741813%5D%2C%20%5B126.98223807916081%2C%2037.509314966770326%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cub3d9%5Cuc791%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211200%22%2C%20%22name%22%3A%20%22%5Cub3d9%5Cuc791%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Dongjak-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.89184663862764%2C%2037.547373974997114%5D%2C%20%5B126.94566733083212%2C%2037.526617542453366%5D%2C%20%5B126.95249990298159%2C%2037.51722500741813%5D%2C%20%5B126.92810628828279%2C%2037.51329595732015%5D%2C%20%5B126.92177893174825%2C%2037.494889877415176%5D%2C%20%5B126.90531975801812%2C%2037.48218087575429%5D%2C%20%5B126.89594776782485%2C%2037.504675281309176%5D%2C%20%5B126.88156402353862%2C%2037.513970034765684%5D%2C%20%5B126.88825757860099%2C%2037.54079733630232%5D%2C%20%5B126.89184663862764%2C%2037.547373974997114%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc601%5Cub4f1%5Cud3ec%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211190%22%2C%20%22name%22%3A%20%22%5Cuc601%5Cub4f1%5Cud3ec%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Yeongdeungpo-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.90156094129895%2C%2037.47753842789901%5D%2C%20%5B126.9167728146601%2C%2037.45490566423789%5D%2C%20%5B126.93084408056525%2C%2037.447382928333994%5D%2C%20%5B126.9025831711697%2C%2037.434549366349124%5D%2C%20%5B126.87683271502428%2C%2037.482576591607305%5D%2C%20%5B126.90156094129895%2C%2037.47753842789901%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuae08%5Cucc9c%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211180%22%2C%20%22name%22%3A%20%22%5Cuae08%5Cucc9c%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Geumcheon-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.82688081517314%2C%2037.50548972232896%5D%2C%20%5B126.88156402353862%2C%2037.513970034765684%5D%2C%20%5B126.89594776782485%2C%2037.504675281309176%5D%2C%20%5B126.90531975801812%2C%2037.48218087575429%5D%2C%20%5B126.90156094129895%2C%2037.47753842789901%5D%2C%20%5B126.87683271502428%2C%2037.482576591607305%5D%2C%20%5B126.84762676054953%2C%2037.47146723936323%5D%2C%20%5B126.83549485076196%2C%2037.474098236975095%5D%2C%20%5B126.82264796791348%2C%2037.4878476492147%5D%2C%20%5B126.82504736331406%2C%2037.50302612640443%5D%2C%20%5B126.82688081517314%2C%2037.50548972232896%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuad6c%5Cub85c%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211170%22%2C%20%22name%22%3A%20%22%5Cuad6c%5Cub85c%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Guro-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.79575768552907%2C%2037.57881087633202%5D%2C%20%5B126.80702115023597%2C%2037.60123001013228%5D%2C%20%5B126.82251438477105%2C%2037.5880430810082%5D%2C%20%5B126.85984199399667%2C%2037.571847855292745%5D%2C%20%5B126.89184663862764%2C%2037.547373974997114%5D%2C%20%5B126.88825757860099%2C%2037.54079733630232%5D%2C%20%5B126.86637464321238%2C%2037.54859191094823%5D%2C%20%5B126.86610073476395%2C%2037.52699964144669%5D%2C%20%5B126.84257291943153%2C%2037.52373707805596%5D%2C%20%5B126.8242331426722%2C%2037.53788078753248%5D%2C%20%5B126.77324417717703%2C%2037.5459123450554%5D%2C%20%5B126.76979180579352%2C%2037.55139183008809%5D%2C%20%5B126.79575768552907%2C%2037.57881087633202%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuac15%5Cuc11c%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211160%22%2C%20%22name%22%3A%20%22%5Cuac15%5Cuc11c%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Gangseo-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.8242331426722%2C%2037.53788078753248%5D%2C%20%5B126.84257291943153%2C%2037.52373707805596%5D%2C%20%5B126.86610073476395%2C%2037.52699964144669%5D%2C%20%5B126.86637464321238%2C%2037.54859191094823%5D%2C%20%5B126.88825757860099%2C%2037.54079733630232%5D%2C%20%5B126.88156402353862%2C%2037.513970034765684%5D%2C%20%5B126.82688081517314%2C%2037.50548972232896%5D%2C%20%5B126.8242331426722%2C%2037.53788078753248%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc591%5Cucc9c%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211150%22%2C%20%22name%22%3A%20%22%5Cuc591%5Cucc9c%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Yangcheon-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.90522065831053%2C%2037.57409700522574%5D%2C%20%5B126.93898161798973%2C%2037.552310003728124%5D%2C%20%5B126.96358226710812%2C%2037.55605635475154%5D%2C%20%5B126.96448570553055%2C%2037.548705692021635%5D%2C%20%5B126.94566733083212%2C%2037.526617542453366%5D%2C%20%5B126.89184663862764%2C%2037.547373974997114%5D%2C%20%5B126.85984199399667%2C%2037.571847855292745%5D%2C%20%5B126.88433284773288%2C%2037.588143322880526%5D%2C%20%5B126.90522065831053%2C%2037.57409700522574%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cub9c8%5Cud3ec%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211140%22%2C%20%22name%22%3A%20%22%5Cub9c8%5Cud3ec%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Mapo-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.9524752030572%2C%2037.60508692737045%5D%2C%20%5B126.95565425846463%2C%2037.576080790881456%5D%2C%20%5B126.96873633279075%2C%2037.56313604690827%5D%2C%20%5B126.96358226710812%2C%2037.55605635475154%5D%2C%20%5B126.93898161798973%2C%2037.552310003728124%5D%2C%20%5B126.90522065831053%2C%2037.57409700522574%5D%2C%20%5B126.9524752030572%2C%2037.60508692737045%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc11c%5Cub300%5Cubb38%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211130%22%2C%20%22name%22%3A%20%22%5Cuc11c%5Cub300%5Cubb38%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Seodaemun-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.9738864128702%2C%2037.62949634786888%5D%2C%20%5B126.95427017006129%2C%2037.622033431339425%5D%2C%20%5B126.9524752030572%2C%2037.60508692737045%5D%2C%20%5B126.90522065831053%2C%2037.57409700522574%5D%2C%20%5B126.88433284773288%2C%2037.588143322880526%5D%2C%20%5B126.90396681003595%2C%2037.59227403419942%5D%2C%20%5B126.90303066177668%2C%2037.609977911401344%5D%2C%20%5B126.91455481429648%2C%2037.64150050996935%5D%2C%20%5B126.956473797387%2C%2037.652480737339445%5D%2C%20%5B126.9738864128702%2C%2037.62949634786888%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc740%5Cud3c9%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211120%22%2C%20%22name%22%3A%20%22%5Cuc740%5Cud3c9%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Eunpyeong-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.0838752703195%2C%2037.69359534202034%5D%2C%20%5B127.09706391309695%2C%2037.686383719372294%5D%2C%20%5B127.09440766298717%2C%2037.64713490473045%5D%2C%20%5B127.11326795855199%2C%2037.639622905315925%5D%2C%20%5B127.10782277688129%2C%2037.61804244241069%5D%2C%20%5B127.07351243825278%2C%2037.61283660342313%5D%2C%20%5B127.05209373568619%2C%2037.62164065487782%5D%2C%20%5B127.04358800895609%2C%2037.62848931298715%5D%2C%20%5B127.05800075220091%2C%2037.64318263878276%5D%2C%20%5B127.05288479710485%2C%2037.68423857084347%5D%2C%20%5B127.0838752703195%2C%2037.69359534202034%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cub178%5Cuc6d0%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211110%22%2C%20%22name%22%3A%20%22%5Cub178%5Cuc6d0%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Nowon-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.05288479710485%2C%2037.68423857084347%5D%2C%20%5B127.05800075220091%2C%2037.64318263878276%5D%2C%20%5B127.04358800895609%2C%2037.62848931298715%5D%2C%20%5B127.01465935892466%2C%2037.64943687496812%5D%2C%20%5B127.02062116141389%2C%2037.667173575971205%5D%2C%20%5B127.01039666042071%2C%2037.681894589603594%5D%2C%20%5B127.01795099203432%2C%2037.69824412775662%5D%2C%20%5B127.05288479710485%2C%2037.68423857084347%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cub3c4%5Cubd09%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211100%22%2C%20%22name%22%3A%20%22%5Cub3c4%5Cubd09%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Dobong-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.99383903424%2C%2037.676681761199085%5D%2C%20%5B127.01039666042071%2C%2037.681894589603594%5D%2C%20%5B127.02062116141389%2C%2037.667173575971205%5D%2C%20%5B127.01465935892466%2C%2037.64943687496812%5D%2C%20%5B127.04358800895609%2C%2037.62848931298715%5D%2C%20%5B127.05209373568619%2C%2037.62164065487782%5D%2C%20%5B127.03892400992301%2C%2037.609715611023816%5D%2C%20%5B127.0128154749523%2C%2037.613652243470256%5D%2C%20%5B126.98672705513869%2C%2037.63377641288196%5D%2C%20%5B126.9817452676551%2C%2037.65209769387776%5D%2C%20%5B126.99383903424%2C%2037.676681761199085%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuac15%5Cubd81%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211090%22%2C%20%22name%22%3A%20%22%5Cuac15%5Cubd81%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Gangbuk-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.977175406416%2C%2037.62859715400388%5D%2C%20%5B126.98672705513869%2C%2037.63377641288196%5D%2C%20%5B127.0128154749523%2C%2037.613652243470256%5D%2C%20%5B127.03892400992301%2C%2037.609715611023816%5D%2C%20%5B127.05209373568619%2C%2037.62164065487782%5D%2C%20%5B127.07351243825278%2C%2037.61283660342313%5D%2C%20%5B127.07382707099227%2C%2037.60401928986419%5D%2C%20%5B127.042705222094%2C%2037.59239437593391%5D%2C%20%5B127.02527254528003%2C%2037.57524616245249%5D%2C%20%5B126.99348293358314%2C%2037.588565457216156%5D%2C%20%5B126.98879865992384%2C%2037.6118927319756%5D%2C%20%5B126.977175406416%2C%2037.62859715400388%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc131%5Cubd81%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211080%22%2C%20%22name%22%3A%20%22%5Cuc131%5Cubd81%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Seongbuk-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.07351243825278%2C%2037.61283660342313%5D%2C%20%5B127.10782277688129%2C%2037.61804244241069%5D%2C%20%5B127.1201246020114%2C%2037.60178457598188%5D%2C%20%5B127.10304174249214%2C%2037.57076342290955%5D%2C%20%5B127.08068541280403%2C%2037.56906425519017%5D%2C%20%5B127.07382707099227%2C%2037.60401928986419%5D%2C%20%5B127.07351243825278%2C%2037.61283660342313%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc911%5Cub791%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211070%22%2C%20%22name%22%3A%20%22%5Cuc911%5Cub791%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Jungnang-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.02527254528003%2C%2037.57524616245249%5D%2C%20%5B127.042705222094%2C%2037.59239437593391%5D%2C%20%5B127.07382707099227%2C%2037.60401928986419%5D%2C%20%5B127.08068541280403%2C%2037.56906425519017%5D%2C%20%5B127.07421053024362%2C%2037.55724769712085%5D%2C%20%5B127.05005601081567%2C%2037.567577612590846%5D%2C%20%5B127.02547266349976%2C%2037.568943552237734%5D%2C%20%5B127.02527254528003%2C%2037.57524616245249%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cub3d9%5Cub300%5Cubb38%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211060%22%2C%20%22name%22%3A%20%22%5Cub3d9%5Cub300%5Cubb38%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Dongdaemun-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.08068541280403%2C%2037.56906425519017%5D%2C%20%5B127.10304174249214%2C%2037.57076342290955%5D%2C%20%5B127.11519584981606%2C%2037.557533180704915%5D%2C%20%5B127.1116764203608%2C%2037.540669955324965%5D%2C%20%5B127.10087519791962%2C%2037.524841220167055%5D%2C%20%5B127.0690698130372%2C%2037.522279423505026%5D%2C%20%5B127.05867359288398%2C%2037.52629974922568%5D%2C%20%5B127.07421053024362%2C%2037.55724769712085%5D%2C%20%5B127.08068541280403%2C%2037.56906425519017%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuad11%5Cuc9c4%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211050%22%2C%20%22name%22%3A%20%22%5Cuad11%5Cuc9c4%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Gwangjin-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.02547266349976%2C%2037.568943552237734%5D%2C%20%5B127.05005601081567%2C%2037.567577612590846%5D%2C%20%5B127.07421053024362%2C%2037.55724769712085%5D%2C%20%5B127.05867359288398%2C%2037.52629974922568%5D%2C%20%5B127.02302831890559%2C%2037.53231899582663%5D%2C%20%5B127.01070894177482%2C%2037.54118048964762%5D%2C%20%5B127.02547266349976%2C%2037.568943552237734%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc131%5Cub3d9%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211040%22%2C%20%22name%22%3A%20%22%5Cuc131%5Cub3d9%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Seongdong-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.01070894177482%2C%2037.54118048964762%5D%2C%20%5B127.02302831890559%2C%2037.53231899582663%5D%2C%20%5B127.01397119667513%2C%2037.52503988289669%5D%2C%20%5B126.98223807916081%2C%2037.509314966770326%5D%2C%20%5B126.95249990298159%2C%2037.51722500741813%5D%2C%20%5B126.94566733083212%2C%2037.526617542453366%5D%2C%20%5B126.96448570553055%2C%2037.548705692021635%5D%2C%20%5B126.98752996903328%2C%2037.55094818807139%5D%2C%20%5B127.01070894177482%2C%2037.54118048964762%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc6a9%5Cuc0b0%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211030%22%2C%20%22name%22%3A%20%22%5Cuc6a9%5Cuc0b0%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Yongsan-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.02547266349976%2C%2037.568943552237734%5D%2C%20%5B127.01070894177482%2C%2037.54118048964762%5D%2C%20%5B126.98752996903328%2C%2037.55094818807139%5D%2C%20%5B126.96448570553055%2C%2037.548705692021635%5D%2C%20%5B126.96358226710812%2C%2037.55605635475154%5D%2C%20%5B126.96873633279075%2C%2037.56313604690827%5D%2C%20%5B127.02547266349976%2C%2037.568943552237734%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc911%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211020%22%2C%20%22name%22%3A%20%22%5Cuc911%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Jung-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.9738864128702%2C%2037.62949634786888%5D%2C%20%5B126.977175406416%2C%2037.62859715400388%5D%2C%20%5B126.98879865992384%2C%2037.6118927319756%5D%2C%20%5B126.99348293358314%2C%2037.588565457216156%5D%2C%20%5B127.02527254528003%2C%2037.57524616245249%5D%2C%20%5B127.02547266349976%2C%2037.568943552237734%5D%2C%20%5B126.96873633279075%2C%2037.56313604690827%5D%2C%20%5B126.95565425846463%2C%2037.576080790881456%5D%2C%20%5B126.9524752030572%2C%2037.60508692737045%5D%2C%20%5B126.95427017006129%2C%2037.622033431339425%5D%2C%20%5B126.9738864128702%2C%2037.62949634786888%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%22%5Cuc885%5Cub85c%5Cuad6c%22%2C%20%22properties%22%3A%20%7B%22base_year%22%3A%20%222013%22%2C%20%22code%22%3A%20%2211010%22%2C%20%22name%22%3A%20%22%5Cuc885%5Cub85c%5Cuad6c%22%2C%20%22name_eng%22%3A%20%22Jongno-gu%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%5D%2C%20%22type%22%3A%20%22FeatureCollection%22%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20var%20color_map_24206dcf730244c29bfd2efd8e6d07e1%20%3D%20%7B%7D%3B%0A%0A%20%20%20%20%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.color%20%3D%20d3.scale.threshold%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B45738.36851363237%2C%2046009.01974766561%2C%2046279.670981698844%2C%2046550.322215732085%2C%2046820.973449765326%2C%2047091.62468379856%2C%2047362.2759178318%2C%2047632.92715186504%2C%2047903.578385898276%2C%2048174.22961993152%2C%2048444.88085396476%2C%2048715.532087998%2C%2048986.18332203123%2C%2049256.834556064474%2C%2049527.485790097715%2C%2049798.13702413095%2C%2050068.78825816419%2C%2050339.43949219743%2C%2050610.090726230665%2C%2050880.741960263906%2C%2051151.39319429715%2C%2051422.04442833038%2C%2051692.69566236362%2C%2051963.34689639686%2C%2052233.998130430096%2C%2052504.64936446334%2C%2052775.30059849658%2C%2053045.95183252981%2C%2053316.60306656305%2C%2053587.254300596294%2C%2053857.905534629535%2C%2054128.55676866277%2C%2054399.20800269601%2C%2054669.85923672925%2C%2054940.510470762485%2C%2055211.161704795726%2C%2055481.81293882897%2C%2055752.4641728622%2C%2056023.11540689544%2C%2056293.766640928676%2C%2056564.41787496192%2C%2056835.06910899516%2C%2057105.7203430284%2C%2057376.37157706164%2C%2057647.022811094874%2C%2057917.674045128115%2C%2058188.32527916135%2C%2058458.97651319459%2C%2058729.62774722783%2C%2059000.27898126107%2C%2059270.930215294306%2C%2059541.58144932755%2C%2059812.23268336078%2C%2060082.88391739402%2C%2060353.53515142726%2C%2060624.1863854605%2C%2060894.83761949374%2C%2061165.48885352698%2C%2061436.14008756021%2C%2061706.79132159345%2C%2061977.442555626694%2C%2062248.093789659935%2C%2062518.745023693176%2C%2062789.39625772641%2C%2063060.047491759644%2C%2063330.698725792885%2C%2063601.349959826126%2C%2063872.00119385937%2C%2064142.65242789261%2C%2064413.30366192584%2C%2064683.95489595908%2C%2064954.60612999232%2C%2065225.25736402556%2C%2065495.9085980588%2C%2065766.55983209204%2C%2066037.21106612528%2C%2066307.86230015851%2C%2066578.51353419176%2C%2066849.16476822499%2C%2067119.81600225823%2C%2067390.46723629147%2C%2067661.11847032471%2C%2067931.76970435795%2C%2068202.42093839118%2C%2068473.07217242442%2C%2068743.72340645766%2C%2069014.3746404909%2C%2069285.02587452414%2C%2069555.67710855739%2C%2069826.32834259061%2C%2070096.97957662385%2C%2070367.6308106571%2C%2070638.28204469034%2C%2070908.93327872358%2C%2071179.58451275682%2C%2071450.23574679004%2C%2071720.88698082328%2C%2071991.53821485653%2C%2072262.18944888977%2C%2072532.84068292301%2C%2072803.49191695623%2C%2073074.14315098949%2C%2073344.79438502272%2C%2073615.44561905596%2C%2073886.0968530892%2C%2074156.74808712244%2C%2074427.39932115568%2C%2074698.05055518891%2C%2074968.70178922216%2C%2075239.35302325539%2C%2075510.00425728863%2C%2075780.65549132187%2C%2076051.30672535511%2C%2076321.95795938835%2C%2076592.60919342158%2C%2076863.26042745482%2C%2077133.91166148806%2C%2077404.5628955213%2C%2077675.21412955454%2C%2077945.86536358779%2C%2078216.51659762103%2C%2078487.16783165425%2C%2078757.8190656875%2C%2079028.47029972074%2C%2079299.12153375398%2C%2079569.77276778722%2C%2079840.42400182044%2C%2080111.0752358537%2C%2080381.72646988693%2C%2080652.37770392017%2C%2080923.02893795341%2C%2081193.68017198665%2C%2081464.33140601989%2C%2081734.98264005312%2C%2082005.63387408636%2C%2082276.2851081196%2C%2082546.93634215284%2C%2082817.58757618608%2C%2083088.2388102193%2C%2083358.89004425256%2C%2083629.54127828579%2C%2083900.19251231904%2C%2084170.84374635227%2C%2084441.49498038551%2C%2084712.14621441875%2C%2084982.79744845198%2C%2085253.44868248522%2C%2085524.09991651846%2C%2085794.7511505517%2C%2086065.40238458494%2C%2086336.05361861819%2C%2086606.70485265143%2C%2086877.35608668465%2C%2087148.00732071791%2C%2087418.65855475113%2C%2087689.30978878436%2C%2087959.96102281762%2C%2088230.61225685084%2C%2088501.2634908841%2C%2088771.91472491733%2C%2089042.56595895057%2C%2089313.21719298381%2C%2089583.86842701705%2C%2089854.51966105029%2C%2090125.17089508352%2C%2090395.82212911677%2C%2090666.47336315%2C%2090937.12459718324%2C%2091207.77583121648%2C%2091478.4270652497%2C%2091749.07829928296%2C%2092019.72953331619%2C%2092290.38076734944%2C%2092561.03200138267%2C%2092831.68323541591%2C%2093102.33446944915%2C%2093372.9857034824%2C%2093643.63693751564%2C%2093914.28817154886%2C%2094184.9394055821%2C%2094455.59063961534%2C%2094726.24187364859%2C%2094996.89310768183%2C%2095267.54434171505%2C%2095538.19557574831%2C%2095808.84680978153%2C%2096079.49804381479%2C%2096350.14927784802%2C%2096620.80051188124%2C%2096891.4517459145%2C%2097162.10297994773%2C%2097432.75421398097%2C%2097703.40544801421%2C%2097974.05668204745%2C%2098244.70791608069%2C%2098515.35915011393%2C%2098786.01038414717%2C%2099056.6616181804%2C%2099327.31285221364%2C%2099597.96408624688%2C%2099868.6153202801%2C%20100139.26655431336%2C%20100409.91778834659%2C%20100680.56902237984%2C%20100951.22025641307%2C%20101221.87149044631%2C%20101492.52272447955%2C%20101763.1739585128%2C%20102033.82519254604%2C%20102304.47642657926%2C%20102575.1276606125%2C%20102845.77889464574%2C%20103116.43012867899%2C%20103387.08136271223%2C%20103657.73259674545%2C%20103928.38383077871%2C%20104199.03506481193%2C%20104469.68629884519%2C%20104740.33753287842%2C%20105010.98876691164%2C%20105281.6400009449%2C%20105552.29123497813%2C%20105822.94246901138%2C%20106093.59370304461%2C%20106364.24493707785%2C%20106634.89617111109%2C%20106905.54740514433%2C%20107176.19863917757%2C%20107446.8498732108%2C%20107717.50110724405%2C%20107988.15234127728%2C%20108258.80357531052%2C%20108529.45480934376%2C%20108800.10604337699%2C%20109070.75727741024%2C%20109341.40851144347%2C%20109612.05974547671%2C%20109882.71097950995%2C%20110153.3622135432%2C%20110424.01344757644%2C%20110694.66468160968%2C%20110965.3159156429%2C%20111235.96714967614%2C%20111506.61838370937%2C%20111777.26961774263%2C%20112047.92085177585%2C%20112318.57208580911%2C%20112589.22331984233%2C%20112859.87455387559%2C%20113130.52578790882%2C%20113401.17702194204%2C%20113671.8282559753%2C%20113942.47949000853%2C%20114213.13072404178%2C%20114483.78195807501%2C%20114754.43319210826%2C%20115025.08442614149%2C%20115295.73566017472%2C%20115566.38689420797%2C%20115837.0381282412%2C%20116107.68936227445%2C%20116378.34059630768%2C%20116648.99183034094%2C%20116919.64306437416%2C%20117190.29429840742%2C%20117460.94553244064%2C%20117731.59676647387%2C%20118002.24800050713%2C%20118272.89923454035%2C%20118543.55046857361%2C%20118814.20170260684%2C%20119084.85293664006%2C%20119355.50417067332%2C%20119626.15540470654%2C%20119896.80663873977%2C%20120167.45787277303%2C%20120438.10910680625%2C%20120708.76034083951%2C%20120979.41157487273%2C%20121250.06280890596%2C%20121520.71404293922%2C%20121791.36527697244%2C%20122062.0165110057%2C%20122332.66774503893%2C%20122603.31897907218%2C%20122873.97021310541%2C%20123144.62144713866%2C%20123415.27268117189%2C%20123685.92391520512%2C%20123956.57514923837%2C%20124227.2263832716%2C%20124497.87761730485%2C%20124768.52885133808%2C%20125039.18008537134%2C%20125309.83131940456%2C%20125580.48255343782%2C%20125851.13378747104%2C%20126121.78502150427%2C%20126392.43625553753%2C%20126663.08748957075%2C%20126933.73872360401%2C%20127204.38995763724%2C%20127475.04119167046%2C%20127745.69242570372%2C%20128016.34365973694%2C%20128286.9948937702%2C%20128557.64612780343%2C%20128828.29736183665%2C%20129098.94859586991%2C%20129369.59982990313%2C%20129640.25106393636%2C%20129910.90229796962%2C%20130181.55353200284%2C%20130452.2047660361%2C%20130722.85600006933%2C%20130993.50723410258%2C%20131264.1584681358%2C%20131534.80970216906%2C%20131805.4609362023%2C%20132076.11217023554%2C%20132346.76340426877%2C%20132617.414638302%2C%20132888.06587233525%2C%20133158.71710636848%2C%20133429.36834040174%2C%20133700.01957443496%2C%20133970.67080846822%2C%20134241.32204250144%2C%20134511.97327653467%2C%20134782.62451056793%2C%20135053.27574460115%2C%20135323.9269786344%2C%20135594.57821266763%2C%20135865.22944670086%2C%20136135.88068073412%2C%20136406.53191476734%2C%20136677.1831488006%2C%20136947.83438283383%2C%20137218.48561686705%2C%20137489.1368509003%2C%20137759.78808493353%2C%20138030.4393189668%2C%20138301.09055300002%2C%20138571.74178703324%2C%20138842.3930210665%2C%20139113.04425509972%2C%20139383.69548913298%2C%20139654.3467231662%2C%20139924.99795719946%2C%20140195.6491912327%2C%20140466.30042526594%2C%20140736.95165929917%2C%20141007.60289333243%2C%20141278.25412736565%2C%20141548.90536139888%2C%20141819.55659543214%2C%20142090.20782946536%2C%20142360.85906349862%2C%20142631.51029753184%2C%20142902.1615315651%2C%20143172.81276559833%2C%20143443.46399963155%2C%20143714.1152336648%2C%20143984.76646769803%2C%20144255.41770173126%2C%20144526.06893576452%2C%20144796.72016979774%2C%20145067.371403831%2C%20145338.02263786423%2C%20145608.67387189745%2C%20145879.3251059307%2C%20146149.97633996393%2C%20146420.6275739972%2C%20146691.27880803042%2C%20146961.93004206364%2C%20147232.5812760969%2C%20147503.23251013012%2C%20147773.88374416338%2C%20148044.5349781966%2C%20148315.18621222986%2C%20148585.8374462631%2C%20148856.48868029634%2C%20149127.13991432957%2C%20149397.79114836283%2C%20149668.44238239605%2C%20149939.09361642928%2C%20150209.74485046254%2C%20150480.39608449576%2C%20150751.04731852902%2C%20151021.69855256224%2C%20151292.3497865955%2C%20151563.00102062873%2C%20151833.65225466195%2C%20152104.3034886952%2C%20152374.95472272843%2C%20152645.60595676166%2C%20152916.25719079492%2C%20153186.90842482814%2C%20153457.5596588614%2C%20153728.21089289463%2C%20153998.86212692785%2C%20154269.5133609611%2C%20154540.16459499433%2C%20154810.8158290276%2C%20155081.46706306082%2C%20155352.11829709407%2C%20155622.7695311273%2C%20155893.42076516052%2C%20156164.07199919378%2C%20156434.723233227%2C%20156705.37446726026%2C%20156976.0257012935%2C%20157246.67693532674%2C%20157517.32816935997%2C%20157787.97940339323%2C%20158058.63063742645%2C%20158329.28187145968%2C%20158599.93310549294%2C%20158870.58433952616%2C%20159141.23557355942%2C%20159411.88680759264%2C%20159682.5380416259%2C%20159953.18927565913%2C%20160223.84050969235%2C%20160494.4917437256%2C%20160765.14297775883%2C%20161035.7942117921%2C%20161306.44544582532%2C%20161577.09667985854%2C%20161847.7479138918%2C%20162118.39914792503%2C%20162389.05038195825%2C%20162659.7016159915%2C%20162930.35285002473%2C%20163201.004084058%2C%20163471.65531809122%2C%20163742.30655212447%2C%20164012.9577861577%2C%20164283.60902019092%2C%20164554.26025422418%2C%20164824.9114882574%2C%20165095.56272229066%2C%20165366.2139563239%2C%20165636.86519035714%2C%20165907.51642439037%2C%20166178.16765842363%2C%20166448.81889245685%2C%20166719.4701264901%2C%20166990.12136052334%2C%20167260.77259455656%2C%20167531.42382858982%2C%20167802.07506262304%2C%20168072.7262966563%2C%20168343.37753068953%2C%20168614.02876472275%2C%20168884.679998756%2C%20169155.33123278923%2C%20169425.9824668225%2C%20169696.63370085572%2C%20169967.28493488894%2C%20170237.9361689222%2C%20170508.58740295543%2C%20170779.23863698868%2C%20171049.8898710219%2C%20171320.54110505513%2C%20171591.1923390884%2C%20171861.84357312162%2C%20172132.49480715487%2C%20172403.1460411881%2C%20172673.79727522135%2C%20172944.44850925458%2C%20173215.0997432878%2C%20173485.75097732106%2C%20173756.4022113543%2C%20174027.05344538754%2C%20174297.70467942077%2C%20174568.35591345403%2C%20174839.00714748725%2C%20175109.6583815205%2C%20175380.30961555374%2C%20175650.960849587%2C%20175921.61208362022%2C%20176192.26331765344%2C%20176462.9145516867%2C%20176733.56578571993%2C%20177004.21701975315%2C%20177274.86825378638%2C%20177545.51948781963%2C%20177816.17072185286%2C%20178086.82195588612%2C%20178357.47318991934%2C%20178628.12442395257%2C%20178898.77565798583%2C%20179169.42689201905%2C%20179440.0781260523%2C%20179710.72936008553%2C%20179981.3805941188%2C%20180252.03182815204%2C%20180522.68306218527%2C%20180793.3342962185%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23f1eef6ff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23d4b9daff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23c994c7ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23df65b0ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23dd1c77ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%2C%20%27%23980043ff%27%5D%29%3B%0A%20%20%20%20%0A%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.x%20%3D%20d3.scale.linear%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B45738.36851363237%2C%20180793.3342962185%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B0%2C%20400%5D%29%3B%0A%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.legend%20%3D%20L.control%28%7Bposition%3A%20%27topright%27%7D%29%3B%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.legend.onAdd%20%3D%20function%20%28map%29%20%7Bvar%20div%20%3D%20L.DomUtil.create%28%27div%27%2C%20%27legend%27%29%3B%20return%20div%7D%3B%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.legend.addTo%28map_976d5813110e4e1aa4a466ca1f688fc5%29%3B%0A%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.xAxis%20%3D%20d3.svg.axis%28%29%0A%20%20%20%20%20%20%20%20.scale%28color_map_24206dcf730244c29bfd2efd8e6d07e1.x%29%0A%20%20%20%20%20%20%20%20.orient%28%22top%22%29%0A%20%20%20%20%20%20%20%20.tickSize%281%29%0A%20%20%20%20%20%20%20%20.tickValues%28%5B45738.36851363237%2C%2068247.52947739672%2C%2090756.69044116107%2C%20113265.85140492543%2C%20135775.0123686898%2C%20158284.17333245414%2C%20180793.3342962185%5D%29%3B%0A%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.svg%20%3D%20d3.select%28%22.legend.leaflet-control%22%29.append%28%22svg%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22id%22%2C%20%27legend%27%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20450%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2040%29%3B%0A%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.g%20%3D%20color_map_24206dcf730244c29bfd2efd8e6d07e1.svg.append%28%22g%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22key%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22transform%22%2C%20%22translate%2825%2C16%29%22%29%3B%0A%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.g.selectAll%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.data%28color_map_24206dcf730244c29bfd2efd8e6d07e1.color.range%28%29.map%28function%28d%2C%20i%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20return%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20x0%3A%20i%20%3F%20color_map_24206dcf730244c29bfd2efd8e6d07e1.x%28color_map_24206dcf730244c29bfd2efd8e6d07e1.color.domain%28%29%5Bi%20-%201%5D%29%20%3A%20color_map_24206dcf730244c29bfd2efd8e6d07e1.x.range%28%29%5B0%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20x1%3A%20i%20%3C%20color_map_24206dcf730244c29bfd2efd8e6d07e1.color.domain%28%29.length%20%3F%20color_map_24206dcf730244c29bfd2efd8e6d07e1.x%28color_map_24206dcf730244c29bfd2efd8e6d07e1.color.domain%28%29%5Bi%5D%29%20%3A%20color_map_24206dcf730244c29bfd2efd8e6d07e1.x.range%28%29%5B1%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20z%3A%20d%0A%20%20%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20%7D%29%29%0A%20%20%20%20%20%20.enter%28%29.append%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2010%29%0A%20%20%20%20%20%20%20%20.attr%28%22x%22%2C%20function%28d%29%20%7B%20return%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20function%28d%29%20%7B%20return%20d.x1%20-%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.style%28%22fill%22%2C%20function%28d%29%20%7B%20return%20d.z%3B%20%7D%29%3B%0A%0A%20%20%20%20color_map_24206dcf730244c29bfd2efd8e6d07e1.g.call%28color_map_24206dcf730244c29bfd2efd8e6d07e1.xAxis%29.append%28%22text%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22caption%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22y%22%2C%2021%29%0A%20%20%20%20%20%20%20%20.text%28%27%27%29%3B%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20layer_control_c8a67ea4af514588b1cb96b4ab0ba94a%20%3D%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20base_layers%20%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22stamentoner%22%20%3A%20tile_layer_19ca6284efbf4a6ca8c3dad93f630138%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20overlays%20%3A%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22macro_element_b9ecb66e73c94d77bd888ff2ba8649d1%22%20%3A%20choropleth_b9ecb66e73c94d77bd888ff2ba8649d1%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20L.control.layers%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20layer_control_c8a67ea4af514588b1cb96b4ab0ba94a.base_layers%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20layer_control_c8a67ea4af514588b1cb96b4ab0ba94a.overlays%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%22autoZIndex%22%3A%20true%2C%20%22collapsed%22%3A%20true%2C%20%22position%22%3A%20%22topright%22%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_976d5813110e4e1aa4a466ca1f688fc5%29%3B%0A%20%20%20%20%20%20%20%20%0A%3C/script%3E onload="this.contentDocument.open();this.contentDocument.write(    decodeURIComponent(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
# 얻은 결과를 html 형식으로 저장할 수 있다.
map.save("./2020_01_map_test.html")
```
