# 병렬처리와 분산처리
- 병렬분산처리라고 합쳐서 많이 부른다.
- Spark는 RDD에서 작업을 수행하고, 함수가 task의 역할을 한다.
  - `datas.map(lambda row: row.split(",")[2])`
  - `datas`는 RDD이고 `lambda ~ [2]`는 task에 해당한다.
  - 함수로 작업을 등록(아직 실행X)하는것
- 클라이언트가 함수를 마스터에게 전달
- 함수를 마스터가 파악해서 스케쥴링 후 적절한 워커한테 TASK로 등록
- 워커는 TASK에 맞춰 실행 
- RDD에서 TASK를 돌린다라고 생각하면 된다.

## Data-Parallel 작동 방식
- 데이터를 한 노드에서 처리하기 버거운 경우
1. 데이터를 여러개로 쪼갠다
2. 여러 데이터가 들어있는 여러 쓰레드가 각자 task를 수행한다
    1. cpu 1개당 작업 1개만 수행한다
        1. cpu에는 thred라는게 있다.
        2. cpu는 thred가 다루는 노예의 손 개수
        3. 한 노드가 고구마밭이라면, 데이터가 4개로 쪼개졌을 때 고구마밭이 4개가 된다 
        4. cpu가 손 한개로 캐는 것보다 나눠진 밭을 4개의 스레드로 동시에 캐는게 더 빠르다
    2. task는 작업 수행 도구 (즉  rdd는 고구마밭, task는 작업 수행도구)

- cpu 하나당 노드 하나를 배치하는게 가장 이상적이다. 
- 때문에 기본적으로는 cpu가 node하나를 맡아야 하지만, 너무 크니까 데이터를 쪼개서 스레드를 이용해서 작업을 나눠서 처리하고 각각의 결과물을 합친다 == 병렬처리 (한대의 컴퓨터 하나의 cpu에서 벌어지는 일)

## Distributed Data-Parallel 작동 방식
- 분산 병렬처리
- 네트워크를 통한 분산 환경에서의 병렬처리
1. 데이터를 여러개로 쪼개서 여러 노드로 보낸다 (분산시스템)
2. 여러 노드에서 각자 독립적으로 task를 수행한다
    1. 이 때 spark context에 작성된 프로그램이 각 워커에 전달된다
3. 각 노드에서 작업된 task 결과물을 합쳐준다
- 상황에 따라서 노드 하나가 크다면 다시 잘라서 스레드로 관리한다
- 노드간 통신에 신경써야할 게 늘어난다
- 하지만 이런 모든 밸런싱도 스파크가 다 알아서 해준다

## 데이터 병렬 모델 추상화
- 데이터 추상화 == 대충 아는 것
- 스파크는 분산된 환경에서의 데이터 병렬 모델을 구현해 추상화시켜준다.
- 이걸 구현해놓은 것이 바로 RDD(Resilient Distributed Datasets)
- RDD라는 개념 자체가 분산된 환경에서도 병렬처리를 할 수 있도록 알아서 처리한다(데이터가 추상화되었기 때문에 가능)
- RDD를 기반으로 만든 sql이나 데이터프레임도 동일하다
- 노드간 통신 등 통신 속도를 신경써야 하긴 하다! 하지만 스파크가 알아서 해주긴함


# 분산처리와 레이턴시
## 분산처리 문제
- 부분 실패 문제
  - 하나가 망가졌다고 전체가 망가지면 안된다
- 속도
  - ⭐**많은 네트워크 통신을 필요로 하는 작업은 속도가 저하된다**
  - 느려지는 요인: 횟수와 네트워크 통신 데이터 크기
- 어떤 작업을 어떻게 코드로 나타내는가에 따라 달라지는 속도
  1. 횟수는 어쩔 수 없다
  2. 데이터의 크기는 프로그래밍적으로 제어할 수 있다
     1. 데이터의 크기 == 네트워크를 통과하는 건수
     2. HDFS에서의 combine과정: 데이터의 속성은 변하지 않으면서 데이터의 절대적인 건수가 줄어들도록 한다
     3. 불러올 데이터를 `filter()` 함수로 줄여준 다음,  데이터를 불러와서 `reduceByKey()` 합쳐준다
        1. 마치 집계과정. 퍼포먼스가 제일 뛰어나다
        2. 초보자의 입장에선 뭘쓰든 상관없다고 생각할 수 있지만 속도면에서 차이가 난다
        3. 무조건 필터링하고 집계를 하는게 속도가 빠르다. 마치 where절에서 데이터를 거르고 group by하는 순서로 sql 쿼리가 진행되는 것과 같다


