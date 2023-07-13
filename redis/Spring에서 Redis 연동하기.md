# Redis 라이브러리 사용

- Lettuce: 가장 많이 사용되는 라이브러리, Spring Data Redis에 내장되어 있음.
- Spring Data Redis는 Redis Template이라는 Redis 조작의 추상 레이어를 제공함.

![제목 없는 다이어그램 drawio](https://github.com/suhjaesuk/til/assets/110963294/bdb8ae50-6844-411d-862b-53b7dd4c52bc)


# 연동하기

- build.gradle에 의존 추가

```bash
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

- application.yml 파일에 host, port 지정

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
```

# 실습

간단한 실습을 위해 모듈을 분리하지 않고 컨트롤러만 이용합니다.

- RedisTemplate 의존성 주입

```java
private final StringRedisTemplate redisTemplate;
```

- fruit이라는 String 변수에 값(name) 넣는 API

```java
    @GetMapping("/setFruit")
    public String setFruit(@RequestParam String name) {
        // 문자열 값을 다루기 위한 기능을 제공해주는 인터페이스
        ValueOperations<String, String> ops = redisTemplate.opsForValue();
        ops.set("fruit", name);

        return "saved";
    }
```

- setFruit API 호출

```java
> http://localhost:8080/setFruit?name=banana
saved
```

- fruit의 값 조회하는 API

```java
@GetMapping("/getFruit")
    public String getFruit() {
        ValueOperations<String, String> ops = redisTemplate.opsForValue();
        String fruitName = ops.get("fruit");

        return fruitName;
    }
```

- getFruit API 호출

```java
> http://localhost:8080/getFruit
banana
```

**전체 코드**

```java
import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequiredArgsConstructor
public class HelloController {
    private final StringRedisTemplate redisTemplate;

    @GetMapping("/setFruit")
    public String setFruit(@RequestParam String name) {
        ValueOperations<String, String> ops = redisTemplate.opsForValue();
        ops.set("fruit", name);

        return "saved";
    }

    @GetMapping("/getFruit")
    public String getFruit() {
        ValueOperations<String, String> ops = redisTemplate.opsForValue();
        String fruitName = ops.get("fruit");

        return fruitName;
    }
}
```
