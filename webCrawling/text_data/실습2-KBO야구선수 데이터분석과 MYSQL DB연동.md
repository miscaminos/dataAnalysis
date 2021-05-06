# 실습: KBO홈페이지에서 팀별로 선수 정보데이터 활용해보기

1. 데이터 추출 후 dataframe 저장 (csv로 저장)
2. VM의 CentOS에 있는 MySQL DB로 저장
3. dataframe내 데이터 전처리 (data type변환)
4. 데이터기반의 유용한 정보 추출 (구단별 평균 연봉,순위)

## My solution

참고 link:

https://yurimkoo.github.io/python/2019/09/14/connect-db-with-python.html

https://towardsdatascience.com/a-really-simple-way-to-edit-row-by-row-in-a-pandas-dataframe-75d339cbd313

### 1. 데이터 추출 후 dataframe 저장


```python
from bs4 import BeautifulSoup
from selenium import webdriver

path = 'C:/Study/Python/datadown/chromedriver.exe'
driver = webdriver.Chrome(path)
driver.get("https://www.koreabaseball.com/Player/Search.aspx")
html = driver.page_source
soup = BeautifulSoup(html,'html.parser')

#선수별 상세 페이지 조회 위해 URL에 들어갈 선수코드 + 선수포지션 추출이 필요함
import time

players=[]
for i in range(2,12):
    driver.find_element_by_xpath('//*[@id="cphContents_cphContents_cphContents_ddlTeam"]/option[{}]'.format(i)).click()
    html = driver.page_source
    soup = BeautifulSoup(html,'html.parser')
    time.sleep(3)
    #pagination 넘어가기
    for n in range(1,6):
        try:
            driver.find_element_by_xpath('//*[@id="cphContents_cphContents_cphContents_ucPager_btnNo{}"]'.format(n)).click()
            html = driver.page_source
            soup = BeautifulSoup(html,'html.parser')
            time.sleep(2)
            for m in range(1,21):
                try:
                    player= soup.select(
                        '#cphContents_cphContents_cphContents_udpRecord > div.inquiry > table > tbody > tr:nth-child({}) > td:nth-child(2) > a'.format(m))[0]['href']
                    #선수 코드 추출
                    player_code = str(player).split("Id=")[1]
                    soup.select('#cphContents_cphContents_cphContents_udpRecord > div.inquiry > table > tbody > tr:nth-child(1) > td:nth-child(4)')
                    #선수 포지션 추출
                    position=soup.select('#cphContents_cphContents_cphContents_udpRecord > div.inquiry > table > tbody > tr:nth-child({}) > td:nth-child(4)'.format(m))[0].text
                    tmp = [player_code, position]
                    players.append(tmp)
                except:
                    #마지막 page에서 20개보다 작은경우
                    pass
        except:
            #pagination이 5보다 작은경우
            pass
```


```python
#몇명의 선수 코드 + 선수 포지션 추출되었는지 확인
len(players)
```


```python
#URL에 들어갈 format에 맞게 선수 포지션명 변경
player_type={'투수': 'Pitcher', 
             '외야수': 'Hitter', 
             '내야수': 'Hitter', 
             '포수': 'Hitter'}

for p in players:
    p[1] = player_type[p[1]]
    
players[:5]
```


```python
#선수별 사세페이지 데이터추출1-requests사용 방법

import requests
import pandas as pd

url1 = "https://www.koreabaseball.com/Record/Player/{}Detail/Basic.aspx?playerId={}"
url2 = "https://www.koreabaseball.com/Futures/Player/{}Detail.aspx?playerId={}"
header = {'user-agent':
         'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36'}

#table column명 한번만 추출 (첫번째 column이름 '구단명'은 manually넣음)
col_names=['구단명']
r = requests.get(url1.format(players[0][1],players[0][0]), headers=header)
profile_infos = r.text.split("player_basic")[1].split("<ul>")[1].split("<strong>")
for title in profile_infos:
    if ":" in title: col_names.append(title.split(":")[0])

#선수별 상세정보 추출
contents_list = []
for p in players:
    contents=[]
    #두type의 선수 상세페이지 URL
    try:
        r = requests.get(url1.format(p[1],p[0]), headers=header)
    except:
        r = requests.get(url2.format(p[1],p[0]), headers=header)
    #구단명 추출 후 첫번째 column에 들어가도록 가장 먼저 contents에 입력   
    team = r.text.split('<div class="player_info">')[1].split('</span>')[1].split('</h4>')[0]
    contents.append(team)
    #선수 상세 데이터 추출
    player_infos = r.text.split("player_basic")[1].split("<ul>")[1].split("</span>")
    for info in player_infos:
        if "<span id=" in info: contents.append(info.split("<span id=")[1].split(">")[1])
    contents_list.append(contents)

driver.close()

#dataframe 저장 및 csv 저장(원본 저장)
df = pd.DataFrame(contents_list, columns=col_names)
df.to_csv('../data3/kbo_players.csv', index=False)
```

