# 1. Membuat Kafka dengan Docker

Official [Docs](https://kafka.apache.org/)

## Buat Image dengan dockerfile

Buat file dengan nama `dockerfile` </br>
Untuk membuild `dockerfile`
```
cd ./kafka
sudo docker build . -t zulfikar4568/kafka:2.8.0
```

## Jelajahi Kafka yang terinstall

Kita jalankan image yang sudah kita buat dan menjadi container
```
sudo docker run --rm --name kafka -it zulfikar4568/kafka:2.8.0 bash

ls -l /kafka/bin
cat /kafka/config/server.properties
```

Lalu buka terminal baru, kita copy keluarkan konfigurasi tersebut keluar container dengan `docker cp`
```
sudo docker cp kafka:/kafka/config/server.properties ./server.properties
sudo docker cp kafka:/kafka/config/zookeeper.properties ./zookeeper.properties
```
Kita buat folder `kafka-1, kafka-2, kafka-3` di `/kafka/config` dan buat folder `zookeeper-1` di `/zookeeper/config` </br> 
Lalu pindahkan `server.properties` ke masing-masing `kafka-1, kafka-2, kafka-3`, dan pindahkan `zookeeper.properties` ke `zookeeper-1` </br>
</br>
Ubah masing2 configurasi `server.properties`
```
broker.id=1
zookeeper.connect=zookeeper-1:2181
```

# Zookeeper - 1

Selanjutnya untuk membuat image Zookeeper sama halnya dengan kafka yang berbeda hanyalah pada menjalankan script nya `start-zookeeper.sh` script
```
cd ./zookeeper
sudo docker build . -t zulfikar4568/zookeeper:2.8.0
cd ..
```
Agar tiap container bisa saling terhubung maka kita harus membuat network docker seperti berikut
```
sudo docker network create kafka
```
Lalu jalankan docker dengan
```
sudo docker run -d \
--rm \
--name zookeeper-1 \
--net kafka \
-v ${PWD}/zookeeper/config/zookeeper-1/zookeeper.properties:/kafka/config/zookeeper.properties \
zulfikar4568/zookeeper:2.8.0

sudo docker ps
sudo docker logs zookeeper-1
```

# Kafka 
Pada kali ini kita akan membuat 3 buah server kafka seperti berikut

## Kafka - 1
Selanjutnya membuat Kafka - 1

```
sudo docker run -d \
--rm \
--name kafka-1 \
--net kafka \
-v ${PWD}/kafka/config/kafka-1/server.properties:/kafka/config/server.properties \
zulfikar4568/kafka:2.8.0

sudo docker ps
sudo docker logs kafka-1
```

## Kafka - 2
Selanjutnya membuat Kafka - 2

```
sudo docker run -d \
--rm \
--name kafka-2 \
--net kafka \
-v ${PWD}/kafka/config/kafka-2/server.properties:/kafka/config/server.properties \
zulfikar4568/kafka:2.8.0

sudo docker ps
sudo docker logs kafka-2
```

## Kafka - 3
Selanjutnya membuat Kafka - 3

```
sudo docker run -d \
--rm \
--name kafka-3 \
--net kafka \
-v ${PWD}/kafka/config/kafka-3/server.properties:/kafka/config/server.properties \
zulfikar4568/kafka:2.8.0

sudo docker ps
sudo docker logs kafka-3
```

# Topic pada Apache Kafka
Selanjutnya kita akan membuat topic `Test`, Untuk membuat topic Kafka dan Zookeeper mempunyai script yang bisa kita jalankan untuk membuat topic seperti berikut:

Masuk terlebih dahulu ke dalam container zookeeper
```
sudo docker exec -it zookeeper-1 bash
```
Membuat Topic dengan Zookeeper
```
/kafka/bin/kafka-topics.sh \
--create \
--zookeeper zookeeper-1:2181 \
--replication-factor 1 \
--partitions 3 \
--topic Test
```
Descripsikan Topic Kita
```
/kafka/bin/kafka-topics.sh \
--describe \
--topic Test \
--zookeeper zookeeper-1:2181
```
Maka akan muncul seperti apa Topic kita
```
Topic: Test	TopicId: J55QtZtEQYai1AFfnJDdWQ	PartitionCount: 3	ReplicationFactor: 1	Configs: 
	Topic: Test	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: Test	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
	Topic: Test	Partition: 2	Leader: 3	Replicas: 3	Isr: 3

```

# Producer & Consumer sederhana
Dengan kita menggunakan network `kafka` yang sama maka topic `Test` yang kita gunakan bisa saling terhubung

## Membuat Consumer untuk mendapatkan message
```
docker exec -it zookeeper-1 bash

/kafka/bin/kafka-console-consumer.sh \
--bootstrap-server kafka-1:9092,kafka-2:9092,kafka-3:9092 \
--topic Test --from-beginning
```

## Membuat Producer untuk mengirim message
Buka Terminal baru lalu ketikan :
```
sudo docker exec -it zookeeper-1 bash

echo "Hallo: 1" | \
/kafka/bin/kafka-console-producer.sh \
--broker-list kafka-1:9092,kafka-2:9092,kafka-3:9092 \
--topic Test > /dev/null
```
## Message pada Kafka
Setelah message masuk ke kafka, kita dapat menjelajahi message yang sudah tersimpan di `partition`
Buka Terminal baru lalu ketikan : </br>
```
sudo docker exec -it kafka-1 bash

apt install -y tree

tree /tmp/kafka-logs
```
Maka akan muncul log
```
├── Test-0
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-1
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-10
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-13
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-16
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-19
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-22
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-25
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-28
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-31
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-34
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-37
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-4
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-40
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-43
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-46
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-49
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-7
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── cleaner-offset-checkpoint
├── log-start-offset-checkpoint
├── meta.properties
├── recovery-point-offset-checkpoint
└── replication-offset-checkpoint
```
Dan untuk melihat detail nya kita bisa gunakan:

### Melihat Log di Kafka - 1
```
sudo docker exec -it kafka-1 bash

ls -lh /tmp/kafka-logs/Test-0
```
Maka akan muncul log :
```
total 4.0K
-rw-r--r-- 1 root root 10M Aug 15 07:50 00000000000000000000.index
-rw-r--r-- 1 root root   0 Aug 15 07:50 00000000000000000000.log
-rw-r--r-- 1 root root 10M Aug 15 07:50 00000000000000000000.timeindex
-rw-r--r-- 1 root root   8 Aug 15 07:50 leader-epoch-checkpoint
```

### Melihat Log di Kafka - 2
```
sudo docker exec -it kafka-2 bash

ls -lh /tmp/kafka-logs/Test-1
```
Maka akan muncul log :
```
total 8.0K
-rw-r--r-- 1 root root 10M Aug 15 07:50 00000000000000000000.index
-rw-r--r-- 1 root root  76 Aug 15 07:53 00000000000000000000.log
-rw-r--r-- 1 root root 10M Aug 15 07:50 00000000000000000000.timeindex
-rw-r--r-- 1 root root   8 Aug 15 07:50 leader-epoch-checkpoint
```

### Melihat Log di Kafka - 3
```
sudo docker exec -it kafka-3 bash

ls -lh /tmp/kafka-logs/Test-2
```
Maka akan muncul log :
```
total 4.0K
-rw-r--r-- 1 root root 10M Aug 15 07:50 00000000000000000000.index
-rw-r--r-- 1 root root   0 Aug 15 07:50 00000000000000000000.log
-rw-r--r-- 1 root root 10M Aug 15 07:50 00000000000000000000.timeindex
-rw-r--r-- 1 root root   8 Aug 15 07:50 leader-epoch-checkpoint
```

### Kesimpulan
Jika di lihat bahwa 0 bytes di partisi 0 dan 1, maka message di simpan pada partition 2 76 Bytes

Untuk melihat message kita bisa lihat dengan 
```
cat /tmp/kafka-logs/Test-2/*.log
```

# 2. Membuat Kafka dengan Docker Compose
Pastikan docker compose telah terinstal, check dengan cara
```
sudo docker-compose --version
```
Sejauh ini kita sudah membuat Kafka dan Zookeeper dengan perintah docker, lalu explorasi tentang konfigurasi kafka dan bagaimana kita mem-produce message dan men-consume message.

Sekarang kita akan membuat kafka dan Zookeeper dengan perintah docker-compose. Docker compose digunakan agar kita bisa membuat configurasi docker secara otomatis, tidak secara manual seperti perintah sebelumnya.

Dengan compose kita akan  membuat container dan di arahkan ke `dockerfile` dengan `build.context`, dan juga menggunakan `volumes` untuk menggunakan config file.

Catatan: Producer dan Consumer berjalan pada installasi image kafka (prepakackage) dengan contoh consumers dan producers sebagai script. Kita timpa entrypoint kafka dengan `bash` dan `stdin` jadi akan dimulai dari keadaan `pause` jadi kita bisa jalankan script sebagai contoh

Buat file dengan `docker-compose.yaml` file :
```
version: "3.8"
services:
```
## Zookeeper
```
zookeeper-1:
    container_name: zookeeper-1
    image: zulfikar4568/zookeeper:2.8.0
    build:
      context: ./zookeeper
    volumes:
    - ./zookeeper/config/zookeeper-1/zookeeper.properties:/kafka/config/zookeeper.properties
    - ./zookeeper/data/zookeeper-1/:/tmp/zookeeper/
    networks:
    - kafka
```
## Kafka-1 sampai 3
```
kafka-1:
    container_name: kafka-1
    image: zulfikar4568/kafka:2.8.0
    build: 
      context: .
    volumes:
    - ./kafka/config/kafka-1/server.properties:/kafka/config/server.properties
    - ./kafka/data/kafka-1/:/tmp/kafka-logs/
    networks:
    - kafka
```
## Producer
```
kafka-producer:
    container_name: kafka-producer
    image: zulfikar4568/kafka:2.8.0
    build: 
      context: ./kafka
    working_dir: /kafka
    entrypoint: /bin/bash
    stdin_open: true
    tty: true
    networks:
    - kafka
```
## Consumer
```
kafka-consumer:
    container_name: kafka-consumer
    image: zulfikar4568/kafka:2.8.0
    build: 
      context: .
    working_dir: /kafka
    entrypoint: /bin/bash
    stdin_open: true
    tty: true
    networks:
    - kafka
```
## Network
```
networks: 
  kafka:
    name: kafka
```

# Membuild Image dan Container dengan Docker Compose

## Stop lalu Hapus semua container
```
sudo docker ps -a

sudo docker container stop 7c6c5495639c
```
## Build kembali image dan buat container, dan jalankan container
```
sudo docker-compose up -d
sudo docker-compose down
```
