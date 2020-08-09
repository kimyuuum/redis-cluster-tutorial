# Redis-cluster-tutorial



## Redis (REmote Dictionary Server)?

 메모리 기반의 key-value 구조로, data를 관리하는 시스템이다.

read / write가 빠르다는 장점을 가지고 있으며, Memcached와 유사한 구조를 가지고 있다.

<br>
<br>

**Memchached의 특징**

- 처리 속도가 빠르다.
- Data는 Memory에만 저장된다.
- Cache이므로, 만료일을 지정하여 그 이후에는 자동으로 Data가 사라진다.
- LRU(Least Recently Used)알고리즘으로, 저장소 메모리를 재사용한다.

→ Redis는 Disk에도 저장되어 **데이터 복구가 가능**하고, 메모리를 재사용하지 않고 명시적으로 Data를 제거하여 사용한다.

또한, 다양한 자료구조 (String, List, Set, Sorted set, Hash)를 사용하여 Data를 저장 가능하다.

하지만 Key-Value라는 구조를 가지고 있기 때문에, 복잡한 Database 설계에는 적합하지 않다. 

→ **Database Caching**에 활용하자!

<br>
<br>

### Caching?

Data를 미리 읽어두었다가, 요청이 올 경우 빠르게 응답하기 위한 목적으로 사용 가능하다. 

Server의 불필요한 traffic을 줄일 수 있다.

→ WAS 부하 감소시키기 → 조회 등의 처리 성능을 확보하자!

**Cache의 대상**

1. 단순한 구조의 정보
2. 반복적으로 동일하게 요청되는 정보
3. 정보의 변경 주기는 빈번하지 않은 정보

에 사용 가능하다.

Cache Info, 유효 기간(Expire Date), Caching 정보의 갱신 시점에 유의하여 사용하자.

<br>

---
<br>
<br>
<br>

## 😎Quick Start Redis With Homebrew

- 해당 문서는 Mac 기반으로 작성되었습니다.
<br>

### 1. Install Homebrew

Homebrew는 맥에서 사용 가능한 패키지 관리자이다.

Page에 나와있는 Link를 Terminal에 복사하여 간단하게 설치 가능하다.

