## 실습: 
**10000개 파일의 파일명 및 파일 내용으로 data 추출하기**

Given:

파일명: 이름_생년월일_성별.txt (e.g., 강가연_19961123_3xxxxxx.txt)

파일내용: 금액 숫자 (e.g., 329,206)

Task:
1. 하나의 DataFrame 생성
columns: 이름, 생년월일, 성별, 금액, 나이, 연령대,
(생년월일 -> datetime, 금액 -> int, 성별 -> 남, 여, 연령대 -> 10대, 20대 .. 50세 이후는 다 50대로 표시)

2. 연령대별 금액 평균

### 잠시 review:

* 함수의 상세내역은 함수내에서 """문자열 표기로 구분하여 정의한다."""

* 현 py외에 다른 파일에서도 함수를 사용할 수 있도록 module을 생성한다.
(beautiful.py에서 정의한 bbb()함수는 beautiful module import후, 사용할 수 있다)


```python
#함수 상세내역(help) 작성하기
def aaa():
    """
    this function is aaa
    """
    pass
```


```python
help(aaa)
```

    Help on function aaa in module __main__:
    
    aaa()
        this function is aaa
    



```python
import beautiful as bf
bf.bbb() 
#bbb()함수 안에 print("You are beautiful no matter what they say")가 실행된다.
```

    You are beautiful no matter what they say


### Teacher's solution


```python
import os
help(os.walk)
```

#### For each directory in the directory tree rooted at top yields a 3-tuple:
(including top itself, but excluding '.' and '..')
    
        dirpath, dirnames, filenames


```python
import sys
```


```python
sys.path
```




    ['/home/encore/workspace',
     '/home/encore/miniconda3/lib/python38.zip',
     '/home/encore/miniconda3/lib/python3.8',
     '/home/encore/miniconda3/lib/python3.8/lib-dynload',
     '',
     '/home/encore/miniconda3/lib/python3.8/site-packages',
     '/home/encore/miniconda3/lib/python3.8/site-packages/IPython/extensions',
     '/home/encore/.ipython']




```python
import os

content_list=[]
for roots, dirs, files in os.walk("./data2"):
    for file in files:
        with open(roots + "/" +file, "r") as f:
            content_list.append(f.read())
```


```python
content_list[:5]
```




    ['83,057', '1,174', '11,812', '8,832,673', '2,128,655']




```python
name, birth, gender = "황해인_19660908_1xxxxxx.txt".split("_")
```


```python
name
```




    '황해인'




```python
birth
```




    '19660908'




```python
gender
```




    '1xxxxxx.txt'




```python
names=[]
births=[]
genders=[]
moneys=[]

for roots, dirs, files in os.walk("./data2"):
    for f in files:
        name, birth, gender = f.split("_")
        names.append(name)
        births.append(birth)
        
        #만약 txt외에 다른 파일이 들어있다면, 아래와 같이 filter
        if gender.split(".")[-1] !="txt": continue
            
        genders.append(gender[0])
        with open(roots + "/" +f, "r") as content:
            moneys.append(content.read())
```


```python
import pandas as pd

df = pd.DataFrame({"이름" : names, "생년월일":births, 
              "성별":genders, "금액": moneys})
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
      <th>이름</th>
      <th>생년월일</th>
      <th>성별</th>
      <th>금액</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강가인</td>
      <td>19710304</td>
      <td>1</td>
      <td>83,057</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강가희</td>
      <td>19600922</td>
      <td>4</td>
      <td>1,174</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강건영</td>
      <td>19750611</td>
      <td>1</td>
      <td>11,812</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강건영</td>
      <td>19950614</td>
      <td>3</td>
      <td>8,832,673</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강건희</td>
      <td>19751115</td>
      <td>2</td>
      <td>2,128,655</td>
    </tr>
  </tbody>
</table>
</div>




```python
from datetime import date, datetime, timedelta

datetime.strptime("20210510","%Y%m%d")
```




    datetime.datetime(2021, 5, 10, 0, 0)




```python
df['생년월일']=df['생년월일'].apply(lambda x: datetime.strptime(x,"%Y%m%d"))
```


```python
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
      <th>이름</th>
      <th>생년월일</th>
      <th>성별</th>
      <th>금액</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강가인</td>
      <td>1971-03-04</td>
      <td>1</td>
      <td>83,057</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강가희</td>
      <td>1960-09-22</td>
      <td>4</td>
      <td>1,174</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강건영</td>
      <td>1975-06-11</td>
      <td>1</td>
      <td>11,812</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강건영</td>
      <td>1995-06-14</td>
      <td>3</td>
      <td>8,832,673</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강건희</td>
      <td>1975-11-15</td>
      <td>2</td>
      <td>2,128,655</td>
    </tr>
  </tbody>
</table>
</div>




```python
help(datetime.strptime)
```


```python
a = datetime.strptime("20210510","%Y%m%d")
a
```




    datetime.datetime(2021, 5, 10, 0, 0)




```python
a.weekday() #0=Monday
```




    0




