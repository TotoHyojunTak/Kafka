# Kafka UI Tool

Kafka 브로커를 모니터링하거나 관리하기 위한 툴 중 Kafka-UI를 학습한다.

[Reference] https://towardsdatascience.com/overview-of-ui-tools-for-monitoring-and-management-of-apache-kafka-clusters-8c383f897e80?gi=9ad65bcd8e85

[Github] https://github.com/provectus/kafka-ui

오픈소스이므로 무료이지만 아직 완성되지 않은 서비스 입니다.

구성은 멀티 클라스터로 구성하며 docker compose로 쉽게 사용할 예정이다.

###[Docker Compose] Kafka Multi Broker 구성하기
```
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:5.5.1
    ports:
      - '32181:32181'
    environment:
      ZOOKEEPER_CLIENT_PORT: 32181
      ZOOKEEPER_TICK_TIME: 2000


  kafka-1:
    image: confluentinc/cp-kafka:5.5.1
    ports:
      - '9092:9092'
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:32181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka-1:29092,EXTERNAL://localhost:9092
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3


  kafka-2:
    image: confluentinc/cp-kafka:5.5.1
    ports:
      - '9093:9093'
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:32181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka-2:29093,EXTERNAL://localhost:9093
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3


  kafka-3:
    image: confluentinc/cp-kafka:5.5.1
    ports:
      - '9094:9094'
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:32181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka-3:29094,EXTERNAL://localhost:9094
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
```


### [Docker Compose] Kafka UI 구성하기
```
version: '2'
services:
  kafka-ui:
    image: provectuslabs/kafka-ui
    container_name: kafka-ui
    ports:
      - "8989:8080"
    restart: always
    environment:
      - KAFKA_CLUSTERS_0_NAME=local
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka-1:29092,kafka-2:29093,kafka-3:29094
      - KAFKA_CLUSTERS_0_ZOOKEEPER=zookeeper:22181
```


위의 내용을 모두 구성한 후에는 localhost:8989로 접속하면 UI 화면을 확인할 수 있다.

## Main 화면
- Online : 현재 Kafka-UI와 연결하여 정상적으로 수행하고 있는 클러스터 갯수를 의미한다
- Cluster name : 클러스터 이름
- Version : 버전 (클러스터 버전, 사용자 임의)
- Brokers Count : 현재 카프카 브로커 개수
- Partitions : 현재 존재하는 파티션 갯수
- Topics : 클러스터에서 사용하는 기본 토픽 (기본값 1개)
- Production : 발행한 메시지 용량
- Consumption : 수신한 메시지 용량

## Broker 화면
- Uptime
  - Total Brokers : 총 브로커 개수
  - Active Controllers : 활동 중인 컨트롤러 갯수
  - Version : 브로커 버전

- Partitions
  - Online : 현재 활동중인 파티션 갯수
  - URP : 복제가 덜 된 파티션 갯수 (Un Replication Partitions)
  - In Sync Replicas : 현재 싱크되어있는 레플리카 갯수
  - Out of Sync Replicas : 현재 싱크가 맞지 않는 레플리카 갯수

- 상세정보
  - Broker: 각각 브로커를 구분해서 볼 수 있음
  - Segment Size(Mb) : 세그먼트 바이트. 기본값 0 Byte. 세그먼트는 파티션 내부에 분리된 영역
  - Segment Count : 세그먼트 갯수
  - Port : 현 브로커 포트 번호
  - Host : 호스트 이름

## TOPIC 화면
- 현재 생성된 모든 토픽 리스트 확인
- _confluent.support.metrics: 은 confluent에서 지원되는 Metric 정보
- 상세정보
  - Topic Name: 현재 존재하는 토픽 이름
  - Total Partitions: 토픽의 총 파티션 개수
  - Out of sync replicas: 복제 싱크가 맞지 않는 복제 개수
  - Replication Factor: 복제 factor (복제 개수는 브로커 개수보다 작게 설정하는 것이 일반적)
  - Number of messages: 메시지 갯수
  - Size: 메시지 총 크기

### TOPIC 생성하기
- 토픽 생성을 위한 화면이 나타난다.
  - Topic Name: 생성할 토픽 이름
  - Number of partitions: 생성할 파티션 개수
  - Replication Factor: 복제 계수
  - Min in Sync Replicas: 최소 복제 싱크 계수
  - Cleanup Policy: 클린업을 위한 정책(메시지 클린을 위한 정책)
     - Delete: 리텐션 기간이 지나면 삭제한다.
     - Compact: 압축을 수행한다.
     - Compact, Delete: 압축 후 삭제한다.
  - Time to retain data (in ms) : 데이터를 저장할 기간을 지정
  - Size on disk in GB: 디스크 최대 크기를 지정한다.
  - Maximum message size in bytes: 최대 메시지 크기
  - Custom parameters: 추가적인 파라미터 정보를 지정할 수 있다.

## Consumer 화면
- 클러스터에 연동된 컨슈머 정보를 나타낸다.
- 상세정보
  - Consumer Group ID: 컨슈머 그룹 아이디
  - Number Of Members: 컨슈머 그룹의 멤버수
  - Num Of Topics: 컨슈머 그룹의 토픽 수
  - Message Behind:
  - Coordinator
  - State: 컨슈머 상태

[출처] https://devocean.sk.com/blog/techBoardDetail.do?ID=163980