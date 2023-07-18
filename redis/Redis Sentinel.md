# Redis Sentinel

- Redis에서 고가용성을 제공하기 위한 장치
- Master-Replica 구조에서 Master가 다운 시 Replica를 Master로 승격시키는 Auto-Failover를 수행
- Sentinel 기능
    - 모니터링
    - 알림
    - 자동 장애 복구
    - 환경 설정 제공자

![1111](https://github.com/suhjaesuk/til/assets/110963294/a74ecf92-f817-4325-9e71-a1291139c00a)


# Redis Sentinel 구상도

- Sentinel 노드는 3개 이상으로 구성 (Quorum 때문)
- Sentinel들은 서로 연결되어 있음
- Sentinel들은 Master와 Replica를 모니터링
- Client는 Sentinel을 통해 Redis에 접근

![2222](https://github.com/suhjaesuk/til/assets/110963294/9023c3b8-6202-4142-8d41-f3a3bf1906ab)


# Sentinel의 특징

- SDOWN(Subjective Down)과 ODOWN(Objective Down)의 2가지 판단이 있음
    - SDOWN: Sentinel 1대가 down으로 판단 (주관적)
    - ODOWN: 정족수가 충족되어 down으로 판단 (객관적)
- Master 노드가 dow된걸로 판단되기 위해서는 Sentinel 노드들이 정족수(Quorum)을 충족해야 함
- 클라이언트는 Sentinel을 통해 Master의 주소를 얻어내야 함

# 실습

**Docker-Compose를 이용한 Sentinel 구성**

```powershell
version: '2'

services:

  # master : bitnami/redis:6.2.6
  redis-master:
    hostname: redis-master
    container_name: redis-master
    image: bitnami/redis:6.2.6
    environment:
      - REDIS_REPLICATION_MODE=master
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6379:6379

  # slave1 : bitnami/redis:6.2.6
  redis-slave-1:
    hostname: redis-slave-1
    container_name: redis-slave-1
    image: bitnami/redis:6.2.6
    environment:
      - REDIS_REPLICATION_MODE=slave
      - REDIS_MASTER_HOST=redis-master
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6480:6379
    depends_on:
      - redis-master

  # slave2 : bitnami/redis:6.2.6
  redis-slave-2:
    hostname: redis-slave-2
    container_name: redis-slave-2
    image: bitnami/redis:6.2.6
    environment:
      - REDIS_REPLICATION_MODE=slave
      - REDIS_MASTER_HOST=redis-master
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6481:6379
    depends_on:
      - redis-master
      - redis-slave-1

  # sentinel1 : bitnami/redis-sentinel:6.2.6
  redis-sentinel-1:
    hostname: redis-sentinel-1
    container_name: redis-sentinel-1
    image: bitnami/redis-sentinel:6.2.6
    environment:
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT_NUMBER=6379
      - REDIS_MASTER_SET=master-name
      - REDIS_SENTINEL_QUORUM=2
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    ports:
      - 26379:26379

  # sentinel2 : bitnami/redis-sentinel:6.2.6
  redis-sentinel-2:
    hostname: redis-sentinel-2
    container_name: redis-sentinel-2
    image: bitnami/redis-sentinel:6.2.6
    environment:
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT_NUMBER=6379
      - REDIS_MASTER_SET=master-name
      - REDIS_SENTINEL_QUORUM=2
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    ports:
      - 26380:26379

  # sentinel3 : bitnami/redis-sentinel:6.2.6
  redis-sentinel-3:
    hostname: redis-sentinel-3
    container_name: redis-sentinel-3
    image: bitnami/redis-sentinel:6.2.6
    environment:
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT_NUMBER=6379
      - REDIS_MASTER_SET=master-name
      - REDIS_SENTINEL_QUORUM=2
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    ports:
      - 26381:26379
```

**Docker-Compose 빌드**

```powershell
PS C:\Users\sivae> docker-compose up --build
[+] Running 26/26
 - redis-slave-2 Pulled                                                                                           11.9s
 - redis-sentinel-1 Pulled                                                                                         7.7s
   - 6e09ff902305 Pull complete                                                                                    2.6s
   - e0f029e85c11 Pull complete                                                                                    2.6s
   - 3daa34cb9f3b Pull complete                                                                                    2.8s
   - 30c13161188f Pull complete                                                                                    2.9s
   - 9bf87270a98c Pull complete                                                                                    2.9s
   - a9d16cc52579 Pull complete                                                                                    3.0s
   - f8a84de1d169 Pull complete                                                                                    3.4s
   - 89c9e60c3b36 Pull complete                                                                                    3.5s
   - 06c5a6b1611c Pull complete                                                                                    4.2s
 - redis-master Pulled                                                                                            11.9s
   - 90aaa860080d Pull complete                                                                                    3.7s
   - 514a72873fa6 Pull complete                                                                                    5.0s
   - ad9a4d05a144 Pull complete                                                                                    5.1s
   - bbfcb0182456 Pull complete                                                                                    5.2s
   - a027d31a7fe1 Pull complete                                                                                    6.3s
   - 5596c760a78c Pull complete                                                                                    6.3s
   - 5debfb7b4154 Pull complete                                                                                    6.4s
   - 307572bb2ea3 Pull complete                                                                                    6.5s
   - f4f5d21ff456 Pull complete                                                                                    6.6s
   - bf7bffe206cd Pull complete                                                                                    7.3s
   - 468c7ee54379 Pull complete                                                                                    7.8s
 - redis-slave-1 Pulled                                                                                           11.9s
 - redis-sentinel-3 Pulled                                                                                         7.7s
 - redis-sentinel-2 Pulled                                                                                         7.7s
[+] Running 7/7
 - Network sivae_default       Created                                                                             0.8s
 - Container redis-master      Created                                                                             0.1s
 - Container redis-slave-1     Created                                                                             0.1s
 - Container redis-slave-2     Created                                                                             0.1s
 - Container redis-sentinel-1  Created                                                                             0.1s
 - Container redis-sentinel-2  Created                                                                             0.1s
 - Container redis-sentinel-3  Created                                                                             0.1s
Attaching to redis-master, redis-sentinel-1, redis-sentinel-2, redis-sentinel-3, redis-slave-1, redis-slave-2
```

**Sentinel 접속**

```powershell
$ winpty docker exec -it redis-sentinel-1 sh
```

```powershell
$ redis-cli -p 26379 #Centinel port로 접속해야 함
```

**Sentinel 정보 조회**

```powershell
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=master-name,status=ok,address=192.168.144.2:6379,slaves=2,sentinels
=3
```

- Master 1개, Slave 2개
- Master IP는 192.168.144.2

**Master IP 확인 (Sentinel Master IP와 Master로 지정한 IP가 같은지 확인)**

```powershell
$ docker inspect redis-master
[
    {
        "Id": "0426a47f6f1c404943af919966f84335fbe25d3d6a4cb1a3e70c06a2c839490a",
        "Created": "2023-07-18T07:23:06.823450034Z",
        "Path": "/opt/bitnami/scripts/redis/entrypoint.sh",
				# ... 생략
				"IPAddress": "192.168.144.2", 
				# ... 생략
				}
      }
	  }
  }
]
```

**FailOver 테스트**

1. **Master 중단**

```powershell
$ docker stop redis-master
```

2. Quorom 정족수가 차면 ODOWN 실행

```powershell
redis-sentinel-2  | 1:X 18 Jul 2023 07:44:16.500 # +sdown master master-name 192.168.144.2 6379
redis-sentinel-3  | 1:X 18 Jul 2023 07:44:16.536 # +sdown master master-name 192.168.144.2 6379
redis-sentinel-1  | 1:X 18 Jul 2023 07:44:16.543 # +sdown master master-name 192.168.144.2 6379
redis-sentinel-3  | 1:X 18 Jul 2023 07:44:16.599 # +odown master master-name 192.168.144.2 6379 #quorum 2/2
```

3. redis-slave-1이 Master로 승격

```powershell
redis-slave-1     | 1:M 18 Jul 2023 07:55:50.521 * MASTER MODE enabled (user request from 'id=4 addr=192.168.176.5:39727 laddr=192.168.176.3:6379 fd=8 name=sentinel-451a2fa0-cmd age=21 idle=0 flags=x db=0 sub=0 psub=0 multi=4 qbuf=6052 qbuf-free=34902 argv-mem=4 obl=13839 oll=0 omem=0 tot-mem=61468 events=r cmd=exec user=default redir=-1')
```

4. redis-slave-2가 redis-slave-1을 Master로 인식

```powershell
redis-slave-2     | 1:S 18 Jul 2023 07:55:50.743 * REPLICAOF 192.168.176.3:6379 enabled (user request from 'id=10 addr=192.168.176.5:48901 laddr=192.168.176.4:6379 fd=9 name=sentinel-451a2fa0-cmd age=0 idle=0 flags=x db=0 sub=0 psub=0 multi=4 qbuf=7624 qbuf-free=33330 argv-mem=4 obl=13714 oll=10 omem=205040 tot-mem=266508 events=r cmd=exec user=default redir=-1')
redis-slave-2     | 1:S 18 Jul 2023 07:55:50.749 # CONFIG REWRITE executed with success.
redis-slave-2     | 1:S 18 Jul 2023 07:55:50.749 * Non blocking connect for SYNC fired the event.
redis-slave-2     | 1:S 18 Jul 2023 07:55:50.749 * Master replied to PING, replication can continue...
redis-slave-2     | 1:S 18 Jul 2023 07:55:50.750 * Trying a partial resynchronization (request 3b642559ab2d34f778fc678f487a8a9f0c224d7d:1910).
redis-slave-2     | 1:S 18 Jul 2023 07:55:50.750 * Successful partial resynchronization with master.
redis-slave-2     | 1:S 18 Jul 2023 07:55:50.750 # Master replication ID changed to 45dd26020ba0cafa14687eed02f965e3dd91aac6
redis-slave-2     | 1:S 18 Jul 2023 07:55:50.750 * MASTER <-> REPLICA sync: Master accepted a Partial Resynchronization.
```

5. redis-slave-1에서 쓰기 작업 시 정상

```powershell
127.0.0.1:6379> set a a
OK
```

6. redis-slave-2에서 쓰기 작업 시 오류

```powershell
127.0.0.1:6379> set a a
(error) READONLY You can't write against a read only replica.
```

7. redis-slave-2에서 모든 key 조회

```powershell
127.0.0.1:6379> keys *
1) "a"
```

8. redis-master 재실행 

```powershell
docker start redis-master
```

9. redis-master 쓰기 작업 시 오류

```powershell
127.0.0.1:6379> set b b
(error) READONLY You can't write against a read only replica.
```

**Spring에서 Sentinel 사용하기**

**application.yml (센티널 정보만 입력해주면 된다.)**

```yaml
spring:
  redis:
    sentinel:
      master: master-name
			# 모든 센티널 노드들
      nodes: 127.0.0.1:26379, 127.0.0.1:26380, 127.0.0.1:26381
```
