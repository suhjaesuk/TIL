# Redis Cluster 기능

- 여러 노드에 자동적인 데이터 분산
- 일부 노드의 실패나 통신 단절에도 계속 작동하는 가용성
- 고성능을 보장하면서 선형 확장성을 제공

# Redis Cluster 특징

- full-mesh 구조로 통시
- cluster bu라는 추가 채널(port) 사용
- gossip protocol 사용
- hash slot을 사용한 키 관리
- DB0만 사용 가능
- multi key 명령어가 제한됨
- 클라이언트는 모든 노드에 접속
    
    ![11.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f06b9a97-02c8-4993-8679-535df2370c8e/11.png)
    

# Sentinel과의 차이점

- 클러스터는 데이터 분산(샤딩)을 제공함
- 클러스터는 자동 장애조치를 위한 모니터링 노드(Sentinel)를 추가 배치할 필요가 없음
- 클러스터에서는 multi key 오퍼레이션이 제한됨
- Sentinel은 비교적 단순하고 소규모의 시스템에서 고가용성이 필요할 때 채택

# 데이터를 분산하는 기준

- 특정 key의 데이터가 어느 노드(shard)에 속할 것인지 결정하는 메커니즘이 있어야 함
- 보통 분산 시스템에서 해싱이 사용됨
- 단순 해싱으로는 노드의 개수가 변할 때 모든 매핑이 새로 계산되어야 하는 문제가 있음

# Hash Slot을 이용한 데이터 분산

- Redis는 16384개의 hash slot으로 key 공간을 나누어 관리
- 각 키는 CRC16 해싱 후 16384로 modulo 연산을 해 각 hash slot에 매핑
- hash slot은 각 노드들에게 나누어 분배됨

# 클라이언트의 데이터 접근

- 클러스터 노드는 요청이 온 key에 해당하는 노드로 자동 redirect를 해주지 않음
- 클라이언트는 MOVED 에러를 받으면 해당 노드로 다시 요청해야 함

```yaml
(error) MOVED 16000 127.0.0.1:7005
```

# 클러스터를 사용할 때 성능

- 클라이언트가 MOVED 에러에 대해 재요청을 해야 하는 문제
    - 클라이언트(라이브러리)는 key-node 맵을 캐싱하므로 대부분의 경우 발생하지 않음
- 클라이언트는 단일 인스터스의 Redis를 이용하 때와 같은 성능으로 이용 가능
- 분산 시스템에서 성능은 데이터 일관성(Consistency)와 trade-off가 있음
    - Redis Cluster는 고성능의 확장성을 제공하면서 적절한 수준의 데이터 안정성과 가용성을 유지하는 것을 목표로 설계됨

# 클러스터의 데이터 일관성

- Redis Cluster는 strong consistency를 제공하지 않음
- 높은 성능을 위해 비동기 복제를 하기 때문
    - 성능을 위해 일관성을 희생

![223.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f61298b1-a4f1-4195-bf15-88baf3674143/223.png)

# 클러스터의 가용성

## auto failover

- 일부 노드(Master)가 실패(또는 네트워크 단절)하더라도 과반수 이상의 Master가 남아있고, 사진 Master의 replica들이 있다면 클러스터는 failover되어 가용한 상태가 된다.
- node timeout동안 과반수의 Master와 통신하지 못한 Master는 스스로 error state로 빠지고 write 요청을 받지 않음

## Replica Migration

- Replica가 다른 Master로 Migrate해서 가용성을 높인다.

# 클러스터의 제약사항

## DB0만 사용가능

- Redis는 한 인스턴스에 여러 데이터베이스를 가질 수 있으며 Defauly는 16
    - ex) databases 16
- Multi DB는 용도별로 분리해서 관리를 용이하게 하기 위한 목적
- 클러스터에서는 해당 기능을 사용할 수 없고 DB0으로 고정됨

## Multi key operation 사용 제약

- key들이 각각 다른 노드에 저장되므로 MSET과 같은 multi-key operation은 기본적으로 사용할 수 없다.
- 같은 노드 안에 속한 key들에 대해서는 multi-key operation이 가능
- hash tags 기능을 사용하면 여러 key들을 같은 hash slot에 속하게 할 수 있음
    - key 값 중 {} 안에 들어간 문자열에 대해서망 해싱을 수행하는 원리

```yaml
MSET {user:a}:age 20 {user:a}:city seoul
```

## 클라이언트 구현의 강제

- 클라이언트는 클러스터의 모든 노드에 접속해야 함
- 클라이언트는 redirect 기능을 구현해야 함 (MOVED 에러 대응)
- 클라이언트 구현이 잘 된 라이브러리가 없는 환경도 있을 수 있음

# 실습

## 클러스터 설정 파일