# Key-Value RDD
- 데이터 집계의 핵심이 되는 RDD 구조
- key value 상을 갖기 때문에 pairs RDD라고도 한다
- 딕셔너리같지만 딕셔너리보단 튜플의 모양이다
  - (key, value) 쌍을 갖는다.
  - `pairs = rdd.map(lambda x: (x, 1))`
  - 리스트도 value가 될 수 있다.
  - `pairs = rdd.map(lambda x: (x, [1, 1]))`
  - map 변환 전에는 짜장면, 짬뽕으로만 적힌 rdd 형태이지만, map이라는 mapping 변환 작업을 거치면 (짜장면, 1)과 같은 key value rdd가 된다(리스트도 가능~짜장면, [1, 1])
- key를 기준으로 고차원적인 연산이 가능하다
- single value rdd는 단순 연산 수행
  - 텍스트 등장 단어수, 날짜수 세기
  - 그룹바이가 불가능하다
  - 그룹핑이 안되기 때문에 조건에 맞게 기준을 세울 수 없다
- key value rdd는 key에 따른 다양한 고차원적 집계 연산이 가능
  - 영화 장르별 평균 점수 계산, 날짜별 주문 개수, 날짜별 승객 수 등
  - 그룹바이가 가능하다
  - 조건에 맞는 기준을 세울 수 있다
## Key-Value RDD의 Reduction 연산
- reduction은 데이터를 줄여준다.
- reduction은 key 값을 기준으로 데이터를 묶어서 처리한다.
- 데이터베이스와 유사하기 때문에 비슷한 형태로 Join도 가능하다.
  - `join`, `rightOuterJoin`, `leftOuterJoin`, `subtractByKey` 등
- reduction 함수 (대부분 action 함수이다.)
  - `reduceByKey(<task>)` - key를 기준으로 task 처리
  - `groupByKey()` - key를 기준으로 value 묶어주기
  - `sortByKey()` - key를 기준으로 정렬
  - `keys()` - key 추출
  - `values()` - value 추출
- reduction 연산 예시
```py
pairs = rdd.map(lambda x: (x, 1))
count = pairs.reduceByKey(lambda a, b: a + b)

# rdd
[ 짜장면, 짜장면, 짬뽕 ]

# pairs
[ (짜장면, 1), (짜장면, 1), (짬뽕, 1), ]

# count
[ (짜장면, 2), (짬뽕, 1), ]
```


# RDD Transformations & Actions
## Spark operation
- `Transformations` + `Actions`
### Transformations
- 새로운 RDD 반환
- 지연 실행 (lazy execution)
- Narrow Transformations + Wide Transformations
  - Narrow Transformations
    - 일대일 변환을 의미한다.
    - 하나의 열을 조작하기 위해 다른 열이나 파티션의 데이터를 사용하지 않는다.
    - `filter()`, `map()`, `flatMap()`, `sample()`, `union()`
  - Wide Transformations
    - Shuffling 된다. (다른 파티션 혹은 다른 노드로 네트워크를 타고 데이터가 이동)
    - 결과 RDD의 파티션에 다른 파티션의 데이터가 들어갈 수 있다.
    - 데이터가 여러 파티션을 넘나들기 때문에 많은 통신과 리소스가 필요하게 된다.
    - 수행 시간이 많이 소요된다.
    - 느리다고 해서 무조건 안 좋은 것은 아니며 집계를 하기 위해 필연적으로 써야 하는 transformation일 수 있지만, 파티션의 변화 즉 shuffling이 최대한 일어나지 않도록 신경써서 사용해야 한다.
    - Shuffling을 줄이는 방식으로 코딩돼야 하기 때문에 대부분이 데이터 집계 함수, 즉 데이터의 개수가 줄어드는 함수이다.
    - 집계에는 필연적으로 파티션 이동이 필요하다.
    - `Intersection and join`, `distinct()`, `cartesian()`, `reduceByKey()`, `groupByKey()`
- 그외 `mapValues()`, `flatMapValues()`, `sortByKey()` 등

### Actions
- 연산 결과 출력 및 저장
- 즉시 실행 (eager execution)
- actions 함수를 실행하고 나면 파이썬 리스트가 되기 때문에 더이상 RDD 함수를 사용할 수 없다.
- 즉 action을 하고 난 후 다시 action을 적용할 수 없다.
- `collect()`, `count()`, `countByValue()`, `take()`, `top()`, `reduce()`, `fold()`, `foreach()`

