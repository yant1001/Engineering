# Spark SQL
- 스파크를 기반으로 만들어진 하나의 패키지이다.
- 크게 3개의 주요 API가 존재한다.
  - SQL
  - DataFrame
  - DataSets


# Spark DataFrame VS Pandas DataFrame
## Spark DataFrame 생성
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
- 예시 문제
  - Q. 문자열 형식의 데이터가 아닌 컬럼에 대해서만 `.describe()` 수행
```py
num_cols = [column_name for column_name, dtype in titanic_sdf.dtypes if dtype != 'string']
titanic_sdf.select(num_cols).describe().show()
```

## Pandas DataFrame 생성
- `import pandas as pd`
- pandas는 별도로 스키마 추정할 필요없이 데이터만 불러온다. (자동으로 추정됨)
  - `titanic_pdf = pd.read_csv(filepath)`
  - spark에서 `.head()`는 Transformation 함수이다. (RDD만 출력)
- pandas DF를 출력하면 DataFrame이 나온다. (Action)
- `.shape()`을 이용해서 행과 열의 개수를 확인한다.
