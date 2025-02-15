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

## 원인

- 

## 해결

- asdf

출처: (우아한형제들) HikariCP Dead lock에서 벗어나기 [이론편](https://techblog.woowahan.com/2664/) [실전편](https://techblog.woowahan.com/2663/)