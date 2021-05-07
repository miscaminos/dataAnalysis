### 공공데이터 API key를 사용해서 data 추출하기

1. Covid19 공공데이터내의 누적확진자수와 사망자 수를 추출해서 dataframe에 저장하기.

2. 일별 확진자 수를 확인할 수 있는 파생열 생성하기

3. VM CENTOS의 MySQL DB에 저장하기


서비스 인증키:
key = "WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D"

REST 방식의 인증키 활용 :
아래와 같은 URL을 입력하면 data (get)방식으로 받아올수있다.

'http://openapi.data.go.kr/openapi/service/rest/Covid19/getCovid19InfStateJson?serviceKey=인증키(URL Encode)&pageNo=1&numOfRows=10&startCreateDt=20200401&endCreateDt=20200506'

### My solution

주의할점! diff()함수를 사용할때에 해당 row에 정확하게 match되는지 확인해야한다.
(shift() 함수로 해당 row의 위 또는 아래칸으로 shift해야할수있음)


```python
import requests
import pandas as pd
from bs4 import BeautifulSoup as BS

url="http://openapi.data.go.kr/openapi/service/rest/Covid19/getCovid19InfStateJson?serviceKey=WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D&pageNo=1&numOfRows=10&startCreateDt=20210401&endCreateDt=20210506"
response = requests.get(url)
type(BS(response.text))
```




    bs4.BeautifulSoup




```python
date=[]
decide=[]
death=[]

for x in BS(response.text).select('createDt'): date.append(x.text)
for y in BS(response.text).select('decideCnt'): decide.append(int(y.text))
for z in BS(response.text).select('deathCnt'): death.append(int(z.text))
```


```python
df = pd.DataFrame({'일시':date,'누적 확진자수':decide,'사망자수':death})
```


```python
df['일별 확진자 수'] = df['누적 확진자수'].diff(-1)
df.head()
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
      <th>일시</th>
      <th>누적 확진자수</th>
      <th>사망자수</th>
      <th>일별 확진자 수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-05-06 09:34:55.519</td>
      <td>125519</td>
      <td>1851</td>
      <td>574.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-05-05 09:37:42.237</td>
      <td>124945</td>
      <td>1847</td>
      <td>676.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-05-04 09:46:01.635</td>
      <td>124269</td>
      <td>1840</td>
      <td>541.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-05-03 09:31:59.949</td>
      <td>123728</td>
      <td>1834</td>
      <td>488.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-05-02 09:23:20.57</td>
      <td>123240</td>
      <td>1833</td>
      <td>606.0</td>
    </tr>
  </tbody>
</table>
</div>



### Teacher's solution

lesson-learned:

1. **xml file 다루기:** 
XML에서 데이터를 추출해오기위해 xml.etree.ElementTree 모듈을 사용한다.

2. **url 주소 다루기:**
url주소를 분해하거나 조각들을 붙여서 url주소를 형성하기위해 urllib의 parse모듈을 사용한다.


```python
import requests
import xml.etree.ElementTree as ET
import pandas as pd
from urllib import parse #url주소를 분해해야하는 경우 사용
```

### URL주소 다루기 

parsing an URL into components & combining components to create  whole url address


```python
key = "WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D"
```


```python
url = "http://openapi.data.go.kr/openapi/service/rest/Covid19/getCovid19InfStateJson?serviceKey=WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D&pageNo=1&numOfRows=10&startCreateDt=20200310&endCreateDt=20200315"
```


```python
result = parse.urlparse(url)

