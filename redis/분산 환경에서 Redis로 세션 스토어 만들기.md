# 세션(Session)

- 네트워크 상에서 두 개 이상의 통신장치간에 유지되는 상호 연결
- 연결된 일정 시간 동안 유지되는 정보를 나타냄
- 적용 대상에 따라 다른 의미를 가짐

![112](https://github.com/suhjaesuk/til/assets/110963294/8f1831cf-591d-4a27-b48d-299b6e1e484b)


## 웹 로그인 세션

- 웹 상에서 특정 유저가 로그인했음을 나타내는 정보
- 브라우저는 쿠키, 서버는 해당 쿠키에 연관된 세션 정보를 저장
- 유저가 로그아웃하거나 세션이 만료될 때 까지 유지되어 유저에 특정한 서비스 가능

![121](https://github.com/suhjaesuk/til/assets/110963294/f32b0f8d-fa2f-4cfb-88a1-6f4bb0a78879)


## Web 로그인 과정

![333](https://github.com/suhjaesuk/til/assets/110963294/f7eeae65-71c5-4dc8-b145-d7a82253a2e3)


# 분산 환경에서의 세션 처리

- 서버는 세션 정보를 저장해야 함
- 서버가 여러 대라면 최초 로그인한 서버가 아닌 서버는 세션 정보를 알지 못함
- 세션 정보를 서버간에 공유할 방법이 필요 (Session Clustering)

![111111](https://github.com/suhjaesuk/til/assets/110963294/c1c4a327-2a17-4f72-b590-a6b64dc10971)


# 세션 스토어로서 Redis 사용

- 세션 데이터는 단순 Key-Value 구조
- 세션 데이터는 영속성이 필요 없음
- 세션 데이터는 변경이 빈번하고 빠른 액세스 속도가 필요

# 세션 관리를 위한 서버의 역할

- 세션 생성: 요청이 들어왔을 때 세션이 없다면 만들어서 응답에 set-cookie로 넘겨줌
- 세션 이용: 요청이 들어왔을 때 세션이 있다면 해당 세션의 데이터를 가져옴
- 세션 삭제: 타임아웃이나 명시적인 로그아웃 API를 통해 세션을 무효화 함

## HttpSession

- 세션을 손쉽게 생성하고 관리할 수 있게 해주는 인터페이스
- UUID로 세션 ID를 생성
- JSESSIONID라는 이름의 쿠키를 설정해 내려줌

![44444](https://github.com/suhjaesuk/til/assets/110963294/9b949ade-4b44-47f1-98d7-196632e9f872)


# 실습

**Controller 전체 코드**

```java
import jakarta.servlet.http.HttpSession;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;

@RestController
public class LoginController {
		// 실습이므로 Hash에 데이터를 저장
    HashMap<String, String> sessionMap = new HashMap<>();

    @GetMapping("/login")
    public String login(HttpSession session, @RequestParam String name) {
        sessionMap.put(session.getId(), name);

        return "saved";
    }

    @GetMapping("/myName")
    public String myName(HttpSession session) {
        String myName = sessionMap.get(session.getId());

        return myName;
    }
}
```

- 로그인 API 호출 시 cookie에 JSESSIONID 값이 저장됨

```java
> http://localhost:8080/login?name=jay
saved
```

![캡처](https://github.com/suhjaesuk/til/assets/110963294/6404c127-97ad-4456-9b6e-83c278c3388e)


- 쿠키에 저장 된 이름 조회 API 호출 시 cookie에 담긴 값으로 데이터를 찾음

```java
> http://localhost:8080/myName
jay
```

하지만 서버가 여러 개일 경우 로컬 변수는 공유가 안되기 때문에 세션ID는 DB에 저장되어야 한다.

- build.gradle 의존 추가

```java
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

- application.yml 설정

```java
spring:
  session:
    store-type: redis # 세션이 redis에 저장됨
  redis:
    host: localhost
    port: 6379
```

**Controller 전체 코드**

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpSession;

@RestController
public class LoginController {

    @GetMapping("/login")
    public String login(HttpSession session, @RequestParam String name) {
        session.setAttribute("name", name);

        return "saved";
    }

    @GetMapping("/myName")
    public String myName(HttpSession session) {
        String myName = (String)session.getAttribute("name");

        return myName;
    }
}
```

- 8080 포트에서 login API 호출

```java
> http://localhost:8080/login?name=jay
saved
```

- 8081 포트에서 쿠키에 저장 된 이름 조회 API 호출

```java
> http://localhost:8081/myName
jay
```
