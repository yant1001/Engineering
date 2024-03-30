# Data Pipeline 정리
- Data Lake 단계는 아무 것도 정리되지 않은 모든 데이터가 다 들어가있다.
- Data Warehouse 단계는 Join에 의해 데이터가 하나로 묶인다.
- Data Mart 단계는 내가 원하는 비즈니스에 맞는 데이터를 만들어낸다.

## 배경
- 데이터 처리 방법인 ETL과 ELT (Extract Load Transform)
- 데이터를 쌓아나가고 싶은 형태로 변환하면서 Load(적재)하는것 == ETL
- ETL은 데이터 수집 시간이 너무 길었던 문제
- 수집한 데이터를 일단 저장 후 변환하는 것 == ELT
- ELT로 인해 Lake, Warehouse, Mart 개념이 생겨난것

## Data Lake
- 데이터F를 적재하는 역할이다.
- Data Lake는 데이터 수집 & 저장소의 역할까지만 수행한다.
  - Flume은 데이터 수집 도구이다.
- 예를 들어, csv 파일을 주피터 파일에 업로드한 상태(아무 작업 안함)는 Data Lake이다.

## Data Warehouse
- Data Lake에 산개되어 있는 데이터를 묶어놓는 역할이다.
- 연관성 있는 데이터들끼리 분석하기 위해 Join으로 준비 데이터를 마련한다. 
- Data Warehouse를 구성할 때, 데이터를 뭉쳐만 놓을수도 있고, Null이나 중복 값 등읙 결측치를 처리할 수도 있다. (선택의 문제)

## Data Mart
- 데이터를 원하는 형태로 걸러주는 역할이다.
- 데이터 분석에 필요없는 데이터를 정제하고 비즈니스에 맞는 데이터만 남긴다.

<hr>

# Data Lake → Data Warehouse
## 원본 데이터 배경
- `trips`: 2021년 뉴욕 택시 운행 기록 데이터
- `zone`: 2021년 뉴욕 장소 관련 코드 데이터
- 원본 csv 데이터 파일이 모인 주피터 노트북 디렉토리가 **데이터 레이크** 역할을 한다.
  - 로컬에 간단히 구현한 모습이며, HDFS 등의 다른 방법으로도 데이터 레이크를 구성할 수 있다.
## 쌓여있던 데이터가 묶이는 과정
- Data Lake → Data Warehouse
1. 현재 csv파일로 저장되어 있는 데이터프레임 형태의 `trips`와 `zone`의 임시 뷰를 생성한다.
```py
trips_df.crateOrReplaceTempView('trips')
zone_df.crateOrReplaceTempView('zone')
```
2. `trips`와 `zone` 데이터를 `Join`하여 웨어하우스를 생성한다.
```py
query = """
SELECT
    t.VendorID AS vendor_id,
    TO_DATE(t.tpep_pickup_datetime) AS pickup_date,
    TO_DATE(t.tpep_pickup_datetime) AS dropoff_date,
    HOUR(t.tpep_pickup_datetime) AS pickup_time,
    HOUR(t.tpep_pickup_datetime) AS dropoff_time,
    t.passenger_count,
    t.trip_distance,
    t.fare_amount,
    t.tip_amount,
    t.tolls_amount,
    t.total_amount,
    t.payment_type,
    pz.Zone as pickup_zone,
    dz.Zone as dropoff_zone
FROM trips AS t
LEFT JOIN zone AS pz ON t.PULocationID = pz.locationID
LEFT JOIN zone AS dz ON t.DOLocationID = dz.locationID
"""
comb_df = spark.sql(query)
comb_df.show(10)
```
3. `comb_df`가 **데이터 웨어하우스** 역할을 한다.
4. 웨어하우스에 SQL을 사용하기 위해 임시 뷰를 등록한다.
- 데이터프레임에 SQL(`sql()`)을 사용하려면 TempView를 만들어서 메모리에 저장해야한다.
```py
comb_df.createOrReplaceTempView('comb')
```

# Data Warehouse → Data Mart
1. 분석에 필요없는 데이터를 걸러내기 위해 여러 정보들을 찾아본다.
- 데이터 중 정제의 대상을 알아내어, ELT 중 T(Transform) 과정을 수행하기 위한 목적이다.
- 예를 들어, (1)데이터들의 날짜와 시간을 확인 (2)요금 데이터에 대한 통계 정보 확인 (3)거리 데이터 확인 (4)월별 운행 수 확인 (5)승객 수 확인 등을 살펴본다.
  - `select()`, `describe()` 등을 이용한다.
- 비즈니스에 따라 데이터 처리 기준을 정해야 한다.
  - 예를 들어, 2020/12/31에 택시를 탔지만 2021/01/01에 내린 고객은 몇년도에 해당하는 고객으로 간주해야 할까?
```py
# 날짜와 시간 확인
query = """
SELECT pickup_date, pickup_time
FROM comb
ORDER BY pickup_date
"""
spark.sql(query).show()

# 월별 운행 수 확인
#   DATE_TRUNC: 데이터를 잘라준다.
query = """
SELECT 
    DATE_TRUNC('MM', pickup_date) AS month,
    COUNT(*) AS trips
FROM comb
GROUP BY month
ORDER BY month DESC
"""
spark.sql(query).show()
```
2. 살펴본 내용을 토대로, 원본 데이터를 실제 분석할 데이터로 정제한다.
```py
query = """
SELECT *
FROM comb AS c
WHERE c.total_amount < 200
  AND c.total_amount > 0
  AND c.passenger_count < 5
  AND c.pickup_date >= '2021-01-01'
  AND c.pickup_date < '2021-08-01'
  AND c.trip_distance < 10
  AND c.trip_distance > 0 
"""
cleaned_df = spark.sql(query)
```
3. `cleaned_df`가 **데이터 마트** 역할을 한다.
4. 데이터프레임 형태인 데이터 마트 자료를 SQL로 사용하기 위해 TemView를 등록한다.
- `cleaned_df.createOrReplaceTempView('cleaned')`


# 시각화
- spark를 사용하는 분산처리 환경에서는 제플린을 활용해서 시각화한다.
  - 제플린: 디자이너와 개발자가 협업하기 위해 사용하는 응용 프로그램
- spark를 모두 사용한 후에는 pandas 기반의 시각화를 jupyter notebook 상에서 수행할 수도 있다.
- `matplotlib`, `seaborn` 등을 이용하여 시각화하는 과정
  1. 시각화 라이브러리를 불러온다.
     - `import numpy as np `
     - `import pandas as pd`
     - `import seaborn as sns`
     - `import matplotlib.pyplot as plt`
  2. pickup_date별 운행 수를 확인한다.
    ```py
    query = """
    SELECT
        pickup_date,
        COUNT(pickup_date) AS trips
    FROM cleaned
    GROUP BY pickup_date
    """
    ```
  3. `toPandas()`를 이용하여 pandas DF로 만들어준다.
     - `toPandas()`는 action 함수이다.
     - `pd_df = spark.sql(query).toPandas()`
  4. 시각화한다.
```py
plt.subplots(figsize=(20, 8))
sns.lineplot(
    x='pickup_date',
    y='trips',
    data=pd_df
)
plt.show()
```