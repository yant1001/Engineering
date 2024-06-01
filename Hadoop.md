# 분산 시스템의 이해
## 3-Tier Architecture
- Remote Call이 가능해지고 데이터의 전문 처리 시스템(DataBase)가 가능해졌다.
- 이로 인해 3-Tier Architecture(Client-Server-Database)가 온라인(웹) 서비스 구현의 하나의 패턴이 되었다.

## 기본 방식의 한계
- 2008년 음향회사였던 애플에서 아이폰을 출시하면서 스마트폰 시대가 시작되었다.
- 이때부터 인터넷을 본격적으로 사용하게 되었고, 데이터가 폭발적으로 증가하였다.
- 컴퓨터로는 쏟아지는 데이터량을 감당할 수 없어서, HDD, RAM 등의 사양을 높이는 방식(Scale-Up)을 시도하였으나, 방대한 트래픽과 데이터를 감당하지 못했다.

## 분산 시스템이 필요한 이유
- 기업 입장에서는 지속적인 Scale-Up 방식은 예산이 커질 뿐 효율적이지 못했다.
- 하드웨어의 덩치를 키우다 못해, 저렴한 컴퓨터를 여러 개 쓰자는 아이디어로 분산시스템이 등장한다.
- 따라서 서비스의 원활한 공급을 위해, 하드웨어가 아닌 소프트웨어에서 해결 방안을 찾게 된다. (GFS=>하둡)
- 분산시스템(Distribute File System : DFS)은 기본적으로 Scale-Out 방식이기 때문에 훨씬 경제적인 해결 방안이 되었다.
- Scale-Out은 물리적으로 떨어져있는 컴퓨터에 랜선을 꼽아 한대처럼 연결시키는 방식이며, 이를 성공시킨 회사가 하둡이다.
- 아직까지도 독보적인 영향력을 가지고 있어, 분산시스템은 곧 하둡이며, 빅데이터를 다룬다는 것은 곧 하둡을 다룬다는 의미와 동일하다.

## 분산시스템의 특징
- Concurrency (동시성)
  - 동시에 여러 작업 수행
- No Global Clock (비동기성)
  - A가 실행되고 있더라도 B와 C가 원하는 때에 병렬적으로 실행 가능 (== 비동기화)
  - 하나가 끝나야지만 다음이 시작될 수 있는 동기화의 경우 한 곳에서 문제가 생기면 실행이 멈추는(Lock) 문제가 있다. 다만 비동기에서도 같은 문제가 발생할 수는 있다.
- Independent Failure (실패로부터 독립)
  - A, B, C가 동작할 때 B에 문제가 생기더라도 A, C가 잘 동작해야 한다는 개념이다.
  - 즉 시스템 한 부분의 실패가 전체 시스템에 영향(장애)을 주면 안된다.

## 분산시스템의 고려 요소
- Heterogeneity (이식성)
  - 분산시스템은 여러 대의 컴퓨터를 사용한다는 의미이기 때문에, 서로 다른 시스템 사이에서 정보와 자원을 공유하며 동작할 수 있어야 한다.
  - 리눅스에서 분산시스템을 수행하고 있으나, 그 외 서로 다른 시스템에서도 분산시스템을 지원하는 경우가 있기 때문이다.
  - 즉 어떠한 환경에도 종속되지 않는 언어가 필요하다.
  - 대표적으로 객체지향 프로그래밍 언어인 java가 해당되며, java는 jvm만 OS에 맞게 설치되면 어디서든 동일한 코드를 실행시킬 수 있어 이식성이 좋은 언어이다. (+ 함수지향 Python 등)
  - C, C++ 등 Native Library에 대한 Dependency가 큰 언어로는 분산시스템을 개발하기 힘들다.
- Openess (개방성)
  - 서로 다른 요소 사이의 연결과 확장이 가능해야 한다.
- Security (보안성)
  - 권한제어, 접근제어 등이 가능해야 한다.
