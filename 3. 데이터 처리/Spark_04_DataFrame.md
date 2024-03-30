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

<hr>

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

<hr>

# 컬럼 이름 변경하기
- `wihtColumnRenamed()` 메소드를 이용하여 컬럼 이름을 변경한다.
  - `wihtColumnRenamed('수정 전 컬럼명', '수정 후 컬럼명')`
  - 이 경우 없는 컬럼의 이름을 넣더라도 오류가 나지 않기 때문에 주의해야 한다.
- `select()`에서 alias로 컬럼 이름을 변경한다.

<hr>

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


## (8) Spark DataFrame의 컬럼과 로우 삭제
- 컬럼을 삭제할 때는 `drop()` 메소드를 사용한다.
  - 다만 여러 개의 컬럼 삭제 시 리스트 사용이 불가능하다.
  - `drop()`은 제거가 아닌 조회의 개념이다.
- 로우(레코드)를 삭제는 원칙적으로 불가능하며, 대신 `filter()`를 이용한다.
  - 로우의 삭제는 곧 데이터의 삭제이며, RDD의 불변성(`immutable`)이라는 특성 상 spark DF에는 삭제라는 개념이 없다.
  - 컬럼의 삭제라는 의미도, 사실은 필요한 부분을 추출하고 필요없는 부분은 선택하지 않은 것일 뿐이다.
- 컬럼 삭제하는 경우
  - `titanic_sdf.drop(F.col('Name'))`
  - 컬럼 속성으로 지정해준다.
- 로우 삭제하는 경우
  - `titanic_sdf.filter(F.col('Pclass') != 1)`
  - 삭제된 게 아니라 Pclass가 1이 아닌 데이터만 가져온 것이다.
  - 행을 제거할 방법이 없기 때문에 대신 `filter()`나 `where()`로 걸러준다.
### (8-1) Spark DataFrame의 dropna()
- 로우(레코드)에 `Null` 또는 `Nan` 값이 하나라도 있다면 삭제 후 DF를 반환한다.
  - `titanic_sdf.dropna()`
- 특정 컬럼에 있는 `Nan` 값을 제거할 때는 `subset=[]`를 이용한다.
  - `titanic_sdf.dropna(subset=['Age', 'Embarked'])`
- `dropna()`가 아닌 `filter()`와 `isNotNull()`을 이용해서도 특정 컬럼의 `Null` 값을 제거할 수 있다.
  ```py
  titanic_sdf.filter(
    F.col('Age').isNotNull() & F.col('Embarked').isNotNull()
  )
  ```

## (8) Pandas DataFrame의 컬럼과 로우 삭제 - drop()
- 컬럼(열)과 로우(행) 모두 `drop()`으로 삭제할 수 있다.
- spark DF는 `immutable`하지만, pandas DF는 파이썬 기반이기 때문에 `mutable`한 것에서 차이가 있다.
- `titanic_pdf.drop('Name', axis=1, inplace=True)`
  - `inplace`는 데이터프레임을 변환하는 옵션이다. 
  - True로 설정하면 spark DF처럼 변형 적용, 즉 별도의 특정 변수에 저장하지 않아도 기존 DF에 변경된 설정이 덮어쓰인다.


# [참고] Spark와 Pandas의 None, Nan, Null
- 크게 중요한 부분은 아니다.
- `isNull()`라는 메소드와 `isNan`이라는 함수를 사용한다.
  - 메소드이기 때문에 `F.col('Age').isNull()`
  - 함수이기 때문에 `F.isnan(F.col('Age'))`
- `None`: 아무것도 없다는 것을 의미하기 위한 (NoneType 클래스의) 객체
  - 객체가 존재하지 않는다는 뜻
- `Nan`: Not a number의 약어로, 값이 없다는 것을 의미하기 위한 파이썬의 값
  - 값이 존재하지 않는다는 뜻
- `Null`: 값이 없다는 것을 의미하기 위한 자바의 값
- 자바에서는 `None`과 `Nan`을 가리지 않고 모두 통틀어서 `Null`이라고 부른다.
  - spark는 pyspark를 설치하여 사용하고, spark DF는 spark SQL의 API이다.
  - `pyspark`를 설치하여, 파이썬 파일로 드라이버 프로그램을 실행하더라도 프로그램 안에 파이썬 코드를 자바로 바꿔주는 라이브러리가 작동되기 때문에 결국 실제로는 자바로 바뀐 영역이 spark master에 제출되며 결과 또한 자바의 결과가 출력되기 때문에 `Null`값이 표현된다.
- 데이터 분석에서는 파이썬의 넘파이를 많이 사용하기 때문에 `Nan` 값을 더 많이 보게 된다.
- 객체가 존재하지 않는 것과 값이 존재하지 않는 것의 차이점
  - `a = 10`은 변수 a 안에 10이라는 값(value)으로 이루어져 있다.
    - a 안에 참조할 값이 없다면, 값 자리에 `Nan` 값을 넣는다.
  - `b = B()`는 변수 b는 메모리 상의 B라는 객체를 의미한다.
    - b 안에 참조할 객체가 없다면 객체 자리에 `None` 값을 넣는다.
    - 객체는 접근연산자인 `.`으로 호출이 가능하다.
