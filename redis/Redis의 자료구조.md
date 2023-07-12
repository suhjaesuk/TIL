# Strings

- 가장 기본적인 데이터 타입으로 제일 많이 사용됨
- 바이트 배열을 저장 (binary-safe)
- 바이너리로 변환활 수 있는 모든 데이터를 저장 가능 (JPG와 같은 파일 등)
- 최대 크기는 512MB

## 주요 명령어

| 명령어 | 기능 | 예제 |
| --- | --- | --- |
| SET | 특정 키에 문자열 값을 저장한다. | SET say hello |
| GET | 특정 키의 문자열 값을 얻어온다. | GET say |
| INCR | 특정 키의 값을 Integer로 취급하여 원자적으로 1 증가시킨다.  | INCR mycount |
| DECR | 특정 키의 값을 Integer로 취급하여 원자적으로 1 감소시킨다. | DECR mycount |
| MSET | 여러 키에 대한 값을 한번에 저장한다.  | MSET mine milk yous coffe |
| MGET | 여러 키에 대한 값을 한번에 얻어온다. | MGET mine yours |

## 실습

- key1의 값으로 hi 저장

```bash
> SET key1 hi
OK
```

- key1의 값 조회

```bash
> GET key1
"hi"
```

- mycount의 값으로 10 저장

```bash
> SET mycount 10
OK
```

- mycount 값 10 증가

```bash
> INCR mycount 10
20
```

- mycount 값 5 감소

```bash
> DECR mycount 5
15
```

- key1에 hi key2에 hello key3에 bye 추가

```bash
> MSET key1 hi key2 hello key3 bye
OK
```

- key1, key2, key3의 값 조회

```bash
> MGET key1 key2 key3
1) "hi"
2) "hello"
3) "bye"
```

# Lists

- Linked-list 형태의 자료구조 (인덱스 접근은 느리지만 데이터 추가/삭제가 빠름)
- Queue와 Stack으로 사용할 수 있음 (Deque)

## 주요 명령어

| 명령어 | 기능 | 예제 |
| --- | --- | --- |
| LPUSH | 리스트의 왼쪽(head)에 새로운 값을 추가한다. | LPUSH mylist apple |
| RPUSH | 리스트의 오른쪽(tail)에 새로운 값을 추가한다. | RPUSH mylist banana |
| LLEN | 리스트에 들어있는 아이템 개수를 반환한다. | LLEN mylist |
| LRANGE | 리스트의 특정 범위를 반환한다. | LRANGE 0 -1 |
| LPOP | 리스트의 왼쪽(head)에서 값을 삭제하고 반환한다. | LPOP mylist |
| RPOP | 리스트의 오른쪽(tail)에서 값을 삭제하고 반환한다. | RPOP mylist |

## 실습

- mylist의 왼쪽에 apple 추가

```bash
> LPUSH mylist apple
1
```

- mylist 오른쪽에 banana 추가

```bash
> LPUSH mylist banana
2
```

- mylist의 길이 조회

```bash
> LLEN mylist
2
```

- mylist의 왼쪽 기준 0번 인덱스 부터 -1번 인덱스(마지막) 까지 조회

```bash
> LRANGE mylist 0 -1
1) "banana"
2) "apple"
```

- mylist 왼쪽 요소를 제거하고 조회

```bash
> LPOP mylist
"banana"
```

- mylist 왼쪽 요소를 제거하고 조회

```bash
> LPOP mylist
"apple"
```

- mylist 왼쪽 요소를 제거하고 조회

```bash
> LPOP mylist
(nil)
```

# Sets

- 순서가 없는 유니크한 값의 집합
- 검색이 빠름
- 개별 접근을 위한 인덱스가 존재하지 않고, 집합 연산이 가능 (교집합, 합집합)

## 주요 명령어