### 2. VM의 CentOS에 있는 MySQL DB로 저장


```python
# import the module
from sqlalchemy import create_engine

# create sqlalchemy engine
engine = create_engine("mysql+pymysql://{user}:{pw}@{host}/{db}"
                       .format(user="root",
                               pw="1234",
                               host='(my centos ipaddr)',
                               db="KBO"))

# Insert whole DataFrame into MySQL
kbo_table.to_sql(con = engine, name='kbo_players', if_exists = 'append', index=False)
```

### 3. dataframe내 데이터 전처리 (data value 및 type변환)


```python
import pandas as pd
df = pd.read_csv('../data3/kbo_players.csv')

#df의 데이터로 전처리: 연봉 column
for idx in df.index:
    if df.loc[idx,'연봉'].endswith('달러'):
        df.loc[idx,'연봉'] = int(df.loc[idx,'연봉'].replace('달러',''))*(1/8.87)
    elif df.loc[idx,'연봉'].endswith('만원'):
        df.loc[idx,'연봉'] = int(df.loc[idx,'연봉'].replace('만원',''))
    else:
        df.loc[idx,'연봉'] = int(df.loc[idx,'연봉'].replace('','0'))

df['연봉']=df['연봉'].astype('int64')

#변경된 dataframe 내용 및 data type 확인
print(df.dtypes)
df.head(10)
```

### 4. 데이터기반의 유용한 정보 추출 (구단별 평균 연봉)


```python
#구단별 평균연봉 계산하기

#연봉 값이 없는 경우, 0이 들어갔기때문에 있다면 제외하고 평균을 구한다. 
df[df['연봉']==0]
sub_df = df[df['연봉']>0]

#구단별 연봉의 평균 확인
team_df = round(sub_df.groupby(['구단명'])['연봉'].mean(),-2)
print(team_df)

#구단별 연봉의 표준편차 확인
round(sub_df.groupby(['구단명'])['연봉'].std(),-2)

#구단별 평균 테이블의 기초 통계값
team_df.describe()
```

## Teacher's solution

lesson-learned: 
1. **USE REGEX!** 
정규식을 사용하면 너무나 빠르고 간단하게 데이터를 추출할 수 있다.

2. **USE TIME의 SLEEP함수!** 
페이지 조회 후 정보 추출을 위해 중간중간 time.sleep()으로 간격을 띄워준다.

3. **CHECK URL patterns carefully!** 
requests의 get으로 반복적으로 선수 상세페이지의 URL을 조회하려면,URL이 최소한으로 변경되도되는 pattern을 찾아야한다. (e.g., 이번경우에는 새로운 선수이든, 기존 선수이든, 선수 position 상관없이 player-id만 URL에서 바꿔주면 상세페이지 조회가 가능했다.)

#### for cleaner and more efficient code:
    * zip: 옷에 달린 zipper를 생각해보면 이해하기 쉽다. Take two(or more) arrays and combine each element at a time to make an array of tuples. 만약 combine하려는 array들 길이가 다르다면, 가장 짧은 array의 element 갯수만큼 zip된다. (실제 zipper처럼)
    
    * enumerate: enumerate(iterable, start index=n) iterable object를 (index, iterable)로 구성된 (enumerated obj)리스트와같은 형태로 반환해준다. start index는 optional(default가 0이다) for-loop과 counter를 따로 만들지 않도고 iterable안에 component하나씩 access할 수 있다.
    
    * extend vs.append: 이번경우와 같이 순차적으로 여러 페이지를 조회래서  각 페이지에서 수집한 데이터를 하나의 리스트에 더하려면, extend를 사용할 수 있다. 
    
    - append를 사용하면 a=[1,2,3,4]를 리스트(b=[a,b,c])의 마지막에 한개의 원소로 더하지만(b=[a,b,c,[1,2,3,4]]) 
    
    - extend를 사용해서 a를 b에 더하면 a의 각각의 원소가 b에 더해진다. 결과는 b=[a,b,c,1,2,3,4]이다. 
    
    * with open() & lazy evaluation: open()은 파일을 access하기위해 파일을 여는 함수이다. 그냥 바로 open()을 사용하는 것 보다, with keyword와 함께 사용하면 더 좋은 syntax와 exceptions 처리가 가능하다고 한다. 그리고 파일을 자동으로 닫아주기 때문에 close()를 따로 호출하지 않아도 된다. with를 사용하면 lazy evaluation 방식을 사용하는 것인데 (lazy evluation은 미리 evaluate하지 않고 value가 필요할때에 evaluate하는 방식임), 파일의 전체를 한꺼번에 가져오는 것이 아니라, 부분적을 가져오는것이라서 대용량의 파일을 가져와야 할때 매우 유용한다.
    
    참고 link:
    
    https://towardsdatascience.com/what-is-lazy-evaluation-in-python-9efb1d3bfed0