- 원본 데이터의 Cabin이라는 문자열 타입의 컬럼같은 경우에는 spark에서는 `None`, pandas에서는 `Null` 값이 출력된다.
  - 문자열은 string 클래스의 객체이기 때문에 `None` 값이 출력되었다.
  - 즉 자바에서는 모두 `Null`로 표시되고, 파이썬에서는 객체와 값인지에 따라 `None`과 `Nan`으로 구분되어 표시된다.

## Pandas와 Spark에서의 Null 값 처리
### Spark DataFrame에서 Null 컬럼명 찾기
- `titanic_sdf.columns`로 spark DF의 컬럼명 리스트를 출력한다.
- List Comprehension을 이용하여 모든 컬럼의 `Null` 값을 검사한다.
  - `null_columns = [F.col(column_nae).isNull() for column_name in titanic_sdf.columns]`
  - `titanic_sdf.select(null_columns).show(10)`
### Pandas DataFrame에서 Null 개수 찾기
- `titanic_pdf.isna().sum()`
- 누락값을 카운트하는 것과 같다.
### Spark DataFrame에서 Null 개수 찾기
- spark에서는 pandas와 같이 간단하게 누락값을 세는 기능이 없다.
- `when()`과 `count()`를 사용하여 CASE WHEN THEN 문과 같은 형태를 만들어준다. (안에서 바깥쪽으로 코드를 작성)
  1) `F.col('Age').isNull()`
     1) Null인 컬럼 속성
  2) `F.when(F.col('Age').isNull(), 'Age')`
     1) 출력값: `Column<'CASE WHEN (Age IS NULL) THEN Age END'>`
     2) Age is Null이 조건이고, 조건 충족시 Age 컬럼을 반환한다.
  3) `F.count(F.when(F.col('Age').isNull(), 'Age'))` 조건에 대한 개수
  4) 상기 코드를 원본 데이터 전체 컬럼에 대한 List Comprehension으로 풀어낸다.
     1) `null_count_conditions = [ F.count(F.when(F.col(c).isNull(), c)).alias(c) for c in titanic_sdf.columns ]`
  5) 위 구문을 select에 넣으면 집계가 완료된다.
     1) `titanic_sdf.select(null_count_conditions).show()`
### Pandas DataFrame에서 Null 값 처리하기
- 특정 컬럼의 `Null` 값을 해당 컬럼의 평균으로 채워준다.
- `titanic_pdf['Age'] = titanic_pdf['Age'].fillna(titanic_pdf['Age'].mean())`
### Spark DataFrame에서 Null 값 처리하기
- 누락값에 숫자 `999`를 채워넣는 경우, 숫자형 컬럼에는 값이 채워지고 문자형 컬럼에는 값이 채워지지 않고 `Null` 값으로 남는다.
  - `titanic_sdf.fillna(value=999).select('Age', 'Cabin').show(10)`
- 반대로 누락값에 문자 `없음`을 채워넣는 경우, 문자형 컬럼에는 값이 채워지고 숫자형 컬럼에는 값이 채워지지 않고 `Null` 값으로 남는다.
  - `titanic_sdf.fillna(value='없음').select('Age', 'Cabin').show(10)`
- 따라서 타입별로 누락값에 넣을 내용을 각기 넣어줘야 한다.
- `fillna()`의 인자로 딕셔너리를 입력하여 여러 컬럼별로 결측치 값을 입력할 수도 있다.
  ```py
  titanic_sdf.fillna({'Age': avg_agg_value,
                      'Cabin': 'C000',
                      'Embarked': 'S'})
  ```
- 특정 컬럼의 `Null` 값을 해당 컬럼의 평균으로 채우는 경우
  - `mean_age = titanic_sdf.select(F.avg(F.col('Age')))`라는 Age 컬럼의 평균값으로는 결측치 값을 처리할 수 없다.
  - 데이터프레임에는 `int`, `str`, `bool`과 같은 값이 들어가야 한다는 에러가 나타난다.
  - `mean_age`는 select로 transformation까지만 진행된 실제 값이 없는 데이터프레임이기 때문에 값으로 사용이 불가능하기 때문이다.
  - collect로 값을 추출한다. `mean_avg_row = mean_age.collect()`
  - 첫번째 값을 지정해온다. `row = mean_avg_row[0]`
  - 다시 첫번째 컬럼을 가져온다. `mean_age_value = row[0]` == `mean_age_value = mean_avg_row[0][0]`
  - fillna를 이용해서 결측치 값을 처리한다.
    - `titanic_sdf.fillna(value=mean_age_value).show()`
  - `subset=[]` 옵션을 이용하면 특정 컬럼에만 값을 채울 수 있다.
    - `titanic_sdf.fillna(value=mean_age_value, subset=['Age']).select('Age').show()`

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
- 문제 3번
```sql
SELECT *
FROM titanic_sdf
WHERE age IS NOT NULL
  AND Embarked IS NOT NULL
```
```PY
titanic_sdf.dropna(subset=['Age', 'Embarked'])
```

