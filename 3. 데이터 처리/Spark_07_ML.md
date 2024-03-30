# MLlib
- Machine Learning Library
- 스파크 기반의 머신러닝 라이브러리로, 최근에는 Spark ML로 이름이 바뀌어서 쓰이는 경우도 종종 있다.
- 데이터프레임과 연동하여 쉽고 빠르게 ML 모델 개발이 가능하다.
## Spark Conponent
- MLlib은 Spark Core(RDD)의 DataFrame API 기반으로 작동되기 때문에 그 위에서만 사용 가능하다.
  - Spark Core(RDD)의 Spark SQL에는 크게 3개의 주요 API가 존재한다.
  - SQL API, DataFrame API, DataSets API가 있으며, 이 중 DataFrame API 안에 MLlib이 있다.
- RDD 기반으로 작동하는 패키지도 존재했었으나 현재는 지원하지 않는다.
- DataFrame을 쓰는 MLlib API를 Spark ML이라고 하며, 이제는 MLlib==Spark ML이다.
## Machine Learning
- 많은 데이터를 기반으로 데이터의 패턴을 분석한다.
- 프로그래밍 구문이 아닌 데이터에 의해 프로그램이 만들어진다.
- 머신러닝의 파이프라인은 아래와 같다.
  - 데이터 로딩 → 전처리 → 학습 → 모델 평가
  - 모델 평가 결과 모델이 충분한 성능을 보이지 않는다면 파라미터 튜닝 후 다시 시도한다.
## Transformer
- MLlib의 주요 컴포넌트 중 하나로, Feature Transformation을 담당한다.
- 데이터를 변형(전처리)하는 모듈이다.
- 일반적으로 Feature Engineering 도구라고 본다.
- 모든 transformer는 `transform()` 함수를 가지고 있다.
- 머신러닝이 가능한 형태로 데이터의 포맷을 바꿔준다.
- 원본 DF를 변형하여 새로운 DF를 생성한다.
  - 보통 하나 이상의 컬럼이 더해지는 작업으로, 사아킷런의 pandas DF와 차이점이다.
  - spark DF는 immutable 속성으로 DF 자체 변형이 안되기 때문에, 변형된 데이터를 컬럼으로 추가시킨다.
## Estimator
- MLlib의 주요 컴포넌트 중 하나로, 모델의 학습을 담당한다.
- 모든 estimator는 `fit()` 함수를 가지고 있다.
- `fit()`은 DataFrame을 입력받아 학습한 다음 모델을 반환한다.
  - 모델에 x를 집어넣어서 f(x)로 y라는 결과값이 나온다.
- `fit()`을 통해 훈련된 모델은 하나의 transformer이기 때문에 predict가 존재하지 않는다.
  - 데이터를 삭제하거나 편집하는 것이 불가능한 RDD이기 때문이다.
  - 지금까지 알던 ML은 모델을 만들면 그 자체가 예측 모델이었는데, Spark에서는 기존 데이터프레임의 변경이 안되기 때문에 만들어진 모델이 transformer이 된다.
  - 이 점이 사이킷런 머신러닝과 스파크 머신러닝의 가장 큰 차이점이다.
## Evaluator
- MLlib의 주요 컴포넌트 중 하나로, 평가 방식(metric)을 기반으로 모델의 성능을 평가한다.
  - RMSE, MSE, MAE 등
- 여러 모델을 만들어서 성능을 평가한 후 가장 좋은 모델을 선택하는 방식으로 모델 튜닝을 자동화시킬 수 있다.

# 사이킷런으로 만든 분류 모델
- 데이터를 자르고, 모델에 넣고, 예측하고, 결과물을 확인하는 모든 과정이 독립적이었다.
```PY
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import  accuracy_score

# 훈련 데이터와 테스트 데이터셋 분할
X_train, X_test, y_train, y_test = train_test_split(
    iris_data,
    iris_label,
    test_size=0.2,
    random_state=42
)

# 분류모델 훈련
tree_clf = DecisionTreeClassifier()
tree_clf.fit(X_train, y_train)

# 분류모델 예측
pred = tree_clf.predict(X_test)

# 성능 확인
#   정답(y_test)과 훈련을 통한 예측(pred) 데이터를 비교해서 정확도 구하기
accuracy = accuracy_score(y_test, pred)
print(pred)
```

# Spark ML으로 만든 분류 모델
## 훈련 & 테스트 데이터셋 분할 - randomSplit()
```py
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName('tree-clf').getOrCreate()
spark

iris_filepath = '<file_address>'
iris_sdf = spark.read.csv(f'file://{iris_filepath}', inferSchema=True, header=True)

# 훈련 데이터와 테스트 데이터셋 분할
#   DataFrame API의 randomsplit 메소드 활용 (transformation)
train_sdf, test_sdf = iris_sdf.randomSplit([0.8, 0.2], seed=42)
```

