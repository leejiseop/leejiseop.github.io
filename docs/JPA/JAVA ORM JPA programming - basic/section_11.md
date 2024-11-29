---
layout: default
title: 섹션 11
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 11
---

# 섹션 11
{: .no_toc }

1. TOC
{:toc}

## 소개

어떤 특정 조건으로 데이터를 뽑으려면 결국 SQL이 실행되어야한다.  
JPA는 다양한 쿼리 방법을 지원한다
- **JPQL**
- JPA Criteria
- **QueryDSL**
- 네이티브 SQL
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

### JPQL

```java
String jpql = "select m from Member m where m.age > 18";
List<Member> result = em.createQuery(jpql, Member.class)
    .getResultList();
```
JPA를 사용하면 엔티티 객체를 중심으로 개발... 문제는 검색 쿼리  
검색을 할 때도 테이블이 아닌 **엔티티 객체를 대상으로 검색**해야한다.  
모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능  
필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요  

이를 위해 JPA는 SQL을 추상화한 JPQL이라는 **객체지향 쿼리 언어**를 제공(**ANSI 표준 SQL** 지원)  
JPQL - **엔티티 객체**를 대상으로 쿼리, SQL을 추상화해서 특정 DB SQL에 의존 X, 즉 '객체지향 SQL'  
SQL - **DB 테이블**을 대상으로 쿼리  
엔티티를 대상으로 쿼리를 날리고, 엔티티의 매핑 정보를 읽어서 적절한 SQL을 만들어준다.  

### Criteria

JPQL은 결국 문자라서 동적쿼리 만들기 까다롭다  
(if username != null 일때 기존 쿼리에 where 절 문자열을 더해서 붙이고 띄어쓰기 신경쓰고 ... 등등등)  
java 표준에서 제공, JPA 공식 제공  
```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

Root<Member> m = query.from(Member.class);

CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
em.createQuery(cq).getResultList();
```
- 장점
  - java 코드로 짠다 -> 잘못 짜면 컴파일 오류로 검출가능
  - 동적쿼리 짜기 좋다. 쿼리를 코드 짜듯 분기문도 넣고..
  - JPQL 빌더 역할 (JPQL을 만들어내는)
- 단점
  - 길어지면 점점 복잡.. 알아보기 힘들다 -> 실무에서 잘 안씀
  - 이런게 있구나
- 그래서 Criteria 대신에 **QueryDSL 사용 권장**(오픈소스 라이브러리)

### [QueryDSL](http://querydsl.com/)

문자가 아닌 자바 코드로 JPQL 작성 가능(빌더 역할)  
컴파일 시점에 오류 확인 가능  
단순, 편리, **실무 사용 권장**  
JPQL 문법을 잘 알면, QueryDSL은 그냥 [메뉴얼](http://querydsl.com/) 보고 쓰면 된다. 메뉴얼 잘 되어있음.  

### 네이티브 SQL

```java
String sql = "SELECT ID, ... WHERE NAME = 'kim'";
em.createNativeQuery(sql, Member.class).getResultList();
```  
JPA가 제공하는 SQL 직접 사용 기능 -> JPQL로 해결할 수 없는 특정 DB에 의존적인 기능  
ex) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트  
사용할거라면 Spring이 제공하는 SpringJdbcTemplate를 사용하는게 더 편하다  
JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, SpringJdbcTemplate, MyBatis등을 함께 사용 가능  
단, 영속성 컨텍스트를 적절한 시점에 **강제로 `flush` 필요**
JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트 수동 플러시 하는게 좋다.  
- **flush가 동작**하는 경우
  - **commit 할 때**
  - **query 날릴 때** (선 flush 후 query를 날린다)

## 기본 문법과 쿼리 API

### 기본 문법

JPQL은 객체지향 쿼리 언어, 테이블이 아닌 엔티티 객체를 대상으로 쿼리한다.  
JPQL은 결국 SQL로 변환된다. (SQL의 추상화: 특정 DB에 종속되지 않는다)  
`select m from Member as m where m.age > 18`
엔티티와 속성은 대소문자를 구분한다(Member, age)  
JPQL 키워드는 대소문자 구분하지 않는다(SELECT, FROM, where)  
테이블이 아닌 엔티티 이름을 사용한다(기본은 클래스명, `@Entity(name = "asdf")`로 지정 가능)  
별칭 필수(m) (as는 생략 가능)  

