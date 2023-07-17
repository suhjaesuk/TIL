# 리더보드 (Leader Board)

- 게임이나 경쟁에서 상위 참가자의 랭킹과 점수를 보여주는 기능
- 순위로 나타낼 수 있는 다양한 대상에 응용 (최다 구매 상품, 리뷰 순위 등)

# 동작 과정 (API 관점)

빠른 업데이트 / 빠른 조회가 필요 

- 점수 생성 / 업데이트
- 상위 랭킹 조회 (범위 기반 조회)
- 특정 대상 순위 조회 (값 기반 조회)

# 데이터 구조와 성능 문제

## R**DB**

- 업데이트의 경우
    - 한 행에만 접근하므로 비교적 빠름
- 랭킹 범위나 특정 대상의 순위 조회의 경우
    - 데이터를 정렬하거나 COUNT() 등의 집계 연산을 수행해야 하므로 데이터가 많아질수록 속도가 느려짐

## Redis

- 순위 데이터에 적합한 Sorted-Set의 자료구조를 사용하면 score를 통해 자동으로 정렬됨
- 용도에 특화된 오퍼레이션(Set 삽입 / 업데이트 / 조회)이 존재하므로 사용이 간단함
- 자료구조의 특성으로 데이터 조회가 빠름 (범위 검색 / 특정 값의 순위 검색)
- 빈번한 엑세스에 유리한 In-Memory DB의 속도

# 실습

**build.gradle redis 의존 추가**

```java
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
```

**application.yml 설정**

```java
spring:
  redis:
    host: localhost
    port: 6379
```

**RankingService**

Sorted-Set(ZSet)을 이용한 userId에 Score 저장 및 랭킹 조회 API 작성

```java
import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ZSetOperations;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Set;

@Service
@RequiredArgsConstructor
public class RankingService {
    private final StringRedisTemplate redisTemplate;
    private static final String LEADERBOARD_KEY = "leaderBoard";

    // zSet에 유저의 점수 저장
    public boolean setUserScore(String userId, int score) {
        ZSetOperations zSetOps = redisTemplate.opsForZSet();
        zSetOps.add(LEADERBOARD_KEY, userId, score);

        return true;
    }

    // 유저의 랭킹 조회
    public Long getUserRanking(String userId) {
        ZSetOperations zSetOps = redisTemplate.opsForZSet();
        Long rank = zSetOps.reverseRank(LEADERBOARD_KEY, userId) + 1;

        return rank;
    }

    // 1위 부터 limit 까지 랭킹 조회
    public List<String> getTopRank(int limit) {
        ZSetOperations zSetOps = redisTemplate.opsForZSet();
        Set<String> rangeSet = zSetOps.reverseRange(LEADERBOARD_KEY, 0, limit - 1);

        return new ArrayList<>(rangeSet);
    }
}
```

**RankingController**

```java
import com.study.redis.service.RankingService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequiredArgsConstructor
public class RankingController {

    private final RankingService rankingService;

    @GetMapping("/setScore")
    public Boolean setScore(
            @RequestParam String userId,
            @RequestParam int score
    ) {
        return rankingService.setUserScore(userId, score);
    }

    @GetMapping("/getRank")
    public Long getUserRank(
            @RequestParam String userId
    ) {
        return rankingService.getUserRanking(userId);
    }

    @GetMapping("/getTopRank")
    public List<String> getTopRank() {
        return rankingService.getTopRank(10);
    }
}
```

**API 호출 (A는 20점, B는 30점, C는 5점 저장)**

```java
> http://localhost:8080/setScore?userId=A&score=20
true
```

```java
> http://localhost:8080/setScore?userId=B&score=30
true
```

```java
> http://localhost:8080/setScore?userId=C&score=5
true
```

**API 호출** (**A의 랭킹 조회)**

```java
> http://localhost:8080/getRank?userId=A
2 # 2등
```

**API 호출** **(10위 까지 랭킹 조회)**

```java
> http://localhost:8080/getTopRank
["B","A","C"]
```

## **RDB와 Redis의 성능 비교**

데이터 백만건 기준으로 성능 비교를 해보자.

**Test 코드 작성 (redis에 1,000,000건 데이터 Insert)**

```java
@SpringBootTest
public class SimpleTest {

    @Autowired
    private RankingService rankingService;
    static final int MILLION = 1000000;

    @Test
		// redis에 백만건 저장 
    void insertScore() {
        for (int i = 0; i < MILLION; i++) {
            int score = (int)(Math.random() * MILLION);
            String userId = "userKey" + i;

            rankingService.setUserScore(userId, score);
        }
    }
}
```

실행 후 Redis 확인

```java
127.0.0.1:6379> ZCOUNT leaderBoard 0 9999999
(integer) 1000003
```

**Test Code 작성 (userService 메서드 호출 시 걸리는 시간 확인)**

```java
@Test
    void getRanks() {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        Long userRank = rankingService.getUserRanking("userKey100");
        stopWatch.stop();

        System.out.println("userRank : " + userRank);
        System.out.println("time : " + stopWatch.getTotalTimeSeconds());

        stopWatch = new StopWatch();
        stopWatch.start();
        List<String> topRanker = rankingService.getTopRank(10);
        stopWatch.stop();

        System.out.println("topRanker : " + topRanker);
        System.out.println("time : " + stopWatch.getTotalTimeSeconds());
    }
```

**결과** 

- 단건 조회: 0.401초
- 범위 조회: 0.005초

```java
userRank : 429848
time : 0.4017581
topRanker : [userKey893193, userKey651165, userKey44075, userKey261224, userKey881552, userKey2392, userKey810000, userKey661065, userKey384512, userKey966526]
time : 0.0056594
```

**Test Code 작성 (ArrayList를 이용하여 userRanks 구현, 랭킹 조회 시 시간 확인)**

```java
@Test
    void inMemorySortPerformance() {
        ArrayList<Integer> userRanks = new ArrayList<>();
        for (int i = 0; i < MILLION; i++) {
            int score = (int)(Math.random() * MILLION);
            userRanks.add(score);
        }

        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
				// 내림차순 정렬
        Collections.sort(userRanks, Collections.reverseOrder());
        Integer userRank = userRanks.get(100);
        stopWatch.stop();

        System.out.println("userRank : " + userRank);
        System.out.println("time : " + stopWatch.getTotalTimeSeconds());

        stopWatch = new StopWatch();
        stopWatch.start();
        Collections.sort(userRanks, Collections.reverseOrder());
        List<Integer> topRanker = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            topRanker.add(userRanks.get(i));
        }
        stopWatch.stop();

        System.out.println("topRanker : " + topRanker);
        System.out.println("time : " + stopWatch.getTotalTimeSeconds());
    }
```

**결과**

- 단건 조회: 0.323초
- 범위 조회: 0.307초

```java
userRank : 999907
time : 0.3232569
topRanker : [999999, 999998, 999997, 999996, 999994, 999994, 999992, 999991, 999991, 999990]
time : 0.3078065
```

**성능 비교**

단건 조회에서는 큰 차이를 보여주지 않지만, 범위 조회의 경우 60배의 성능 차이를 보여준다.