- cluster-enabled <yes/no>: 클러스터 모드로 실행할지 여부를 결정
- cluster-config-file <filename>: 해당 노드의 클러스터를 유지하기 위한 설정을 저장하는 파일로, 사용자가 수정하지 않음.
- cluster-node-timeout <milliseconds>
    - 특정 노드가 정상이 아닌 것으로 판단하는 기준 시간
    - 이 시간동안 감지되지 않는 master는 replica에 의해 failover가 이루어짐
- cluster-replica-validity-factor <factor>
    - master와 통신한지 오래된 replica가 failover를 수행하지 않게 하기 위한 설정
    - cluster-node-timeout 만큼 master와 통신이 없었던 replica는 failover 대상에서 제외된다.
- cluster-migration-barrier <count>:
    - 한 master가 유지해야 하는 최소 replica의 개수
    - 이 개수를 충족하는 선에서 일부 replica는 replica를 가지지 않은 master의 replica로 migrate 될 수 있다.
- cluster-require-full-coverage <yes/no>
    - 일부 hash slot이 커버되지 않을 때 write 요청을 받지 않을지 여부
    - no로 서정하게 되면 일부 노드에 장애가 생겨 해당 hash slot이 정상 작동하지 않더라도 나머지 hash slot에 대해서는 작동하도록 할 수 있다.
- cluster-allow-reads-when-down <yes/no>
    - 클러스터가 정상 상태가 아닐 때도 read 요청은 받도록 할지 여부
    - 어플리케이션에서 read 동작의 consistency가 중요치 않은 경우에 yes로 설정.

**.conf 파일 수정**

- 총 6개의 port를 활용하여 Master 3개, Replica 3개로 클러스터를 구성할 것이다.

```yaml
port 7000 ~ 7005
cluster-enabled yes
```

## **Cluser 구성**

- cluster create: 모든 노드 나열(ip, port 번호)
- cluster-replicas: 클러스터당 레플리카를 몇 개 가질 건지

```yaml
sudo redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
```

- Master 3개, Replica 3개로 클러스터가 구성되었다.

```yaml
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica localhost:7004 to 127.0.0.1:7000
Adding replica localhost:7005 to localhost:7002
Adding replica localhost:7003 to 127.0.0.1:7001
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 6b2a32d678cd4b11563ffb10a59f66106ebb7d78 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
M: 4de5f87ced8b35ab747318e555f9e1a0ebd9e0de 127.0.0.1:7001
   slots:[5461-16383] (5461 slots) master
M: 8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b 127.0.0.1:7002
   slots:[5461-16383] (5462 slots) master
S: ff2be0a009d89485cf0fe68a8dce805366841d60 127.0.0.1:7003
   replicates 4de5f87ced8b35ab747318e555f9e1a0ebd9e0de
S: cebfac516b520248b1d1248639c4cad88d908384 127.0.0.1:7004
   replicates 6b2a32d678cd4b11563ffb10a59f66106ebb7d78
S: 5d9dfeff4cfc7f865657c2360f20dc3da3f804bb 127.0.0.1:7005
   replicates 8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
```

- 정상적으로 만들어졌다면 아래처럼 나온다.

