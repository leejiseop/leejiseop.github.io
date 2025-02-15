---
layout: default
title: 1 ~ 3. 소개
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 1
---

# 소개
{: .no_toc }

1. TOC
{:toc}

## SQL 중심적인 개발의 문제점

1. 객체를 관계형 DB에 관리하려면 SQL 쿼리를 날려야한다. 객체에 필드가 추가되면 다시 쿼리문 다 고치고...  
이런 과정이 무한 반복되는 지루한 **SQL 의존적인 CRUD 작업**
2. 패러다임의 불일치
객체지향 프로그래밍은 추상화, 캡슐화, 정보은닉, 상속, 다형성 등 복잡성을 제어할 수 있는 다양한 장치들을 제공  
객체들을 영구 저장하기에 가장 현실적인 대안은 관계형 데이터베이스  
사실상 개발자는 객체와 RDB 사이의 SQL Mapper이다.

### 객체와 관계형 데이터베이스의 차이
1. 상속
   - RDB에는 객체같은 상속관계가 없다. -> 테이블을 분해해서 상속처럼 각각 관리해야한다.
   - 조회시에는 슈퍼타입과 서브타입 join... 복잡하다.
2. 연관관계
   - 객체는 참조를 사용 `member.getTeam()`
   - 테이블은 외래 키를 사용 `join on member.team_id = team.team_id`
   - 객체를 테이블에 맞추어 모델링 하게된다 (`Long teamId`).
     - 하지만 그것은 객체다운 모델링이 아니다.
   - 객체는 key가 아닌 참조(.)로 연관관계를 맺는다 (`Team team`)
     - 근데 그러면 CRUD 쿼리 만들기가 너무 까다로워진다...
3. 엔티티 신뢰 문제
   - 모든 객체를 미리 전부 로딩할 수는 없다. 비효율적. -> 일부 계층만 로딩하면 객체의 다음 계층을 신뢰할 수 없다. -> 로딩하는 쿼리 어떻게 짜여져있나 DAO 일일이 들어가서 확인해봐야함 -> 모든 경우의 수에 대비 (`getMember`, `getMemberWithTeam ...`) -> 번거롭다
   - 객체는 자유로운 **객체 그래프 탐색**이 가능해야한다.
     - (`member.order.orderitem.item.category ... `)
4. 데이터 식별 방법
   ```java
    // 객체를 RDB에 저장한 경우
    class MemberDAO {
        public Member getMember(String memberId) {
            String sql = "select * from member where member_id = ?";
            // ...
            return new Member(...); // new로 새로운 인스턴스 반환
        }
    }

    Member member1 = memberDAO.getMember(100);
    Member member2 = memberDAO.getMember(100);

    member1 == member2; // 다르다! (같은 데이터, 다른 인스턴스)
    ```

    ```java
    // 객체를 Collection에 저장한 경우
    Member member1 = list.get(100);
    Member member2 = list.get(100);

    member1 == member2 // 같다! (같은 인스턴스 참조)
    ```
- 객체를 Collection에 저장하듯이 DB에 저장할 수는 없을까?
  - 그게 JPA

## JPA 소개

Java Persistence API  
자바 진영의 ORM 기술 표준  

### ORM

- Object Relational Mapping
- 객체는 객체대로, DB는 DB대로 설계
- ORM 프레임워크가 중간에서 매핑

- JPA는 애플리케이션과 JDBC 사이에서 동작
  - `[JAVA application] <--> [JPA] <--> [JDBC API] <--query--> [DB]`
- **JPA는 표준 명세** 즉, **인터페이스의 모음**이다
  - **Hibernate**, EclipseLink, DataNucleus ... 다양한 **구현체**들

1.0(2006) -> 2.0(2009) -> 2.1(2013) -> 이후부터 큰 변화는 없음

### **JPA를 왜 사용해야 하는가?**

- SQL 중심 개발 -> **객체 중심 개발**
- 생산성
  - Java Collection 사용하듯 편리하게 코딩
- 유지보수
  - SQL 쿼리 일일이 변경할 필요 X
- 패러다임의 불일치 해결
  - 상속, 연관관계, 객체 그래프, 객체 비교
    - 상속: JPA가 대신 super type과 sub type 테이블에 insert 쿼리 두번 날려줌
    - 조회: JPA가 대신 JOIN 해줌
    - 연관관계: `member.setTeam(team)`
    - 객체 그래프 탐색 `member.getTeam()` + 지연 로딩
    - 객체 비교: 동일 트랜잭션에서 조회한 엔티티는 같음을 보장
- 성능 최적화
  - 1차 캐시와 동일성 보장 (identity)
    - 중간에 뭔가 생긴다? 버퍼와 캐시로 활용 가능. 바로 JPA가 그런 것.
    - 같은 트랜잭션 안에서의 조회: 같은 엔티티를 반환(영속성 컨텍스트)
  - 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)
    - 데이터를 버퍼로 모아서 한방에 보낸다(JDBC batch SQL 기능 사용) (insert, update 등)
    ```java
    transaction.begin();
    em.persist(memberA);
    em.persist(memberB);
    em.persist(memberC);
    transaction.commit(); // 커밋하는 순간 insert query를 한번에 DB로 보낸다 - buffer writing
    ```
  - 지연 로딩 (Lazy Loading)
    - 객체가 실제 사용될 때 로딩
- 데이터 접근 추상화와 벤더 독립성
- 표준

### JPA는 어떻게 동작하나?

`persistence.xml`에서 설정 정보 조회 -> `EntityManagerFactory` 클래스 생성 -> `EntityManager` 생성  

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcwZNc4%2FbtrvhMSJJBE%2FqUGhjtwf1T5qpKjqeECHiK%2Fimg.png)

```java
// 로딩 시점에 하나만 만들어둔다
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
// 고객 요청이 올 때마다(트랜잭션 시점마다) 만들고 지운다
// em 은 쓰레드에서 공유하면 안된다 -> 사용하고 바로바로 버리는 개념
EntityManager em = emf.createEntityManager(); // DB connection과 비슷. 객체를 저장해주는 collection 느낌.
EntityTransaction tx = em.getTransaction();

tx.begin();
try { // ---> 이런건 스프링이 다 해준다

  // code

  tx.commit();
} catch (Exception e) {
  tx.rollback();
} finally {
  em.close(); // DB connection 해제
}

emf.close();
```