# kafka_infra
- 간단한 single broker 기반 Kafka 사용 예시

![image](https://user-images.githubusercontent.com/36991763/222956950-b37aec83-ef9b-41f6-8825-c4b6f37ad110.png)

# How to use?

```yaml
version: "3.8"

services:
  zookeeper:
    container_name: my_zookeeper
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEPER_SYNC_LIMIT: 2
    ports:
      - "22181:2181"
  kafka:
    container_name: my_kafka
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - "29092:29092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: "zookeeper:2181"
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0

```
1. zookeeper, kafka 를 위와 같이 세팅한다. 
  - zookeeper 환경변수 목록
     - ZOOKEEPER_SERVER_ID : zookeeper 클러스터에서 주키퍼를 식별할 아이디
     - ZOOKEEPER_CLIENT_PORT : 주키퍼 클라이언트 포트
     - ZOOKEEPER_TICK_TIME : 동기화를 위한 기본 틱 타임 (ms)
     - ZOOKEEPER_INIT_LIMIT : 주키퍼 초기화를 위한 제한시간
     - ZOOKEEPER_SYNC_LIMIT : 주키퍼 리더와 나머지 서버들의 싱크 타임
  - kafka 환경변수 목록
     - KAFKA_BROKER_ID: kafka 브로커 아이디
     - KAFKA_ZOOKEEPER_CONNECT: kafka가 zookeeper에 커넥션하기 위한 대상을 지정
     - KAFKA_ADVERTISED_LISTENERS: 외부에서 접속하기 위한 리스너 설정
     - KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 보안을 위한 프로토콜 매핑
     - KAFKA_INTER_BROKER_LISTENER_NAME: 도커 내부에서 사용할 리스너 이름을 지정한다
     - KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: single 브로커인경우에 지정. 멀티브로커일 땐 필요없음.
     - KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 카프카 그룹이 초기 리밸런싱할때 컨슈머들이 컨슈머 그룹에 조인할때 대기 시간

2. docker-compose 명령을 통해 zookeeper, kafka 실행
```shell
docker-compose up -d
```
3. topic 생성
```shell
$ docker-compose exec kafka kafka-topics --create --topic my-topic --bootstrap-server kafka:9092 --replication-factor 1 --partitions 1
```
4. 생성 된 토픽 확인
```shell
$ docker-compose exec kafka kafka-topics --describe --topic my-topic --bootstrap-server kafka:9092 

Topic: my-topic TopicId: 0K_QBf2aTCmKm0tY_nxyew PartitionCount: 1       ReplicationFactor: 1    Configs:  
Topic: my-topic Partition: 0    Leader: 1       Replicas: 1     Isr: 1  
```
5. 컨슈머 실행
```shell
$ docker-compose exec kafka bash
$ kafka-console-consumer --topic my-topic --bootstrap-server kafka:9092
```
6. 프로듀서 실행 및 결과 확인
```shell
$ docker-compose exec kafka bash
$ kafka-console-producer --topic my-topic --broker-list kafka:9092
> Hello
> This is Producer
```
- 컨슈머에서 메시지를 확인할 수 있다. 
![image](https://user-images.githubusercontent.com/36991763/222957381-c7f8714d-6cfe-4aaf-bc18-0e571861a041.png)
