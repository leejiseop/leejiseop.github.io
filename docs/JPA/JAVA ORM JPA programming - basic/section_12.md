---
layout: default
title: 섹션 12
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 12
---

# 섹션 12
{: .no_toc }

1. TOC
{:toc}

## 경로 표현식

점(.)을 찍어 객체 그래프를 탐색하는 것

- 상태 필드(state field): 단순히 값 저장
- 연관 필드(association field): 연관관계를 위한 필드
  - 단일 값 연관 필드: @ManyToOne, @OneToOne, 대상이 엔티티
  - 컬렉션 값 연관 필드: @OneToMany, @ManyToMany, 대상이 컬렉션

- 상태 필드: 경로 탐색의 끝, 추가 탐색 X
- 연관 필드
  - 단일 값 연관 경로: 묵시적 내부 조인, 탐색 O
    - `select m.team.name from Member m`
  - 컬렉션 값 연관 경로: 묵시적 내부 조인, 탐색 X (t.members.size 정도까지는 가능)
    - `select t.members from Team t`
    - FROM 절에서 **명시적 조인**을 통해 별칭을 얻으면 **별칭을 통해 탐색 가능**
      - `select m.username from team t join t.members m`
      - 별칭 m을 얻고 m.username 탐색  

**묵시적 내부 조인은 웬만하면 피하자** -> 성능 튜닝하기 힘듬  
**항상 명시적 조인을 사용하자!**  

```sql
select m.username -- 상태 필드 (값)
from Member m
join m.team t -- 단일 값 연관 필드 (엔티티)
join m.orders o -- 컬렉션 값 연관 필드 (컬렉션)
where t.name = '팀A'
```

## 페치 조인 1 - 기본

**Fetch join - 실무에서 엄청 중요함** -> N + 1 문제 해결

- SQL join 종류 X
- JPQL에서 **성능 최적화**를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 SQL 한번에 함께 조회하는 기능
- join fetch 명령어 사용
- `페치 조인 ::= [ left [ outer ] | inner ] join fetch 조인경로`

### 엔티티 페치 조인
- 회원을 조회하면서 연관된 팀도 함께 조회
- eager join과 같지만 fetch join은 명시적으로 내가 직접 원할때 사용하는것(동적 타이밍)

```sql
-- jpql
select m from Member m join fetch m.team
-- sql
select m.*, t.*
from member m
inner join team t on m.team_id = t.id
```

### 컬렉션 페치 조인
- 일대다 or 컬렉션인 경우

```sql
-- jpql
select t from Team t join fetch t.members where t.name = 'team A'
-- sql
select t.*, m.*
from Team t
inner join member m on t.id = m.team_id
where t.name = 'team A'
-- 회원 수 만큼 데이터가 뻥튀기 된다 (distinct로 중복제거 가능) - 객체와 RDB의 차이
-- 하지만 같은 team이라면 영속성 컨텍스트 공유
```

### 페치 조인과 DISTINCT

일대다 관계에서 데이터 뻥튀기가 될 수 있다.  
다대일은 상관없음.

- SQL의 DISTINCT: 중복된 결과를 제거하는 명령
- JPQL의 DISTINCT 2가지 기능
  - SQL에 DISTINCT를 추가 -> 데이터가 완전히 같아야 distinct가 적용된다.. 사실상 의미없음
  - 애플리케이션에서 엔티티 중복 제거 (ex. **같은 식별자**를 가진 Team 엔티티 제거)
```sql
select distinct t
from Team t join fetch t.members
where t.name = 'team a'
```

```java
Team team1 = new Team();
team1.setName("team1");
em.persist(team1);

Team team2 = new Team();
team2.setName("team2");
em.persist(team2);

Member member1 = new Member();
member1.setUsername("회원1");
member1.setTeam(team1);
em.persist(member1);

Member member2 = new Member();
member2.setUsername("회원2");
member2.setTeam(team1);
em.persist(member2);

Member member3 = new Member();
member3.setUsername("회원3");
member3.setTeam(team2);
em.persist(member3);

em.flush();
em.clear();

String query = "select t from Team t join fetch t.members";
List<Team> result = em.createQuery(query, Team.class).getResultList();
System.out.println(result.size()); // 왜 3이 아니라 2가 나오지??? distinct 자동 적용??
for (Team team : result) {
    System.out.println("Team: " + team.getName());
    System.out.println("Members: " + team.getMembers().size());
}
```