### 집합과 정렬

group by, having, order by

### TypeQuery, Query

- TypeQuery: 반환 타입이 **명확할 때** 사용
- Query: 반환 타입이 **명확하지 않을 때** 사용

```java
TypedQuery<Member> query = 
    em.createQuery("SELECT m FROM Member m", Member.class);
Query query = 
    em.createQuery("SELECT m.username, m.age from Member m");

TypedQuery<Member> query1 = em.createQuery("select m from Member m", Member.class);
TypedQuery<String> query2 = em.createQuery("select m.username from Member m", String.class);
Query query3 = em.createQuery("select m.username, m.age from Member m");
```

### 결과 조회 API

- query.getResultList()
  - 결과가 하나 이상, 리스트 반환
  - 없으면 빈 리스트 반환
- query.getSingleResult()
  - 결과가 정확히 하나, 단일 객체 반환
  - 없으면 javax.persistence.NoResultException -> jakarta?
  - 둘 이상이면 javax.persistence.NonUniqueResultException -> try-catch 필요
  - spring data jpa에서는 결과가 없으면 exception 대신 null이나 optional 반환

### 파라미터 바인딩 - 이름 기준, 위치 기준

```java
// 이름 기준
SELECT m From Member m where m.username= :username
query.setParameter("username", usernameParam);

// 위치 기준 -> 비추천
SELECT m From Member m where m.username=?1
query.setParameter(1, usernameParam);

```

## 프로젝션(SELECT)

SELECT 절에 조회할 대상을 지정하는 것  
(엔티티, 임베디드 타입, 스칼라 타입(숫자 문자 등 기본 데이터 타입))  
관계형 DB에서는 스칼라 타입만 가능하다  
`select m from Member m` -> **엔티티** 프로젝션  
`select m.team from Member m` -> **엔티티** 프로젝션 -> **묵시적 join 주의 필요**  
(`"select t from Member m join m.team t", Team.class` -> **명시적 join**)  
`select m.address from Member m` -> **임베디드 타입** 프로젝션  
(임베디드 타입은 엔티티가 아니니 단독 조회는 불가능하다. 어디 소속인지 관계 명시 필요)  
`select m.username, m.age from Member m` -> **스칼라 타입** 프로젝션  
Type 떼고, DISTINCT로 중복 제거
**조회 결과는 영속성 컨텍스트에서 관리된다**

### 프로젝션 - 여러 값 조회

1. **Query 타입**으로 조회
2. **Object[] 타입**으로 조회
3. **new 명령어**로 조회 **(추천)** (표준으로 지원하는 기능)
    - 단순 값을 DTO로 바로 조회
    - `SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m`
    - 패키지 명을 포함한 전체 클래스 명 입력
    - 순서와 타입이 일치하는 생성자 필요

```java
// Query
// Object를 List 형태로 받아오고
List resultList = 
    em.createQuery("SELECT m.username, m.age from Member m").getResultList();
// List 안에 들어있는 Object를 꺼내고
Object o = resultList.get(0);
// 그걸 배열로 조회 가능하도록 Object[]로 캐스팅 후 접근
Object[] result = (Object[]) o;

System.out.println("username = " + result[0]);
System.out.println("age = " + result[1]);
```

```java
// TypeQuery 명시
// List로 가져오면서 동시에 캐스팅
List<Object[]> resultList = 
    em.createQuery("SELECT m.username, m.age from Member m").getResultList();
Object[] result = resultList.get(0);

System.out.println("username = " + result[0]);
System.out.println("age = " + result[1]);
```

```java
// new 명령어로 조회
// 코드가 너무 길어지면 -> QueryDSL로 해결 가능 (자바 코드로 짤 수 있다. 패키지명 import 가능)
List<MemberDTO> result = em.createQuery("select new jpql.MemberDTO(m.username, m.age) from Member m", MemberDTO.class)
    .getResultList();
MemberDTO memberDTO = result.get(0);
System.out.println("memberDTO.getUsername() = " + memberDTO.getUsername());
System.out.println("memberDTO.getAge() = " + memberDTO.getAge());
```