print(result)
type(result)
```

    ParseResult(scheme='http', netloc='openapi.data.go.kr', path='/openapi/service/rest/Covid19/getCovid19InfStateJson', params='', query='serviceKey=WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D&pageNo=1&numOfRows=10&startCreateDt=20200310&endCreateDt=20200315', fragment='')
    




    urllib.parse.ParseResult




```python
result.geturl()
```




    'http://openapi.data.go.kr/openapi/service/rest/Covid19/getCovid19InfStateJson?serviceKey=WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D&pageNo=1&numOfRows=10&startCreateDt=20200310&endCreateDt=20200315'




```python
result.scheme
```




    'http'




```python
result.hostname
```




    'openapi.data.go.kr'




```python
result.path
```




    '/openapi/service/rest/Covid19/getCovid19InfStateJson'




```python
result.query
```




    'serviceKey=WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D&pageNo=1&numOfRows=10&startCreateDt=20200310&endCreateDt=20200315'



#### ParseResult를 사용하여 url주소 구조 완성


```python
parse_result = parse.ParseResult(scheme  = result.scheme, netloc= result.hostname, path=result.path, 
                 params = result.params, query=result.query, fragment=result.fragment)
parse_result
```




    ParseResult(scheme='http', netloc='openapi.data.go.kr', path='/openapi/service/rest/Covid19/getCovid19InfStateJson', params='', query='serviceKey=WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D&pageNo=1&numOfRows=10&startCreateDt=20200310&endCreateDt=20200315', fragment='')



#### urlunparse() 메소드를 사용하여 주소 형태로 전달


```python
#### URL의 scheme, hostname, path, query등 모두 합해서 single url address를 만든다
parse.urlunparse(parse_result)
```




    'http://openapi.data.go.kr/openapi/service/rest/Covid19/getCovid19InfStateJson?serviceKey=WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D&pageNo=1&numOfRows=10&startCreateDt=20200310&endCreateDt=20200315'



### 주소 parameter 수정하기
dictionary 형태로 url에 들어갈 components를 manage하고,
components를 합해서 url address를 생성한다


```python
#URL의 result.query부분의 components를 dictionary형태로 확보
query_dict = parse.parse_qs(result.query)

query_dict
```




    {'serviceKey': ['WRvGBkn9UEtw+Asg0tYo210Etxvb1QEcAX+wfWvOxVGYJkh1CNZ/Y4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q=='],
     'pageNo': ['1'],
     'numOfRows': ['10'],
     'startCreateDt': ['20200310'],
     'endCreateDt': ['20200315']}




```python
#dictionary에 날짜를 수정한다
query_dict['endCreateDt'] = '20210507'
```


```python
query_dict
```




    {'serviceKey': ['WRvGBkn9UEtw+Asg0tYo210Etxvb1QEcAX+wfWvOxVGYJkh1CNZ/Y4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q=='],
     'pageNo': ['1'],
     'numOfRows': ['10'],
     'startCreateDt': ['20200310'],
     'endCreateDt': '20210507'}




```python
parse.urlencode(query_dict) #default doseq=False 상태
```




    'serviceKey=%5B%27WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D%27%5D&pageNo=%5B%271%27%5D&numOfRows=%5B%2710%27%5D&startCreateDt=%5B%2720200310%27%5D&endCreateDt=20210507'



#### 주의할 점: doseq 옵션
False가 기본값이며, False일시 dict의 value값의 [' '] 까지 인코딩을 한다.

[' 은 %5B%27 

'] 은 %27%5D

url에 필요없는 encoding된 값들이 있음을 확인할 수 있다.

불필요한 %5B%27 나 %27%5D를 포함하지 않으려면 **True**로 설정한다. 실제 값만 인코딩을 한다


```python
#doseq = True로 해야지만 []도 encode되지않아서 제대로 url을 만들수있다
parse.urlencode(query_dict, doseq=True) 
```




    'serviceKey=WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D&pageNo=1&numOfRows=10&startCreateDt=20200310&endCreateDt=20210507'




```python
api_url = parse.ParseResult(scheme  = result.scheme, netloc= result.hostname, path=result.path, 
                 params = result.params, query=parse.urlencode(query_dict, doseq=True), fragment=result.fragment)
