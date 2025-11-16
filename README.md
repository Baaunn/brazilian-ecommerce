# 🇧🇷 Olist 브라질 이커머스 데이터 분석
### : 고객 만족도의 핵심 동인과 VIP/이탈 고객 세그먼트 분석



## 1. 프로젝트 개요 (Introduction)

본 프로젝트는 브라질 최대 이커머스 플랫폼인 **Olist**의 공개 데이터셋을 활용하여, **고객 만족도에 영향을 미치는 핵심 요인을 파악**하고 **고객 이탈 원인을 규명**하는 것을 목표로 합니다.

9개의 분리된 테이블을 병합, 정제하고 RFM 고객 세분화, EDA, 지리공간 분석을 통해 "어떻게 하면 고객을 만족시키고 유지할 수 있는가?"라는 비즈니스 질문에 대한 데이터 기반의 답을 찾고자 했습니다.

* **데이터 소스:** [Kaggle: Olist Brazilian E-commerce Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

---

## 2. 핵심 분석 결론 

분석 결과, 3가지 핵심 인사이트를 도출했습니다.

### 1. 배송 경험이 고객 만족도의 핵심
`review_score`(리뷰 점수)는 **`delivery_time`(배송 시간)과 가장 강력한 음의 상관관계**를 보였습니다. 고객 만족(5점)의 평균 배송 시간은 약 10일인 반면, 불만족(1점)의 평균 배송 시간은 20일을 초과했습니다.

특히, `R_Score=3`(중간 고객) 그룹에서 1점 리뷰가 비정상적으로 높게 나타났는데, **해당 그룹 1점 리뷰의 평균 배송 시간이 24일**로 전 그룹 중 가장 길었음을 증명하며 가설을 입증했습니다.

### 2. VIP와 이탈 고객은 완전히 다른 고객
RFM 분석을 통해 고객을 세분화하고 행동 패턴을 교차 분석한 결과, 두 그룹은 구매 품목과 가격대에서 명확한 차이를 보였습니다.

* **VIP 고객 (`555` 그룹):** `health_beauty`, `bed_bath_table` 등 **반복 구매성 생활용품**을 다양한 가격대에서 구매했습니다.
* **이탈 고객 (`111` 그룹):** `telephony`, `computers_accessories` 등 **일회성 전자제품**을 특정 (저가~중가) 가격대에 집중적으로 구매했습니다.

### 3. "만족한 1회성 고객"의 이탈이 문제
이탈 고객 그룹의 평균 리뷰 점수는 **4.10점**으로, 전체 평균(4.01점)보다 오히려 높았습니다. 이는 고객이 '불만족'해서 이탈한 것이 아니라, **첫 구매에 '만족'했음에도 불구하고 재구매로 이어지지 않았음**을 의미합니다. Olist가 일회성 구매 고객을 플랫폼에 락인시킬 CRM 전략이 부재함을 시사합니다.

---

## 3. 분석 프로세스 (Analysis Process)

1.  **데이터 로딩 및 정제 (Data Cleaning & Merging)**
    * 9개의 `.csv` 파일을 로드하고, 9개 테이블을 `order_id`, `customer_id` 등을 기준으로 하나의 마스터 테이블(`df_merged`)로 병합했습니다.
    * `null` 값의 의미를 파악했습니다. (예: `order_status`가 `canceled`일 때 `delivery_time`이 `null`인 것은 '배송 실패'를 의미하는 정상 데이터임을 확인)

2.  **피처 엔지니어링 (Feature Engineering)**
    * `delivery_time` (실제 배송 소요 시간), `delivery_diff_time` (예상 대비 실제 배송 시간) 등 고객 경험을 측정할 핵심 파생 변수를 생성했습니다.

3.  **RFM 고객 세분화 (RFM Analysis)**
    * `Recency`, `Frequency`, `Monetary` 값을 계산했습니다.
    * 데이터 쏠림(Skewness) 문제를 해결하기 위해 `rank(method='first')`로 강제 등수를 부여한 뒤, `pd.qcut`을 사용해 1-5점의 RFM 스코어를 부여했습니다.
    * `'555'` 그룹을 (VIP), `'111'` 등 R이 낮은 그룹을 (Churn)으로 정의했습니다.

4.  **탐색적 데이터 분석 (EDA)**
    * **배송과 만족도:** `boxplot`을 통해 배송 시간이 길어질수록 리뷰 점수가 급락함을 시각화했습니다.
    * **주문 상태와 만족도:** '배송 실패'(`processing`, `unavailable`) 주문의 평균 리뷰 작성 시간이 30일로 가장 긴 것을 발견, "리뷰 생성 시간 = 고객이 기다린 시간"임을 증명했습니다.
    * **VIP vs. Churn 교차 분석:**
        * 구매 카테고리 비교 (`barh` plot)
        * 구매 가격대 비교 (Log-Transformed `kdeplot`)
        * 평균 리뷰 점수 비교

5.  **지리공간 분석 (Geospatial Analysis)**
    * `Folium`을 활용해 브라질 지도 위에 고객 분포(`HeatMap`)와 '주(State)'별 총매출(`Choropleth`), '주(State)'별 VIP 고객 수(`Choropleth`)를 시각화했습니다.
    * 매출과 VIP 고객 모두 'SP'(상파울루) 등 남동부 지역에 압도적으로 밀집되어 있음을 확인했습니다.

---

## 4. 사용 기술 (Tech Stack)

* **Data Handling:** `Python`, `Pandas`, `Numpy`
* **Data Visualization:** `Matplotlib`, `Seaborn`, `WordCloud`, `Folium`
* **Environment:** `Jupyter Notebook`