- Scalability (확장성)
  - 시스템 자원이나 사용자 수에 따라 확장 가능해야 한다.
  - 빅데이터 시스템에서는 주로 수평적 확장(Scale-Out)을 사용한다.
- Failure Handling (실패 제어)
  - 장애 및 실패에 대해 자동화된 방식으로 대응할 수 있어야 한다.
- Concurrency (동시성)
  - 여러 클라이언트가 하나의 공유 자원에 접근하는 등의 동시성에 대한 문제를 해결할 수 있어야 한다.
  - 예를 들어 클라이언트가 2명 이상일 때, 하나의 파일을 여러 컴퓨터에 나눠 담는 경우라도 함께 작업할 수 있어야 한다.
- Transparency (투명성)
  - 내부에 있는 정보를 보이지 않게 해야 한다.
  - 동일한 요청 한번에 모든 시스템을 전부 사용할 수 있어야 한다. (접근 투명성)
  - 컴퓨터들에 대한 정보 없이 리소스에 접근할 수 있어야 한다. (위치 투명성)
  - 원본과 복제본이 항상 똑같아야 한다. (복제 투명성)
  - 컴퓨터 중 하나에 문제가 생겨도 나머지 시스템은 영향을 받지 않고 작업을 마칠 수 있어야 한다. (실패 투명성) (복제 투명성 덕분에 가능)


# 빅데이터 엔지니어링
## 빅데이터와 빅데이터 플랫폼의 필요성
- 빅데이터는 물리적으로 Petabyte 이상의 굉장히 큰 데이터를 의미하며, 이러한 데이터로 효율적인 의사 결정과 기회 창출을 도모해야 한다.
- 오랜 시간 누적된 데이터는 기존의 방식대로는 분석과 예측이 불편하기 때문에 이를 위해 사용자들이 데이터를 처리하고 분석을 쉽게 할 수 있는 빅데이터 플랫폼을 구축해야 한다.
- 일반적으로 빅데이터 플랫폼을 구축하기 위한 요구 사항은 다음과 같다.
  - 데이터 수집, 처리 및 저장
  - 데이터 발견, 검색, 보안 제공
  - 데이터 분석 및 ML 지원
- 요구 사항을 충족한 이후에는 자동화를 통해 하나의 플랫폼을 만들어야 한다.
- 데이터 엔지니어는 이러한 플랫폼을 구축하여, 데이터로 하는 모든 업무를 서포팅하는 역할을 수행한다.
- 즉 엔지니어링에서 가장 중요한 부분은 설계이다. 
## Hadoop Eco System
- 하둡은 현 시점에서 가장 유명하고 탄탄한 빅데이터 플랫폼으로, 크게 원천 데이터, 수집 레이어, 처리 레이어, 저장 레이어, 분석&예측 레이어, 출력 레이어로 구성된다.
- `ELT`: 처리를 하지 않고 저장하는 것 (요즘 트렌드)
- `ETL`: 처리를 하고 저장하는 것

### 빅데이터 파이프라인
1. Data Source (원천 데이터)
   - 데이터 소스 선택
   - 데이터베이스, 파일 시스템, 네트워크 기반 데이터 소스 등 어디서 수집해야 하는지, 어떻게 수집해야 하는지, 어떤 데이터를 수집해야 하는지 결정해야 한다.
2. Data Ingestion & Processing (수집&처리 레이어)
   - 데이터 수집에는 실시간 데이터 수집과 배치 데이터 수집이 있다.
     - 실시간 데이터 수집과 처리
       - `realtime data` == 현재와 미래의 데이터, 즉 들어오고 있고 앞으로 들어올 데이터들
       - 대규모의 데이터 스트림에서 데이터를 실시간으로 수집하고 처리
       - Kafka, Flume, Nifi 등의 분산 메시징 시스템이나 Spark Streaming, Flink, Storm 등의 스트리밍 프레임워크가 사용된다.
     - 배치 데이터 수집과 처리
       - `batch data` == 이미 쌓여져 있는 과거의 데이터 (DataBase, File 등)
       - 큰 양의 데이터를 한 번에 수집하고 처리
       - Sqoop, Flume 등으로 배치 데이터 수집하며, 처리 속도는 느리지만 대규모 데이터 처리에 최적화되어 있다.
       - Hadoop MapReduce, Hive, Spark의 batch processing 등으로 배치 데이터 처리하며, 판다스도 동일한 역할을 수행한다.
