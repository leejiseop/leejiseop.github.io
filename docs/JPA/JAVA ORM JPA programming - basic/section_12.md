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

**Fetch join - 실무에서 엄청 중요함**

- SQL join 종류 X
- JPQL에서 **성능 최적화**를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 SQL 한번에 함께 조회하는 기능
- join fetch 명령어 사용
- `페치 조인 ::= [ left [ outer ] | inner ] join fetch 조인경로`

## 페치 조인 2 - 한계



## 다형성 쿼리



## 엔티티 직접 사용



## Named 쿼리



## 벌크 연산

