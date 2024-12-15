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

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcwZNc4%2FbtrvhMSJJBE%2FqUGhjtwf1T5qpKjqeECHiK%2Fimg.png)

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
    - 영속성 컨텍스트에 의해 관리되었다가 분리된 상태(id 값만 존재)
  - 삭제 (removed)
    - 객체가 삭제됨

- 영속성 컨텍스트를 사용하는 이점?
  - 1차 캐시
  - 동일성 보장
  - 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
  - 변경 감지(Dirty Checking)
  - 지연 로딩(Lazy Loading)



## 플러시

## 준영속 상태