## 행벡터로 transform - VectorAssembler()
- 데이터셋 분할 후 `VectorAssembler()`라는 transformer을 이용해서 행벡터화한다.
  - spark는 column으로 묶여있고, pandas는 row로 묶여있다.
  - 즉 열벡터로 되어있는 spark DF를 Pandas DF처럼 행벡터로 바꿔준다.
- `VectorAssembler()`와 같이 데이터프레임을 변형하는 클래스를 transformer라고 한다. transformer는 데이터프레임을 변형할 수 있는 `transform` 메소드를 갖는다.
- 만들어진 행벡터는 하나의 엔티티이며, 이 엔티티를 머신러닝 알고리즘에 투입하는 모델을 추정기라고 한다.
```py
from pyspark.ml.feature import VectorAssembler

# 열벡터를 행벡터로 만들기
iris_columns = ['sepal_length', "sepal_width", "petal_length", "petal_width"]
vec_assembler = VectorAssembler(inputCols=iris_columns, outputCol='features')

# 훈련 데이터에 transform() 메소드로 새로운 컬럼을 추가
train_feature_vector_sdf = vec_assembler.transform(train_sdf)
train_feature_vector_sdf.show(5)
```

## 모델 학습 - fit()
- 예측을 할 수 있는 추정기(Estimator)를 적용한다.
  - pandas DF의 fit 과정
- Estimator는 예측을 할 수 있는 모델을 의미하며, 데이터를 변환시키는 transformer에 해당한다.
  - 예측을 할 때 사이킷런처럼 예측값만 출력되는게 아니라, feature가 들어있는 DF를 받은 후 예측값 컬럼을 추가해주는 변환(transformation) 과정을 수행하기 때문이다.
- 모든 estimator는 `fit()`이라는 action 함수를 갖는다.
- 훈련(fit) 이후 수행되는 예측 과정은 transformation 과정이다.
```py
from pyspark.ml.classification import DecisionTreeClassifier

# 모델의 껍데기만 일단 생성한다.
#   훈련 데이터프레임 컬럼 중에서 feature와 target을 지정한다.
#   feature에는 행벡터화된 컬럼이 들어간다.
dt = DecisionTreeClassifier(
    featuresCol = 'features',
    labelCol = 'target',
    maxDepth = 5
)

# 모델 학습
#   fit() 함수가 실행되면서 실제 연산이 일어난다.
#   사이킷런 ML모델에서 fit(X_train, y_train)으로 훈련 데이터를 넣었던 것처럼, fit(train_feature_vector_sdf)를 넣는다.
#   train_feature_vector_sdf는 행벡터화로 transform 시켰던 train 데이터셋이다.
dt_model = dt.fit(train_feature_vector_sdf)
```

## 테스트 데이터 예측
- 훈련(train) 과정에서 진행했던 transform 과정이 동일하게 적용된다.
- 절대로 테스트 셋을 위한 transformer를 새로 만들면 안된다.
  - 앞에서 행벡터화 할 때 `VectorAssembler()`로 만들었던 transformer인 `vec_assembler`을 그대로 사용해서 transform 한다.
```py
# 테스트 데이터를 행벡터화 transform
test_feature_vector_sdf = vec_assembler.transform(test_sdf)

# 테스트 데이터셋으로 예측
#   행벡터화된 테스트 데이터를 훈련 데이터로 학습시킨 모델로 예측 transform
predictions = dt_model.transform(test_feature_vector_sdf)
predictions.show(5)
```
- 예측까지 수행하게 되면 기존 데이터프레임에 아래 컬럼이 추가된다.
  - `rawPrediction` : 머신러닝 모델 알고리즘 별로 다를 수 있다.
      - 머신러닝 알고리즘에 의해서 계산된 값
      - 값에 대한 정확한 의미는 없다.
      - `LogisticRegression`의 경우 예측 label 별로, 예측 수행 전 `sigmoid` 함수 적용 전 값
          - $ \hat{y} = \sigma(WX + b) $
          - $ WX + b $의 결과가 `rawPrediction`
  - `probability` : 예측 label 별 예측 확률 값
  - `prediction` : 최종 예측 label 값

## 모델 평가
```py
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

evaluator_accuracy = MulticlassClassificationEvaluator(
    labelCol = 'target',
    predictionCol = 'prediction',
    metricName = 'accuracy'
)
accuracy = evaluator_accuracy.evaluate(predictions)
accuracy
```


# Machine Learning 파이프라인 구성 (4/1)