```


```python
url = parse.urlunparse(api_url)
print(url) #생성된 링크를 클릭해보면 xml data 조회 가능
```

    http://openapi.data.go.kr/openapi/service/rest/Covid19/getCovid19InfStateJson?serviceKey=WRvGBkn9UEtw%2BAsg0tYo210Etxvb1QEcAX%2BwfWvOxVGYJkh1CNZ%2FY4QFa0r7j4bhT4NjPu7z1i1ck8ZgsKMt2Q%3D%3D&pageNo=1&numOfRows=10&startCreateDt=20200310&endCreateDt=20210507
    


```python
requests.get(url).text
```

### xml parsing


```python
r = requests.get(url)
```


```python
root = ET.fromstring(r.text)
```


```python
items = root.iter(tag='item')
items
```




    <_elementtree._element_iterator at 0x225c255b408>




```python
total = []
items = root.iter(tag='item')
for element in items:
    #print(element.find("accDefRate").text)
    covid_dict = {}
    for x in element:
        covid_dict[x.tag] = x.text
    total.append(covid_dict)
```


```python
total[:3]
```




    [{'accDefRate': '1.4047878518',
      'accExamCnt': '9051354',
      'accExamCompCnt': '8972458',
      'careCnt': '8162',
      'clearCnt': '116022',
      'createDt': '2021-05-07 09:38:57.839',
      'deathCnt': '1860',
      'decideCnt': '126044',
      'examCnt': '78896',
      'resutlNegCnt': '8846414',
      'seq': '504',
      'stateDt': '20210507',
      'stateTime': '00:00',
      'updateDt': 'null'},
     {'accDefRate': '1.4049684926',
      'accExamCnt': '9010992',
      'accExamCompCnt': '8933937',
      'careCnt': '8177',
      'clearCnt': '115491',
      'createDt': '2021-05-06 09:34:55.519',
      'deathCnt': '1851',
      'decideCnt': '125519',
      'examCnt': '77055',
      'resutlNegCnt': '8808418',
      'seq': '503',
      'stateDt': '20210506',
      'stateTime': '00:00',
      'updateDt': 'null'},
     {'accDefRate': '1.4015026684',
      'accExamCnt': '8994069',
      'accExamCompCnt': '8915074',
      'careCnt': '8154',
      'clearCnt': '114944',
      'createDt': '2021-05-05 09:37:42.237',
      'deathCnt': '1847',
      'decideCnt': '124945',
      'examCnt': '78995',
      'resutlNegCnt': '8790129',
      'seq': '502',
      'stateDt': '20210505',
      'stateTime': '00:00',
      'updateDt': 'null'}]




```python
# 다음과 같이 items의 내부 함수들 중, __iter__를 통해 items를(iterable object로) traverse할 수 있는것이다
dir(items)
```




    ['__class__',
     '__delattr__',
     '__dir__',
     '__doc__',
     '__eq__',
     '__format__',
     '__ge__',
     '__getattribute__',
     '__gt__',
     '__hash__',
     '__init__',
     '__init_subclass__',
     '__iter__',
     '__le__',
     '__lt__',
     '__ne__',
     '__new__',
     '__next__',
     '__reduce__',
     '__reduce_ex__',
     '__repr__',
     '__setattr__',
     '__sizeof__',
     '__str__',
     '__subclasshook__']




```python
for element in items:
    print(element.find("accDefRate").text)
    break
```

    1.4049684926
    


```python
#element도 마찬가지, 
#element의 내부 함수중에는 iterate을 가능하게하는 내부함수가 있기에 iterable obj로 활용될 수 있다.
for x in element:
    print(x.tag)
