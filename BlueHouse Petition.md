```python
# Open-Knowledge-Korea/covid-19-our-memory
# https://github.com/Open-Knowledge-Korea/covid-19-our-memory/blob/master/code/society/blue-house-petition-analysis/%EA%B5%AD%EB%AF%BC%EC%B2%AD%EC%9B%90_%ED%81%AC%EB%A1%A4%EB%A7%81.ipynb
# 청와대 홈페이지 크롤링 코드
```


```python
from urllib.request import urlopen
from bs4 import BeautifulSoup
import pandas as pd
from selenium.webdriver.common.keys import Keys

from selenium import webdriver    #동적 구현을 위한 셀리늄 활용
from webdriver_manager.chrome import ChromeDriverManager
import time
```


```python
#driver = webdriver.Chrome(#'C:/chromedriver.exe')
driver = webdriver.Chrome(ChromeDriverManager().install())
driver.get('https://www1.president.go.kr/search')
```

    [WDM] - Current google-chrome version is 91.0.4472
    [WDM] - Get LATEST driver version for 91.0.4472
    [WDM] - Driver [/Users/seongy.yoon/.wdm/drivers/chromedriver/mac64/91.0.4472.101/chromedriver] found in cache


     



```python
# driver.find_element_by_id(" ") # 아이디 입력란의 id 를 찾아서 커서를 두겠다는 의미 
# driver.find_element_by_class_name() 클래스를 기준으로 HTML 요소를 찾는다
# driver.find_element_by_css_select() css 선택자를 기준으로 HTML요소를 찾는다
# driver.find_element_by_xpath() XPath 를 기준으로 HTML요소를 찾는다
# elem.send_key(pwd) # 현재의 커서가 위치한 곳에 넣겠다는 뜻
# elem.send_key(keys.RETURN) # Enter키를 누르게 합니다.
# .page_source 해당 요소의 HTML소스를 가져온다
# .get() 해당 URL로 이동한다.
```


```python
search_box = driver.find_element_by_css_selector("li.search_PG_bar input")
search_box.send_keys("우한폐렴")
search_box.send_keys(Keys.RETURN)
```


```python
detail_search_button = driver.find_element_by_css_selector("#contents > div.search_PG_contens > div > ul > li:nth-child(5) > a")
#detail_search_button = driver.find_element_by_css_selector("#contents > div.search_PG_contens > div.PG_contens > a")
detail_search_button.click()
```


```python
titles = []
start_date = []
end_date = []
num_consent = []
address = []

for n in range(1,20): #n은 페이지 수    
# get titles, address(url)
    html = driver.page_source
#    print(html)
    soup = BeautifulSoup(html, "html.parser")
    contents = soup.find_all('div', class_='PG_contens_pd_list')
    for i in range(0,len(contents)):
        content = contents[i]
        titles.append(content.h2.get_text())
        add_url = str(content.find('a')['href'])
        address.append('https://www1.president.go.kr'+add_url)
    
    # 청원시작과 끝 날짜, 참여인원, 답변일 정보 얻기 
    infos = soup.find_all('ul', class_='PG_peti_list')
    for info in infos: 
        date = info.find_all('span')
        start_date.append(date[0].get_text())
        #print(start_date)
        end_date.append(date[1].get_text())
        #print(end_date)
        num_consent.append(date[2].get_text())
        
    print(n)    
    page_bar = driver.find_elements_by_css_selector('div.p_btn > a')
    try:
        if n%7 != 0:
            page_bar[n%7+1].click()
            time.sleep(1)
        else:
            page_bar[8].click()
            time.sleep(2)
    except:
        print("수집완료")
        break
    time.sleep(0.5)
#driver.close()
```

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15
    16
    17
    18
    수집완료



```python
data = {'제목':titles, '청원시작일':start_date, '청원종료일': end_date,
       '참여인원': num_consent,'URL':address}
df = pd.DataFrame(data)
df.describe()
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
      <th>제목</th>
      <th>청원시작일</th>
      <th>청원종료일</th>
      <th>참여인원</th>
      <th>URL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>168</td>
      <td>168</td>
      <td>168</td>
      <td>168</td>
      <td>168</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>164</td>
      <td>34</td>
      <td>34</td>
      <td>158</td>
      <td>164</td>
    </tr>
    <tr>
      <th>top</th>
      <td>중국에서시작한 '우한폐렴' 코로나바이러스  한국에퍼지지않도록해주세요</td>
      <td>2020-02-03</td>
      <td>2020-03-04</td>
      <td>553</td>
      <td>https://www1.president.go.kr/petitions/584598</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>2</td>
      <td>56</td>
      <td>56</td>
      <td>2</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
df = df.drop_duplicates('제목', keep='first')
df.describe()
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
      <th>제목</th>
      <th>청원시작일</th>
      <th>청원종료일</th>
      <th>참여인원</th>
      <th>URL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>164</td>
      <td>164</td>
      <td>164</td>
      <td>164</td>
      <td>164</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>164</td>
      <td>34</td>
      <td>34</td>
      <td>158</td>
      <td>164</td>
    </tr>
    <tr>
      <th>top</th>
      <td>코로나바이러스(우한 폐렴)</td>
      <td>2020-02-03</td>
      <td>2020-03-04</td>
      <td>553</td>
      <td>https://www1.president.go.kr/petitions/588060</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>1</td>
      <td>55</td>
      <td>55</td>
      <td>2</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.to_csv('BlueHouse Petition data.csv', sep=',', encoding='utf-8')
```