| 명령어 | 기능 | 예제 |
| --- | --- | --- |
| SADD | Set에 데이터를 추가한다. | SADD myset apple |
| SREM | Set에서 데이터를 삭제한다. | SREM myset apple |
| SCARD | Set에 저장된 아이템 개수를 반환한다. | SCARD myset |
| SMEMBERS | Set에 저장된 아이템들을 반환한다. | SMEMBERS myset |
| SISMEMBER | 특정 값이 Set에 포함되어 있는지 반환한다. | SISMEMBER myset apple |

## 실습

- myset에 apple 추가

```bash
> SADD myset apple
1
```

- myset에 banana 추가

```bash
> SADD myset banana
1
```

- myset의 길이 조회

```bash
> SCARD myset
2
```

- myset의 멤버 전체 조회

```bash
> SMEMBERS myset
1) "apple"
2) "banana"
```

- myset에서 apple 제거

```bash
> SREM myset apple
1
```

- apple이 myset의 멤버인지 확인

```bash
> SISMEMBER myset apple
0
```

- banana가 myset의 멤버인지 확인

```bash
> SISMEMBER myset banana
1
```

# Hashes

- 하나의 key 하위에 여러개의 field-value 쌍을 저장
- 여러 필드를 가진 객체를 저장하는 것으로 생각할 수 있음
- HINCRBY 명렁어를 사용해 카운터로 활용 가능

## 주요 명령어

| 명령어 | 기능 | 예제 |
| --- | --- | --- |
| HSET | 한개 또는 다수의 필드에 값을 저장한다. | HSET user1 name bear age 10 |
| HGET | 특정 필드의 값을 반환한다. | HGET user1 name |
| HMGET | 한개 이상의 필드 값을 반환한다. | HMGET user1 name age |
| HINCRBY | 특정 필드의 값을 Integer로 취급하여 지정한 숫자를 증가시킨다. | HINCRBY user1 viewcount 1 |
| HDEL | 한개 이상의 필드를 삭제한다. | HDEL user1 name age |

## 실습

- user의 name 필드에 bear,  age 필드에 10 추가

```bash
> HSET user name bear age 10
2
```

- user의 name 필드의 값 조회

```bash
> HGET user name
"bear"
```

- user의 name, age 필드의 값 조회

```bash
> HMGET user name age
1) "bear"
2) "10"
```

- user의 viewcount 필드에 15 추가

```bash
> HSET user viewcount 15
1
```

- user의 viewcount 필드의 값 3 증가

```bash
> HINCRBY user viewcount 3
18
```

- user의 모든 필드 조회

```bash
> HKEYS user
1) "name"
2) "age"
3) "viewcount"
```

- user의 name, age 필드 삭제

```bash
> HDEL user name age
2
```

- user의 모든 필드 조회

```bash
> HKEYS user
1) "viewcount"
```

# Sorted Sets

- Set과 유사하게 유니크한 값의 집합
- 각 값은 연관된 score를 가지고 정렬되어 있음
- 정렬된 상태이기에 빠르게 최소/최대값을 구할 수 있음
- 순위 계산, 리더보드 구현 등에 활용

## 주요 명령어

| 명령어 | 기능 | 예제 |
| --- | --- | --- |
| ZADD | 한개 또는 다수의 값을 추가 또는 업데이트한다. | ZADD myrank 10 apple 20 banana |
| ZRANGE | 특정 범위의 값을 반환한다. (오름차순으로 정렬된 기준) | ZRANGE myrank 0 1 |
| ZRANK | 특정 값의 위치(순위)를 반환한다. (오름차순으로 정렬된 기준) | ZRANK myrank apple |
| ZREVRANK | 특정 값의 위치(순위)를 반환한다. (내림차순으로 정렬된 기준) | ZREVRANK myrank apple |
| ZREM | 한개 이상의 값을 삭제한다. | ZREM myrank apple |

## 실습

- myrank에 apple에 10점, banana에 20점, grape에 30점 추가

```bash
> ZADD myrank 10 apple 20 banana 30 grape
3
```

- myrank의 0번째 인덱스부터 2번째 인덱스 조회

```bash
> ZRANGE myrank 0 2
1) "apple"
2) "banana"
3) "grape"
```