3. 빅데이터 적재 (저장 레이어)
   - 데이터를 저장할 저장소를 선택하고, 데이터를 적재한다.
   - 데이터를 모델링, 인덱싱, 필터링, 병합하는 등의 작업을 수행한다.
   - 적재를 위한 저장소로 Hadoop HDFS, NoSQL의 DB인 MongoDB, Cassandra, HBase 등이 사용된다.
   - 데이터 레이크 => 데이터 웨어하우스 => 데이터 마트 순으로 구축된다.
   - 데이터 웨어하우스를 만들 때는 스파크나 하이브, 데이터 마트를 만들 때는 스파크를 주로 사용하며, 스파크 하나만으로도 웬만한 툴을 대체할 수 있다. (ML 구현, 실시간 분석도 가능)
   - 최근에는 다른 플랫폼과의 연결다리 혹은 환경을 구축하는 플랫폼 역할로만 하둡을 사용한다.
   - 하둡을 기반으로 한 분산시스템 핸들링은 필수적이다.
   - 데이터 레이크 (Data Lake)
     - 대규모 데이터를 실시간으로 수집, 저장, 처리한다.
     - `ELT`, `ETL` 등의 가공되지 않은 데이터를 데이터 레이크에 쌓아놓는다.
   -  데이터 웨어하우스 (Data Warehouse)
      -  대규모의 데이터를 저장하고 필요한 데이터를 위한 쿼리 실행을 위해 설계된 데이터베이스이다.
      -  데이터 레이크에 쌓여있는 데이터를 분류하고 전처리해야 하는데 이것을 컨테이너라고 하며, 곧 데이터 웨어하우스를 의미한다.
   -  데이터 마트 (Data Mart)
      -  데이터 웨어하우스에 분류된 데이터를 완벽한 상품으로 만드는 단계를 상품화 단계라고 한다.
      -  상품화된 데이터가 쌓여있는 형태를 데이터 마트라고 한다.
      -  데이터 마트로 저장된 데이터를 분석하게 된다.
4. 데이터 분석&예측 레이어
   - 데이터 마트까지 데이터 적재가 완료되면 분석&예측 레이어에서 데이터를 분석하고 예측 모델을 구축한다.
   - 데이터 시각화, 통계 분석, ML, DL 등의 기술을 활용한다.
5. 데이터 출력 레이어
   - 예측 모델을 바탕으로 데이터 시각화와 보고서 작성 등이 수행된다.
   - 분석에 대한 결과물은 태블로, 파워 BI, 아마존 퀵사이트 등으로 나타내며, 시각화 및 보고서 자료를 의사 결정에 활용하게 된다. +
  

# Hadoop
## 구조
- 클러스터
  - 여러 대가 연결되어 하나의 시스템처럼 동작하는 컴퓨터들의 집합
- Master/Worker 아키텍쳐
  - 하둡의 대표 아키텍쳐
  - 관리자의 역할을 하는 Master 노드, 실제 데이터에 대한 처리 업무를 담당하는 Worker 노드로 이루어져 있다. (사장님과 직원 관계)
  - 클라이언트(고객)가 워커(직원)에게 직접 요청할 필요 없이, 클라이언트(고각)가 마스터(사장)에게 요청하고, 마스터(사장)가 워커(직원)에게 명령을 내리는 구조
  - 마스터 노드는 워커 노드의 메타데이터를 가지고 있고 이를 토대로 클라이언트 요청을 수행할 수 있는 데이터를 가진 워커 노드에게 작업을 명령한다. (메타데이터: 데이터를 설명하기 위한 데이터)
  - 클라이언트는 마스터하고만 통신하면 되기 때문에 인터페이스가 간단해지고, 워커(직원, 실무자)들의 주소를 직접 알 필요가 없게 된다.

