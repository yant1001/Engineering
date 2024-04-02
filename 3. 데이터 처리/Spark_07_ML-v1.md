# Spark Ml 기반 예측 모델 - 단일 회귀
- 원본 데이터: `Spark_07_ML`의 `trips` 데이터셋
- `trips`가 데이터 레이크 역할
- 현재 데이터를 따로 join할 필요가 없기 때문에 데이터 레이크 상태 그대로 데이터 마트 작업 수행
  - L-W-M 3단계는 상황에 따라 유동적으로 수행된다.


## 순서
1. 데이터 정제
   - 데이터프레임을 SQL에서 새용하기 위해서는 TempView를 생성해주어야 한다.
     - `createOrReplaceTempView()`
   - 원하는 목적에 맞게 데이터를 추출한다.
     - `query =""" <SQL query> """`
     - `data_df = spark.sql(query)`
2. Train/Test 분할
   - 데이터셋 분할 후 train 데이터셋만 캐시에 삽입
     - `train_sdf, test_sdf = data_df.randomSplit([0.8, 0.2], seed=42)`
     - `train_sdf.cache()`
   - ML은 보통 train/test 데이터셋을 분할 후, train 셋을 이용하여 각종 전처리를 수행한다.
   - 전처리할 때도 transform이 반복되는데, 이후에도 여러 모델까지 훈련시키기 때문에 train 데이터셋이 변형되는 작업이 굉장히 많아지며 시간도 많이 소요된다.
     - 똑같은 train_sdf가 각각 다른 전처리 루틴만큼, 훈련시킨 모델의 개수만큼 만들어진다.
     - train 데이터셋은 어떻게 변환이 되더라도 하나만 존재하는 게 좋다.
   - 그렇기 때문에 train을 cache에 저장하여 사용한다. (캐싱)
     - 모델마다 공통되는 전처리 과정을 모두 수행한 후 train_sdf를 캐시에 넣어놓는다.
     - 실제 훈련 직전까지 사용할 데이터는 캐싱을 하는 것이 효율적이다.
3. 특성 벡터 어셈블링
   - 행벡터화
     - feature가 1개라고 하더라도 반드시 어셈블링(행벡터화)을 해야한다.
     - ML에는 무조건 2차원 형태로 진행되기 때문
       - `R^N*M`에서의 `M`을 만들어주는 과정
        ```py
        from pyspark.ml.feature import VectorAssembler

        # 행벡터화
        vec_assembler = VectorAssembler(
            inputCols = ['trip_distance'],
            outputCol = 'features'
        )
        ```
   - 행벡터화 후 변환까지
     - `vec_train_sdf = vec_assembler.transform(train_sdf)`
     - features라는 이름으로 데이터를 어셈블링
     - features 컬럼 외 다른 컬럼(trip_distance, total_amount 등)은 내부적으로 올라가지 않는 데이터들이며, 실제 ML에는 features 컬럼만 올라간다.
4. 모델 생성 및 훈련
   - estimator는 데이터를 예측하는 형태로 변환시켜주므로 transformer이다.
    ```py
    from pyspark.ml.regression import LinearRegression

    # 모델 생성
    lr = LinearRegression(
        maxIter = 50,
        featuresCol = 'features',
        labelCol = 'total_amount'
    )

    # 모델 훈련
    lr_model = lr.fit(vec_train_sdf)
    ```
5. 예측
   - test용을 따로 만들지 않고 앞서 assembler 단계에서 만들었던 transformer를 그대로 사용한다.
     - features 컬럼이 추가 생성된다.
     - `vec_test_sdf = vec_assembler.transform(test_sdf)`
   - lr_model 예측(transform)을 수행한다.
     - 예측 결과 컬럼이 뒤에 추가 생성된다.
     - `predictions = lr_model.transform(vec_test_sdf)`
6. 평가
   - 오차 확인
     - `lr_model.summary.rootMeanSquaredError`
   - 정확도 확인
     - `lr_model.summary.r2`
