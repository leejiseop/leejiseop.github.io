---
layout: default
title: Hikari pool locking
parent: DB
---

# Hikari pool locking
{: .no_toc }

1. TOC
{:toc}

## 현상

```bash
o.h.engine.jdbc.spi.SqlExceptionHelper : SQL Error: 0, SQLState: null
o.h.engine.jdbc.spi.SqlExceptionHelper : hikari-pool-1 – Connection is not available, request timed out after 30000ms.
org.hibernate.exception.JDBCConnectionException: unable to obtain isolated JDBC connection
Could not open JPA EntityManager for transaction; nested exception is org.hibernate.exception.JDBCConnectionException: Unable to acquire JDBC Connection
```

- Connection timeout, Insert 실패, Rollback 주기적 발생
- **간헐적으로 1~2건씩 정상 insert**

## 원인

- **한 트랜잭션에서 2개 이상의 Connection 사용** -> Thread 간 Dead lock
  - **Id 값 채번을 위한 Connection이 하나 더 필요했다**
  - Hikari는 스레드가 이전에 사용했던 Connection을 빠르게 return 해주거나, idle 상태의 connection을 찾아서 return 해준다
  - idle connection이 없다면 handoffQueue에서 대기 -> 30초 뒤 timeout Exception

| Th1 | Th2 | Th3 | Th4 | . . . | Th16 |
|**connection**|**connection**|**connection**|wait| . . . |**connection**|
|wait|wait|wait|wait| . . . |wait|

- 부하 상황에서 16개의 Thread가 거의 동시에 HikariCP로부터 Connection을 얻어냄
- Max Connection이 10개이므로 6개의 Thread는 대기중
- 트랜잭션 진행을 위한 두번째 Connection 연결 불가 -> 30초 뒤 반납

| Th1 | Th2 | Th3 | Th4 | . . . | Th16 |
|**connection**|wait|**connection**|wait| . . . |**connection**|
|wait|**connection**|**connection**|wait| . . . |wait|

- 반납된 Connection을 다시 받아가면서 우연히 2개의 Connection을 취득 -> 트랜잭션 정상 진행

## 해결

- pool size를 늘린다
  - **최소**: Tn * (Cm - 1) + 1
    - (필요 커넥션 수 - 1개)만큼 모든 쓰레드를 연결하고, 하나의 여분 Connection
  - **권장: Tn * (Cm - 1) + (Tn / 2)**
- SequenceGenerator에 대한 Pooled-lotl optimizer 적용
  - `NoopOptimizer`, `HiLoOptimizer`, `PooledOptimizer`, `PooledLoOptimizer`, `PooledLoThreadLocalOptimizer` -> 적용

출처: (우아한형제들) HikariCP Dead lock에서 벗어나기 [이론편](https://techblog.woowahan.com/2664/) [실전편](https://techblog.woowahan.com/2663/)