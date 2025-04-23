---
layout: default
title: 4. 실무 필수 최적화
parent: 실전! 스프링부트와 JPA 활용2
---

# 4. 실무 필수 최적화
{: .no_toc }

1. TOC
{:toc}

- Open Session In View
- Open EntityManager In View: JPA
- `spring.jpa.open-in-view: true` (default)
- JPA는 언제 DB connection을 가져오나?
  - DB 트랜잭션을 시작할 때
- 그럼 언제 돌려주나?
  - OSIV가 켜있으면: @Transactional 서비스 밖으로 나가도 DB connection 반환 안함
  - **트랜잭션이 끝나도 영속성 컨텍스트를 살려둔다**
  - (API의 경우)API가 유저에게 반환될 때까지, (화면의 경우)렌더링 할 때까지
  - **컨트롤러, 뷰에서도 지연로딩이 가능**
- 단점
  - 너무 오랫동안 커넥션을 물고있는다 **(DB 커넥션 풀 차지)**
  - 컨트롤러에서 외부 API 호출하는 경우(소요시간 3초) -> 이떄도 DBCP 계속 차지한다
- **OSIV 끄면?**
  - 서비스 단계에서 @Transactional 끝나면 DB 커넥션 반환
  - DBCP를 잠시만 차지
  - **모든 지연로딩을 트랜잭션 안에서만 처리해야한다**
    - `could not initialize proxy ... - no session`
    - `org.hibernate.LazyInitializationException`
  - 해결방법
    - OSIV를 켠다
    - 트랜잭션 안에서 다 해결한다
    - fetch join으로 애초에 다 다져온다
    - **커맨드와 별도의 쿼리 service 분리**
      - 비즈니스 로직 자체는 크게 변함 없다
      - 화면 관련 쿼리는 자주 바뀐다 -> **따로 분리하여 관리**
- 실시간성이 중요한 API 서버는 OSIV를 큰다
- admin 시스템은 트래픽이 적으므로 OSIV 켜서 편리하게 관리
