# Sqoop
- 아이스크림 스쿱처럼 데이터를 퍼서 하둡에 올리는 데이터 수집 도구
- RDBMS와 Hadoop 사이에서 데이터 전송을 쉽게 할 수 있도록 도와준다.
  - RDBMS에 있는 데이터를 Hadoop의 HDFS로 옮기는 역할을 한다. (반대의 경우도 가능)
- 데이터 전송을 위한 다양한 옵션과 데이터 포맷을 지원하여, 데이터 전송 및 변환 과정을 간편하게 처리할 수 있도록 한다.
- 가장 처음에 주목받았던 빅데이터 툴이었으나 (spark의 등장으로) 점차 사라져가는 추세이다.
- 하둡에서는 스쿱을 (2.7 이상까지 나왔는데도) 2.6 버전까지만 지원하여, 하둡과 버전 차이가 날 경우 오류가 발생한다. (별도 라이브러리 다운받아 스쿱에 넣어줘야 한다.)