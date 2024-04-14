# Spark Ml 기반 예측 모델 - 다중 회귀

## 순서
1. 데이터 정제
   - 데이터프레임을 SQL에서 새용하기 위해서는 TempView를 생성해주어야 한다.
     - `createOrReplaceTempView()`
   - 원하는 목적에 맞게 데이터를 추출한다.
     - `query =""" <SQL query> """`
     - `data_df = spark.sql(query)`
2. Train/Test 분할
  에라이~
3. 파이프라인 정의
4. OneHotEncoding Stage
5. Standard Scaling Stage
6. Feature Assemble Stage
7. 파이프라인 구성
8. 데이터를 파이프라인에 통과시키기
9.  모델 생성 및 훈련
10. 예측
11. 평가