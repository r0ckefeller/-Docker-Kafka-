### 1. Створити docker-мережу

```bash
docker network create kafka-net
```

### 2. Запустити Kafka 

```bash
docker run -d --name kafka --network kafka-net -p 9092:9092 `
>> -e KAFKA_NODE_ID=1 `
>> -e KAFKA_PROCESS_ROLES=broker,controller `
>> -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@kafka:9093 `
>> -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER `
>> -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:29092,CONTROLLER://0.0.0.0:9093,PLAINTEXT_HOST://0.0.0.0:9092 `
>> -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092 `
>> -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT `
>> -e KAFKA_INTER_BROKER_LISTENER_NAME=PLAINTEXT `
>> -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 `
>> -e KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1 `
>> -e KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1 `
>> -e KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS=0 `
>> -e KAFKA_AUTO_CREATE_TOPICS_ENABLE=true `
>> -e CLUSTER_ID=MkU3OEVBNTcwNTJENDM2Qk `
>> -v kafka-data:/var/lib/kafka/data `
>> confluentinc/cp-kafka:7.7.1
```

### 3. Зібрати образи Producer'а і Consumer'а

```bash
docker build -t kafka-demo-consumer -f DockerfileConsumer .
docker build -t kafka-demo-producer -f DockerfileProducer .
```

### 4. Запустити сервіси

```bash
docker run -d --name kafka-consumer --network kafka-net kafka-demo-consumer
docker run -d --name kafka-producer --network kafka-net kafka-demo-producer
```

## 5. Перевірка, що працює

```bash
docker logs kafka-producer
docker logs kafka-consumer
```

Очікуваний вивід Producer:

```
-> Запит надіслано: start=10 finish=100 (id=...)
<- Отримано відповідь: avgSteps=33
Готово. Контейнер живе.
```

Очікуваний вивід Consumer:

```
Чекаю запитів у 'demo-requests'.
<- Отримано запит: start=10 finish=100
-> Надіслано відповідь: avgSteps=33