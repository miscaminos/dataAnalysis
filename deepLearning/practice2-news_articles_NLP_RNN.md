## 실습

네이버 경제 뉴스 데이터 수집 후, 뉴스기사 텍스트를 말뭉치로 활용하여 KoNLPy 라이브러리 도구들을 통해 말뭉치(corpus)를 분석해서 형태소 분석 및 유사한 단어를 찾아볼 수있다. 

### 1. naver경제 뉴스 수집 

- 카테고리 : 금융, 증권, 산업/재계,.... (속보만빼고 모두)
- 각 카테고리별 모든 페이지의 뉴스기사를 수집

	-> 수집된 기사는 본문을 txt로 지정
    
	-> 파일 제목은 본문의 링크주소

위의 기능을 함수로 만들어주고 매개변수로 날짜를 지정하면 그 날짜의 경제 뉴스 수집하는 함수를 작성하여 해당 날짜의 경제 뉴스 수집


```python
from bs4 import BeautifulSoup
from datetime import date,datetime, timedelta
import re, os, requests
import pandas as pd
```


```python
category = {'금융':'259','증권':'258','산업/재계':'261','중기/벤쳐':'771',
           '부동산':'260','글로벌 경제':'262','생활경제':'310','경제일반':'263'}

base_url = 'https://news.naver.com/main/list.nhn?mode=LS2D&mid=shm&sid2={}&sid1=101&date={}&page={}'
header = {'user-agent':
          'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36'}
```


```python
def news_crawling(start_date=None, end_date=None):
# tip: selenium으로 url을 조회하는 방법은 사용하지 않는게 좋다. 
# 속도가 매우 느림(수동으로 클릭하는 수준으로 속도가 떨어짐)    

    for date in pd.date_range(start_date, end_date):
        date = str(date).replace("-", "")[:8]
        for c in category:
            response= requests.get(base_url.format(category[c], date, 200 ), headers= header)
            soup = BeautifulSoup(response.text,'lxml')
            last_page = soup.find('div','paging').find("strong").text
            print('last page:{}'.format(last_page))
            for page in range(1, int(last_page)+1):
                article_listing = requests.get(base_url.format(category[c], date, page), headers= header)
                soup2 = BeautifulSoup(article_listing.text, 'lxml')
                for x in soup2.find('div','list_body newsflash_body').findAll('dt'):
                    article(date, x.a['href'])  
```


```python
def article(date, url):
    # 뉴스기사의 불필요한 부분은 수집되지 않도록 정규식으로 규칙 정의 
    reporter = re.compile("[가-힣]{2,4}\s*기자")
    email   = re.compile("[\w._%+-]+@[\w.-]+\.[a-zA-Z]{2,4}")
    
    # 기사 내용 추출
    response = requests.get(url, headers= header)
    bs = BeautifulSoup(response.text)
    try:
        rt = bs.find('div',id = 'articleBodyContents')
        text = rt.text.strip()
    except: 
        return
    try: 
        text = text[:text.find(rt.select_one('a').text)]
    except:
        pass
    
    # sub함수 : email, reporter 변수에 compile된 string pattern에 해당하는 부분을 ""로 치환(제거한다)
    text = re.sub(email, "", text)
    text = re.sub(reporter, "", text)  
    
    #해당 날짜의 폴더 생성 후 기사내용 저장
    directory = './economy_{}'.format(date)
    if not os.path.isdir(directory):
        os.mkdir(directory)
        
    f = open(directory + "/" + url.split("?")[-1] + ".txt", "w", encoding='utf-8')
    f.write(text)
    f.close()
```


```python
news_crawling('2021-05-26', '2021-05-26')
```

### 2. KoNLPy를 사용한 텍스트 분석

참고 link: https://konlpy.org/en/latest/


KoNLPy = 시중에 공개된 꼬꼬마(서울대), 코모란, 트위터, 한나눔, 은전한닢 다섯개 형태소 분석기를 한꺼번에 묶어서 편리하게 사용할 수 있도록 한 패키지

KoNLPYy의 Komoran 형태소 분석기를 적용하여 형태소 분석을 진행

