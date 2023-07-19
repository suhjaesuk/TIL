# Eviction이란?

- 메모리가 한계에 도달했을 때 어떤 조치가 일어날지 결정하는 정책
- 처음부터 메모리가 부족한 상황을 만들지 않는 것이 중요함
- 캐시로 사용할 때는 적절한 eviction policy가 사용될 수 있음

# Redis 메모리 관리

- Memory 사용 한도 설정
    - 지정하지 않을 시 32bit → 3GB, 64bit → 무제한으로 설정됨

```yaml
maxmemory 100mb
```

- maxmemory에 도달한 경우 eviction 정책 설정

```yaml
maxmemory-policy noeviction
```

# maxmemory-policy 옵션

- noeviction: eviction 없음. 추가 데이터는 저장되지 않고 에러 발생
- allkeys-lru: 가장 최근 사용된 키들을 남기고 나머지 삭제 (권장)
- allkeys-lfu: 가장 빈번하게 사용된 키들을 남기고 나머지 삭제
- volatile-lru: LRU를 사용하되 expire field가 true로 설정된 항목들 중에서만 삭제
- volatile-lfu: LFU를 사용하되 expire field가 true로 설정된 항목들 중에서만 삭제
- allkeys-random: 랜덤하게 삭제
- volatile-randon: expire field가 true로 설정된 항목들 중에서 랜덤하게 삭제
- volatile-ttl: expire field가 true로 설정된 항목들 중에서 짧은 TTL 순으로 삭제