# UDF
- User Defined Function (사용자 정의 함수)
- 사용자가 만든 함수를 Worker의 Task에서 사용 가능하도록 한다.
  - Driver Program은 Spark Session 상에서 만들어지고 그 위에서 작동한다.
  - 이는 Spark Master에 전달되어, Spark Master가 Spark Worker에게 분배해준다.
  - 때때로 개발자가 직접 Driver Program에서 새로 기능을 정의하는 경우가 있는데, 이렇게 개발자가 만든 함수의 경우 Spark Session이랑 관계가 없는 함수이기 때문에 D.P-M-W로 전달되어 동작하지 않고 Client 환경에서만 동작하게 된다.
  - 개발자가 만든 함수를 Worker에 보낼 수 있도록 등록하는 것이 UDF(User Defined Function)이다.
- 분산 병렬 처리 환경에서 사용할 수 있는 함수, 즉 실제 데이터를 다루는 Worker에서 작동하는 함수를 만들 때 사용한다.
  - Worker에서 작동하는 함수는 전체 데이터를 다루는 함수이다.
- 리턴 타입을 지정하지 않으면 기본적으로 `string`을 반환한다.
- Spark DataFrame과 SQP 모두 사용 가능하며, 분산 처리를 지원하는 거의 모든 프로그램에는 UDF 기능이 있다.

## Spark SQL에서 사용할 UDF 생성
1. `from pyspark.sql.types import LongType, StringType`
2. SQL에서 사용할 함수를 생성한다. (UDF)
  - 반드시 리턴이 있어야 한다.
        ```py
        def squared(n):
            return n * n
        ```
3. `register()`을 이용하여 스파크 세션에서 사용자가 만든 함수를 등록한다.
  - 드라이버 프로그램을 보낼 수 있는 환경을 의미한다.
  - `register('worker에서 사용할 함수의 이름', 마스터(클라이언트)에서 정의된 함수의 이름, 리턴 타입)`
  - worker에서 사용할 함수의 이름과 마스터에서 정의된 함수의 이름은 동일하게 설정하는게 관례이다.
  - `spark.udf.register('squared', squared, LongType())`
4. UDF로 정의된 함수 `squared`는 SQL 환경에서 사용 가능, 즉 Worker에서 사용 가능한 함수가 된다.
```py
query = """
SELECT price, squared(price)
FROM transactions
"""
spark.sql(query).show()
```

## Spark DataFrame에서 사용할 UDF 생성
1. 파일을 불러온다.
   1. `filepath = 'file_address'`
   2. `spark.read.csv(filepath, inferSchema=True, header=True)`
2. 결측치를 처리한다.
   1. `import pyspark.sql.functions as F`
   2. `fillna()`를 이용하여 `Nan` 값 채워넣는다.
3. 필요한 함수를 정의한다. (UDF)
   1. `def`를 이용한다. (ex. `get_category()`)
   2. 반드시 return 값이 있어야 한다.
4. `udf()`를 이용하여 데이터프레임 API에서 UDF를 호출할 수 있도록 등록한다
   1. `udf_get_category = F.udf(lambda x: get_category(x), StringType())`
5. DataFrame 환경에서 사용 가능, 즉 Worker에서 사용 가능한 함수가 된다.
   1. `titanic_sdf_filled.withColumn('AgeCategory'. udf_get_category(F.col('Age'))).select('Age', 'AgeCategory').show()`