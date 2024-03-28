# Spark SQL
- 스파크를 기반으로 만들어진 하나의 패키지이다.
- 크게 3개의 주요 API가 존재한다.
  - SQL
  - DataFrame
  - DataSets
## Spark SQL이 아닌 Spark DataFrame을 쓰는 이유
- Spark DataFrame 사용 시 프로그래밍 문법 적용이 가능하다는 아주 큰 장점이 있다.
- 두 가지 모두 사용법을 알고 있어야 하지만, DataFrame 위주로 알고 있으면 유용하다.
  - Spark Streaming의 경우 Spark DataFrame 밖에 사용 안되기 때문


# Spark DataFrame VS Pandas DataFrame
## (1) Spark DataFrame 생성
- 원본 데이터를 불러온다. (ex. kaggle의 titanic.csv 데이터셋)
- `.inferschema()`로 불러온 데이터의 컬럼 타입(스키마)을 추정한다.
  - 기본적으로 csv 파일은 전부 문자열이기 때문에, 최대한 데이터들의 타입을 추론하여 나타내주는 역할을 한다.
  - `titanic_sdf = spark.read.csv(filepath, inferSchema=True, header=True)`
- spark DF를 출력하면 DataFrame의 정보가 나온다. (Transformation)
  - 아직 Action 전이기 때문에 데이터프레임이 만들어지지 않은 RDD 상태이다.
- `.toPandas()`를 사용하면 Spark DF에서 Pandas DF로 변환할 수 있다.
  - 스파크에서 판다스로 데이터의 변환을 수행하기 위해서는 실제로 데이터가 메모리 상에 올라와야 하기 때문에 `.toPandas()`는 Action 함수이다.
  - 분산 처리 시스템이 더 이상 필요없을 때, spark에서 pandas로 변환한다.
  - spark가 훨씬 대용량의 데이터를 다룰 수 있긴 하지만, pandas가 훨씬 빠르기 때문에 분산 처리 시스템이 필요없다면 굳이 spark DF를 사용할 필요가 없기 때문이다.
- `.select()`를 이용하여 원하는 컬럼을 추출할 수 있다.
  - `titanic_sdf.select(['Name', 'Fare]).show()`
- `.shape()`이 동작하지 않기 때문에 행과 열의 개수를 따로 확인해야 한다.
  - `titanic_sdf.count(), len(titanic_sdf.columns)`
- example Q. 문자열 형식의 데이터가 아닌 컬럼에 대해서만 `.describe()` 수행해본다면?
```py
num_cols = [column_name for column_name, dtype in titanic_sdf.dtypes if dtype != 'string']
titanic_sdf.select(num_cols).describe().show()
```

## (1) Pandas DataFrame 생성
- `import pandas as pd`
- pandas는 별도로 스키마 추정할 필요없이 데이터만 불러온다. (자동으로 추정됨)
  - `titanic_pdf = pd.read_csv(filepath)`
  - spark에서 `.head()`는 Transformation 함수이다. (RDD만 출력)
- pandas DF를 출력하면 DataFrame이 나온다. (Action)
- `.shape()`을 이용해서 행과 열의 개수를 확인한다.

## (2) Spark DataFrame의 조회 - select()
- DataFrame 조회 후 추출하지 않고, 변환(Transformation) 작업이 수행된다.
- `.select()`로 조회할 때에도 컬럼 속성으로 조회한다.
- 함수의 인자로 개별 컬럼명을 문자열 형태 또는 DataFrame의 컬럼 속성을 지정한다.
- `.select()`를 이용해서 개별 컬럼명 혹은 컬럼 속성
  - `titanic_sdf.select('Name').show()`
  - `titanic_sdf.select('Name', 'Age').show()`
  - 위 구문을 SQL 쿼리로 표현하면 `SELECT Name FROM titanic_sdf;`
- 리스트를 활용하여 select 할 수 있다.
  - 2.xx의 구버전은 리스트를 이용한 조회가 불가능했기 때문에 `*`를 이용하여 `[]`를 unpacking 해야만 했다.
    - `select_columns = ['Name', 'Age', 'Pclass', 'Fare']`
    - `titanic_sdf.select(*select_columns).show()`
  - 3.xx의 신버전은 리스트를 이용한 조회가 가능하다.
    - `titanic_sdf.select(select_columns).show()`
## (2) Pandas DataFrame의 조회 - []
- 즉시 DataFrame을 조회해서 추출 작업이 실행되는 Action 함수이다.
- `[]`을 이용해서 컬럼 값을 가져온다.
  - `titanic_pdf['Name']`
  - `titanic_pdf[['Name', 'Age']]`


