
# 📊 [노트북 가격과 인기의 상관관계 & 최근 3년간 노트북 가격을 비교하여 26년도 3사 플래그십_모델 가격을 예측해보자!(진행중)[]()


> 한 줄 요약: 다나와 웹사이트의 3사 노트북데이터들을 뷰티풀수프+셀리니움으로 크롤링 한후, 데이터 프레임에 저장및, .csv파일에 저장한후, 데이터 전처리및 시각화를 통해서 상관관계를 도출하자.

<a href="https://github.com/sehyeogg365/mini_project/commit/70cc3eb239d81e82eada67b9c703ab5ec7841f0d">기획안</a>


## 🎯 프로젝트 개요
[간단한 소개]
다나와 웹사이트의 3사 노트북데이터들을 뷰티풀수프+셀리니움으로 다나와 사이트 접속후, 3사 노트북을 체크박스 선택하게 끔 하여 크롤링 한다. 크롤링한 데이터를 데이터 프레임에 저장및, .csv파일에 저장한다. 그후, 데이터 전처리및 시각화를 통해서 상관관계를 도출하자.

## 📂 프로젝트 구조
[디렉토리 트리]
```
.
├── docs
├── .gitignore
├── data.csv
├── data2.csv
└── mini_project.ipynb
```
## 🔄 데이터 파이프라인

### 전체 프로세스
```
웹 크롤링 → 데이터프레임 저장 → CSV 저장 → EDA → 인사이트 도출
```

---

## 1️⃣ 웹 크롤링

### 구현 내용
- **대상 사이트**: [다나와]
- **수집 데이터**: [데이터 종류]
- **수집량**: 500건이상
- **사용 기술**: BeautifulSoup / Selenium

### 코드 예시
```python
# 핵심 크롤링 로직
[간단한 코드 스니펫]
```
```
MAX_PAGES_TO_SCRAPE = 20  # 수동으로 크롤링할 최대 페이지 수 (약 500개 데이터)

try:
    # 1. 브라우저 실행
    driver = webdriver.Chrome()
    driver.maximize_window()
    driver.get("https://prod.danawa.com/list/?cate=112758")
    time.sleep(3)
    
    print("페이지 로딩 완료")
    
    # 2. 체크박스 선택 (공백 제거하고 비교)
    # 2. ul > li > label 구조에서 title 속성으로 찾기
    target_brands = ['삼성전자', 'LG전자', 'APPLE']
    
    # ul.item_list 안의 모든 li 찾기
    li_elements = driver.find_elements(By.CSS_SELECTOR, 'ul.item_list.item_list_popular > li.sub_item')

    for li in li_elements:
        try:
            label = li.find_element(By.TAG_NAME, 'label')

            # title 속성 값 가져오기
            title = label.get_attribute('title')
            
            # title이 None이면 건너뛰기
            if not title:
                continue
            # 공백 제거
            title = title.strip()
            
            # target_brands에 있으면 클릭
            if title in target_brands:
                driver.execute_script("arguments[0].scrollIntoView(true);", label)
                time.sleep(0.3)
                driver.execute_script("arguments[0].click();", label)
                print(f"✓ {title} 체크박스 선택 완료")
                time.sleep(0.5)
        
        except Exception as e:
            continue 

        # print(f"✗ {brand} 선택 실패: {e}")
    
    print("\n필터 적용 대기 중...")
    time.sleep(3)
        
    # # 3. 현재 페이지 HTML 가져오기
    # html = driver.page_source
    # soup = BeautifulSoup(html, "html.parser")

    print("크롤링 준비 완료!")
    
except Exception as e:
    print(f"에러 발생: {e}")

finally:
    pass
```


### 🔧 트러블슈팅