### 페치 조인과 일반 조인의 차이

일반 조인 실행시 연관된 엔티티를 함께 조회하지 않음 (컬렉션은 프록시 객체 아니다)
```sql
-- jpql
select t
from Team t join (fetch) t.members m
where t.name = 'team a'

-- sql
select t.* (, m.*)
from team t
inner join member m on t.id = m.team_id
where t.name = 'team a'
```

- jpql은 결과 반환시 연관관계를 고려하지 않는다. 단지 select 절에 지정한 엔티티만 조회.
- fetch join을 사용할 때만 연관된 엔티티도 함께 조회한다.(즉시 로딩)
- fetch join은 객체 그래프를 sql 한번에 조회하는 개념

## 페치 조인 2 - 한계

- 페치 조인 대상에는 별칭을 줄 수 없다.
  - (hibernate는 가능, but **가급적 사용 X**. 다중 fetch join 할 때나 사용한다.)
  - JPA가 의도한 fetch join은 기본적으로 연관된걸 전부 가져오는것. (객체 탐색 동일하게 되도록. 데이터 정합성 이슈)
  - 일부만 가져오려면 따로 조회.
```sql
select t from Team t join fetch t.members as m
```
- 둘 이상의 컬렉션은 페치 조인 할 수 없다.
  - 일대다 조회도 데이터 뻥튀기가 되는데.. 배수의 배수로 팍팍 늘어날 수 있다.
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다. (setFirstResult, setMaxResults) (데이터 정합성 이슈)
  - 일대일, 다대일 같은 단일 값 연관 필드들은 fetch join 해도 페이징 가능 (데이터 뻥튀기 X)
  - 일대다, 컬렉션은 hibernate가 경고 로그를 남기고 메모리에서 페이징 (**매우 위험**)
    - 정합성을 지키기 위해 **테이블 전체를 메모리에 올리고 페이징**한다 (ex. 10만건)
  - 해결방법
    - 일대다를 뒤집어서 다대일로 조회한다 -> 정합성 문제없음 -> 페이징 가능
    - 또는 `@BatchSize(size = 100)` 사용 -> lazy loading 할때 한번에 100개씩 -> **N + 1 문제도 해결**
      - persistence.xml에서 글로벌 세팅 가능, **값은 1000 이하로**
        - `<property name="hibernate.default_batch_fetch_size" value="100" />`
  - DTO 만들기

- 정리
  - 실무에서 -> 글로벌 로딩 전략은 모두 지연 로딩 + **최적화**가 필요한 곳에 fetch join 적용 -> 대부분 해결
  - 모든 것을 fetch join으로 해결할 수는 없음. fetch join은 **객체 그래프를 유지할 때 사용**하면 효과적
  - 여러 테이블을 join해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하면, fetch join보다는 일반 join을 사용하고, 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적
  - **fetch join은 100% 이해해야한다 ! ! !**

## 다형성 쿼리

- TYPE
  - 조회 대상을 특정 자식으로 한정
  - ex. Item 중에 Book, Movie를 조회해라

```sql
-- jpql
select i from Item i where type(i) in (Book, Movie)

-- 실제 날아가는 sql 쿼리
select i fron i where i.dtype in ('B', 'M')
```

- TREAT
  - 자바의 type casting과 유사
  - 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용
  - ex. 부모인 Item과 자식 Book

```sql
-- jpql
select i from Item i where treat(i as Book).auther = 'kim'

-- sql (싱글 테이블 전략인 경우)
select i.* from Item i where i.dtype = 'B' and i.auther = 'kim'
```

## 엔티티 직접 사용

### 기본 키 값
- jpql에서 엔티티를 직접 사용하면 sql에서 해당 엔티티의 기본키를 사용한다