## (3) Spark DataFrame의 정렬 - orderBy(), sort()
- `import pyspark.sql.functions as F` 불러오기
- `orderBy()`, `sort()`를 사용한다.
    ```py
    # 정렬 기준이 1개
    titanic_sdf.orderBy(
        F.col('Pclass'),
        ascending=False
    ).show()

    # 정렬 기준이 2개
    #   orderBy() 예시
    #   대문자가 소문자보다 값이 작아서 오름차순 정렬 시 앞단에 위치한다.
    titanic_sdf.orderBy(
        [F.col('Pclass'), F.col('Age')],
        ascending=[False, True]
    ).show()

    #   sort() 예시
    titanic_sdf.sort(
        F.col('Pclass').asc(),
        F.col('Age').desc()
    ).show()
    ```


## (3) Pandas DataFrame의 정렬 - sort_values()
- `sort()`, `sort_values()`를 사용한다.
    ```py
    titanic_pdf.sort_values(by='Name', ascending=True)
    titanic_pdf.sort_values(
        by=['Pclass', 'Age'], 
        ascending=[True, False]
        )
    ```

## (4) Spark DataFrame의 집계(Aggregation) - select()~F.집계함수
- 집계와 관련된 메소드는 거의 대부분 `pyspark.sql.functions` 패키지의 함수를 활용한다.
    ```py
    titanic_sdf_agg = titanic_sdf.select(
        F.max(F.col('Age')),
        F.min(F.col('Age')),
        F.sum(F.col('Age'))
    ).show()
    ```
- 집계 메소드 중 `.count()`만 예외적으로 따로 존재한다.
  - 다만, `.count()`를 이용할 경우 spark df에서는 전체 행의 개수만 집계 가능하고 pandas df에서는 컬럼별 행의 개수 집계가 가능하다.
 
## (4) Pandas DataFrame의 집계(Aggregation) - 집계함수
- 특정 컬럼에 대한 집계가 가능하다.
  - `titanic_pdf['Age'].mean()`
- 전체 컬럼별 집계가 가능하다.
  - `titanic_pdf.mean()`
  - `titanic_pdf.count()`
- `.agg()` 함수를 이용하여 특정 컬럼별 서로 다른 기준을 적용한 집계가 가능하다.
    ```py
    agg_format = {
        'Age': 'max',
        'SibSp': 'sum',
        'Fare': 'mean'
    }
    titanic_pdf_groupby.agg(agg_format)
    ```


## (5) Spark DataFrame의 집계2 - groupBy()
- 컬럼 속성과 함께 `.groupBy()`를 사용한다.
- `.groupby()`에 집계 함수를 함께 적용할 수 있다.
    ```py
    titanic_sdf.groupBy(
        F.col('Pclass')
    ).count().show()

    titanic_sdf.groupBy(
        F.col('Pclass')
    ).max().show()


    # 두 개 이상의 집계 기준 적용
    titanic_sdf.groupBy(
        [F.col('Pclass'), F.col('Embarked')]
    ).max().show()

    # function package의 집계 함수 적용
    titanic_sdf.groupby('Pclass').agg(
        F.max(F.col('Age')).alias('max_age'),
        F.min('Age').alias('min_age'),
        F.sum(F.col('Age')).alias('sum_age'),
        F.avg(F.col('Age')).alias('avg_age')
    ).show()
    ```
- `.groupby()`에서 튀어나온 집계 함수에는 컬럼 속성을 사용할 수 없다.
- function package에서 나와 `F`와 함께 사용되는 집계 함수에는 컬럼 속성을 사용할 수 있다.

## (5) Pandas DataFrame의 집계2 - groupby()
- `.groupby()`를 이용한다.
  - `titanic_pdf.groupby(by='Pclass')`
  - `titanic_pdf.groupby.count()`

## (6) Spark DataFrame의 데이터 조작 - withColumn()
- spark dataframe에서 데이터 조작을 위해 copy를 하는 경우, `select('*')`를 사용한다.
  - RDD에서 전체 조회를 하면 그 자체가 데이터의 변형을 의미하므로 RDD 특성상 새로운 RDD가 생성되는 원리를 이용하는 것!
  - `titanic_sdf_copy = titanic_sdf.select('*')`