```python
from selenium import webdriver
from bs4 import BeautifulSoup as BS
import re
import time

pattern = re.compile("playerId=([0-9]+)")

path = 'C:/Study/Python/datadown/chromedriver.exe'
driver = webdriver.Chrome(path)
driver.get("http://www.koreabaseball.com")
driver.find_element_by_css_selector("#lnb > li:nth-child(4) > a").click()

select_team = "#cphContents_cphContents_cphContents_ddlTeam > option:nth-child({})"
select_page = "#cphContents_cphContents_cphContents_ucPager_btnNo{}"

playid = []
for x in range(2,12):
    for_1 = select_team.format(x)
    driver.find_element_by_css_selector(for_1).click()
    time.sleep(2)
    #playid.extend(pattern.findall(driver.page_source))
    for y in range(1,6):
        f2 = select_page.format(y)
        try:
            driver.find_element_by_css_selector(f2).click()
            time.sleep(1)
            playid.extend(pattern.findall(driver.page_source))
        except Exception as e:
            print (e)
        time.sleep(2)
```


```python
len(playid)
print(playid[:5])
```


```python
#pickle 형태로 선수 id 저장하기
import pickle

with open("../data3/kbo_1700.pkl", "wb") as f:
    pickle.dump(playid,f)
    
#선수 id 입력한 URL로 각 선수 상세페이지 데이터 추출해오기
import requests
from urllib import request

#note: 선수 상세 페이지의 프로파일 사진 가져오는 방법
r = requests.get("https://www.koreabaseball.com/Record/Player/HitterDetail/Basic.aspx?playerId=62356")
request.urlretrieve("http:"+BS(r.text).find("div",\
            class_ = "player_basic").find("img")['src'], "./김규민.jpg")

#선수 상세 페이지의 프로파일 정보 가져오는 함수 정의
def playerID(id_):
    play_dict = {}
    url = "https://www.koreabaseball.com/Record/Player/HitterDetail/Basic.aspx?playerId={}".format(id_)
    r = requests.get(url)
    #print (r.url)
    for x in BS(r.text).find("div",\
                class_ = "player_basic").findAll("li"):
        rt = [y.strip() for y in x.text.strip().split(":")]
        play_dict[rt[0]] = rt[1]
    #play_dict['Team'] = BS(r.text).find("h4", class_="team regular/2021/emblem_NC").text
    play_dict['Team'] = BS(r.text).find("h4", id='h4Team').text
    return play_dict

#선수 상세페이지 데이터 추출 및 dataframe에 저장하기
import pandas as pd
total=[]
for idx,play in enumerate(playid):
    print(idx)
    total.append(pd.DataFrame([playerID(play)]))
    
kbo = pd.concat(total)
print(kbo.columns)
# 필요시 column위치 바꾸기
kbo = kbo[['Team', '선수명', '등번호', '생년월일', '포지션', '신장/체중', '경력', '입단 계약금', '연봉', '지명순위',
       '입단년도']]

#결과 dataframe 확인
kbo.head()
```


```python
# tip: 원본 데이터 테이블을 바로 사용하기 보단, copy를 하나 만들어서 전처리 진행하는게 더 안전함
kbo_salary = kbo[kbo['연봉'] != ''].copy()

kbo_salary['연봉'] = kbo_salary['연봉'].apply(lambda x : int(x.replace("달러", "")) * 1125  if x.find('달러') > -1 else x)
kbo_salary['연봉'] = kbo_salary['연봉'].apply(lambda x : int(x.replace("만원", "")) * 10000 if str(x).find("만원") > -1 else x)

import numpy as np
kbo_salary['연봉'] = kbo_salary['연봉'].astype(np.int64)

#가장 연봉이 높은 선수는?
kbo_salary.sort_values(by=['연봉'], ascending=False)
```


```python
pd.set_option('display.float_format', lambda x: '%.3f' % x)
 
#가장 평균 연봉이 높은 구단은? 
kbo_salary.groupby('Team')['연봉'].mean().sort_values(ascending=False) #높은 것 부터 순서대로
```


```python
# centos의 MySQL DB에 dataframe 넣기

import pymysql

kbo_db = pymysql.connect(
    user='root', 
    passwd='1234', 
    host='(my centos ipaddr)', 
    db='KBO', 
    charset='utf8'
)

cur = kbo_db.cursor()

sql="INSERT INTO kbo_players VALUES (%s, %s, %s, %s, %s, \
            %s, %s, %s, %s, %s, %s)"

for idx, row in kbo.iterrows():
    print(row)
    cur.execute(sql, tuple(row))
    
kbo_db.commit()
```