## 변천사
### 하둡1
- 하둡은 쉽게 말해 디렉토리에 파일을 삽입, 삭제, 조회하는 File System과 유사한 형태이다.
- 클라이언트는 네임노드하고만 소통한다. 클라이언트가 하둡의 파일을 읽고 쓸 때 네임노드에 요청해서 처리하게 된다.
- 클라이언트의 요청을 받은 네임노드는 워커노드에게 작업내역을 지시한다.
- 실제 데이터는 네임노드가 아닌 워커노드에 존재한다.
- 이 때 워커의 작업 현황을 파악하여 적절하게 일을 분배시켜주는 것을 스케쥴링(계획)이라고 하며, 이 역할을 JobTracker가 수행한다.
- JobTracker는 전체적인 Job의 작업을 계획 및 모니터링하며, Job에 어떤 Task가 있는지 분석하고 체크한 후 쪼개진 Task들을 워커노드에게 분배한다.
- 워커노드에 위치한 TaskTracker에서 전달받은 Task 간 우선순위를 정하고 스케쥴링한다.
  - Task: 예를들어, SQL 쿼리를 Job이라고 하며 쿼리 안에서 일어나는 연산들을 Task라고 지칭한다.
- 하둡1의 가장 큰 문제점
  - 네임노드의 단일 장애 지점 (SPOF: Single Point Of Failure)
    - 네임노드에 장애가 발생하면 일부 데이터가 유실되는 현상
    - 네임노드는 한개만 활성화되기 때문에 네임노드가 망가지면 클라이언트 요청을 처리할 서버가 사라진다.
    - Secondary NameNode가 있지만, 둘 사이의 시간 차이가 존재한다.
  - JobTracker 부하
    - 클러스터의 모든 작업을 관리하므로 부하가 일어나기 쉽다.
  - 맵리듀스 이외의 작업은 수행 불가능
    - 맵리듀스: 하둡의 파일 저장소라고 할 수 있는 HDFS 내의 데이터를 처리하는 모듈
    - 과거 데이터 분석가들은 SQL만 할 수 있는 경우가 대부분이었는데, 하둡1에서는 맵리듀스 프로그래밍만 지원했기 때문에 분석가들이 코딩을 하거나, 개발자가 쿼리를 할 줄 알아야 했다.
- 네임노드는 1대만 있어야 하는데 하는 일이 많기 때문에 금방 과부화가 올 위험이 있었다. 이걸 보완하여 2, 3년 만에 하둡2가 등장한다.

### 하둡2
- 네임노드를 Active와 Standby로 관리하여, Active 네임노드가 다운되면 Standby 네임노드가 작동할 수 있게 하였다.
- Active와 Standby 네임노드 사이에 JournalNode까지 연결시켜 조금 더 견고한 네임노드 구조가 완성되었다.
- 하둡1의 JobTracker와 TaskTracker가 사라지고 ResourceManager와 NodeManager로 교체되었다.
  - ResourceManager: YARN이라고 하며, 워커노드 작업을 스케쥴링 해주는 네임노드의 비서 역할이 분리되었다. 리소스매니저가 등장함으로써 네임노드가 훨씬 여유로워졌기 때문에 하둡1보다 훨씬 많은 컴퓨터를 추가할 수 있게 되었다.(~수만대) 워커 노드들이 최대한 효율적으로 데이터를 처리할 수 있도록 한다. 노드매니저는 주기적으로 리소스매니저에게 상태를 전송한다. (하트비트 패킷)
  - NodeManager: 데이터노드의 있는 노드매니저 안에는 Container와 Application Master가 있다. 이는 맵리듀스뿐만 아니라 다양한 분산 처리 어플리케이션을 사용할 수 있도록 지원한다.
- 일례로 카카오 데이터센터 화재 사건을 통해 구조를 이해할 수 있다. 
  - SK 사옥이 화재 => 카카오톡 서버 마비 => 서울 서버(Active NameNode)에 불이 났으니 스탠바이된 지방 서버(Active NameNode)가 활성화되어야 하는데 이에 실패 => 데이터 이중화에는 성공했으나 서버 이중화에는 실패했다고 입장 발표
