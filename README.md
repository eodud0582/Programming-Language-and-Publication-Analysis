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
- 1. 도서 제목, 도서 URL, 도서 이미지, 저자, 가격, 할인 가격, 출판사, 출판일, ISBN, 도서 설명/특징
- 2. 도서 페이지 수

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

## 1. 데이터 수집

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
## 2. 데이터 전처리

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
## 3. 데이터 분석
















