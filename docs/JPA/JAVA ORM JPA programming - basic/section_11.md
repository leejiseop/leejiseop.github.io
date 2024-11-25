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



## 페이징



## 조인



## 서브 쿼리



## JPQL 타입 표현과 기타식



## 조건식(CASE 등)



## JPQL 함수