- 데이터가 변형되기 때문에 Transformation 과정이다.
- `withColumn()` 메소드를 이용하여 기존의 컬럼 값을 수정, 타입 변경, 신규 컬럼 추가 등을 수행한다.
  - 신규 컬럼을 추가하는 경우에는 `select()`도 사용 가능하다.
    - 생성한다는 개념이므로, 마찬가지로 앞단에 전체 조회(*)를 넣어준다.
    - 아래는 `전체 컬럼 + 신규 컬럼(별칭 설정)` 데이터프레임 예시이다.
    - `titanic_sdf_copy.select('*', F.col('Embarked').alias('E')).show()`
    - 아래는 Cabin의 첫 글자만 추출해서 신규 컬럼을 추가하는 예시이다. (SQL의 `SUBSTR()` 역할) 파이썬과 SQL의 인덱스 기준이 다름에 유의한다.
    - `titanic_sdf_copy = titanic_sdf_copy.select('*', F.substring(F.col('Cabin'), 0, 1).alias('CabinSection'))`
- `withColumn('신규 또는 수정되는 컬럼명', '신규 또는 수정되는 값')`
  - '신규 또는 수정되는 컬럼명'이 신규 컬럼인 경우 **문자열**로 지정한다.
  - '신규 또는 수정되는 컬럼명'이 기존에 있는 컬럼을 수정하는 경우 컬럼 속성 `F.col()`을 이용한다.
  - 컬럼명은 문자열, 값은 `F.col`을 이용한다.
- 데이터를 삽입하는 경우는 아래와 같다.
    ```py
    titanic_sdf_copy = titanic_sdf.select('*')
    titanic_sdf_copy = titanic_sdf_copy.withColumn(
                    'Extra_Fare', F.col('Fare') * 10
                    ) 
    ```
- 데이터를 수정하는 경우는 아래와 같이 데이터를 삽입하는 문법과 동일하다.
    ```py
    titanic_sdf_copy = titanic_sdf_copy.withColumn(
                    'Extra_Fare', F.col('Fare') + 10
                    ) 
    ```
- 데이터를 변환하는 경우는 아래와 같이 `cast()`를 사용한다.
    ```py
    titanic_sdf_copy = titanic_sdf_copy.withColumn(
                    'Extra_Fare', F.col('Extra_Fare').cast('integer')
                    ) 
    ```
- 신규 컬럼을 추가하는 경우는 아래와 같다.
    ```py
    # F.split(컬럼명, 나눌 문자)
    # 개행 문자는 `\` 사용, 개행 문자를 넣지 않으려면 대괄호 `{}` 적용
    titanic_sdf_copy.withColumn(
        "first_name", F.split(F.col("Name"), ",").getItem(0))\
                    .withColumn(
        "last_name", F.split(F.col("Name"), ",").getItem(1)
        ).show(5)
    ```


## (6) Pandas DataFrame의 데이터 조작 - []
- 대괄호를 사용하여 데이터의 삽입, 수정을 손쉽게 수행 가능하다.
- 데이터 삽입하는 경우이다.
  - `titanic_pdf_copy['Extra_Fare'] = titanic_pdf_copy['Fare'] * 10`
- 데이터 수정하는 경우이다.
- `titanic_pdf_copy['Extra_Fare'] = titanic_pdf_copy['Fare'] + 20`


## (7) Spark DataFrame의 literal - F.lit()
- 자료 구조가 아닌 어떠한 값, 즉 상수(Scala) 값을 의미한다.
- 프로그래밍 언어에서 코드에 등장하는 직접적인 값을 의미한다.
- 예를 들어, `a = 10`에서 `a`는 변수, `10`은 literal(상수)를 의미한다.
- spark에서는 브로드캐스트가 불가능하므로 데이터 조작을 위한 `withColumn()` 함수 적용 후 `F.lit()` 함수를 호출하여 스칼라 값을 강제로 넣어줘야 한다.
  - `titanic_sdf_copy = titanic_sdf_copy.withColumn('Extra_Fare', F.lit(10))`

## (7) Pandas DataFrame의 literal
- pandas df의 브로드캐스트 덕분에 아래 과정이 가능하다.
  - `titanic_pdf_copy['Extra_Fare'] = 10`





# 연습: SQL 구문을 Spark SQL 구문으로
- 기존 SQL 구문의 실행 순서와 동일하게 Spark SQL 구문을 왼쪽에서 오른쪽으로 작성해야 한다.
  - SQL은 FROM 절부터 실행되기 때문에 Spark SQL에서는 FROM 절에 해당하는 구문이 가장 먼저 등장한다.