- 하둡2로 인해 Scale-Out일 때 훨씬 비용 절감이 가능하다는 것이 증명되었다.
- YARN의 분리와 Standby NameNode의 추가가 하둡2의 혁신적인 변혁이었다.
- Application Master (/DataNode/NodeManager/Container&ApplicationMaster): 하둡이 맵리듀스 이외의 다른 빅데이터 도구도 지원받을 수 있도록 해주었다. (하이브, 스파크 등) 이로 인해 빅데이터 전문가들에게 자바 프로그래밍이 필수적 요소가 아니게 되었다.
  - 다만 하둡 자체는 맵리듀스로만 작동되기 때문에 하이브 등으로 작업하더라도 결국 맵리듀스로 프로그래밍된다.

### 하둡3
- 하둡2와 동일한 구조에 기능이 추가된 업데이트 버전이다.
- Erasure Coding 지원
  - 데이터 복제본의 사이즈를 줄여 용량의 효율적인 관리 지원
- Java 8
  - 하둡2는 Java 1.7을 지원했으나 하둡3부터는 Java 1.8 이상을 지원
  - Java 1.7까지는 객체 지향 프로그래밍만 지원하였다.
  - 객체 지향 프로그래밍은 마치 레고처럼 각각의 클래스로 객체를 만들어서 필요한 부분을 조립하는 방식이다. 객체 지향 프로그래밍은 객체가 없이 기능만 있어도 되는 경우라고 하더라도 반드시 객체가 필요하다는 단점이 있다.
  - Java 1.8부터는 함수형 프로그래밍으로, lambda 식이 추가되었다. 함수형 프로그래밍은 데이터 처리에서 필수적이다.
- NameNode 이중화 기능 강화
  - 네임노드 이중화를 2개까지만 지원했으나, 여러 개를 동시에 이중화할 수 있도록 지원하기 시작

## 실습
### 하둡 설치
`wget https://dlcdn.apache.org/hadoop/common/hadoop-3.2.4/hadoop-3.2.4.tar.gz` pass (wget == web get의 약어)

`tar xvfz hadoop-3.2.4.tar.gz` 하둡 설치

`cd ~/.bashrc` vi editor 열기

```python
# HADOOP_HOME
export HADOOP_HOME=/home/ubuntu/hadoop-3.2.4
export PATH=$PATH:$HADOOP_HOME/bin
```

`esc + :wq` 환경변수 생성 확인 후 나오기

`source ~/.bashrc` 환경변수 불러오기

`echo $HADOOP_HOME` 잘 생성되었는지 확인 (출력)

#### ssh 로그인 설정

`cat ~/.ssh/authorized_keys` authorized_keys (key box) 확인

`ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa` pass

`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys` 퍼블릭키(pub)를 키박스(authorized_keys) 추가

`chmod 0600 ~/.ssh/authorized_keys` pass

`cat ~/.ssh/authorized_keys` authorized_keys (key box) 잘 들어갔는지 확인

`ls ~/.ssh/`

`ssh [localhost](http://localhost)` 로컬호스트로 로그인

`w` 접속 확인

`exit` 로컬호스트 로그아웃

`w` 접속 확인

### 하둡 설정 (core-site/hdfs-site/yarn-site/mapred-site)

#### core-site.xml

`vim $HADOOP_HOME/etc/hadoop/core-site.xml` core-site.xml 열기

`i` 인서트 모드로 변경

```python
<configuration>
    **<property>
        <name>fs.defaultFS</name>
        <value>hdfs://$your_namenode_host:9000</value>
    </property>**
</configuration>
```

`esc + :wq`

#### hdfs-site.xml

`vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml`  hdfs-site.xml 파일열기

`i` 인서트 모드