## 페이징

JPA의 **페이징 추상화 API**
**hibernate.dialect** 설정된 대로 적절히 **에뮬레이팅 되어 실행**된다
MySQL, Oracle 3-depth rownum ... 등등

- `setFirstResult(int startPosition)`: 조회 시작 위치 (0부터 시작)
- `setMaxResults(int maxResult)`: 조회할 데이터 수

```java
String jpql = "select m from Member m order by m.name desc";
List<Member> resultList = em.createQuery(jpql, Member.class)
    .setFirstResult(10) // 시작 지점
    .setMaxResults(20) // 몇 개
    .getResultList();
```

메모 - toString 무한루프 주의

## 조인

- 내부 조인(inner join)
  - `select m from Member m (inner) join m.team t`
- 외부 조인(outer join)
  - `select m from Member m left (outer) join m.team t`
- 세타 조인(from 절이 두 개, 카르테시안 곱?)
  - `select count(m) from Member m, Team t where m.username = t.name`

- ON 절
  - join 대상 **필터링**
  - ex) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인
```sql
-- JPQL
select m, t from Member m left join m.team t on t.name = 'A'
-- SQL
select m.*, t.* from Member m left join Team t on m.TEAM_ID=t.id and t.name = 'A'
```
  - **연관관계 없는 엔티티** 외부 조인(hibernate 5.1부터)
  - ex) 회원의 이름과 팀의 이름이 같은 대상 외부 조인
```sql
-- JPQL
select m, t from Member m left join Team t on m.username = t.name
-- SQL
select m.*, t.* from Member m left join Team t on m.username = t.name
```

## 서브 쿼리

나이가 평균보다 많은 회원
```sql
select m from Member m
where m.age > (select avg(m2.age) from Member m2)
-- 서브 쿼리와 메인 쿼리가 관계 없다
```

한 건이라도 주문한 고객
```sql
select m from Member m
where (select count(o) from Order o where m = o.member) > 0
-- 서브 쿼리가 메인 쿼리와 관계 있음 (성능 down)
```

### 서브 쿼리 지원 함수

- (not) exists (subquery): 서브쿼리에 결과가 존재하면 참
  - \{all \| any \| some\} (subquery)
  - all: 모두 만족하면 참
  - any, some: 같은 의미, 조건을 하나라도 만족하면 참
- (not) in (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

```sql
-- 팀A 소속인 회원
select m from Member m
where exists (select t from m.team t where t.name = '팀A')
-- 전체 상품 각각의 재고보다 주문량이 많은 주문들
select o from Order o
where o.orderAmount > ALL (select p.stockAmount from Product p)
-- 어떤 팀이든 팀에 소속된 회원
select m from Member m
where m.team = ANY (select t from Team t)
```

### JPA 서브 쿼리의 한계

- JPA는 where, having 절에서만 서브 쿼리 사용 가능
- select 절 서브 쿼리도 가능(JPA 표준 스펙은 아니지만, hibernate에서 지원)  
`select (select avg(m1.age) from Member m1) as avgAge from Member m join Team t ... `
- from 절의 서브 쿼리는 현재 JPQL에서 불가능
  - (가능하다면) join으로 풀어서 해결, 애플리케이션으로 가져와서 일부만 사용, 쿼리 두번 분해해서 날리기, 네이티브 sql
  - sql 내에 여러 로직(데이터 타입 변경, 문자 조작 등...)을 넣는 경우 from 절 서브쿼리가 자주 쓰인다
    - 그런건 분리해서 애플리케이션으로 가져오면 from 절 서브쿼리를 줄일 수 있다

## JPQL 타입 표현과 기타식

- 문자: 'HELLO', 'She''s'
- 숫자: 10L(Long), 10D(Double), 10F(Float)
- Boolean: TRUE, FALSE
- enum: jpabook.MemberType.Admin (패키지명 포함)
- 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)
  - `em.createQuery("select i from Item i where type(i) = Book", Item.class)`


## 조건식(CASE 등)



## JPQL 함수

