# Pub/Sub 패턴

- 메시징 모델 중의 하나로 발행(Publish)과 구독(Subscribe) 역할로 개념화한 형태
- 발행자와 구독자는 서로에 대한 정보 없이 특정 주제(토픽 or 채널)를 매개로 송수신

![1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58d8e7c3-3fb4-4630-aa78-a831e3abd0e9/1.png)

# 메시징 미들웨어 사용의 장점

- 비동기: 통신의 비동기 처리
- 낮은 결합도: 송신자와 수신자가 직접 서로 의존하지 않고 공통 미들웨어에 의존
- 탄력성: 구성원들간에 느슨한 연결로 인해 일부 장애가 생겨도 영향 최소화

# Redis의 Pub/Sub 특징

- 메시지가 큐에 저장되지 않음
- Kafka의 컨슈머 그룹같은 분산처리 개념이 없음
- 메시지 발행 시 push 방식으로 subscriber들에게 전송
- subscriber가 늘어날수록 성능이 저하

# Usecase

- 실시간으로 빠르게 전송되어야 하는 메시지
- 메시지 유실을 감내할 수 있는 케이스
- 최대 1회 전송(at-most-once) 패턴이 적합한 경우
- Subscriber가 다양한 채널을 유동적으로 바꾸면서 한시적으로 구독하는 경우

# 채팅방 기능 요구사항

- 채팅 클라이언트와 채팅 서버가 존재하고 통신 방식을 정해야 함.
- 채팅 서버는 채팅방 관리 로직을 작성해야 함

![3.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ebe78aa5-647f-4e7f-a93d-c6573462238a/3.png)

## Redis Pub/Sub을 이용한 채팅방 구현

- 채팅방 기능을 Pub/Sub 구조를 이용해 쉽게 구현 가능

![5.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/73e03bd5-6529-4cc1-9883-ef583f389ad2/5.png)

# 실습

**build.gradle에 redis 의존 추가**

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

**RedisChatConfig 코드 작성**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;

@Configuration
public class RedisChatConfig {

    @Bean
		// Lettuce 라이브러리 사용
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory();
    }

    @Bean
		// 구독 및 송 수신 관리해주는 컨테이너
    RedisMessageListenerContainer redisContainer() {
        final RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(redisConnectionFactory());
        return container;
    }
}
```

**Service 코드 작성**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.Message;
import org.springframework.data.redis.connection.MessageListener;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.listener.ChannelTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.stereotype.Service;

import java.util.Scanner;

@Service
public class ChatService implements MessageListener {

    @Autowired
    private RedisMessageListenerContainer container;
    @Autowired
    RedisTemplate<String, String> redisTemplate;

    public void enterChatRoom(String chatRoomName) {
        // onMessage에 메세지를 뿌려주는 역할 
        container.addMessageListener(this, new ChannelTopic(chatRoomName));
        Scanner scanner = new Scanner(System.in);
        while(scanner.hasNextLine()) {
            String line = scanner.nextLine();
						// quit을 입력할 경우 while문 종료
            if (line.equals("quit")) {
                System.out.println("quit...");
                break;
            }
            // Redis에서 채널을 구독할 수 있게 해주는 기능 
            redisTemplate.convertAndSend(chatRoomName, line);
        }
        // 채널 구독 취소
        container.removeMessageListener(this);
    }
		
    @Override
    public void onMessage(Message message, byte[] pattern) {
        System.out.println("Message: " + message.toString());
    }
}
```

**Main Class에 Comman Line을 이용하여 채팅 서비스 구현**

```java
@SpringBootApplication
public class RedisApplication implements CommandLineRunner {

    @Autowired
    private ChatService chatService;

    public static void main(String[] args) {
        SpringApplication.run(RedisApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println("chat started...");
        chatService.enterChatRoom("chatRoom1");
    }
}
```

**Redis에서** **채팅 보내기**

```java
127.0.0.1:6379> publish chatRoom1 "안녕하세요 "
(integer) 1
```

**Command Line**

```java
Message: 안녕하세요
```

**Redis에서 채팅방 구독하기**

```java
127.0.0.1:6379> subscribe chatRoom1
```

**Command Line**

```java
hi
```

**Redis 결과값**

```java
1) "message"
2) "chatRoom1"
3) "hi"
```

**채팅방 나가기**

**Command Line**

```java
quit
Quit...
```