```python
petition = pd.read_csv('BlueHouse Petition data.csv', thousands=',', encoding='utf-8', index_col=0)
petition.head()
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
      <th>제목</th>
      <th>청원시작일</th>
      <th>청원종료일</th>
      <th>참여인원</th>
      <th>URL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>경제 외교 방역실패한 문재인대통령을 탄핵해주세요 !!</td>
      <td>2021-05-03</td>
      <td>2021-06-02</td>
      <td>1865</td>
      <td>https://www1.president.go.kr/petitions/598086</td>
    </tr>
    <tr>
      <th>1</th>
      <td>세계적인 재난상황앞에서 어째서 국민들만 희생을 해야합니까? 대통령이하 모든 공무원분...</td>
      <td>2021-04-27</td>
      <td>2021-05-27</td>
      <td>562</td>
      <td>https://www1.president.go.kr/petitions/597951</td>
    </tr>
    <tr>
      <th>2</th>
      <td>제 동생의 죽음의 억울함을 풀어주세요</td>
      <td>2020-10-27</td>
      <td>2020-11-26</td>
      <td>34725</td>
      <td>https://www1.president.go.kr/petitions/593648</td>
    </tr>
    <tr>
      <th>3</th>
      <td>문재인 대통령님 스스로 하야 해주세요</td>
      <td>2020-10-02</td>
      <td>2020-11-01</td>
      <td>2366</td>
      <td>https://www1.president.go.kr/petitions/593190</td>
    </tr>
    <tr>
      <th>4</th>
      <td>정부 주도의 중국산 코로나 백신 (시노팜) 수입 검토를 철회해 주십시오.</td>
      <td>2020-09-18</td>
      <td>2020-10-18</td>
      <td>16428</td>
      <td>https://www1.president.go.kr/petitions/592917</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.to_excel('BlueHouse Petition data.xlsx')
```


```python
import re 
import time
from tqdm import tqdm_notebook
from urllib.request import urlopen
from bs4 import BeautifulSoup

category =[]
content = []
person = []

for n in tqdm_notebook(petition.index): 
    url = urlopen(petition['URL'][n]) 
    if n / 2 == 0:
        time.sleep(1)
    else: 
        time.sleep(2)
    soup = BeautifulSoup(url, "html.parser") 
    
    # get category 
    sector = soup.find('ul', class_='petitionsView_info_list')
    try: 
        category.append(sector.li.get_text()[4:])
    except AttributeError:
        print('AttributeError: index-', n, 'URL-', petition['URL'][n])
    person.append(sector.find_all('li')[3].get_text()[3:-6])

    # get petition content 
    content_raw = soup.find_all('div', class_='View_write')
    try: 
        if len(content_raw) > 1:
            content_raw_text = content_raw[1].get_text()
        else: 
            content_raw_text = content_raw[0].get_text()   
    except IndexError: 
        print('IndexError:', petition['URL'][n])
    
    content_split = re.split('\r|\t|\n|\xa0', content_raw_text)
    content.append(''.join(content_split))
```

    <ipython-input-13-a23a5f3a6f0a>:11: TqdmDeprecationWarning: This function will be removed in tqdm==5.0.0
    Please use `tqdm.notebook.tqdm` instead of `tqdm.tqdm_notebook`
      for n in tqdm_notebook(petition.index):



    HBox(children=(FloatProgress(value=0.0, max=164.0), HTML(value='')))


    



```python
print(len(category), len(content), len(person))
```

    164 164 164



```python
petition['카테고리'] = category
petition['청원내용'] = content
petition['청원인'] = person
```


```python
petition.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 164 entries, 0 to 163
    Data columns (total 8 columns):
     #   Column  Non-Null Count  Dtype 
    ---  ------  --------------  ----- 
     0   제목      164 non-null    object
     1   청원시작일   164 non-null    object
     2   청원종료일   164 non-null    object
     3   참여인원    164 non-null    int64 
     4   URL     164 non-null    object
     5   카테고리    164 non-null    object
     6   청원내용    164 non-null    object
     7   청원인     164 non-null    object
    dtypes: int64(1), object(7)
    memory usage: 16.5+ KB



```python
petition.to_csv('Final BlueHouse Petition data.csv', sep=',', encoding='utf-8')
```