```python
    **<property>
        <name>dfs.replication</name> 
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/path/to/hadoop-3.2.4/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/path/to/hadoop-3.2.4/dfs/data</value>
    </property>
	  <property>
				<name>dfs.namenode.http-address</name>
				<value>$your_hdfs_host:9870</value>
		</property>**
```

`esc + :wq`

- 네임노드, 데이터노드를 위한 디렉토리 생성

`mkdir -p $HADOOP_HOME/dfs/name` 네임노드를 위한 디렉토리 생성

`mkdir -p $HADOOP_HOME/dfs/data` 데이터노드를 위한 디렉토리 생성

`ls $HADOOP_HOME/dfs` name과 data 디렉토리 생성 확인

#### yarn-site.xml

`vi $HADOOP_HOME/etc/hadoop/yarn-site.xml` yarn-site.xml 파일 열기

`i`

```python
    **<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
    </property>
		<property>
				<name>yarn.resourcemanager.webapp.address</name>
				<value>$your_nodemanager_host:8088</value>
		</property>
		<property>
				<name>yarn.nodemanager.webapp.address</name>
				<value>$your_nodemanager_host:8042</value>
		</property>**
```

`esc + :wq`

#### mapred-site.xml

`vi $HADOOP_HOME/etc/hadoop/mapred-site.xml` mapred-site.xml 파일열기 

```python
<configuration>
    **<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
		<property>
				<name>yarn.resourcemanager.webapp.address</name>
				<value>0.0.0.0:8088</value>
		</property>
		<property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>0.0.0.0:19888</value>
    </property>**
</configuration>
```

`esc + :wq` 

- hadoop-env.sh

`vim $HADOOP_HOME/etc/hadoop/[hadoop-env.sh](http://hadoop-env.sh)` hadoop-env.sh 파일열기 

`shift + G` 스크롤 맨 아래로 내리기

`JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre` 맨 아래에 삽입

`esc + :wq` 

#### HDFS 실행

`hdfs namenode -format` HDFS 실행 전 네임노드 포맷 (반드시 처음 시작하고 1회만 포맷)

- 포맷은 디스크 초기화에 해당한다. 하둡의 DFS 사용하기 전 필요한 환경을 구성하기 위한 필요작업으로 생각하면 된다.

`$HADOOP_HOME/sbin/start-dfs.sh` 네임노드 실행

`jps` Jps / DataNode / NameNode / SecondaryNameNode 정상 실행 확인 

#### YARN 실행

`$HADOOP_HOME/sbin/start-yarn.sh` YARN (리소스 매니저) 실행

`jps` ResourceManager / NodeManager 추가적으로 2개 정상 실행 확인 

#### JobHistoryServer 시작

`mapred --daemon start historyserver` jobhistoryserver 시작

`jps` JobHistoryServer 추가적으로 1개 정상 실행 확인 (총 7개)

### 맵리듀스 예제 실행

`hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.4.jar pi 16 10000` 맵리듀스 예제 실행

