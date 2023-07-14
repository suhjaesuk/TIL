# 캐싱 (Caching)

- 캐시: 성능 향상을 위해 값을 복사해놓는 임시 기억 장치
- 캐시에 복사본을 저장해놓고 읽음으로서 속도가 느린 장치로의 접근 횟수를 줄임
- 캐시의 데이터는 원본이 아니며 언제든 사라질 수 있음

![111](https://github.com/suhjaesuk/til/assets/110963294/9cc853d5-7bca-4959-b143-2f59e97d63a5)


# 적용

![222](https://github.com/suhjaesuk/til/assets/110963294/f4919b79-407b-45cd-a067-c77f8f24843a)


# 용어

- 캐시 적중(Cache Hit): 캐시에 접근해 데이터를 발견함
- 캐시 미스(Cache Miss): 캐시에 접근했으나 데이터를 발견하지 못함
- 캐시 삭제 정책(Eviction Policy): 캐시의 데이터 공간 확보를 위해 저장된 데이터를 삭제
- 캐시 전략: 환경에 따라 적합한 캐시 운영 방식을 선택할 수 있음
    - Cache-Aside
    - Write-Through

## 전략

### Cache-Aside(Lazy Loading)

- 항상 캐시를 먼저 체크하고, 없으면 원본에서 읽어온 후 캐시에 저장함
- 장점: 필요한 데이터만 캐시에 저장되고, Cache Miss가 있어도 치명적이지 않음
- 단점: 최초 접근이 느림, 업데이트 주기가 일정하지 않기 때문에 캐시가 최신 데이터가 아닐 수 있음

![444](https://github.com/suhjaesuk/til/assets/110963294/68ee66c1-a721-4dce-abef-963dcff3fd38)


### Write-Through

- 데이터를 쓸 때 항상 캐시를 업데이트하여 최신 상태를 유지함
- 장점: 캐시가 항상 동기화되어 있어 데이터가 최신 상태
- 단점: 자주 사용하지 않는 데이터도 캐시됨, 쓰기 지연 시간이 증가

![5555](https://github.com/suhjaesuk/til/assets/110963294/3db9f1fe-a590-42d2-9e67-402dbbf124f4)


### Write-Back

- 데이터를 캐시에만 쓰고, 캐시의 데이터를 일정 주기로 DB에 업데이트
- 로그 파일에 주로 사용
- 장점: 쓰기가 많은 경우 DB 부하를 줄일 수 있음
- 단점: 캐시가 DB에 쓰기 전에 장애가 생기면 데이터 유실 가능

![666](https://github.com/suhjaesuk/til/assets/110963294/dc899bdb-86c1-4ac8-85e4-64e5efff813b)


## 삭제 방식

- Expiration: 각 데이터에 TTL(Time-To-Live)을 설정해 시간 기반으로 삭제
- Eviction Algorithm: 공간을 확보해야 할 경우 어떤 데이터를 삭제할지 결정하는 방식
    - LRU(Least Recently Used): 가장 오랫동안 사용되지 않은 데이터를 삭제
    - LFU(Least Frequently Used): 가장 적게 사용된 데이터를 삭제
    - FIFO(First In First Out): 먼저 들어온 데이터를 삭제

# 실습

- Cache-Aside 전략으로 구현
    - 요청에 대해 캐시를 먼저 확인하고, 없으면 원천 데이터 조회 후 캐시에 저장
- API로 사용자의 프로필(이름, 나이)를 얻어온다.
    - 이름은 캐시로 저장하지만, 비교를 위해 나이는 캐시에 저장하지 않는다.
- 이 실습에서는 Controller, Service, Dto, Repository로 패키지를 분리하여 진행한다.

**Controller 전체코드**

```java
import com.study.redis.dto.UserProfile;
import com.study.redis.service.UserService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequiredArgsConstructor
public class ApiController {
    private final UserService userService;

    @GetMapping("/users/{userId}/profile")
    public UserProfile getUserProfile(@PathVariable String userId) {
        return userService.getUserProfile(userId);
    }
}
```

**Dto 전체코드**

```java
import lombok.Getter;

@Getter
public class UserProfile {
    private String name;

    private int age;

    public UserProfile(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

**Service 전체코드**

```java
import com.study.redis.dto.UserProfile;
import com.study.redis.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final StringRedisTemplate redisTemplate;
    private final String NAME_KEY = "nameKey::";
    public UserProfile getUserProfile(String userId) {
        String userName;

        ValueOperations<String, String> ops = redisTemplate.opsForValue();
        String cachedName = ops.get(NAME_KEY + userId);
				// 캐시로 조회된 이름이 있다면 userName에 저장
        if (cachedName != null) {
            userName = cachedName;
        } else {
						// 캐시로 조회된 이름이 없다면 repository에서 값 불러오기
            userName = userRepository.findNameById(userId);
            // 캐시를 10초간 저장 (TTL)
            ops.set(NAME_KEY + userId, userName, 10, TimeUnit.SECONDS);
        }
				// 비교를 위해 나이는 캐시로 저장하지 않는다.
        int userAge = userRepository.findAgeById(userId);

        return new UserProfile(userName, userAge);
    }
}
```

**Repository 전체코드**

```java
@Service
public class UserRepository {

    public String findNameById(String userId) {
        try {
						// 비교를 위해 0.5초간 대기
            Thread.sleep(500);
        } catch (InterruptedException e){
        }
        if (userId.equals("A")) {
            return "Adam";
        }
        if (userId.equals("B")) {
            return "Bob";
        }
        return null;
    }

    public int findAgeById(String userId) {
        try {
            Thread.sleep(500);
        } catch (InterruptedException e){
        }
        if (userId.equals("A")) {
            return 28;
        }
        if (userId.equals("B")) {
            return 27;
        }
        return 0;
    }
}
```

1. 캐시를 사용안할 때 API 호출 시
    
    ```java
    > http://localhost:8080/users/A/profile
    {"name":"Adam","age":28}
    ```
    
![1초](https://github.com/suhjaesuk/til/assets/110963294/ef4f63a4-a839-449e-8769-950b45c99c92)

    
    name과 age를 모두 repository에서 가져오기 때문에 1초가 걸림
    
    ```java
    127.0.0.1:6379>  keys *
    1) "nameKey::A"
    ```
    
    redis를 확인해보면 A 유저가 저장됨
    
    ```java
    127.0.0.1:6379> GET "nameKey::A"
    "Adam"
    ```
    
    A 유저의 name을 조회하면 Adam이 값으로 저장됨
    
2. 캐시를 사용할 때 API 호출 시
    
    ```java
    > http://localhost:8080/users/A/profile
    {"name":"Adam","age":28}
    ```
    
![0 5ch](https://github.com/suhjaesuk/til/assets/110963294/8352edfb-b034-4ca5-acd1-253cf7713e33)

    
    name은 redis에서 가져오고 age만 repository에서 가져오기 때문에 0.5초가 걸림
    
- 캐시에 저장되고 만료 후
    
    ```java
    127.0.0.1:6379>  keys *
    (empty array)
    ```
    
    redis에 A 유저의 캐시가 저장되어 있지 않음
    
    API 호출 시 
    
    ```java
    > http://localhost:8080/users/A/profile
    {"name":"Adam","age":28}
    ```
    
![1초](https://github.com/suhjaesuk/til/assets/110963294/5438cda3-3af9-4a17-a542-61c04be25384)

    
    1번과 똑같음. name과 age를 모두 repository에서 가져오기 때문에 1초가 걸림
