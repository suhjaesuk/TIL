# Docker란?

- 경량화된 가상 환경인 컨테이너를 이용해 프로그램을 실행할 수 있는 기술
- 실행 환경을 매번 새로 설정할 필요가 없이 간편하게 실행
- 여러 대의 PC를 사용하는 것 처럼 테스트 환경 설정에 용이

# Docker 설치

- Windows: WSL2 (Windows Subsystem for Linux 2)를 활용해 작동
- Linux: apt-get, yum과 같은 패키지 관리자 사용 가능
- Mac: brew와 같은 패키지 관리자 사용 가능
- Docker 홈페이지에서 각 OS에 맞는 Docker Desktop 설치
    - https://www.docker.com

# Docker로 Redis 설치 및 실행하기

- Docker로 Redis 설치

```bash
$ docker pull redis
```

- Redis 컨테이너 생성
    - -p 6379:6379: 포트 번호 6379:6379에 생성
    - —name my-redis: 이름을 my-redis로 지정

```bash
//이름 지정 x
$ docker run -p 6379:6379 redis
```

```bash
//이름 지정 o
$ docker run --name my-redis -p 6379:6379 redis
```

- Redis 중단

```bash
$ docker stop my-redis
```

- Redis 시작

```bash
$ docker start my-redis
```

# redis-cli 접속
- Window 이용
- redis-server: 레디스 서버
- redis-cli: 레디스 서버에 커맨드를 실행할 수 있는 인터페이스
    - redis-cli를 실행하기 위해서 Docker Container 안에서 실행해야함.
- Docker Container 안에서 쉘 실행

```bash
$ winpty docker exec -it my-redis sh
```

- Container 내부의 쉘에서 redis-cli 실행
    - 호스트와 포트를 지정하지 않으면 127.0.0.1:6379 사용

```docker
// 호스트와 포트 지정 o
# redis-cli -h 127.0.0.1 -p 6379
```

```docker
// 호스트와 포트 지정 x
# redis-cli
```