```

    accDefRate
    accExamCnt
    accExamCompCnt
    careCnt
    clearCnt
    createDt
    deathCnt
    decideCnt
    examCnt
    resutlNegCnt
    seq
    stateDt
    stateTime
    updateDt
    


```python
len(total)
```




    424



### updateDt에 'null' 문자열 값이 들어가있다. 처리가 필요함.

datframe을 보면, 가장 최신 데이터에는 updateDt 항목에 'null'이 들어가있다. 

이것은 우리에겐 결측치이지만, dataframe안에서는 결측치로 여겨지지 않는다. 'null' 문자열 이기때문이다.

python이 인식할 수 있는 결측치로 None으로 바꿔줘야한다.


```python
df = pd.DataFrame(total)
df.head(2) #'null'문자열 값 확인
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
      <th>accDefRate</th>
      <th>accExamCnt</th>
      <th>accExamCompCnt</th>
      <th>careCnt</th>
      <th>clearCnt</th>
      <th>createDt</th>
      <th>deathCnt</th>
      <th>decideCnt</th>
      <th>examCnt</th>
      <th>resutlNegCnt</th>
      <th>seq</th>
      <th>stateDt</th>
      <th>stateTime</th>
      <th>updateDt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.4047878518</td>
      <td>9051354</td>
      <td>8972458</td>
      <td>8162</td>
      <td>116022</td>
      <td>2021-05-07 09:38:57.839</td>
      <td>1860</td>
      <td>126044</td>
      <td>78896</td>
      <td>8846414</td>
      <td>504</td>
      <td>20210507</td>
      <td>00:00</td>
      <td>null</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.4049684926</td>
      <td>9010992</td>
      <td>8933937</td>
      <td>8177</td>
      <td>115491</td>
      <td>2021-05-06 09:34:55.519</td>
      <td>1851</td>
      <td>125519</td>
      <td>77055</td>
      <td>8808418</td>
      <td>503</td>
      <td>20210506</td>
      <td>00:00</td>
      <td>null</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.isnull().head(2) #결측치로 인식하지 않는다는것이 확인됨.
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
      <th>accDefRate</th>
      <th>accExamCnt</th>
      <th>accExamCompCnt</th>
      <th>careCnt</th>
      <th>clearCnt</th>
      <th>createDt</th>
      <th>deathCnt</th>
      <th>decideCnt</th>
      <th>examCnt</th>
      <th>resutlNegCnt</th>
      <th>seq</th>
      <th>stateDt</th>
      <th>stateTime</th>
      <th>updateDt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>1</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>




```python
df = df.where(~(df=='null'),None) #'null'을 None으로 바꿔준다.
```


```python
df.head(2)
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
      <th>accDefRate</th>
      <th>accExamCnt</th>
      <th>accExamCompCnt</th>
      <th>careCnt</th>
      <th>clearCnt</th>
      <th>createDt</th>
      <th>deathCnt</th>
      <th>decideCnt</th>
      <th>examCnt</th>
      <th>resutlNegCnt</th>
      <th>seq</th>
      <th>stateDt</th>
      <th>stateTime</th>
      <th>updateDt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.4047878518</td>
      <td>9051354</td>
      <td>8972458</td>
      <td>8162</td>
      <td>116022</td>
      <td>2021-05-07 09:38:57.839</td>
      <td>1860</td>
      <td>126044</td>
      <td>78896</td>
      <td>8846414</td>
      <td>504</td>
      <td>20210507</td>
      <td>00:00</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.4049684926</td>
      <td>9010992</td>
      <td>8933937</td>
      <td>8177</td>
      <td>115491</td>
      <td>2021-05-06 09:34:55.519</td>
      <td>1851</td>
      <td>125519</td>
      <td>77055</td>
      <td>8808418</td>
      <td>503</td>
      <td>20210506</td>
      <td>00:00</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.isnull().head(2) #None으로 바꿔준 후, 결측치로 여겨지게 되었다.
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
      <th>accDefRate</th>
      <th>accExamCnt</th>
      <th>accExamCompCnt</th>
      <th>careCnt</th>
      <th>clearCnt</th>
      <th>createDt</th>
      <th>deathCnt</th>
      <th>decideCnt</th>
      <th>examCnt</th>
      <th>resutlNegCnt</th>
      <th>seq</th>
      <th>stateDt</th>
      <th>stateTime</th>
      <th>updateDt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>



#### np.nan의 datatype은 float이다?!


```python
import numpy as np
type(np.nan)

