---
layout: default
title: 4. 영속성 관리
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 4
---

# 영속성 관리
{: .no_toc }

1. TOC
{:toc}

## 영속성 컨텍스트

고객마다 entityManager를 생성하고 DB의 connection pool에 연결하여 DB를 사용하게 된다  
영속성 컨텍스트? - "엔티티를 영구 저장하는 환경"  

- J2SE
  - 엔티티 매니저와 영속성 컨텍스트가 1 : 1
- J2EE, Spring Framework
  - 엔티티 매니저와 영속성 컨텍스트가 N : 1

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb4EkGZ%2FbtrvkHDroDJ%2FzYl4aBQ4N0Xto2wSkIXyrk%2Fimg.png)

- 엔티티의 생명주기
  - 비영속 (new/transient)
    - 영속성 컨텍스트와 전혀 관계 없음
    - 객체를 생성만 한 상태
  - 영속 (managed)
    - 영속성 컨텍스트에 의해 관리됨
    - 객체가 영속성 컨텍스트에 들어감
  - 준영속 (detached)
    - 영속성 컨텍스트에 의해 관리되었다가 분리된 상태
    - id 값만 존재
  - 삭제 (removed)
    - 객체가 삭제됨
    - `em.remove` (delete query)

- 영속성 컨텍스트를 사용하는 이점?
  - 1차 캐시
    - 고객 혼자서 사용
    - 애플리케이션 전체(여러 고객들)에서 공유 가능한 캐시는 2차 캐시
    - 성능에 큰 이점은 없지만, 객체지향적 코드를 만드는데 유리
  - 동일성 보장
    - 영속 엔티티의 비교 동일성 보장
    - 1차 캐시로 반복 가능한 읽기(Repeatable Read) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다.
  - 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
    - 쓰기 버퍼처럼 동작
      - A persist 후 쓰기 지연 sql 저장소에 쌓임
      - B persist 후 쓰기 지연 sql 저장소에 쌓임
      - commit 시점에 flush 되면서 순차적으로 한번에 날라간다
      - 그때그때마다 쿼리를 날리는게 아니라 한번에 모아 보내서 네트워크 비용 절약(JDBC batch?)
  - 변경 감지(Dirty Checking)
    - flush -> entity와 snapshot 비교 -> update query 생성
  - 지연 로딩(Lazy Loading)

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FDdlOS%2Fbtrvl88LKkb%2FWbkuWG65iAj0UZAPpNFjX0%2Fimg.png)

## 플러시

**영속성 컨텍스트의 변경사항을 DB에 반영** (보통 commit 시)  
= 영속성 컨텍스트의 변경점과 **DB를 동기화**한다

- flush 발생
  - 변경 감지 ->
  - 수정된 엔티티를 쓰기지연 SQL 저장소에 등록 ->
  - **쓰기지연 SQL 저장소의 쿼리를** DB에 전송
  - **영속성 컨텍스트가 비워지는게 아니다** -> 이거는 `em.clear()`
- 언제 flush가 발생하나?
  - em.flush() 직접 호출
  - tx.commit() 자동 호출
  - JPQL query 실행시 자동 호출
    - 데이터베이스 일관성 확보
- flush mode option
  - FlushModeType.AUTO
    - 커밋이나 쿼리를 실행할 때 flush (기본값)
  - FlushModeType.COMMIT
    - 커밋할 때만 flush (쓸 일 별로 없음)

## 준영속 상태

영속 상태의 엔티틱가 영속성 컨텍스트에서 분리  
- 준영속 상태로 만드는 방법
  - `em.detach(entity)`
  - `em.clear()`
  - `em.close()`
