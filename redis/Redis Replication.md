# Replication

- 백업만으로는 장애 대비에 부족함 (백업 실패 가능성, 복구에 소요되는 시간)
- Redis도 복제를 통해 가용성을 확보하고 빠른 장애조치가 가능
- master가 죽었을 경우 replica 중 하나를 master로 전환해 즉시 서비스 정상화 가능
- 복제본(replica)는 read-only 노드로 사용 가능하므로 traffic 분산도 가능

![1](https://github.com/suhjaesuk/til/assets/110963294/56da0225-1c3c-4116-9141-3cbd49e008a3)


# Redis Replication 사용

- Replica 노드에서만 설정을 적용해 master-replica 복제 구성이 가능

**Replica로 동작하도록 설정**

```bash
replicaof 127.0.0.1 6379
```

**Replica는 read-only로 설정**

```bash
replica-read-only
```

- Master 노드에는 RDB나 AOF를 이용한 백업 기능 활성화가 필수
    - 재시작 후에 비어있는 데이터 상태가 복제되지 않도록

# 실습

**Master 노드 생성 (5000번 포트에 생성 , 6379포트가 엑세스 할 수 있도록 설정)**

```bash
$ docker run --name my-redis-master -p 5000:6379 redis
```

**Replica 생성을 위한 redis.conf 설정** 

- [redis.conf 링크](https://raw.githubusercontent.com/redis/redis/7.0/redis.conf)

```bash
replicaof 127.0.0.1 5000
# 마스터(port:5000)를 replica
```

**Replica 생성 (powershell 이용)**

- **redis.conf 파일이 C:\Users\sivae에 담겨있음**

```bash
PS C:\Users\sivae> docker run --network host -v ${pwd}/redis.conf:/redis.conf --name my-redis-replica redis redis-server /redis.conf
1:C 18 Jul 2023 03:46:14.659 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 18 Jul 2023 03:46:14.659 # Redis version=7.0.12, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 18 Jul 2023 03:46:14.659 # Configuration loaded
1:S 18 Jul 2023 03:46:14.660 * monotonic clock: POSIX clock_gettime
1:S 18 Jul 2023 03:46:14.660 * Running mode=standalone, port=6379.
1:S 18 Jul 2023 03:46:14.660 # Server initialized
1:S 18 Jul 2023 03:46:14.660 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:S 18 Jul 2023 03:46:14.661 * Ready to accept connections
# MASTER와 연결 중...
1:S 18 Jul 2023 03:46:14.662 * Connecting to MASTER 127.0.0.1:5000
1:S 18 Jul 2023 03:46:14.663 * MASTER <-> REPLICA sync started
1:S 18 Jul 2023 03:46:14.663 * Non blocking connect for SYNC fired the event.
1:S 18 Jul 2023 03:46:14.663 * Master replied to PING, replication can continue...
1:S 18 Jul 2023 03:46:14.663 * Partial resynchronization not possible (no cached master)
1:S 18 Jul 2023 03:46:19.712 * Full resync from master: e7e1d193709284aeb94096f9d9b2536ff47c90cb:14
# 이전 데이터를 불러옴
1:S 18 Jul 2023 03:46:19.715 * MASTER <-> REPLICA sync: receiving streamed RDB from master with EOF to disk
1:S 18 Jul 2023 03:46:19.715 * MASTER <-> REPLICA sync: Flushing old data
1:S 18 Jul 2023 03:46:19.715 * MASTER <-> REPLICA sync: Loading DB in memory
1:S 18 Jul 2023 03:46:19.722 * Loading RDB produced by version 7.0.12
1:S 18 Jul 2023 03:46:19.722 * RDB age 0 seconds
1:S 18 Jul 2023 03:46:19.722 * RDB memory usage when created 0.92 Mb
1:S 18 Jul 2023 03:46:19.722 * Done loading RDB, keys loaded: 0, keys expired: 0.
1:S 18 Jul 2023 03:46:19.722 * MASTER <-> REPLICA sync: Finished with success
```

**Master에서 쓰기 작업 시 Replica에 복사**

1. **Master 쓰기**

```bash
127.0.0.1:6379> set a a
OK
```

1. **Replica 키 값 조회**

```bash
127.0.0.1:6379> keys *
1) "a"
```

**Replica에서 쓰기 작업 시 쓰기 안됨. (readOnly)**

```bash
127.0.0.1:6379> set a a
(error) READONLY You can't write against a read only replica.
```

**Replica 중단 / Master 쓰기 후 Replica 재시작**

1. **Replica 중단**
2. **Master 쓰기 작업**

```bash
127.0.0.1:6379> set b b
OK
```

1. **Replica 재시작 및 키 확인**

```bash
127.0.0.1:6379> keys *
1) "b"
2) "a"
```

# Docker-Compose로 쉽게 Replication 구성하기

# Docker-Compose

- 여러 개의 컨테이너로 구성된 어플리케이션을 정의하고 실행할 수 있는 도구
- YAML 파일을 통해 설정

# 실습

**Docker Compose 작성**

```powershell
PS C:\Users\sivae> code docker-compose.yml
```

```bash
version: "3"
services:
  my-redis-a:
    hostname: redis-master
    container_name: redis-master
    image: "bitnami/redis"
    environment:
      - REDIS_REPLICATION_MODE=master
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 7000:6379
  my-redis-b:
    hostname: redis-replicas-1
    container_name: redis-replicas-1
    image: "bitnami/redis"
    environment:
      - REDIS_REPLICATION_MODE=slave
      - REDIS_MASTER_HOST=redis-master
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 7001:6379
    depends_on:
      - my-redis-a
```

**bitnami/redis 이미지 다운로드**

```bash
PS C:\Users\sivae> docker pull bitnami/redis
Using default tag: latest
latest: Pulling from bitnami/redis
f36608bf2eed: Pull complete
Digest: sha256:f167ac4ed65b0761317f62b792bc0dc0cf0c1a41dc418608751e74e98a971159
Status: Downloaded newer image for bitnami/redis:latest
docker.io/bitnami/redis:latest
```

**docker-compose 빌드**

```powershell
PS C:\Users\sivae> docker-compose up --build
[+] Running 3/2
 - Network sivae_default       Created                                                                             0.7s
 - Container redis-master      Created                                                                             0.1s
 - Container redis-replicas-1  Created                                                                             0.1s
Attaching to redis-master, redis-replicas-1
redis-master      | redis 04:10:21.32
redis-master      | redis 04:10:21.32 Welcome to the Bitnami redis container
redis-master      | redis 04:10:21.33 Subscribe to project updates by watching https://github.com/bitnami/containers
redis-master      | redis 04:10:21.33 Submit issues and feature requests at https://github.com/bitnami/containers/issues
redis-master      | redis 04:10:21.33
redis-master      | redis 04:10:21.33 INFO  ==> ** Starting Redis setup **
redis-master      | redis 04:10:21.35 WARN  ==> You set the environment variable ALLOW_EMPTY_PASSWORD=yes. For safety reasons, do not use this flag in a production environment.
redis-master      | redis 04:10:21.35 INFO  ==> Initializing Redis
redis-master      | redis 04:10:21.36 INFO  ==> Setting Redis config file
redis-master      | redis 04:10:21.38 INFO  ==> Configuring replication mode
redis-master      |
redis-master      | redis 04:10:21.39 INFO  ==> ** Redis setup finished! **
redis-master      | redis 04:10:21.40 INFO  ==> ** Starting Redis **
redis-master      | 1:C 18 Jul 2023 04:10:21.413 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-master      | 1:C 18 Jul 2023 04:10:21.413 # Redis version=7.0.12, bits=64, commit=00000000, modified=0, pid=1, just started
redis-master      | 1:C 18 Jul 2023 04:10:21.413 # Configuration loaded
redis-master      | 1:M 18 Jul 2023 04:10:21.414 * monotonic clock: POSIX clock_gettime
redis-master      | 1:M 18 Jul 2023 04:10:21.414 * Running mode=standalone, port=6379.
redis-master      | 1:M 18 Jul 2023 04:10:21.414 # Server initialized
redis-master      | 1:M 18 Jul 2023 04:10:21.414 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis-master      | 1:M 18 Jul 2023 04:10:21.418 * Creating AOF base file appendonly.aof.1.base.rdb on server start
redis-master      | 1:M 18 Jul 2023 04:10:21.423 * Creating AOF incr file appendonly.aof.1.incr.aof on server start
redis-master      | 1:M 18 Jul 2023 04:10:21.423 * Ready to accept connections
redis-replicas-1  | redis 04:10:22.38
redis-replicas-1  | redis 04:10:22.38 Welcome to the Bitnami redis container
redis-replicas-1  | redis 04:10:22.38 Subscribe to project updates by watching https://github.com/bitnami/containers
redis-replicas-1  | redis 04:10:22.38 Submit issues and feature requests at https://github.com/bitnami/containers/issues
redis-replicas-1  | redis 04:10:22.38
redis-replicas-1  | redis 04:10:22.38 INFO  ==> ** Starting Redis setup **
redis-replicas-1  | redis 04:10:22.39 WARN  ==> You set the environment variable ALLOW_EMPTY_PASSWORD=yes. For safety reasons, do not use this flag in a production environment.
redis-replicas-1  | redis 04:10:22.40 INFO  ==> Initializing Redis
redis-replicas-1  | redis 04:10:22.40 INFO  ==> Setting Redis config file
redis-replicas-1  | redis 04:10:22.42 INFO  ==> Configuring replication mode
redis-replicas-1  |
redis-replicas-1  | redis 04:10:22.44 INFO  ==> ** Redis setup finished! **
redis-replicas-1  | redis 04:10:22.45 INFO  ==> ** Starting Redis **
redis-replicas-1  | 1:C 18 Jul 2023 04:10:22.460 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-replicas-1  | 1:C 18 Jul 2023 04:10:22.460 # Redis version=7.0.12, bits=64, commit=00000000, modified=0, pid=1, just started
redis-replicas-1  | 1:C 18 Jul 2023 04:10:22.460 # Configuration loaded
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.460 * monotonic clock: POSIX clock_gettime
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.461 * Running mode=standalone, port=6379.
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.461 # Server initialized
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.461 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.465 * Creating AOF base file appendonly.aof.1.base.rdb on server start
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.469 * Creating AOF incr file appendonly.aof.1.incr.aof on server start
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.469 * Ready to accept connections
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.469 * Connecting to MASTER redis-master:6379
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.470 * MASTER <-> REPLICA sync started
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.470 * Non blocking connect for SYNC fired the event.
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.470 * Master replied to PING, replication can continue...
redis-replicas-1  | 1:S 18 Jul 2023 04:10:22.470 * Partial resynchronization not possible (no cached master)
redis-master      | 1:M 18 Jul 2023 04:10:22.470 * Replica 172.21.0.3:6379 asks for synchronization
redis-master      | 1:M 18 Jul 2023 04:10:22.470 * Full resync requested by replica 172.21.0.3:6379
redis-master      | 1:M 18 Jul 2023 04:10:22.470 * Replication backlog created, my new replication IDs are '73ce249b2f5778c6d2ff7b2373b0efce2c62b3e0' and '0000000000000000000000000000000000000000'
redis-master      | 1:M 18 Jul 2023 04:10:22.470 * Delay next BGSAVE for diskless SYNC
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.448 * Full resync from master: 73ce249b2f5778c6d2ff7b2373b0efce2c62b3e0:0
redis-master      | 1:M 18 Jul 2023 04:10:27.448 * Starting BGSAVE for SYNC with target: replicas sockets
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.450 * MASTER <-> REPLICA sync: receiving streamed RDB from master with EOF to disk
redis-master      | 1:M 18 Jul 2023 04:10:27.449 * Background RDB transfer started by pid 49
redis-master      | 49:C 18 Jul 2023 04:10:27.451 * Fork CoW for RDB: current 0 MB, peak 0 MB, average 0 MB
redis-master      | 1:M 18 Jul 2023 04:10:27.451 # Diskless rdb transfer, done reading from pipe, 1 replicas still up.
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.453 * MASTER <-> REPLICA sync: Flushing old data
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.453 * MASTER <-> REPLICA sync: Loading DB in memory
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.461 * Loading RDB produced by version 7.0.12
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.461 * RDB age 0 seconds
redis-master      | 1:M 18 Jul 2023 04:10:27.462 * Background RDB transfer terminated with success
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.461 * RDB memory usage when created 0.90 Mb
redis-master      | 1:M 18 Jul 2023 04:10:27.462 * Streamed RDB transfer with replica 172.21.0.3:6379 succeeded (socket). Waiting for REPLCONF ACK from slave to enable streaming
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.461 * Done loading RDB, keys loaded: 0, keys expired: 0.
redis-master      | 1:M 18 Jul 2023 04:10:27.462 * Synchronization with replica 172.21.0.3:6379 succeeded
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.461 * MASTER <-> REPLICA sync: Finished with success
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.461 * Creating AOF incr file temp-appendonly.aof.incr on background rewrite
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.461 * Background append only file rewriting started by pid 62
redis-replicas-1  | 62:C 18 Jul 2023 04:10:27.465 * Successfully created the temporary AOF base file temp-rewriteaof-bg-62.aof
redis-replicas-1  | 62:C 18 Jul 2023 04:10:27.465 * Fork CoW for AOF rewrite: current 0 MB, peak 0 MB, average 0 MB
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.494 * Background AOF rewrite terminated with success
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.494 * Successfully renamed the temporary AOF base file temp-rewriteaof-bg-62.aof into appendonly.aof.2.base.rdb
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.494 * Successfully renamed the temporary AOF incr file temp-appendonly.aof.incr into appendonly.aof.2.incr.aof
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.500 * Removing the history file appendonly.aof.1.incr.aof in the background
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.500 * Removing the history file appendonly.aof.1.base.rdb in the background
redis-replicas-1  | 1:S 18 Jul 2023 04:10:27.505 * Background AOF rewrite finished successfully
```
