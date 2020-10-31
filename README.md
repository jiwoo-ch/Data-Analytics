# 데이터 수집과 분석 및 DB 구축
---
- 웹 페이지와의 연동성을 위해 MySQL DB를 구축하여 데이터를 저장하였으며, github에는 일부 데이터를 업로드함  
- 데이터베이스의 자세한 구조는 `database_description.pdf`에서 확인 가능
## Requirements
- selenium
- PyKoSpacing
  - https://github.com/haven-jeon/PyKoSpacing
- konlpy
- krwordrank
  - https://github.com/lovit/kr-wordrank
    
## 리뷰 텍스트 마이닝을 통한 키워드 추출 

### Step1. 생리대 제품 리뷰 크롤링
- 오픈마켓(쿠팡) 웹사이트에서 생리대 제품의 리뷰를 크롤링하여 수집하고 DB에 저장
  - 프로젝트 초기에 수집하고자 하는 생리대 제품 목록 `product(code).csv` 을 확정하였으나, 쿠팡 웹사이트의 잦은 변동으로 인해 기존 목록의 제품들이 사라지는 경우가 발생함
  - 이에 따라 위 목록을 검색하였을 때 검색어가 자동 변환되어 기존 검색어와는 다른 제품들이 조회됨
  - 변환되는 검색어를 저장하고 이를 이용하여 다양한 제품의 리뷰를 크롤링하는 방향으로 수정
- `search_query_crawler.py` 변환된 검색어를 수집하는 크롤러
- `create_post_table.py` 검색어를 쿠팡 사이트에 검색하였을 때 판매량 순으로 상위 노출된 1~3위 게시글을 수집하는 크롤러
- `create_review_table.py` 해당 게시글의 리뷰를 베스트 순으로 150개 수집하는 크롤러 

### Step2. 텍스트 마이닝을 위한 리뷰 전처리
- 수집된 리뷰 데이터에서 특수문자, 숫자, 영문자, 한글 자음 및 모음만 있는 경우를 제거
- PyKoSpacing 라이브러리를 이용하여 맞춤법과 띄어쓰기 교정

### Step3. 제품 키워드 풀(pool) 구축
- Referenced Code
  - https://github.com/lovit/kr-wordrank
- krwordrank 라이브러리를 이용하여 15개 제품의 제품별 리뷰에서 키워드를 추출
- 추출된 키워드를 검사하여 불용어(ex. 생리대, 사용, 구매, 배송 등) 및 동의어 등을 제거하고 약 50개의 키워드를 포함한 키워드 풀(pool) 구축

### Step4. 제품별 특성 키워드 선정
- 제품별 리뷰를 분석하여 리뷰에서 자주 언급되는 키워드들을 찾음
- 키워드풀에 포함되는 5개 키워드를 제품 특성 키워드로 선정

### Step5. 해당 키워드를 포함하는 리뷰 선별 
- 해당 키워드 클릭 시 관련된 리뷰를 확인할 수 있도록 함 

## 제품 구성성분 조사 및 분석 

### Step1. 제품별 구성성분 조사
- 제품 상세페이지 및 제조사 사이트를 확인하여 제품별 구성성분을 조사하고 저장

### Step2. 전체 구성성분 목록 확정
- 모든 제품의 구성성분 조사 후, 같은 구성성분임에도 표기법이 다르거나 한 경우 표기를 통일하여 전체 구성성분 목록을 확정

### Step 3. 구성성분에 관한 설명 및 유해성, 위해성 여부 조사
- 출처
  - http://msds.kosha.or.kr/kcic/msdssearchAll.do 안전보건공단 화학물질정보
- 보다 정확한 정보 제공을 위해 국가기관의 자료 및 유해성, 위험성 분류 사용
- 유해성, 위험성이 있는 성분의 경우 해당 분류에 해당하는 문구 확인 가능
