# Flume
- 스쿱보다 많이 사용하는 데이터 수집 도구

## Log
- 운영체제나 다른 소프트웨어가 실행 중에 발생하는 이벤트나 각기 다른 사용자의 통신 소프트웨어 간의 메시지를 기록한 데이터
- Flume은 정확히는 Log Flume이라고 말해야 한다.
- Log Flume은 통나무 수로, 즉 통나무(로그)를 옮기기 위한 수로(플롬)을 의미한다.
- 로그 플롬의 목적지는 하둡이 될 수도 있고 콘솔창이 될 수도 있다.

## Flume의 주요 구성 요소
### Event
- 데이터 전달 단위
- 이벤트는 곧 데이터라고 생각하면 된다.
- 로그를 이벤트화 시켜 보내게 된다.

### Agent
- Flume의 가장 기초적인 작업 단위
- 하나의 프로세스라고 생각하면 된다.
- Source, Channel, Sink, Interceptor(선택적)로 구성된 작업 단위
- Agent 하나가 로그 데이터를 옮겨주는 수로 역할을 한다.

### Source
- 어디에서 어떤 데이터를 보낼지 지정해준다.
- 하나 이상의 채널로 데이터 전달이 가능하다.
- 여러 형식의 다양한 데이터를 수집할 수 있어야 하기 때문에 여러 컴포넌트를 제공한다.
- 데이터를 수신한 후 채널로 전달한다.
- Source => Channel => Sink 순으로 데이터가 흘러간다.

### Interceptor (선택적)
- 소스로부터 오는 데이터를 가공해서 채널에 넘기는 역할

### Channel
- 소스로부터 받은 데이터가 잠시 머물다 가는 공간
- 어느 곳으로 보낼지 채널에서 최종적으로 결정한다.
- 소스에서 받은 데이터를 잠시 동안 쌓아둔 뒤 정리 후 내보내게 된다.

### Sink
- 수집한 데이터를 최종 목적지로 보내는 맨 마지막 모듈
- 최종 목적지는 HDFS, Kafka, Elastic search, HBase나 다른 Flume Agent가 될 수도 있다.

## Flume Agent 구성 예시
1. 하나의 로그를 받아, 하나의 소스, 채널, 싱크로 구성된 Flume Agent가 있을 수 있다. 싱크로 전달된 데이터는 HDFS에 적재된다.
   - Agent에는 최소한 소스, 채널, 싱크가 하나씩은 존재해야 한다.
   - 소스는 여러 채널로 전달이 가능하다.
   - 싱크는 오로지 하나의 채널에서만 데이터를 전달받을 수 있다.
2. 하나의 로그를 받아, 하나의 소스로부터 두 개의 채널로 전달되는 Flume Agent가 있을 수 있다. 하나의 채널은 기존 구성대로 채널, 싱크를 타고 HDFS에 적재된다. 나머지 하나는 채널에 들어가기 전 인터셉터로 가공된 뒤 채널과 싱크를 타고 Elastic search에 적재된다. (예시)
   - 하나의 로그를 소스가 받아오더라도 여러 갈래로 나뉠 수 있다.
3. 3개의 로그를 받아, 각각 하나의 소스, 채널, 싱크로 총 3개의 Flume Agent가 이루어지고, 최종적으로 하나의 Flume Agent로 집계 저장될 수도 있다.
   - 작업 단위는 로그 소스의 개수와 동일하다.