- 오름차순 정렬된 myrank의 grape 순위 조회

```bash
> ZRANK myrank grape
2
```

- 내림차순 정렬된 myrank의 grape 순위 조회

```bash
> ZREVRANK myrank grape
0
```

- myrank에서 banana 삭제

```bash
> ZREM myrank banana
1
```

- myrank의 0번째 인덱스부터 2번째 인덱스 조회

```bash
> ZRANGE myrank 0 2
1) "apple"
2) "grape"
```

# Bitmaps

- 비트 벡터를 사용해 N개의 Set을 공간 효율적으로 저장
- 하나의 비트맵이 가지는 공간은 4,294,967,295 (2^32 - 1)
- 비트 연산이 가능

## 주요 명령어

| 명령어 | 기능 | 예제 |
| --- | --- | --- |
| SETBIT | 비트맵의 특정 오프셋에 값을 변경한다. | SETBIT visit 10 1 |
| GETBIT | 비트맵의 특정 오프셋의 값을 반환한다. | GETBIT visit 10 |
| BITCOUNT | 비트맵에서 set(1) 상태인 비트의 개수를 반환한다. | BITCOUNT visit |
| BITOP | 비트맵들간의 비트 연산을 수행하고 결과를 비트맵에 저장한다. | BITOP AND result today yesterday |

## 실습

- today_visit에 2번 비트를 활성화

```bash
> SETBIT today_visit 2 1
0
```

- today_visit에 3번 비트를 활성화

```bash
> SETBIT today_visit 3 1
0
```

- today_visit에 4번 비트를 활성화

```bash
> SETBIT today_visit 4 1
0
```

- today_visit에 활성화된 비트 수 반환

```bash
> BITCOUNT today_visit
3
```

- yesterday_visit에 1번 비트를 활성화

```bash
SETBIT yesterday_visit 1 1
0
```

- yesterday_visit에 4번 비트를 활성화

```bash
SETBIT yesterday_visit 4 1
0
```

- total_visit에 today_visit과 yesterday_visit의 교집합 저장

```bash
> BITOP AND total_visit today_visit yesterday_visit
1
```

- total_visit에 활성화된 비트 수 반환

```bash
> BITCOUNT total_visit
1
```

- total_visit에 4번 비트 값 반환

```bash
> GETBIT total_visit 4
1
```

- total_visit에 1번 비트 값 반환

```bash
> GETBIT total_visit 1
0
```

# HyperLogLog

- 유니크한 값의 개수를 효율적으로 얻을 수 있음
- 확률적 자료구조로서 오차가 있으며, 매우 큰 데이터를 다룰 때 사용
- 18,446,744,073,709,551,616(2^64)개의 유니크 값을 계산 가능
- 12KB까지 메모리를 사용하며 0.81%의 오차율을 허용

## 주요 명령어

| 명령어 | 기능 | 예제 |
| --- | --- | --- |
| PFADD | HyperLogLog에 값들을 추가한다. | PFADD visit Jay Peter Jane |
| PFCOUNT | HyperLogLog에 입력된 값들의 cardinality(유일값의 수)를 반환한다. | PFCOUNT visit |
| PFMERGE | 다수의 HyperLogLog를 병합한다. | PFMERGE result visit1 visit2 |

## 실습

- today_visit에 Jay, Peter, Jane 추가

```bash
> PFADD today_visit Jay Peter Jane
1
```

- today_visit에 입력된 값들의 수 조회

```bash
> PFCOUNT today_visit
3
```

- today_visit에 Jay 추가

```bash
> PFADD today_visit Jay
0
```

- today_visit에 입력된 값들의 수 조회

```bash
> PFCOUNT today_visit
3
```

- yesterday_visit에 Jay, Peter, John 추가

```bash
> PFADD yesterday_visit Jay Peter John
0
```

- total_visit에 today_visit과 yesterday_visit을 병합

```bash
> PFMERGE total_visit today_visit yesterday_visit
OK
```

- total_visit에 입력된 값들의 수 조회

```bash
> PFCOUNT total_visit
4
```