- 실행 최종 결과는 [리소스 매니저 web(퍼블릭IPv4DNS) FINISHED 탭](http://<퍼블릭IPv4DNS>:8088/cluster/apps/FINISHED)에서 확인 가능

**다끝나면 꼭 start⇒stop으로 바꿔서 종료하고 터미널 꺼야 한다.

# YARN
- 하둡2부터 도입된 리소스매니저
- 맵리듀스1
  - 마스터 역할은 JobTracker가 수행하고 워커의 역할은 TaskTracker가 수행했다. 클라이언트가 Job을 jobtracker에게 제출하면 jobtracker는 자원을 할당하여 tasktracker가 작업을 수행할 수 있도록 해준다. tasktracker는 task를 실행하고 작업 상황을 jobtracker에게 전달한다.
  - 클라이언트가 작업 요청을 하게 되면 JobTracker가 작업 할당을 하게 된다. 워커들에게 할당된 작업을 토대로 스케쥴링이 이루어지는데, 이떼 jobtracker의 역할이 과중되어 워커노드를 늘리는 클러스터 확장에 제한을 받게 된다는 단점이 있었다.
- YARN (Yet Another Resource Negotiator) 등장
  - Negotiator 협상가의 등장
  - YARN이 등장, 즉 리소스매니저 노드가 따로 분리되면서 jobtracker가 하던 자원 관리는 ResourceManager, 작업 관리 책임은 ApplicationMaster가 하게되어 마스터노드의 부담이 덜어졌다.
  - resource manager는 자원 관리, application master는 작업 관리(실행할 작업 내용 관리)를 맡게 되었다.
  - 기존의 맵리듀스에서는 자원(CPU, 메모리, 디스크, 네트워크)을 슬롯으로 관리하였는데, resource manager는 자원을 리소스 컨테이너라는 단위로 추상화하여 배분하여 관리하였다.
    - 마치 창고에 호미를 여럿 두고 작업에 필요할 때마다 호미를 가져갈 수 있는 것처럼 컨테이너 시스템으로 바뀌어, 컨테이너를 통해 클러스터 이용률이 개선되었다.
## 구조
- 리소스매니저는 모든 클러스터의 자원을 중재한다.
- 플러그인 가능한 스케쥴러와 클러스터 사용자의 Job을 관리하는 어플리케이션 매니저로 구성되어 있다.
- Scheduler
  - 제약에 대한 다양한 어플리케이션에 자원을 할당한다.
  - 실시간 데이터가 일정시간 쌓이면 하둡에게 보낸다.
  - 스케쥴러는 플러그인(컴퓨터에 추가 프로그램을 설치하여 기능 수행이 가능하도록 하는 것)이 가능하다.
  - 스케쥴러의 플러그인
    - FIFO: first in first out 방식이며, 뒤따라오는 작업은 무한정 기다려야 한다는 단점이 있다. 기본 스케쥴링 방식으로 적용된다.
    - Capacity: 사용할 작업 용량을 항상 보장할 수 있으며, 순환 작업 처리에 많이 사용된다.
    - Fair: 모든 어플리케이션이 시간이 지남에 따라 자원을 평균적으로 균등하게 공유하는 방식이다.
- Application Manager
  - 제출된 다수의 어플리케이션의 유지를 책임진다.
    - 어플리케이션에 필요한 모듈(자원, 작업 도구)을 쌓아놓는 곳이 컨테이너(창고)
    - 파일이나 쿼리를 마스터노드에게 업로드한다.
    - 앱을 실행하기 위한 컨테이너 초기화 설정 역할을 한다.
    - 문제가 생겼을때 컨테이너의 재구성까지 책임진다.
- 노드매니저는 노드들을 관리한다. (like 개인 매니저)
- 노드의 작업 상태(바쁘고 한가한 정도, 자원 차지 정도 등)를 리소스매니저와 공유하고, 어플리케이션 컨테이너를 관리 감독(노드의 상태를 컨테이너로 관리)한다.
- 노드가 정상적으로 살아있는 상태임을 마스터노드에게 알리기 위해 HeartBeat를 전송한다.
- 하둡2에서는 job을 application이라고 부른다.
- 어플리케이션 마스터는 리소스 매니저와 자원을 협상한다. 어플리케이션 마스터 입장에서는 가용 자원이 많을수록 좋고, 리소스 매니저는 적당한 자원을 제공해야 하기 때문에 서로 협의가 필요하다.

## 동작 방식
- 클라이언트로부터 어플리케이션 제출
- 리소스 매니저의 어플리케이션 매니저가 이를 받아 최초의 어플리케이션 마스터를 위한 컨테이너 할당을 요청
- 어플리케이션 마스터가 어플리케이션 실행을 위한 컨테이너를 스케쥴러에게 요청
- 스케쥴러는 어플리케이션 실행을 스케쥴링하면서 노드의 자원 상태에 따라 컨테이너 할당
- 워커노드에서 노드매니저가 컨테이너 생성 담당
- 컨테이너는 리소스를 사용하여 어플리케이션 실행
- 어플리케이션 마스터는 컨테이너 실행 상태를 모니터링 및 추적
- 실행이 완료되면 어플리케이션 매니저에게 알림