## [번외] Lazy Execution의 유리함
- 메모리를 최대한 활용할 수 있다.
- 연산을 지연 시킴으로써 디스크와 네트워크의 연산을 최소화할 수 있다.
- 디스크에 계속 데이터를 저장하고 불러오는 반복적인 작업 방식은 속도를 저하시킨다.
- 메모리 내에서 task끼리 데이터를 교환하여 더 속도를 빠르게 한다. (인메모리 방식)
- transformations는 지연 실행되기 때문에 메모리에 저장해 둘 수 있다.
- `Cache()`, `Persist()`로 데이터를 메모리에 저장해두고 사용이 가능하다.


# Cluster Topology
- 스파크의 마스터-워커 구조
- 데이터는 여러곳에 분산되어 있다
- 같은 연산이어도 여러 노드에 걸쳐 실행 가능
- 분산된 위치 즉 데이터 노드에는 워커가 존재하며 마스터 명령을 수행
- **스파크 클러스터 구조**
  - Driver Program
    - 개발할 때 driver program을 중심으로 사용
    - 개발자, 사용자가 프로그램과 상호작용
    - 우리가 작성하는 건 드라이버 프로그램 (주피터노트북 스파크)
    - 모든 프로세스를 조작하며, 메인 프로세스를 수행
    - sparkcontext 생성, 이를 통해 RDD 생성
    - transformations, actions를 생성하고 worker에 전송
  - Cluster Manager
    - 수행되는 작업들에 대한 스케줄링 담당
    - 클러스터 전반에 대한 자원 관리
    - Yarn, Mesos 등
  - Worker Node
    - 실제 연산이 일어나는 노드
    - master가 조작한 task를 수행
    - executor, cache, task 존재
    - executor가 task를 실행하고, 데이터를 저장하고, 그 결과를 driver program에 전송
    - 연산을 하면서 필요한 저장공간을 제공하기 위해 cache를 가지고 있다, cache에 넣어서 빠르게 전송 가능
- **스파크 프로그램 실행 과정**
  - 마스터-워커 구조에서 크게 벗어나진 않는다
  - 실제 스파크 코드에서 나타나는 부분
  - 워커노드까지도 코드에 표시된다
  - 간단하게, sparkcontext가 executor들에게 수행할 task를 전송하고, 수행된 결과물을 다시 driver program에 보내게 된다.
  1. driver program이 sparkcontext를 이용해 spark application 생성
  2. spark context cluster manager에 연결
  3. cluster manager가 자원을 할당
  4. cluster manager가 worker node들의 executor들을 수집
  5. executor가 연산을 수행하고 데이터 저장



# Reduction
## Transformations
- Parallel Transformations
- 주로 데이터의 변형을 일으키는 작업들 (map, flatmap, filter 등)

## Action
- 액션의 대부분은 reduction이다
  - 파일 저장, `collect()` 등과 같이 reduction이 아닌 액션도 있다. 
- 액션을 통해 실제 물리적인 reduction 작업이 일어난다
- 그래서 액션할 때 시간이 더 걸리는 것
- 데이터 분석은 대부분 reduction 작업을 요구하는 것
- reduction이란? 근접하는 요소 즉 연관성 있는 것을 모아서 하나의 결과로 만드는 것

## 병렬 처리가 가능한 Reduction
- 병렬 처리 가능 여부에 따라 결과물이 달라진다
- task끼리 서로 독립적이며, 자체적으로 연산이 가능하다.
- 각각의 task가 각각의 역할을 수행하고 결과를 합쳐서 새로운 task를 수행한다
- 새롭게 만들어진 task는 각각의 역할 수행 후 다시 결과를 합쳐서 또 새로운 task 수행
- 각 단계별 task들은 서로간 간섭이 일어나지 않는다
- 다만 이런 병렬 처리 상황에서는 파티션이 중요하다
- task가 파티션마다 돌아간다

## 병렬 처리가 불가능한 Reduction
- 파티션이 다른 파티션의 결과값에 의존한다 
- task가 각각 진행되었지만, 파티션이 다른 파티션의 결과값에 의존하기 때문에 병렬로 실행할 수 없다 즉 직렬이다
- task의 종류가 다른 것이지, 직렬이고 병렬이라고 뭐가 좋고 나쁘고 하진 않다
- 다만 둘다 reduction 즉 데이터가 줄어든다
- 이왕이면 병렬처리가 빠르긴 하다
- 이렇게 직렬의 경우 분산 병렬 처리로 할 필요 없고, 컴퓨터 한대로 하는게 더 빠를 수 있다

## 대표적인 Reduction Actions
- `Reduce()`, `Fold()`, `GroupBy()`, `Aggregate()`
- transformation과 관계없이, 명령어 입력하자마자 바로 action된다 (그냥 바로 실행)