수집한 경제 뉴스 기사 (말뭉치)의 의미, 문법 정보등이 응축된 embedding결과를 얻을 수 있다.

**embedding**: 자연어를 숫자의 나열인 vector로 바꾼 결과 or vector로 변환시키는 그 과정이다. 단어나 문장 각각을 vector로 변환해서 vector 공간에 '끼워넣는다'라는 의미. 숫자로 소통해야하는 컴퓨터가 자연어를 처리할 수 있도록 embedding과정이 진행되어야한다.


```python
import sys
import requests
from bs4 import BeautifulSoup
from datetime import date, datetime, timedelta
import pandas as pd
from urllib import parse
import os
```


```python
from konlpy.tag import Mecab, Komoran
```


```python
# komoran 객체 생성
tokenizer = Komoran()
```


```python
stop_words = {'금리'}
```


```python
'금리' in stop_words
```


```python
total = []
for roots, dirs, files in os.walk("./economy_20210526"):
    for idx,file in enumerate(files):
        if idx % 100 ==0 : print(idx)
        with open (roots + "/" + file, "r", encoding='utf-8') as f:
            for text in f:
                tmp = []
                for word, morpheme in tokenizer.pos(text.strip()):
                    if morpheme in ['NNG', 'NNP', 'NNB', "NNM"] and len(word) > 1 :
                        #print(word, end=", ")
                        if word not in stop_words: tmp.append(word)
        total.append(tmp)
```


```python
len(total)
```

### genism이란?

genism: open source python library for natural language processing

genism을 활용해서 특정 corpus (말뭉치)로 word2vec model을 훈련시키고 word embedding을 만들 수 있다.

genism introduction: https://towardsdatascience.com/a-beginners-guide-to-word-embedding-with-gensim-word2vec-model-5970fa56cc92

### 특정 명사 단어와 연관 단어 찾기

Word2Vec 모델을 사용하면 단어를 백터화 할때 단어의 문맥적 의미를 보존할 수 있다.
Word2Vec은 분포가정 (distributional hypothesis)를 확인하는데에 사용된다.
문장에서 어떤 단어가 같이 쓰였는지를 구별해서 결국 단어의 의미를 해성할때에 주변 문맥을 통해서 유추해 보는 것이다.

Word2Vec으로 임베딩한 명사 단어 vector와 cosine유사도가 가장 높은 단어를 확인할 수 있다.

word2vec활용 실험내용을 정리한 블로그:
https://ratsgo.github.io/natural%20language%20processing/2017/03/08/word2vec/



```python
from gensim.models import word2vec
```

#### Hierarchical Softmax의 활용
: 계산량이 많은 Softmax function 대신 보다 빠르게 계산가능한 multinomial distribution function을 사용하는 테크닉이다. 이 방법에서는 각 단어들을 leaves로 가지는 binary tree를 하나 만들어놓은 다음(complete 할 필요는 없지만, full 할 필요는 있을 것 같다), 해당하는 단어의 확률을 계산할 때 root에서부터 해당 leaf로 가는 path에 따라서 확률을 곱해나가는 식으로 해당 단어가 나올 최종적인 확률을 계산한다.

**min_count**: 등장횟수가 특정 값 이해는 제외

**sample**: 빈번하게 등장하는 단어에 대한 다운 샘플링 : Google 문서는 .00001에서 .001 사이의 값을 권장한다. 여기에서는 0.001에 가까운 값이 최종 모델의 정확도를 높이는 것으로 보여진다.

**size**: 많은 feature를 사용한다고 항상 좋은 것은 아니지만 대체적으로 좀 더 나은 모델이 된다.

**window**:학습 알고리즘이 고려해야 하는 컨텍스트의 단어 수. hierarchical softmax 를 위해 좀 더 큰 수가 좋지만 10 정도가 적당하다.


```python
model = word2vec.Word2Vec(total, workers = 4, vector_size=100, min_count=40, 
                 window=10, sample=0.001)
```


```python
# 같은 vector안에 물려있는 단어들 확인
model.wv.most_similar("금리")
```


```python

```
