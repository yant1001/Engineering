# Data Pipeline 정리
- Data Lake 단계는 아무 것도 정리되지 않은 모든 데이터가 다 들어가있다.
- Data Warehouse 단계는 Join에 의해 데이터가 하나로 묶인다.
- Data Mart 단계는 내가 원하는 비즈니스에 맞는 데이터를 만들어낸다.

## Data Lake
데이터 적재 즉 쌓아나간다

데이터를 쌓아나가고 싶은 형태로 데이터를 변환하면서 Load(적재)하는것 == ETL

ETL은 데이터 수집 시간이 너무 길었던 문제

수집한 데이터를 일단 저장 후 변환하는 것 == ELT

ELT로 인해 lake, warehouse, mart 개념이 생겨난것

csv 파일을 주피터 파일에 업로드한 상태 (아무 작업 안함) == **Data Lake**

즉 데이터 레이크는 데이터 수집 & 데이터 저장소의 역할까지만 수행

flume은 데이터 수집 도구이다.


## Data Warehouse
산개되어있는 데이터를 묶어놓는 역할

웨어하우스를 구성할 때, 데이터를 뭉쳐만 놓을수도 있고, Null이나 중복 데이터값 등을 처리할 수도 있다 (선택의 문제)


## Data Mart
원하는 형태로 데이터를 걸러준다



<hr>

# Data Lake → Data Warehouse


# Data Warehouse → Data Mart


# 시각화