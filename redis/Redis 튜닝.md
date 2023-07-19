# Redis 성능 측정

- `redis-benchmark` 유틸리티를 이용해 Redis의 성능을 측정할 수 있음.
    - `redis-benchmark [-h host] [-p port] [-c clients] [-n requests] [-t test]`

```yaml
redis-benchmark -c 100 -n 100 -t SET
# 100개의 클라이언트가 100개의 SET 요청 실행 
```

```yaml
====== SET ======
	# 100개의 요청이 0.00초만에 완료
  100 requests completed in 0.00 seconds
  100 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 3600 1 300 100 60 10000
  host configuration "appendonly": no
  multi-thread: no

Latency by percentile distribution:
0.000% <= 0.831 milliseconds (cumulative count 1)
50.000% <= 1.239 milliseconds (cumulative count 51)
75.000% <= 1.383 milliseconds (cumulative count 75)
87.500% <= 1.463 milliseconds (cumulative count 88)
93.750% <= 1.495 milliseconds (cumulative count 94)
96.875% <= 1.519 milliseconds (cumulative count 97)
98.438% <= 1.543 milliseconds (cumulative count 100)
100.000% <= 1.543 milliseconds (cumulative count 100)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 0)
6.000% <= 0.903 milliseconds (cumulative count 6)
21.000% <= 1.007 milliseconds (cumulative count 21)
36.000% <= 1.103 milliseconds (cumulative count 36)
46.000% <= 1.207 milliseconds (cumulative count 46)
60.000% <= 1.303 milliseconds (cumulative count 60)
79.000% <= 1.407 milliseconds (cumulative count 79)
95.000% <= 1.503 milliseconds (cumulative count 95)
100.000% <= 1.607 milliseconds (cumulative count 100)

Summary:
  throughput summary: 50000.00 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        1.210     0.824     1.239     1.503     1.543     1.543

# 99% 확률로 1.5초 걸림
```

# 성능에 영향을 미치는 요소들

- Network bandwidth & latency: Redis의 throughput은 주로 network에 의해 결정되는 경우가 많음.
    - 운영 환경에 런치하기 전에 배포 환경의 network 대역폭과 실제 throughput을 체크하는 것이 좋음
- CPU: 싱글 스레드로 동작하는 Redis 특성 상 CPU 성능이 중요.
    - 코어 수보다는 큰 cache를 가진 빠른 CPU가 선호됨
- RAM 속도 & 대역폭: 10KB 이하 데이터 항목들에 대해서는 큰 영향이 없음
- 가상화 환경의 영향: VM에서 실행되는 경우 개별적인 영향이 있을 수 있음
    - non-local disk, 오래된 hypervisor의 느린 fork 구현 등

# 성능에 영향을 미치는 설정들

- `rdbcompression <**yes**/no>`: RDB 파일을 압축할지 여부로, CPU를 절약하고 싶은 경우 no 선택
- `rdbchecksum <**yes**/no>`: 사용 시 RDB의 안정성을 높일 수 있으나 파일 저장/로드 시에 10% 정도의 성능 저하가 있음
- `save <seconds> <changes>`: RDB 파일 생성 시 시스템 자원이 소모되므로 성능에 영향이 있음

# 쿼리 튜닝

## SLOWLOG 설정

- 수행시간이 설정한 기준 시간 이상인 쿼리의 로그를 보여줌
- 측정 기준인 수행시간은 I/O 동작을 제외함

**로깅되는 기준 시간(microseconds)**

```yaml
slowlog-log-slower-than 10000
```

**로그 최대 길이**

```yaml
slowlog-max-len 128
```

## SLOWLOG 명령어

**slowlog 개수 확인**

```yaml
slowlog len
```

**slowlog 조회**

```yaml
slowlog get [count]
# 일련번호, 시간, 소요시간, 명령어, 클라이언트 IP, 클라이언트 이름 순으로 조회
```

- **예시**

```yaml
slowlog get 1
1) 1) (integer) 10 # 일련번호
	 2) (integer) 1672082741 # 쿼리를 수행한 시간
   3) (integer) 5 # 쿼리를 수행하는데 걸린 시간
	 4) 1) "set"         # 명령어
			2) "aaa"
			3) "123"
	 5) "172.17.0.1:59964" # 클라이언트 IP
	 6) "" # 클라이언트 이름
```

**slowlog 초기화**

```yaml
slowlog reset
```