[Homebrew](https://brew.sh/index_ko)

```bash
$ brew --version
```

![image](https://user-images.githubusercontent.com/46887352/89724939-73950a00-da44-11ea-993e-44d145e5edba.png)

`$ brew --version` 을 통해 설치를 확인 할 수 있다.

<br>
<br>

### 2. Install & start Redis

**Homebrew로 Redis 설치하기**

```bash
$ brew install redis
```

```bash
$ brew services start redis #start redis
$ brew services restart redis #restart redis
$ brew services stop redis #stop redis
```

<br>
<br>

**Redis 실행**

```bash
$ redis-server
```

![image](https://user-images.githubusercontent.com/46887352/89724945-890a3400-da44-11ea-9583-f1cd41478a40.png)

위와 같이 실행 가능합니다.
<br>
<br>
<br>

## 👻Redis Cluster

### Cluster?

각각 다른 서버를 하나로 묶어, 하나의 시스템처럼 동작하게 한다.
→ Client에게 고가용성을 제공한다.

- 여러대의 서버에 Data가 분산되어 저장된다. (= 트래픽 분산)
- 특정 서버에 장애가 일어나더라도, 백업 서버가 보완을 통해 data의 유실 없이 서비스를 이어나갈 수 있다.

Redis Cluster는 내부적으로 slot을 가지고 있으며, 특정 알고리즘으로 key값을 변환하여 실제 데이터가 저장될 슬롯을 결정한다. 데이터를 저장하거나 가져올 때, 저장 슬롯을 가진 노드가 아닐 경우, 자동으로 redirect된다. cluster key가 해당 Key가 속한 노드의 서버로 이동시켜 주기 때문에, 데이터가 어느 서버에 저장되었는지 몰라도 된다.
<br>
<br>
###Master-Slave 구조의 Cluster 구성

만약 master로만 구성할 경우, 하나의 노드에 장애가 발생한다면 해당 노드의 데이터 유실이 발생하게 된다. → 이를 방지하기 위해 slave도 같이 구축하는게 좋다.

**Slave**

: slave를 추가로 구성하면, master의 내용을 복제하여 가진다. master가 장애가 날 경우, 연결되어 있는 slave가 master로 승격하게 된다. 그렇기 때문에 **하나의 서버에 master-slave가 동시에 존재하면 안된다.**

<br>
<br>

### Redis Cluster Configuration Parameters

`port` : port number

`cluster-enabled <yes/no>` : cluster 여부

`cluster-config-file <filename>` : cluster 구성에 변동이 있을 때 마다 Redis cluster 노드가 자동적으로 cluster의 구성을 유지한다. 사용자가 직접 편집할 수 없다.

`cluster-node-timeout <milliseconds>` : 일정 시간 master node가 응답이 없으면, slave node가 fail over를 진행한다.

Cluster 최소 구성 노드 개수 : 3개

- 3 master / 3 slave
<br>
<br>
<br>

## 🏃🏻‍♂️ Creating and using a Redis Cluster

### Redis Cluster config file

redis 설정은 user의 redis 경로로 접근한 뒤, config file을 수정한다.

```bash
$ cd /usr/local/etc
$ vi redis.conf
```

최소 Cluster 구성을 위해 6개의 노드 (3 master, 3 slave)를 생성하고, port 지정은 임의로 7000 ~ 7005로 바인딩을 하였다.

```bash
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

기본 설정으로는 cluster 옵션이 **disabled** 되어있다. **nodes.conf**는 사용자가 설정하지 않고, redis cluster 인스턴스가 시작할 때 생성되고, 필요한 경우 자동으로 업데이트 된다.

위와 같은 방식으로 최소 option을 설정하고, file을 복사해서 각각 별도의 파일로 생성한다. 

```bash
$ cp redis.conf redis-cluster-1.conf

port 7001
cluster-enabled yes
cluster-config-file nodes-7001.conf
cluster-node-timeout 5000
pidfile /var/run/redis_7001.pid
dbfilename dump-cluster01.rdb
...
```
<br>
<br>

### Instance 실행하기

각각의 redis.conf file을 Redis 서버로 실행한다.

```bash
$ redis-server ./redis-cluster-1.conf
```
<br>
<br>

### Cluster 구성하기

Redis Version 5 이상을 사용하는 경우, **redis-cli**를 통하여 새 클러스터를 구성하거나, 기존 클러스터의 정보를 확인 가능하다. 3 또는 4 버전일 경우, **redis-trib.rb**를 사용한다. ( redis-cli를 사용하여 구성하였습니다.)

redis gem을 설치한 뒤 redis-cli를 사용하자. 

```bash
$ gem install redis
```

`--cluster-replicas N` 으로 master의 slave 개수를 정한다. 생략 또는 0으로 설정하면 slave가 생성되지 않는다. replicas = 1일 경우, 총 6개의 노드가 필요하다.

```bash
$ redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
```
<br>
redis-trib.rb를 사용하는 경우

```bash
./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 \ 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```
<br>
<br>

**Can I set the above configuration? (type 'yes' to accept)** 문구가 떴다면 **yes**를 입력한다.
<br>
<br>

### Cluster Info 조회하기

```bash
$ redis-cli --cluster info ip:port
```
<br>
<br>
위 명령어를 이용하여 cluster 정보를 조회 가능하다.

![image](https://user-images.githubusercontent.com/46887352/89724947-91626f00-da44-11ea-9210-2ce2ee759ba3.png)
<br>
<br>
<br>

 Ref.

[Redis cluster tutorial - Redis](https://redis.io/topics/cluster-tutorial)

[redis redis-cli cluster](http://redisgate.kr/redis/cluster/redis-cli-cluster.php#info)