```yaml
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 6b2a32d678cd4b11563ffb10a59f66106ebb7d78 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 5d9dfeff4cfc7f865657c2360f20dc3da3f804bb 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b
M: 8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b 127.0.0.1:7002
   slots:[5461-16383] (10923 slots) master
   2 additional replica(s)
M: ff2be0a009d89485cf0fe68a8dce805366841d60 127.0.0.1:7003
   slots: (0 slots) master
S: cebfac516b520248b1d1248639c4cad88d908384 127.0.0.1:7004
   slots: (0 slots) slave
   replicates 6b2a32d678cd4b11563ffb10a59f66106ebb7d78
S: 4de5f87ced8b35ab747318e555f9e1a0ebd9e0de 127.0.0.1:7001
   slots: (0 slots) slave
   replicates 8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

- 클러스터가 구성 됬다면 redis-cli에 들어가서 cluster 정보 확인

```yaml
sudo redis-cli -p 7000
127.0.0.1:7000> cluster nodes
```

```yaml
5d9dfeff4cfc7f865657c2360f20dc3da3f804bb 127.0.0.1:7005@17005 slave 8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b 0 1689744929184 3 connected
8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b 127.0.0.1:7002@17002 master - 0 1689744927172 3 connected 5461-16383
ff2be0a009d89485cf0fe68a8dce805366841d60 127.0.0.1:7003@17003 master - 0 1689744931195 4 connected
6b2a32d678cd4b11563ffb10a59f66106ebb7d78 127.0.0.1:7000@17000 myself,master - 0 1689744928000 1 connected 0-5460
cebfac516b520248b1d1248639c4cad88d908384 127.0.0.1:7004@17004 slave 6b2a32d678cd4b11563ffb10a59f66106ebb7d78 0 1689744928178 1 connected
4de5f87ced8b35ab747318e555f9e1a0ebd9e0de 127.0.0.1:7001@17001 slave 8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b 0 1689744930000 3 connected
```

## 데이터 저장

- 7000번 포트에서 aa에 값 저장

```yaml
127.0.0.1:7000> set aa bb
OK
# 잘된다.
```

- 7000번 포트에서 aaa에 값 저장

```yaml
127.0.0.1:7000> set aaa dd
(error) MOVED 10439 127.0.0.1:7002
# 안된다. 7002번 포트로 이동하라 알려준다.
```

- 7002번에서 aaa에 값 저장

```yaml
127.0.0.1:7002> set aaa bb
OK
# 잘된다.
```

- 7002번 포트에서 aa에 값 저장

```yaml
127.0.0.1:7002> set aa bb
(error) MOVED 1180 127.0.0.1:7000
# 안된다. 7000번 포트로 이동하라 알려준다.
```

## 데이터 조회

- 7000번 포트에서 aa의 값 조회

```yaml
127.0.0.1:7000> get aa
"bb"
# 잘된다.
```

- 7000번 포트에서 aaa의 값 조회

```yaml
127.0.0.1:7000> get aaa
(error) MOVED 10439 127.0.0.1:7002
# 안된다. 7002번 포트로 이동하라 알려준다.
```

- 7002번 포트에서 aaa의 값 조회

```yaml
127.0.0.1:7002> get aaa
"bb"
# 잘된다.
```

- 7002번 포트에서 aa의 값 조회

```yaml
127.0.0.1:7000> get aa
(error) MOVED 10439 127.0.0.1:7000
```

- 7004번 포트(7000번 포트의 replica)에서 모든 키 조회

```yaml
127.0.0.1:7004> keys *
1) "aa"
```

- 7004번 포트(7000번 포트의 replica)에서 aa의값 조회

```yaml
127.0.0.1:7004> get aa
(error) MOVED 1180 127.0.0.1:7000
# 키 값을 조회 시 7000번 포트로 이동하라 한다.
# replica의 경우 readOnly 옵션을 설정하여 조회가 가능하다.
```

- 7004번 포트 readOnly 허용

```yaml
127.0.0.1:7004> readOnly
OK
```

- 7004번 포트에서 aa의값 조회

```yaml
127.0.0.1:7004> get aa
"bb"
```

## failover 테스트

- 7000번 포트 서버 종료

```yaml
# Redis is now ready to exit, bye bye...
```

- 클러스터 노드 정보 확인

```yaml
127.0.0.1:7001> cluster nodes
```

```yaml
4de5f87ced8b35ab747318e555f9e1a0ebd9e0de 127.0.0.1:7001@17001 myself,slave 8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b 0 1689746490000 3 connected
5d9dfeff4cfc7f865657c2360f20dc3da3f804bb 127.0.0.1:7005@17005 slave 8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b 0 1689746491713 3 connected
# 7004번이 마스터로 변경
cebfac516b520248b1d1248639c4cad88d908384 127.0.0.1:7004@17004 master - 0 1689746434396 7 connected 0 - 5460
8dae9b84ced14fba0ff8bfbcd9fb98c3ac10728b 127.0.0.1:7002@17002 master - 0 1689746492721 3 connected 5461-16383
ff2be0a009d89485cf0fe68a8dce805366841d60 127.0.0.1:7003@17003 master - 0 1689746493727 4 connected
6b2a32d678cd4b11563ffb10a59f66106ebb7d78 127.0.0.1:7000@17000 master,fail - 1689746434396 1689746431000 1 disconnected
```

## 새로운 노드 생성

- 7006번 포트 생성 후 클러스터 노드 추가
    - sudo redis-cli --cluster add-node `새로운 노드` `기존 연결된 노드`

```yaml
sudo redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7005

```

- 7007번 포트 생성 후 7006번에 slave로 지정
- sudo redis-cli --cluster add-node `새로운 노드` `기존 연결된 노드` —cluster-slave [대상 노드 지정 가능(없다면 앞에있는 노드가 지정)]

```yaml
sudo redis-cli --cluster add-node 127.0.0.1:7007 127.0.0.1:7006 --cluster-slave

```

## Spring에서 Cluster 사용 시 설정

**aplication.yml 설정**

```yaml
spring:
	redis:
		cluster:
			nodes: 127.0.0.1:7000, 127.0.0.1:7001 .... 
# 노드들의 ip와 port를 적어주면 된다.
```

설정만 해주면 나머지는 기존 Redis와 똑같이 사용하면 된다.