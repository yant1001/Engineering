# Spark SQL

# 구조화된 데이터
## 데이터를 합치고 추출하는 순서
- 예를 들어, join과 filter를 함께 사용하는 경우, filter로 데이터량을 줄인 다음 join으로 합치는 순서가 효율적이다.
- 먼저 데이터를 걸러내고 합치기 때문에 데이터의 셔플링을 최소화할 수 있다.
- 매번 셔플링으로 인한 효율성을 고려한다면 개발자에 따라 성능 편차가 심해지기 때문에, 데이터를 구조화시킨다면 자동 최적화할 수 있다.

## 데이터 구조화
1. Unstructured
   - 비 구조화된 데이터로, 형식이 없다.
   - 로그 파일, 이미지 등
2. Semi Structured
   - 준 구조화된 데이터로, 행과 열만 갖는다.
   - csv, json, xml 등
     - 예를 들어, csv는 타입이 없고 전부 문자열로 구성되어있다. pandas DF로 csv를 불러올 때 데이터 타입을 자동으로 부여하기 때문에 타입이 있는 것처럼 보였던 것이다.
3. Structured
   - 구조화된 데이터로, 행과 열과 데이터 타입(스키마)를 갖는다.


### RDD와 구조화된 데이터의 차이
- RDD
  - 데이터 구조를 모르기 때문에 (스키마 즉 형식이 없기 때문에) 개발자에게 의존한다.
  - `map`, `flatMap`, `filter` 등을 통해 사용자가 만든 function을 수행한다.
- 구조화된 데이터 (Structured Data)
  - 이미 데이터의 구조를 알고 있기 때문에 어떤 task를 수행할 것인지 정의만 하면 된다.
  - 자동으로 최적화되기 때문에 reduction 과정이 자동 수행된다.

# Spark SQL
- 스파크를 기반으로 만들어진 하나의 패키지이다.
- 크게 3개의 주요 API가 존재한다.
  - SQL
  - DataFrame
  - DataSets
- spark sql을 사용하면 일일이 function을 정의할 일 없이, 구조화된 데이터를 다룰 수 있게 해준다.
- SQL만 작성하면 자동으로 연산이 최적화된다.

## 목적
- 스파크 프로그래밍 내부에서 RDBMS(관계형 처리)처럼 사용하기 위함이다.
  - Spark SQL이 RDBMS는 아니다.
  - 스파크 내부 인메모리 안에 데이터프레임이 존재하기 때문에 스파크가 종료되면 데이터가 사라진다.
- 스키마의 정보를 이용해 자동으로 최적화하기 위함이다.
- csv, tsv, json 등의 외부 데이터를 쉽게 사용하기 위함이다.

## DataFrame
- Spark Core에 RDD가 있다면, Spark SQL에는 DataFrame이 있다.
  - RDD를 기반으로 만들어졌으며, RDD의 성질을 알고 있으면 더 최적화된 DF 사용이 가능하다.
  - RDD에 스키마가 적용된 것이라고 개념적으로 이해 가능하다.
- RDD에 스키마를 정의한 다음 DataFrame으로 변형 가능하다.
  - RDD를 이용해 DataFrame을 만드는 방법은 아래 두 가지이다.
  - Schema를 자동으로 유추해서 DataFrame을 만들 수 있다.
  - 개발자가 직접 Schema를 정의해서 DataFrame을 만들 수 있다.
- text, csv, json 등의 데이터를 받아 DataFrame으로 만들어낼 수 있다.
  - `spark.read.text` `spark.read.csv` `spark.read.json`
- 데이터프레임을 마치 RDBMS의 테이블처럼 사용하기 위해서는 `createOrReplaceTempView()`함수를 이용해서 임시 뷰(temporary view)를 만들어줘야 한다.
### Spark DataFrame의 강점
- Spark DataFrame 사용 시 프로그래밍 문법 적용이 가능하다는 아주 큰 장점이 있다.
- Spark Streaming의 경우 Spark DataFrame 밖에 사용 안되는 등, 다른 Spark 모듈과의 이식성이 뛰어나다.
- 파티셔닝과 셔플링 등 최적화가 필요한 부분을 자동으로 수행해주기 때문에, 점차 RDD보다 Spark SQL을 지원하는 추세이다.

