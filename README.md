# 프로그래밍 언어와 출판 도서 분석

## 분석 배경 및 목적
- TIOBE라는 소프트웨어 코드 품질을 관리하는 회사에서 주기적으로 발표하는 컴퓨터 언어 순위가 있는데, 이를 많은 사람들이 인용하면서 TIOBE Index라고도 부른다.
- 순위는 지속적으로 업데이트 되는데, 최근의 발표에 따르면 가장 인기있는 언어는 Python이라고 한다. 보다 하드웨어 친화적이고 빌드 후 실행시 효율이 높은 언어인 C는 2위, C의 상위 버전으로 인식되는 C++은 4위이고, 통계 전용 언어에서 데이터사이언스의 부각으로 각광받았지만 Python의 부상으로 인기가 조금 하락한 R도 14위에 있다.
- 매년 다양한 프로그래밍 언어에 대한 책이 출판되는데, 이러한 도서 출판과 프로그래밍 언어에 대한 관심 사이의 관계가 어떠한지 알아보았다.
    - Python 같이 인기가 많은 언어의 경우 출판물 수도 많을까?
    - 그렇다면, Python은 정말로 핫한 프로그래밍 언어일까?
- 그리고 어떤 도서는 가격이 매우 비싸다. 또한, 동일한 프로그래밍 언어일지라도 출판된 도서의 가격 차이가 많이 나기도 한다. 도서 가격에 영향을 미치는 요인은 무엇일까? 도서의 페이지 수가 가격과 관계가 있을 것이라 생각하여, 가설에 대한 증명도 해보았다.

## 데이터 소개

