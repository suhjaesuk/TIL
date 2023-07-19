1. **apt 업그레이드**

```yaml
sudo apt update
```

```yaml
sudo apt upgrade
```

1. **build-essential 설치 (소스코드 빌드 툴)**

```yaml
sudo apt-get install build-essential pkg-config -y
```

- 모두 yes할것이기 때문에 pkg-config -y 옵션을 설정한다.

1. **redis 설치**
- redis 설치할 폴더 만들기

```yaml
sudo mkdir -p home/ubuntu/app
```

- 경로 지정

```yaml
cd /home/ubuntu/app
```

- redis 다운로드

```yaml
sudo wget https://download.redis.io/releases/redis-6.2.6.tar.gz
```

```yaml
sudo tar xvfz redis-*.tar.gz
```

```yaml
sudo rm redis-*.tar.gz
```

- 심볼릭 링크 생성

```yaml
ln -s /home/ubuntu/app/redis* redis
```

- 경로 지정

```yaml
cd redis
```

- 소스 컴파일

```yaml
sudo make
```

1. **redis-server /usr/bin 복사**

redis-server를 실행하면 경로를 찾지 못하는 경우가 있는데 이럴 경우 /usr/bin에 redis-server를 복사하면 된다.

```yaml
sudo cp /home/ubuntu/app/redis/src/redis-server /usr/bin
```

```yaml
sudo cp /home/ubuntu/app/redis/src/redis-cli /usr/bin
```

1. **redis-server 실행**

redis 서버를 킬 때 원하는 설정 파일을 입력하면 해당 설정 파일을 읽어 실행된다.

```yaml
redis-server /etc/reids.conf
```