```python
#생년월인이 월요일인 경우만 filter해보면:
df[df['생년월일'].apply(lambda x: x.weekday()==0)].head()
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
      <th>이름</th>
      <th>생년월일</th>
      <th>성별</th>
      <th>금액</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>16</th>
      <td>강규리</td>
      <td>1993-10-11</td>
      <td>2</td>
      <td>638,169</td>
    </tr>
    <tr>
      <th>21</th>
      <td>강노훈</td>
      <td>1962-10-22</td>
      <td>4</td>
      <td>887,773</td>
    </tr>
    <tr>
      <th>22</th>
      <td>강다아</td>
      <td>1965-02-01</td>
      <td>2</td>
      <td>83,051,738</td>
    </tr>
    <tr>
      <th>34</th>
      <td>강대인</td>
      <td>1992-07-20</td>
      <td>1</td>
      <td>3,614,098</td>
    </tr>
    <tr>
      <th>37</th>
      <td>강도경</td>
      <td>1982-11-22</td>
      <td>3</td>
      <td>8,465,939</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['생년월일'] = df['생년월일'].apply(lambda x: x + timedelta(days=100))
```


```python
df['성별']= df['성별'].apply(lambda x: '남' if x in ['1','3'] else '여')
```


```python
df['금액'] = df['금액'].apply(lambda x : int(x.replace(",","")))
```


```python
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
      <th>이름</th>
      <th>생년월일</th>
      <th>성별</th>
      <th>금액</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강가인</td>
      <td>1971-03-04</td>
      <td>남</td>
      <td>83057</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강가희</td>
      <td>1960-09-22</td>
      <td>여</td>
      <td>1174</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강건영</td>
      <td>1975-06-11</td>
      <td>남</td>
      <td>11812</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강건영</td>
      <td>1995-06-14</td>
      <td>남</td>
      <td>8832673</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강건희</td>
      <td>1975-11-15</td>
      <td>여</td>
      <td>2128655</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['생년월일'][0].year
```




    1971




```python
def ages(x):
    age = 2021 - x.year +1
    if age<=20:
        return '10대'
    elif age>20 and age <=30:
        return '20대'
    if age>30 and age <=40:
        return '30대'
    if age>40 and age<=50:
        return '40대'
    else:
        return'50대'
```


```python
df['연령대'] = df['생년월일'].apply(lambda x: ages(x))
```


```python
pd.set_option('display.float_format', lambda x: '%.1f' %x)
```


```python
df.groupby('연령대').금액.mean()
```




    연령대
    20대   11162347.0
    30대   12753210.8
    40대   12864653.2
    50대   12307138.5
    Name: 금액, dtype: float64



### My solution


```python
import os
entries = os.listdir('./data2/')
```


```python
len(entries)
```




    10000




```python
import datetime

name=[]
birth=[]
gender=[]
age=[]
for n in entries: 
    infos = n.split('_')
    name.append(infos[0])
    infos[1]=infos[1].format(datetime.datetime)
    birth.append(infos[1])
    gender.append(infos[2][0])
    age.append(2021-int(infos[1][:4])+1)
```


```python
amount=[]
for e in entries:
    f = open('./data2/{}'.format(e),'r')
    x=f.read()
    x=int(x.replace(",",""))
    amount.append(x)
    f.close()
```


```python
age_group=[]
for b in birth:
    x=int(b[:4])
    if(x<=1972): age_group.append("50대")
    elif(x>1972 and x<=1982): age_group.append("40대")
    elif(x>1982 and x<=1992): age_group.append("30대")
    elif(x>1992 and x<=2002): age_group.append("20대")
    else: age_group.append("10대")
```


```python
len(age_group)
```




    10000




```python
!pip install pandas
```


```python
import pandas as pd

data = {'이름':pd.Series(name),'생년월일':pd.Series(birth),
        '성별':pd.Series(gender),'금액':pd.Series(amount),
        '나이':pd.Series(age),'연령대':pd.Series(age_group)}

result = pd.DataFrame(data)
result.head()
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
      <th>이름</th>
      <th>생년월일</th>
      <th>성별</th>
      <th>금액</th>
      <th>나이</th>
      <th>연령대</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강가인</td>
      <td>19710304</td>
      <td>1</td>
      <td>83057</td>
      <td>51</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강가희</td>
      <td>19600922</td>
      <td>4</td>
      <td>1174</td>
      <td>62</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강건영</td>
      <td>19750611</td>
      <td>1</td>
      <td>11812</td>
      <td>47</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강건영</td>
      <td>19950614</td>
      <td>3</td>
      <td>8832673</td>
      <td>27</td>
      <td>20대</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강건희</td>
      <td>19751115</td>
      <td>2</td>
      <td>2128655</td>
      <td>47</td>
      <td>40대</td>
    </tr>
  </tbody>
</table>
</div>




```python
#바로 list를 넣어도 dataframe 생성 가능
data = {'이름':name,'생년월일':birth,
        '성별':gender,'금액':amount,
        '나이':age,'연령대':age_group}

result = pd.DataFrame(data)
result.head()
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
      <th>이름</th>
      <th>생년월일</th>
      <th>성별</th>
      <th>금액</th>
      <th>나이</th>
      <th>연령대</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강가인</td>
      <td>19710304</td>
      <td>1</td>
      <td>83057</td>
      <td>51</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강가희</td>
      <td>19600922</td>
      <td>4</td>
      <td>1174</td>
      <td>62</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강건영</td>
      <td>19750611</td>
      <td>1</td>
      <td>11812</td>
      <td>47</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강건영</td>
      <td>19950614</td>
      <td>3</td>
      <td>8832673</td>
      <td>27</td>
      <td>20대</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강건희</td>
      <td>19751115</td>
      <td>2</td>
      <td>2128655</td>
      <td>47</td>
      <td>40대</td>
    </tr>
  </tbody>
</table>
</div>




```python
round(result.groupby('연령대').금액.mean(),1)
```




    연령대
    20대   11067352.7
    30대   12850701.6
    40대   12629094.9
    50대   12376219.9
    Name: 금액, dtype: float64




```python

```