## SparkSession
- Spark Core에 SparkContext가 있다면, Spark SQL에는 SparkSession이 있다.
- 데이터가 정형화되었다면, 컬럼이 필요한 차례다. 각각의 데이터가 무엇인지 정의, 즉 스키마를 정의하는 역할을 수행한다.



# Spark SQL 실습
## 데이터프레임 생성
- SparkSession을 만들어준다.
  - `from pyspark.sql import SparkSession`
  - `spark = SparkSession.builder.master('local').appName('appName').getOrCreate()`
- 데이터프레임을 생성한다.
  - `.createDataFrame()`
  - 즉시 실행(action)되는 pandas DF와 달리 spark DF는 변환(transformation)만 되어있는 RDD 상태이다.
- `.dtypes()`으로 스키마의 정보 확인이 가능하다.
- 실행시켜 내용을 확인할 때 `.show()`을 사용한다.
  - action 함수이며, `.collect()`와 동일한 역할을 수행한다.
 
## Spark SQL 사용
- DataFrame에 SQL을 사용할 수 있는 view를 만들어 준다.
  - `.createOrReplaceTempView('view_name')`
  - Spark는 Lazy Evaluation이라 action 함수를 이전까지는 실행되지 않고 있는 상태이기 때문에, TempView(임시 뷰)를 만들어서 메모리에 두고 사용할 수 있도록 한다.
  - 쿼리를 적용하고 싶은 데이터프레임의 임시 뷰를 만들어 이용한다.
    1) 스파크 세션에서 `.createDataFrame()`으로 데이터를 생성한다.
       - `movies_sdf = spark.createDataFrame(data=movies, schema=movie_schema)`
       - Transformation 상태
    2) 해당 데이터프레임의 임시 뷰를 생성한다.
       - `movie_sdf.createOrReplaceTempView('movies')`
       - Transformation 상태 
    3) 쿼리를 입력한다. 
       - ex `query="""SELECT name FROM movies"""`
       - SQL과 동일하게 쿼리를 작성하여 결과를 출력할 수 있다.
    4) `spark.sql(query)`를 이용하여, 생성된 쿼리를 임시 DF에 적용한다.
       - ex `result = spark.sql(query)` => Transformation
            `result.show()`             => Action


## Join 구현
1. 자료형의 데이터 타입을 불러온다.
  - `from pyspark.sql.types import StringType, FloatType, IntegerType`
  - 내부 구조는 컬럼으로 잡는다. (자료형의 데이터 타입)
2. 구조를 만들기 위한 타입 불러온다.
  - `from pyspark.sql.types import StructField, StructType`
  - 전체적인 구조를 `StructType`으로 잡는다.
3. `JOIN`을 수행할 외부 데이터를 불러온다. (ex. 기존데이터: data, 외부데이터: join_data)
4. 외부에서 데이터를 받을 때는 `StructType`과 `StructField` 설정이 반드시 필요하다.
```py
join_schema = StructType([
    StructField('one', IntegerType(), True),
    StructField('two', FloatType(), True),
    StructField('three', StringType(), True)
])
```
5. 외부데이터에 대한 데이터프레임을 생성한다.
   - `join_df = spark.createDataFrame(data=join_data, schema=join_schema)`
6. 해당 데이터프레임의 임시 뷰를 생성한다. (ex. join_tempview)
   - `.createOrReplaceTempView(join_df)`
7. `JOIN`을 포함한 쿼리를 입력한다.
   - 기존데이터 임시뷰를 left(기준데이터), 외부데이터 임시뷰를 right로 가정한다.
```py
query = """ 
SELECT *
FROM data_tempview
JOIN join_tempview ON data_tempview.one = join_tempview.one
ORDER BY data_tempview.one ASC
"""
spark.sql(query).show()
```