**수집 방법**
- 네이버 Open API를 활용하여 도서 데이터 크롤링(https://openapi.naver.com/v1/search/book?)

**수집 데이터**
- 총 10개 언어에 대한 출판 책 정보 수집 (상위 9개 프로그래밍 언어 + R)
    - 도서 제목, 도서 URL, 도서 이미지, 저자, 가격, 할인 가격, 출판사, 출판일, ISBN, 도서 설명/특징
    - 도서 페이지 수

**데이터 수집시 적용 내용**
- 네이버 검색 API는 하루 25,000번 검색 제한이 있으며, 한 검색당 최대 1,000개의 결과만 반환
- 이에 따라, 책 ‘상세 검색’ 기능을 할 수 있는 관련 요청을 query에 포함하여 필요한 데이터만 수집
- 키워드를 사용하여 책 제목에 키워드가 포함되는 도서에 대해서 데이터 수집
- C와 R 언어에 대해선 ‘C 프로그래밍’, ‘R 프로그래밍’ 등 검색에 같이 사용 될 수 있는 관련 용어들을 포함하여 검색
- 파이썬, Python 등과 같이 한글 외에 영어 명칭도 자주 사용되는 프로그래밍 언어엔 영문도 검색
- 프로그래밍 관련 도서들은 카테고리 도서 분야 '컴퓨터/IT' 하위의 'IT 전문서(208820)'에 대부분 위치하기 때문에, 이를 데이터 검색시 활용
- 한 검색시 결과가 1,000개를 초과하게 될 경우를 대비해, 최근에 출간된 도서 데이터들부터 먼저 수집되도록 출간일순서로 요청 변수 설정
- 1차로 API를 사용하여 가져온 데이터에서 도서 URL을 사용해 도서 페이지 수 데이터 별도로 크롤링

**수집 데이터 정리**
- API로 수집된 데이터 중 [도서 제목, 출판사, ISBN, 출판일, 가격, 저자, 도서 URL] 컬럼과 도서 제목 검색에 사용한 키워드, 그리고 따로 크롤링한 도서 페이지 수 컬럼을 추가하여 DataFrame으로 정리
- 최초 7,966 rows x 9 columns

## 분석 방향
- 프로그래밍 언어별 출판 도서 수 및 가격 분석
- Python 관련 출판물 분석
- 국내 출판사 분석
- 연도별 및 기간별 분석
- 도서 페이지 수와 가격 관계 분석

---

## I. 데이터 수집

### 네이버 Open API로 데이터 수집

네이버 Open API를 통해 프로그래밍 언어와 관련된 출판 도서 데이터를 수집하였다. API 코드 값을 생성할 함수, URL 요청과 읽어들일 함수, 그리고 읽어들인 데이터를 DataFrame으로 정리할 함수를 생성하여 데이터 크롤링과 정리를 진행하였다.

```python
# 반복문으로 관려 도서 데이터 수집
result_books = []

search_books = ['파이썬','Python','C 알고리즘','C 언어','C 프로그래밍','C 입문','C 기초','C 개발','C Programming','C 자료구조','JavaScript','자바스크립트','Java','자바','C++','C#','Visual Basic','비주얼 베이직','SQL','PHP','R 언어','R 데이터','R 분석','R 프로그래밍','R 입문','R 기초']
#search_start = 1 # 1~1000; 이후엔 1000
#search_display = 100
search_category = 280020
# start_pubdate = 20170101

for search_book in search_books:
    for n in range(1, 1000, 100): # n = search_start
        url = gen_search_url('book_adv', search_book, n, 100, search_category) 
        result = get_result_onepage(url)
        pd_result = get_fields(result)
        pd_result['language'] = search_book
        result_books.append(pd_result)
    time.sleep(0.5)

result_book_data = pd.concat(result_books)
```
![image](https://user-images.githubusercontent.com/38115693/148677381-2f86a82c-a924-4f45-84cc-bafaf576506b.png)

### 수집한 데이터 중 도서 URL을 통해 도서별 총 페이지 수 정보 수집

```python
# 총 페이지 수 데이터 수집 시작
import time

page_num_col = []

for url in result_book_data['link']:
    print(url)
    print(time.time())
    
    try:
        page_num = get_page_num(BeautifulSoup(urlopen(url), 'html.parser'))
        page_num_col.append(page_num)
    except:
        print('==> Error in get_page_num!!')
        page_num_col.append(np.nan)
    
    print(len(page_num_col))
    time.sleep(0.5)
result_book_data['page_num'] = page_num_col
```
![image](https://user-images.githubusercontent.com/38115693/148677435-72d5af5a-acdc-4166-843c-1732ef985e17.png)

수집간 오류가 발생한 부분은 크롤링을 재시도하였다. 끝끝내 수집이 되지 않은 데이터들은 제거하였으며, 7,966개 데이터로 전처리를 진행하였다.

---

## II. 데이터 전처리

**결측치 처리**
- author 컬럼은 공백으로 채웠다.
- price, isbn 컬럼의 결측치는 제거하였다.

**추가 변수 생성**
- 데이터 수집 때 사용한 검색어를 사용하여 프로그래밍 언어 이름 데이터를 추가하였다.
- 날짜 데이터에서 연도와 월 정보를 추출하여 추가적인 컬럼을 생성하였다.
- ISBN 코드에서 국가 정보를 가져와 국내/해외 출판사 분류 데이터를 추가하였다.

**중복 데이터 처리**
- Java - JavaScript, C - C++/C# 중복 데이터를 정리하였다. 검색어로 데이터를 가져올 때, JavaScript 데이터가 Java 검색에 포함되어 가져와지거나, C++ 또는 C#인데 C 관련 데이터로 가져와진 데이터들을 찾아서 삭제하였다.
- ISBN 중복 데이터를 제거하여 동일한 출판물들을 제거하였다.

---

## III. 데이터 분석

일단 분석에 필요한 변수들만 가져와 DataFrame으로 복사하여 진행하였다.

### 1. 프로그래밍 언어 관련 출판 도서 수 분석

**프로그래밍 언어별 출판 도서 총 개수**

![image](https://user-images.githubusercontent.com/38115693/148678693-1ae36997-56e7-4922-abef-48f01efbe126.png)

- 지금까지 출판된 도서 중, C 언어와 관련된 도서가 가장 많으며, 이어서 Java, C++, Python 순서로 출판수가 많다.

**언어별 출판 도서 수 추이**

![image](https://user-images.githubusercontent.com/38115693/148677926-da28e4f4-10d4-4f6a-bc84-31edbf4313d7.png)

- 역사가 오래 된 C 언어와 관련된 출판물은 오래 전부터 있어온 것을 볼 수 있다.
- Visual Basic의 경우 1999년에, Java의 경우 2002년에 가장 많은 출판물이 발행되었다.
- Python 관련 출판물 수는 2010년부터 증가하기 시작하였고, 급격하게 증가하다 2019년 피크를 찍었다.

**Python 출판물 수 및 전년대비 증가율**

![image](https://user-images.githubusercontent.com/38115693/148678595-7540999a-0acb-47f0-8d56-67b605159d72.png)

- Python 관련 출판물 수는 2010년부터 2019년까지 매년 증가세를 보여주었는데, 특히 전년대비 2014년엔 157%, 2016년엔 121% 증가하였다. 가장 많은 출판물 수를 기록한 2019년도 전년대비 40% 증가한 수치다.
- 2019년 이후 Python 관련 출판물 수는 전년대비 감소세를 보이고 있지만, 매년 100권이 넘는 출판이 지속되고 있어 그 인기가 여전함을 알 수 있다. 

### 2. 프로그래밍 언어 관련 출판 도서 가격 분석

**프로그래밍 언어별 도서 가격**

![image](https://user-images.githubusercontent.com/38115693/148678642-5a55c1de-04f3-44b5-9fde-d18f8babb2f7.png)

- 도서 평균 가격이 가장 높은 언어는 C#(26,535원), R(26,355원)이며, Python은 25,077원으로 나타났다.
- 평균 가격이 가장 낮은 언어는 C(17,300원)이다. 10개 언어 중 역사가 가장 오래된 언어인 만큼, 출판물 수도 많을 테니 그만큼 가격도 낮은 것으로 생각한다.

**최근 7년간 연도별/월별 도서 가격 분포**

![image](https://user-images.githubusercontent.com/38115693/148678324-6c9938e2-70f1-41bc-88ab-b297284b5759.png)

- 최근 7년간 가격 분포는 눈에 띄는 차이를 보이지 않는다.

**최근 7년간 Python 관련 연도별/월별 도서 가격 분포**

![image](https://user-images.githubusercontent.com/38115693/148678409-33d06360-c3f9-4d46-8b4e-021baca0bbb8.png)

- 최근 7년간 Python 관련 도서 가격 분포도 큰 차이를 보이지 않는다.

### 3. 국내 출판사 분석

**프로그래밍 도서 출판물 수 기준 국내 top 10 출판사**

![image](https://user-images.githubusercontent.com/38115693/148678629-442f3cc9-6eed-493b-b33a-b57913e385c8.png)

- 국내 출판사 중에선 한빛미디어가 가장 많은 프로그래밍 언어 관련 도서를 출판했으며, 이어 에이콘출판, 정보문화사 등이 있다.

**국내 Top 10 출판사별 출판물 수 추이**

![image](https://user-images.githubusercontent.com/38115693/148678748-03f069d3-d300-4234-87c5-e078d216b6f6.png)

**국내 Top10 출판사별 Python 관련 출판물 수 추이**

![image](https://user-images.githubusercontent.com/38115693/148678759-1e4c2898-fc6e-41f1-9654-cef31c454d0c.png)

- 국내 Top10 출판스들은 2013년 이후 Python 관련 도서를 많이 출판하기 시작했다. 특히, 에이콘출판사가 Python 관련 도서 출판 수에 있어 많은 출판물을 보유하고 있고, 이어서 한빛미디어, 위키북스, 길벗 출판사가 많은 출판을 하였다.

**국내vs.해외 출판사 Python 출판물 수 비교**

![image](https://user-images.githubusercontent.com/38115693/148678900-2c72a279-b2d2-4ae4-9a64-0473d92a461d.png)

- 2015년 이후 국내 출판사들의 Python 관련 도서 출판 수가 급격히 증가하였는데, 비슷한 시기에 해외에서 출판된 Python 관련 도서들도 국내로 유입되기 시작한 것으로 보인다.

### 4. 2016-18년 vs. 2019-21년 기간별 분석

**2016-18 vs. 2019-21 언어별 출판물 수**

![image](https://user-images.githubusercontent.com/38115693/148679029-ac8d9fe3-40e5-474c-b936-e1bf0cefea6c.png)

- Python과 관려된 출판물 수가 2016-2018년, 2019-2021년 두 기간 모두 가장 많다.
- 대부분의 프로그래밍 언어가 2016-2018년 대비 2019-2021년 출판물 수가 감소하거나 비슷하게 나타난 반면, Python은 약 +84% 증가하였다.

![image](https://user-images.githubusercontent.com/38115693/148678988-00b99d88-c9d1-4fe4-9d1f-a103ad9d1414.png)

**2016-18 vs. 2019-21년 언어별 출판물 평균 가격**

![image](https://user-images.githubusercontent.com/38115693/148679153-4263c576-fa0b-41dc-bf17-f78bfdb18c37.png)

- 2016-18년 대비 2019-21년에 Python 관련 출판물의 평균 가격은 8.6% 증가

### 5. 도서 페이지 수와 가격 관계 분석
- 페이지 수가 가격에 영향을 주지 않을까?
- 페이지 수와 가격 회귀분석 진행
- 페이지 수로 가격 예측 진행

![image](https://user-images.githubusercontent.com/38115693/148680893-bdc0c107-ce46-49f4-9d5e-e1a14c0db565.png)

**페이지 수와 가격 관계 분석 (회귀분석)**

```python
# IQR 기반 이상치 검출
def get_outlier(data, rate=1.5): #이상치 기준 범위를 줄이고 싶으면 1.5보다 작게 잡으면 됨
    q1 = np.quantile(data, q=0.25)
    q3 = np.quantile(data, q=0.75)
    IQR = q3 - q1
    return (data < q1 - IQR*rate) | (data > q3 + IQR*rate) # q1 - IQR*rate: lower fence / q3 + IQR*rate: upper fence
    
book_df[get_outlier(book_df['price'])]['page_num'].value_counts().index
```
![image](https://user-images.githubusercontent.com/38115693/148679478-ab25ab61-4a7e-40e9-93db-f9fabde552a7.png)

- 가격이 특별히 비싼 책들은 페이지 수가 많아 보인다.

![image](https://user-images.githubusercontent.com/38115693/148679525-3c125a8e-d52d-451a-911d-f4a6758f9410.png)

- 회귀분석 결과, 페이지 수와 가격 사이에 관계가 있다고 볼 수 있다.

**출판사별 페이지 수와 가격 관계 분석 (회귀분석)**

![image](https://user-images.githubusercontent.com/38115693/148679571-e961e9d7-1f39-4e67-8994-a09a7fab2485.png)

- 출판사별 편중이 심한 것 같다.

특정 출판사들만 페이지 수와 가격 관계에 대한 회귀분석을 진행해 보았다.

![image](https://user-images.githubusercontent.com/38115693/148679616-fed1be46-63f3-4f22-b06a-e8c2064d9bae.png)

![image](https://user-images.githubusercontent.com/38115693/148679628-3676f1b9-2435-4f4f-a1e6-411ccd93eeaf.png)

![image](https://user-images.githubusercontent.com/38115693/148679637-5bd300f0-0ba2-47bf-9713-b8aa40134e8e.png)

![image](https://user-images.githubusercontent.com/38115693/148679651-8658681a-ac56-431e-90be-5d50bf6d4934.png)

- 만약 출판사별로, 페이지 수를 이용한 가격 예측을 진행한다면, 가격을 보다 잘 예측 할 수 있을 것 같다.

**도서 페이지 수 데이터로 도서 가격 예측**

우선 전체 데이터에서 페이지 수 데이터를 사용하여 가격 예측을 진행해 보았다. 

```python
# 회귀 모델 구성을 위한 데이터 나누기
from sklearn.model_selection import train_test_split

X = book_df['page_num'].values
y = book_df['price'].values

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=13)
X_train = X_train.reshape(-1,1)
X_test = X_test.reshape(-1,1)

# 모델 학습
from sklearn.linear_model import LinearRegression

reg = LinearRegression()
reg.fit(X_train, y_train)

# 에러 계산
from sklearn.metrics import mean_squared_error

pred_tr = reg.predict(X_train)
pred_test = reg.predict(X_test)

rmse_tr = (np.sqrt(mean_squared_error(y_train, pred_tr)))
rmse_test = (np.sqrt(mean_squared_error(y_test, pred_test)))

print('RMSE of Train Data :', rmse_tr)
print('RMSE of Test Data :', rmse_test)
```
![image](https://user-images.githubusercontent.com/38115693/148679881-f2070ddd-b431-43a6-821e-3eb64f032c0d.png)

```python
# 참값과 예측값
plt.scatter(y_test, pred_test)
plt.xlabel('Actual')
plt.ylabel('Predict')
plt.plot([0,80000],[0,80000],'r')
plt.show();
```
![image](https://user-images.githubusercontent.com/38115693/148679905-32650768-5c8b-43c2-9765-b79142887d54.png)

다음으로, 특정 출판사에 대한 예측을 진행한 결과는 아래와 같다.

![image](https://user-images.githubusercontent.com/38115693/148679960-c42bc9f5-6f55-40aa-9a9e-b59e4c749699.png)

- 의미가 있다고 보인다.
- 도서 페이지 수는 도서 가격에 영향이 있으며, 페이지 수가 많을수록 가격도 높다고 생각한다. 

---

## IV. 결론

- TIOBE Index와 같이 Python은 가장 인기가 많은 프로그래밍 언어이며, 그 인기는 관련 출판 도서 수에서도 나타난다. 최근 6년간 프로그래밍 언어 중에서 매년 가장 많은 도서가 출판되었으며, 2018년부터 매년 100권 넘게 출판되고 있는 것으로 보아 그 인기는 여전한 것으로 생각한다. 
- 도서 가격에 페이지 수가 큰 영향을 준다. 페이지 수가 많을수록 높은 가격을 보인다.