#IEEE에서 missing value를 소숫점 집합에 넣도록 정의함. IEEE규격을 따르기때문에 float 타입이다.
```




    float




```python
df = df.where(pd.notnull(df), None)
```


```python
df.keys()
```




    Index(['accDefRate', 'accExamCnt', 'accExamCompCnt', 'careCnt', 'clearCnt',
           'createDt', 'deathCnt', 'decideCnt', 'examCnt', 'resutlNegCnt', 'seq',
           'stateDt', 'stateTime', 'updateDt'],
          dtype='object')



### 일별 확진자 확인


```python
df['일별 확진자'] = df['decideCnt'].astype(int).diff(-1)
```


```python
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
      <th>accDefRate</th>
      <th>accExamCnt</th>
      <th>accExamCompCnt</th>
      <th>careCnt</th>
      <th>clearCnt</th>
      <th>createDt</th>
      <th>deathCnt</th>
      <th>decideCnt</th>
      <th>examCnt</th>
      <th>resutlNegCnt</th>
      <th>seq</th>
      <th>stateDt</th>
      <th>stateTime</th>
      <th>updateDt</th>
      <th>일별 확진자</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.4047878518</td>
      <td>9051354</td>
      <td>8972458</td>
      <td>8162</td>
      <td>116022</td>
      <td>2021-05-07 09:38:57.839</td>
      <td>1860</td>
      <td>126044</td>
      <td>78896</td>
      <td>8846414</td>
      <td>504</td>
      <td>20210507</td>
      <td>00:00</td>
      <td>None</td>
      <td>525.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.4049684926</td>
      <td>9010992</td>
      <td>8933937</td>
      <td>8177</td>
      <td>115491</td>
      <td>2021-05-06 09:34:55.519</td>
      <td>1851</td>
      <td>125519</td>
      <td>77055</td>
      <td>8808418</td>
      <td>503</td>
      <td>20210506</td>
      <td>00:00</td>
      <td>None</td>
      <td>574.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.4015026684</td>
      <td>8994069</td>
      <td>8915074</td>
      <td>8154</td>
      <td>114944</td>
      <td>2021-05-05 09:37:42.237</td>
      <td>1847</td>
      <td>124945</td>
      <td>78995</td>
      <td>8790129</td>
      <td>502</td>
      <td>20210505</td>
      <td>00:00</td>
      <td>None</td>
      <td>676.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.3995739863</td>
      <td>8957155</td>
      <td>8879059</td>
      <td>8301</td>
      <td>114128</td>
      <td>2021-05-04 09:46:01.635</td>
      <td>1840</td>
      <td>124269</td>
      <td>78096</td>
      <td>8754790</td>
      <td>501</td>
      <td>20210504</td>
      <td>00:00</td>
      <td>None</td>
      <td>541.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.4018098628</td>
      <td>8915326</td>
      <td>8826304</td>
      <td>8538</td>
      <td>113356</td>
      <td>2021-05-03 09:31:59.949</td>
      <td>1834</td>
      <td>123728</td>
      <td>89022</td>
      <td>8702576</td>
      <td>500</td>
      <td>20210503</td>
      <td>00:00</td>
      <td>None</td>
      <td>488.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#diff()값을 넣은 '일별확진자'값이 날짜에 맞도록 1칸씩 아래로 shift