> 막간 메모
> 현상황 == aws 인스턴스에 스파크를 설치해서, 로컬에서 스파크를 사용하고 있는 상황 (노션 240327 참고)
> 서로 다른 데이터노드가 동일한 RDD일 수 있다. (RDD는 곧 데이터의 추상화이므로)
> 데이터노드에 있는 데이터를 하나의 RDD로 모아 놓은것
> RDD에 있는 파티션에서는 물리적으로 서로 다른 데이터노드에 있는 데이터들을 마치 하나의 파티션처럼 개념적으로 사용할 수도 있다.
> 파티션은 데이터의 개념적인 분할방법이다
> RDD는 추상화된 정보이기 때문에 물리적 상황과 상관없이 파티션을 꾸릴 수 있다
> 다만 RDD 상태에서 reduce를 하게 되면 데이터의 교환이 잦을 수 있다 즉 shuffling이 많이 일어난다
> 물리적으로는 떨어져있지만 개념적으로는 붙어있다
> task는 파티션에 하나씩 분배가 되는 것
> 그래서 과거에는 이러한 파티션들을 다 고려한 코딩을 했어야 했지만  지금은 스파크 최적화로 그러지 않아도됨
> 데이터노드는 워커노드는 같은 개념이다
> 데이터노드가 워커노드와 다르게 존재해도 상관없다 다만 그렇게 되면 네트워크를 사용하게 된다..! 그래서 그렇게 안하기 위해 데이터노드 안에 워커노드를 띄우는것이다 (한지붕 두가족)
> (인프라 엔지니어를 안하려고 하다면 몰라도 되는 지식이지만 사람일 모르는거니까~?)



# Key Value RDD Operation & Joins
## Key Value RDD의 Transformations & Actions
- Transformations
  - groupbykey
  - reducebykey
  - mapvalues
  - key
  - join, leftouterjoin, rightouterjoin
- Actions
  - countbykey



# Shuffling & Partitioning
## Shuffling
- 한 노드에서 다른 노드로 옮길 때 네트워크 연산이 발생하는 것
- 노드마다 데이터들이 섞여있는 상태에서, 종류별로 묶으려고 한다면 정리가 필요
- 여러 네트워크 연산을 일으키기 때문에 성능을 많이 저하시킨다
- 이때 종류별로 데이터가 모아지는, 섞여지는 과정을 shuffling이라고 한다
- 네트워크를 태우는 데이터의 개수를 줄이는게 중요하다
- RDD를 여러 개 사용하는 경우에 셔플이 일어난다
  - 결과로 나오는 RDD가 원본 RDD의 다른 요소 또는 다른 RDD를 참조할 때 발생
- 특정 데이터가 다양한 노드에 걸쳐 존재하는 경우에 셔플이 일어난다
- 안 좋은 예시
  - groupbykey 후 reducebykey를 수행한다면? groupbykey에 의해 먼저 셔플이 일어난다
  - groupbykey때문에 병목이 일어나서 느려진다 (이러한 상황이 안좋은 예시)
- 좋은 예시는
  - reduce 하기 전 각각의 파티션에서 먼저 reduce과정을 한번 거치게 하는 것(데이터를 줄여놓고 섞는다)
  - 즉 shuffle을 최소화하기 위해 reducing하고 섞어야 한다는 의미
## Partitioning
- 목적
  - 비슷한 것끼리 놔둬야 검색이 쉬워지니까! (정렬과는 다르다)
  - 데이터를 최대한 균일하게 퍼트리기 위해 쿼리가 같이 적용 가능한 데이터를 최대한 곁에 두기
  - key-value RDD일 때만 의미있다
- 특징
  - RDD는 쪼개져서 여러 파티션에 저장된다
  - 하나의 노드는 여러 개의 파티션을 가질 수 있다
  - 하나의 파티션은 하나의 노드(서버)에 존재한다
  - 파티션의 크기와 배치는 자유롭게 설정 가능하고 성능에 큰 영향을 미친다
    - 파티션이 잘 되어야 RDD가 잘됨. 비슷한것끼리 예쁘게 모으는 알고리즘 == hash partitioning, range partitioning
  - key-value RDD일 때만 의미있다
- 종류
  - hash partitioning, range partitioning
  - hash partitioning
    - 두 종류 중 디폴트 방식
    - 데이터를 여러 파티션에 균일하게 분배하는 방식이다 이걸 만들어주는게 해쉬 function
    - hash function ⇒ 어떤 정수 % 파티션 개수
    - 데이터의 성질에 맞게 파티션 개수를 결정하는 것도 능력이다 (hash partitioning의 잘못된 사용이 될 수 있으니 주의)
    - 이런 잘못된연산을 skew라고 하고 이럴때는 병합을 해야하며 그에 따른 자원 낭비가 생길수도 있다
  - range partitioning
    - 범위 파티셔닝
    - 순서가 있는 정렬된 파티셔닝