- 문제 1번 (order by)
```SQL
SELECT Pclass, Name
FROM (SELECT *
      FROM titanic_sdf
      ORDER BY Pclass ASC, Name DESC)
```
```py
titanic_sdf.orderBy(
    [F.col('Pclass'), F.col('Name')],
    ascending = [True, False]
).select(
    F.col('Pclass'), F.col('Name')
).show()
```
- 문제 2번 (sort)
```SQL
SELECT Pclass, Name
FROM titanic_sdf
ORDER BY Pclass ASC, Name DESC
```
```py
titanic_sdf.select(
    F.col('Pclass'), F.col('Name')
).sort(
    F.col('Pclass').asc(),
    F.col('Name').desc()
).show()
```


# 컬럼 속성 이해하기
## 컬럼 속성이란?
- `titanic_sdf['Name']`과 같은 형식을 컬럼 속성이라고 한다.
- 컬럼 속성으로 slect 절에 있는 컬럼과 같은 역할을 수행할 수 있다.
  - `titanic_sdf['Fare']*100`
  - `titanic_sdf.Fare*100`
- select 절에 넣어서 적용할 수 있다.
```py
titanic_sdf.select(
    titanic_sdf['Fare'],
    titanic_sdf['Fare']*100
).show()
```
- 컬럼 속성을 만드는 함수로는 `F.col()`이 있다.
- `F` 즉 `pyspark.sql.functions` 패키지와 함께 나오는 커맨드는 함수가 많다.

## Spark `pyspark.sql.functions` 패키지의 `.col()`
- `import pyspark.sql.functions as F`로 불러와서 `F.col('column_name')` 형태로 사용된다.
- Spark SQL에서 사용되는 모든 연산이 들어있는 `pyspark.sql.functions` 패키지에서 불러온다.
- spark DF에서 컬럼을 다룰 때 가장 많이 사용하는 방식이다.
  - `titanic_sdf['Fare']*100`처럼 브라켓을 사용하거나 `titanic_sdf.Fare*100`처럼 온점을 사용하는 방식보다 `.col()` 함수를 사용하는게 일반적으로 더 좋은 방식이다.
  - `titanic_sdf.select(F.col('Name'), F.col('Fare')).show()`
- SQL 구문을 Spark SQL 구문으로 표현하는 예시
  - `F.upper('Name')`이 아닌 컬럼 속성으로 표현한 이유는 스타일의 차이이다.
    ```SQL
    SELECT Name, UPPER(Name) 
        AS Cap_Name
    FROM titanic_sdf
    ```
    ```py
    titanic_sdf.select(
        F.col('Name'),
        F.upper(F.col('Name')).alias('Cap_Name')
    ).show()
    ```

## Spark DF의 `.filter()`
- SQL의 WHERE과 아주 흡사하다.
  - `.filter()`의 alias 버전이 `.where()` 메소드이다.
- 특정 조건을 만족하는 값을 DataFrame으로 반환한다.
- `.filter()` 내의 조건 컬럼을 컬럼 속성으로 지정한다.
- 복합 조건인 and는 `&`, OR은 `|`, 개별 조건은 `()`로 감싸줘야 한다.
    ```py
    titanic_sdf.filter(
        (F.col('Embarked')=='S') & (F.col('Pclass')==1)
    ).show()

    titanic_sdf.filter(
        (F.col('Embarked')=='S') | (F.col('Pclass')==2)
    ).show()
    ```
- `.filter()`와 함께 **연산자** `LIKE`을 사용한 예시는 아래와 같다.
    ```py
    F.col('Name').like('%Miss%')

    titanic_sdf.filter(
        F.col('Name').like('%Miss%')
    ).show()

    # 아래와 같이 입력해도 동일하게 출력은 된다.
    titanic_sdf.filter(
        "Name like '%Miss%'"
    ).show()
    ```
- SQL 구문을 Spark SQL 구문으로 표현하는 예시
    ```SQL
    SELECT UPPER(Name)
    FROM titanic_sdf
    WHERE LOWER(Name) LIKE 'h%'
    ```
    ```py
    # 필터링으로 데이터를 줄인 다음 셀렉트로 조회하는 순서
    titanic_sdf.filter(
        F.lower('Name').like('h%')
    ).select(
        F.upper('Name')
    ).show()

    # 아래와 같이 입력해도 동일하게 출력된다. (스타일 차이!)
    titanic_sdf.filter(
        F.lower(F.col('Name')).like('h%')
    ).select(
        F.upper(F.col('Name'))
    ).show()
    ```

# 컬럼 이름 변경하기
- `wihtColumnRenamed()` 메소드를 이용하여 컬럼 이름을 변경한다.
  - `wihtColumnRenamed('수정 전 컬럼명', '수정 후 컬럼명')`
  - 이 경우 없는 컬럼의 이름을 넣더라도 오류가 나지 않기 때문에 주의해야 한다.
- `select()`에서 alias로 컬럼 이름을 변경한다.