```sql
-- jpql
select count(m.id) from Member m -- 엔티티의 아이디를 사용
select count(m) from Member m -- 엔티티를 직접 사용

-- sql
select count(m.id) as cnt from Member m
```

```java
// 엔티티를 파라미터로 전달
String jpql = "select m from Member m where m = :member";
List resultList = em.createQuery(jpql)
                    .setParameter("member", member)
                    .getResultList();

// 식별자를 직접 전달
String jpql = "select m from Member m where m.id = :memberId";
List resultList = em.createQuery(jpql)
                    .setParameter("memberId", memberId)
                    .getResultList();
```

```sql
-- 실행된 SQL
select m.* from Member m where m.id=?
```

### 외래 키 값

```java
Team team = em.find(Team.class, 1L);
String qlString = "select m from Member m where m.team = :team";
List resultList = em.createQuery(qlString)
                    .setParameter("team", team)
                    .getResultList();

String qlString = "select m from Member m where m.team.id = :teamId";
List resultList = em.createQuery(qlString)
                    .setParameter("teamId", teamId)
                    .getResultList();
```

```sql
-- 실행된 SQL
select m.* from Member m where m.team_id=?
```

## Named 쿼리

- 미리 정의해서 이름을 부여해두고 사용하는 jpql, 정적 쿼리만 가능
- 어노테이션, xml에 정의
- 애플리케이션 로딩 시점에 파싱 및 초기화 후 캐싱해둔다 -> 재사용
- **애플리케이션 로딩 시점에 jpa가 파싱 -> syntax error로 쿼리를 검증 가능**
- spring data JPA 에도 있다
  ```java
  public interface UserRepository extends JpaRepository<User, Long> {
    @Query("select u from User u where u.emailAddress = ?1")
    User findByEmailAddress(String emailAddress);
  }
  ```

```java
// 어노테이션으로 정의하는 경우 -> 엔티티 코드 더러워져서 비추.. spring data jpa 사용 추천
@Entity
@NamedQuery( // 이름으로 쿼리 불러와서 재활용 가능
  name = "Member.findByUsername", // 이름 이렇게 짓는게 관례 (엔티티명.쿼리이름)
  query = "select m from Member m where m.username = :username"
)
publid class Member { ... }

List<Member> resultList = 
  em.createNamedQuery("Member.findByUsername", Member.class)
    .setParameter("username", "회원1")
    .getResultList();

// xml에 정의하는 경우는 생략
// 어노테이션보다 xml이 우선권을 가진다. 바뀌는 상황에 맞춰 배포 시 편리하다고 함.
```

## 벌크 연산

- "재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면?"
- JPA 변경 감지(더티체킹) 기능으로 실행하려면 너무 많은 SQL 실행
  - ex. 100만건 전부 조회, for문 돌면서 가격 변경, 커밋시점에 변경감지 동작
  - 변경된 데이터가 100건이라면 100번의 UPDATE 문이 날라간다
  - 벌크 연산? 쿼리 하나로 100개를 UPDATE 가능
- 쿼리 한 번으로 여러 테이블 로우 변경(엔티티)
- `executeUpdate()` 결과 : 영향받은 엔티티 수 반환시
```java
int resultCount = em.createQuery("update Member m set m.age = 20")
                    .executeUpdate();
```
- UPDATE, DELETE 지원
- INSERT("insert into .. select" -> hibernate 지원)
- 벌크 연산은 **영속성 컨텍스트를 무시**하고 **DB에 직접 쿼리**
  - 해결법
    - 빈 영속성 컨텍스트 상태에서 벌크 연산을 가장 먼저 수행
    - 벌크 연산 수행 후 **즉시 영속성 컨텍스트 초기화 em.clear()** -> 데이터 정합성 바로 반영해주기
      - 연봉이 5천만원이었는데 벌크 연산 수행 후 6천만원으로 변경됨
      - 하지만 영속성 컨텍스트에는 아직 5천만원으로 남아있는 상태
      - 영속성 컨텍스트 초기화 시켜주면 이후 조회 시 정상적으로 6천만원으로 조회됨
- springDataJPA: @Modifying @Query