#### 문제 1: [페이징 구현법]
- **상황**: [페이징을 하면서 웹 크롤링을 해야하는데 페이징 구현과 선택자 추출이 어려운 상황.]
- **원인**: [왜 발생했는지: 실제 웹 크롤링과 예제 내 웹크롤링과 난이도 자체가 달랐다.]
- **해결**: [페이징도 onclick="javascript:movePage() 함수로 자동화 하는 방법, 그리고 수동방식이 있다고 하는데 나는 수동으로 페이지를 넘기는 방식을 선택했다.]
```python
# Before
[문제 있던 코드]
 # 반복문으로 각각 타겟, 피처 변수들 추출
    for idx, product in enumerate(product_list, 1):
        
        product_main = product.select_one("div.prod_main_info")
        product_data = {}

        if not product_main:
            continue

        # 모델명
        name_elem = product_main.select_one("p.prod_name a")
        if not name_elem:
            continue
        
        name = name_elem.get_text(strip=True)
        product_data['모델명'] = name

        spec_divs = product_main.select_one("div.spec_list")
# After  
[해결된 코드]
```
```
for current_page in range(1, MAX_PAGES_TO_SCRAPE + 1):
        
    print(f"\n{'='*60}")
    print(f"📄 {current_page}페이지 크롤링 중...")
    print(f"{'='*60}")
    
    # 현재 페이지 HTML 가져오기
    time.sleep(2)
    html = driver.page_source
    soup = BeautifulSoup(html, "html.parser")
    
    # ul class="product_list" 내에 li class="prod_item"
    product_list = soup.select('ul.product_list > li.prod_item')
    
    if not product_list:
        print("더 이상 제품이 없습니다.")
        break
    
    print(f"현재 페이지 제품 수: {len(product_list)}개")
        
    # 반복문으로 각각 타겟, 피처 변수들 추출
    for idx, product in enumerate(product_list, 1):
        
        product_main = product.select_one("div.prod_main_info")
        product_data = {}

        if not product_main:
            continue
            
        # 모델명
        name_elem = product_main.select_one("p.prod_name a")
        if not name_elem:
            continue
        
        name = name_elem.get_text(strip=True)
        product_data['모델명'] = name

        spec_divs = product_main.select_one("div.spec_list")    
```

#### 문제 2: [웹 크롤링 자체 문제]
- **상황**: 위에 말한대로 예제 웹 크롤링과 실제 웹 크롤링은 난이도가 다르다.
램 데이터를 추출 한다거나, 화면 크기를 추출해야 한다. /////이런 구조라면 split['/']함수를 활용한 인덱싱 자체만으로도 추출 안되는 데이터들이 있었다. 
- **해결**: 정규표현식을 사용하여 제품 사양 데이터들을 추출하여 딕셔너리 형태로 저장했다.

```
		# 램 
        ram_match = re.search(r'램[:\s]*(\d+)\s*GB|(\d+)\s*GB.*?램', spec_text, re.IGNORECASE)
        if ram_match:
            product_data['RAM'] = ram_match.group(1) if ram_match.group(1) else ram_match.group(2)
        else:
            product_data['RAM'] = ''

        # 용량 (SSD)
        ssd_pattern = r'(SSD|NVMe|용량)[:\s]*(\d+)\s*(GB|TB)'
        ssd_match = re.search(ssd_pattern, spec_text, re.IGNORECASE)
        if ssd_match:
            capacity = ssd_match.group(2) 
            unit = ssd_match.group(3)
            if capacity and unit:
                product_data['SSD'] = str(int(capacity) * 1024 if 'TB' in unit.upper() else int(capacity))
            else:
                product_data['SSD'] = ''
        else:
            product_data['SSD'] = ''
```
데이터 크롤링 과정 출력 및 데이터 저장

```
============================================================
📄 1페이지 크롤링 중...
============================================================
현재 페이지 제품 수: 30개
  [10/30] 추출 중...
  [20/30] 추출 중...
  [30/30] 추출 중...
✓ 1페이지 완료 (누적: 30개)

→ 2페이지로 이동 중...

============================================================
데이터 저장 중...
============================================================
✅ 총 30개 제품 저장 완료!
   파일: data.csv

상위 5개 데이터:
                              모델명  화면크기    무게        CPU    그래픽 RAM  SSD  \
0     삼성전자 갤럭시북5 프로 NT940XHA-K51A    14  1.23    코어 울트라5  내장그래픽  16  256   
1         LG전자 울트라PC 15U50T-GA5HK  15.6   1.7  코어i5-13세대     기타  16  256   
2  삼성전자 갤럭시북4 NT750XGR-A51A WIN11  15.6  1.55  코어i5-13세대     기타  16  256   
3        삼성전자 갤럭시북4 NT750XGR-A51A  15.6  1.55  코어i5-13세대     기타  16  256   
4     LG전자 2025 그램15 15Z90T-GA5HK  15.6  1.29    코어 울트라5  내장그래픽  16  512   
...
  [30/30] 추출 중...
✓ 17페이지 완료 (누적: 510개)

🎯 목표 달성! 총 510개 수집 완료
```
---


## 2️⃣ 데이터프레임 저장

### 구현 내용
- **라이브러리**: Pandas
- **데이터 구조**: [모델명,화면크기,무게,CPU,그래픽,RAM,SSD,최저가,등록년월,평점,댓글개수]
- **데이터 형태**: [행 510개, 열 11개]

### 🔧 트러블슈팅

#### 문제: [크롤링 데이터를 데이터 프레임에저장]
- **상황**: 말 그대로 웹크롤링한 내역을 콘솔창에서 출력은 해봤지만 이런건 생소했다.
- **해결**: 반복문을 순회하며 노트북 하나의 데이터를 키:밸류 형태로 product_data 딕셔너리에 저장후 하나의 노트북 데이터를 all_products_data 리스트에 한 행씩 저장시킨다. 그후 all_products_data리스트를 데이터프레임에 저장.
```
all_products_data.append(product_data)
.
.
.
# ============================================================
    # 데이터 저장
    # ============================================================
    print(f"\n{'='*60}")
    print("데이터 저장 중...")
    print(f"{'='*60}")
    
    df = pd.DataFrame(all_products_data)
```

---

## 3️⃣ CSV 파일 저장

### 구현 내용
```python
df = pd.DataFrame(all_products_data)
df.to_csv('data.csv', index=False, encoding='utf-8-sig')
```

### 저장 전략
- **인코딩**: UTF-8-sig (한글 깨짐 방지)
- **경로**: `/` (원본 데이터 보존)

---

## 4️⃣ EDA (탐색적 데이터 분석)

### 📌 EDA 프로세스 요약
```
1. 데이터 수집 ✅
   ↓
2. 기본 정보 확인 ✅
   ↓
3. 기술 통계 ✅
   ↓
4. 결측치/이상치 확인 ✅
   ↓
5. 시각화 ✅
   ↓
6. 변수 간 상관관계 분석 ✅
   ↓
7. 인사이트 도출 ✅
```

---

### 4-1. 데이터 수집

- **원본 데이터**: 510건
- **수집 기간**: 2025.12.05(오늘 날짜까지)
- **데이터 출처**: [https://prod.danawa.com/list/?cate=112758]

---

### 4-2. 기본 정보 확인
```python
print(df.head())
print(df.shape)
```

**발견 사항**
- 총 480개 행, 11개 열
- 데이터 타입: [int64]
- 메모리 사용량: Z MB

---

### 4-3. 기술 통계
```python
# 기본통계량
print('LG:', df['모델명'].str.contains('LG').sum())
print('Samsung:', df['모델명'].str.contains('삼성').sum())
print('APPLE:', df['모델명'].str.contains('APPLE').sum())
```

LG: 320
Samsung: 136
APPLE: 24


**주요 통계량**<br>
LG: 320<br>
Samsung: 136<br>
APPLE: 24<br>
<!--
| 변수 | 평균 | 표준편차 | 최솟값 | 최댓값 |
|------|------|----------|--------|--------|
| A    | XX   | YY       | ZZ     | WW     |
| B    | XX   | YY       | ZZ     | WW     |
-->
**발견 사항**
- [변수명]: 평균 XX, 표준편차가 크다 → 데이터 분산이 큼
- [변수명]: 최댓값이 비정상적으로 큼 → 이상치 의심

---

### 4-4. 결측치/이상치 확인

#### 결측치 분석
```python
print(df.isna().sum())
```


| 변수   | 결측치 수 |
| ---- | ----- |
| 모델명  | 0     |
| 화면크기 | 0     |
| 무게   | 9     |
| CPU  | 10    |
| 그래픽  | 0     |
| RAM  | 0     |


**처리 방법**

- [CPU]: [처리 방법 + 근거: 임의 대체 시 통계적 왜곡 및 분석 신뢰도 저하 우려]
- [RAM]: [처리 방법 + 근거: 임의 대체 시 통계적 왜곡 및 분석 신뢰도 저하 우려]

```
# CPU, 무게 중 하나라도 결측이면 삭제
df = df.dropna(subset=['CPU', '무게'])
print(f"\n결측치 제거 후 shape: {df.shape}")
```
결측치 제거 후 shape: (461, 11)
전체 데이터 개수: 461

#### 이상치 분석

**탐지 방법**: IQR / Z-score
```python
# 이상치
df['최저가'] = pd.to_numeric(df['최저가'], errors='coerce') # 컬럼을 숫자(float)로 변환
df['평점'] = pd.to_numeric(df['평점'], errors='coerce') # 컬럼을 숫자(float)로 변환

# 파생 컬럼: 브랜드, 인기 
df['브랜드'] = df['모델명'].str.split(' ').str[0]
df['인기'] = pd.to_numeric(df['평점'], errors='coerce') * pd.to_numeric(df['댓글개수'], errors='coerce') # 숫자형으로 변환 

df['출시 연도'] = df['등록년월'].astype(str).str.split('.').str[0].astype(int)# 출시연도 파생 컬럼 추가
df['출시 연도'] = pd.to_numeric(df['출시 연도'], errors='coerce')

df.to_csv('data2.csv')

# 25년도 데이터 추출
df_2025 = df[df['등록년월'].str.startswith('2025')]

# 그 다음에 이상치 기준 계산
Q1 = df_2025['최저가'].quantile(0.25)# 하한
Q3 = df_2025['최저가'].quantile(0.75)# 상한 
IQR = Q3 - Q1

lower_bound_price = Q1 - 1.5 * IQR
upper_bound_price = Q3 + 1.5 * IQR

Q1 = df_2025['평점'].quantile(0.25)# 하한
Q3 = df['평점'].quantile(0.75)# 상한 
IQR = Q3 - Q1

lower_bound_score = Q1 - 1.5 * IQR
upper_bound_score = Q3 + 1.5 * IQR

```

**발견된 이상치**
- [변수명: 최저가, 평점]: 82건 발견
- **판단 기준**: [왜 이상치로 판단했는지: 최저가 이상치는 입력 오류 또는 특수 프로모션 가격일 가능성이 있어서? 평점 이상치는 일반적인 제품 품질 범위를 벗어난 극단값]
- **처리**: [제거] + 근거: 이상치 유지 시 평균/상관관계 등 통계량 왜곡

2025년 데이터 개수: 227
2025년 이상치 제거 후 데이터 개수: 145

---

### 4-5. 시각화

#### 분포 확인

![박스 플롯](./images/histogram.png)

**해석**
- [최저가]: 25년도 3사 모델 최저가는 150~225만원 분포
- [평점]: 25년도 3사 모델 평점은4.8~4.9사이 분포
![](https://velog.velcdn.com/images/shlim55/post/0131b66e-ebcb-4c9f-890a-6d5faff86c9d/image.png)

#### 변수 간 관계
![히스토그램](./images/histogram.png)

**해석**
- [브랜드]: 정규분포에 가까움
- [상품 개수]: LG전자, 삼성전자, APPLE 순서대로 많았다.


![산점도](./images/scatter.png)

**해석**
- [가격대]와 [인기(평점*리뷰)]: 약한 상관관계 보임

![박스 플롯](./images/histogram.png)

**해석**
- [최저가]: LG는 150~225만원사이, 삼성전자는 175~225만원 사이, APPLE은 150~200만원 사이였다.


<img width="1621" height="577" alt="image" src="https://github.com/user-attachments/assets/c9311d5b-875b-4c5b-8f56-a0cf6f49f8e3" />


#### 시간의 흐름에 따른 데이터 변화
![선 그래프](./images/scatter.png)

**해석**
- [출시 연도]와 [최저가]: LG 노트북은 3년간 꾸준히 상승중이다. 삼성 노트북은 24년도에 약간 상승했으며, 24년도와 25년도는 거의 유사하다. APPLE 노트북은 24년도에 상승하여 제일 비쌌으며, 23년도와 25년도 가격은 유사하다.

![](https://velog.velcdn.com/images/shlim55/post/e81d8d4e-2b74-4bc7-a709-5c4e4f3ae24d/image.png)

---

### 4-6. 변수 간 상관관계 분석

#### 상관계수 히트맵
![상관관계](./images/correlation_heatmap.png)

![](https://velog.velcdn.com/images/shlim55/post/dc35b0e6-756f-461b-bf2d-76641742b6b2/image.png)

#### 주요 발견
| 변수 쌍 | 상관계수 | 해석 |
|---------|----------|------|
| RAM - 최저가   | 0.73     | 강한 양의 상관관계 |
| SSD - 최저가   | 0.68    | 강한 양의 상관관계 |
| 출시 연도 - 무게   | -0.40    | 강한 음의 상관관계 |
| 평점 - 무게   | -0.10    | 약한 음의 상관관계 |

**통계적 유의성**
- p-value < 0.05: 통계적으로 유의미함 ✅
- [구체적 검증 방법]

---

### 4-7. 인사이트 도출

#### 💡 주요 발견사항

**1. [가격과 인기의 상관관계]**
- **발견**: [상관계수 0.04로 약한 양의 상관관계 있음]
- **의미**: [리뷰, 평점이 높다고 가격이 비싼것은 아니다.]
- **근거**: [510건 데이터로 산점도및, 히트맵 상관분석 수행]

**2. [RAM 성능,SSD 성능과 최저가의 상관관계]**
- **발견**: [RAM은 상관계수 0.73로 강한 양의 상관관계 있음 분석한 하드웨어 스펙 중 가장 높은 상관관계],  [SSD는 상관계수 0.68로 강한 양의 상관관계 있음 (RAM 0.73 다음으로 높음)]
- **의미**: [RAM, SSD 성능이 높으면 가격이 올라간다.]
- **근거**: [510건 데이터로 히트맵 상관관계 분석 수행]


**3. 실제 사례로 보는 RAM/SSD 업그레이드의 가격 효율성**

### 📊 동일 모델 스펙별 가격 비교

| 모델 | RAM | SSD | 가격 | 리뷰 수 | 별점 |
|------|-----|-----|------|---------|------|
| 16Z90TS | 16GB | 256GB | 157만원 | 142개 | 4.9 |
| 16Z90TP | 32GB | 1024GB | 220만원 | 403개 | 4.9 |
| **차이** | **2배↑** | **4배↑** | **+63만원 (40%↑)** | **2.8배↑** | **동일** |

**4. 장기적 관점**
```
5년 사용 가정:
- 추가 비용: 63만원
- 연간 비용: 12.6만원
- 월간 비용: 1만원

→ 월 1만원으로 쾌적한 작업 환경 확보

월 1만원, 커피 한 잔 값으로 업그레이드하세요

5년 사용 시:
- 총 투자: 63만원
- 연간: 12.6만원
- 월간: 약 1만원 (커피 2-3잔 값)

💬 이렇게 생각해보세요:
"매달 스타벅스 한두 번 덜 마시면
 향후 5년간 쾌적한 업무 환경을 
 보장받을 수 있습니다."

✅ 작업 효율 ↑
✅ 대기 시간 ↓
✅ 스트레스 ↓

→ 월 1만원의 가치, 충분하지 않을까요?
```

---

## 🛠️ 기술 스택

**데이터 수집**
- Python 3.x
- BeautifulSoup4 / Selenium


**데이터 처리 & 분석**
- Pandas, NumPy
- Matplotlib, Seaborn


**시각화**
- Seaborn
- Matplotlib (선택)

---

## 🚀 실행 방법

[기존 내용]

### 환경 요구사항
- Python 3.8 이상
- Jupyter Notebook

### 1. 패키지 설치

**필수 패키지:**
```bash
pip install pandas numpy matplotlib seaborn
```

**크롤링용 패키지:**
```bash
pip install beautifulsoup4 requests
```

**또는 한 번에:**
```bash
pip install pandas numpy matplotlib seaborn beautifulsoup4 requests
```

### 2. 노트북 실행
```bash
# Jupyter Notebook 실행
jupyter notebook
```

브라우저가 자동으로 열리면 `mini_project.ipynb` 파일을 클릭

### 3. 셀 실행 순서

1. **데이터 수집** (크롤링 셀) - 약 2분 소요
2. **데이터 전처리** - 약 1분
3. **EDA 분석** - 약 1분
4. **시각화 및 인사이트 도출**

각 셀을 위에서부터 순서대로 `Shift + Enter`로 실행

### 4. 실행 결과
- 크롤링된 데이터: `data.csv, data2.csv` 파일 생성
- 분석 그래프: 노트북 내에서 바로 확인
- 최종 인사이트: 노트북 마지막 섹션

---

## ⚠️ 참고사항

- 크롤링 대상 사이트의 정책에 따라 실행이 제한될 수 있습니다
- 이미 수집된 데이터로 분석만 하고 싶다면 크롤링 셀은 건너뛰세요
---

## 💭 회고

### 잘한 점
- [ ] [실제 웹 페이지 크롤링+ 데이터프레임 활용및 .csv 저장 한 데이터 바탕으로 전처리, 시각화 진행, 배운 모든 그래프 활용해서 분석 및 인사이트 도출 ]
- [ ] 경력직 피드백: "데이터 분석이 잘 됐다"

### 아쉬운 점 & 개선 방향
- [ ] 현재: CSV 저장 → 개선: PostgreSQL로 DB 구축
- [ ] 현재: 1회성 실행 → 개선: Airflow 스케줄링
- [ ] 현재: 머신러닝 구현 못함 → 개선: 26년도 3사 플래그쉽 가격 예측 등
---

## 📚 참고 자료
- [사용한 데이터셋 출처: 다나와]
- [참고한 분석 방법론: EDA 과정]