df['일별 확진자'] = df['일별 확진자'].shift(1)
```


```python
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
      <th>accDefRate</th>
      <th>accExamCnt</th>
      <th>accExamCompCnt</th>
      <th>careCnt</th>
      <th>clearCnt</th>
      <th>createDt</th>
      <th>deathCnt</th>
      <th>decideCnt</th>
      <th>examCnt</th>
      <th>resutlNegCnt</th>
      <th>seq</th>
      <th>stateDt</th>
      <th>stateTime</th>
      <th>updateDt</th>
      <th>일별 확진자</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.4047878518</td>
      <td>9051354</td>
      <td>8972458</td>
      <td>8162</td>
      <td>116022</td>
      <td>2021-05-07 09:38:57.839</td>
      <td>1860</td>
      <td>126044</td>
      <td>78896</td>
      <td>8846414</td>
      <td>504</td>
      <td>20210507</td>
      <td>00:00</td>
      <td>None</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.4049684926</td>
      <td>9010992</td>
      <td>8933937</td>
      <td>8177</td>
      <td>115491</td>
      <td>2021-05-06 09:34:55.519</td>
      <td>1851</td>
      <td>125519</td>
      <td>77055</td>
      <td>8808418</td>
      <td>503</td>
      <td>20210506</td>
      <td>00:00</td>
      <td>None</td>
      <td>525.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.4015026684</td>
      <td>8994069</td>
      <td>8915074</td>
      <td>8154</td>
      <td>114944</td>
      <td>2021-05-05 09:37:42.237</td>
      <td>1847</td>
      <td>124945</td>
      <td>78995</td>
      <td>8790129</td>
      <td>502</td>
      <td>20210505</td>
      <td>00:00</td>
      <td>None</td>
      <td>574.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.3995739863</td>
      <td>8957155</td>
      <td>8879059</td>
      <td>8301</td>
      <td>114128</td>
      <td>2021-05-04 09:46:01.635</td>
      <td>1840</td>
      <td>124269</td>
      <td>78096</td>
      <td>8754790</td>
      <td>501</td>
      <td>20210504</td>
      <td>00:00</td>
      <td>None</td>
      <td>676.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.4018098628</td>
      <td>8915326</td>
      <td>8826304</td>
      <td>8538</td>
      <td>113356</td>
      <td>2021-05-03 09:31:59.949</td>
      <td>1834</td>
      <td>123728</td>
      <td>89022</td>
      <td>8702576</td>
      <td>500</td>
      <td>20210503</td>
      <td>00:00</td>
      <td>None</td>
      <td>541.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
rt = df[['createDt','일별 확진자']]
```


```python
rt = rt.where(pd.notnull(rt),0)
```


```python
rt.head(5)
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
      <th>createDt</th>
      <th>일별 확진자</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-05-07 09:38:57.839</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-05-06 09:34:55.519</td>
      <td>525.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-05-05 09:37:42.237</td>
      <td>574.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-05-04 09:46:01.635</td>
      <td>676.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-05-03 09:31:59.949</td>
      <td>541.0</td>
    </tr>
  </tbody>
</table>
</div>



### Database 연결 및 Create table, Insert dataframe

GRANT ALL PRIVILEGES ON . TO 'root'@'%' IDENTIFIED BY '(my password)'


```python
import pymysql

try:
    con = pymysql.connect(host='192.168.219.110', user='root', port=3306,
                         password='1234', db='SQOOP', charset='utf8')
    cur = con.cursor()
except Exception as e:
    print(e)
```


```python
for name in df.keys():
    print(name, end=", ")
```

    accDefRate, accExamCnt, accExamCompCnt, careCnt, clearCnt, createDt, deathCnt, decideCnt, examCnt, resutlNegCnt, seq, stateDt, stateTime, updateDt, 일별 확진자, 


```python
sql_create_table = """
    CREATE TABLE covid19 (
        `accDefRate` DOUBLE(15,10),
        `accExamCnt` INT(4),
        `accExamCompCnt` INT(4),
        `careCnt`      INT(4),
        `clearCnt`     INT(4),
        `createDt`     DATETIME, 
        `deathCnt`     INT(4),
        `decideCnt`    INT(4),
        `examCnt`      INT(4),
        `resutlNegCnt` INT(4),
        `seq`          INT(4),
        `stateDt`      DATE,
        `stateTime`    TIME,
        `updateDt`     DATETIME
    ) 
"""
```


```python
cur.execute(sql_create_table)
```




    0




```python
con.commit()
```


```python
sql = """INSERT INTO covid19 (accDefRate, accExamCnt, accExamCompCnt, 
         careCnt, clearCnt, createDt, deathCnt, decideCnt, examCnt, resutlNegCnt, seq, stateDt, stateTime, updateDt) VALUES
         (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
"""
```


```python
for row in df.iterrows():
    s = row[1].tolist()
    print(s)
```


```python
for row in df.iterrows():
    cur.execute(sql, row[1].tolist())
```


```python
con.close()
```
