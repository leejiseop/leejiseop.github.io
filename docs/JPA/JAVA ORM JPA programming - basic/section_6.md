---
layout: default
title: 6. 연관관계 매핑 기초
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 6
---

# 연관관계 매핑 기초
{: .no_toc }

1. TOC
{:toc}

{: .highlight}
객체지향 설계의 목표는 **자율적인 객체들의 협력 공동체**를 만드는 것이다.

- 객체의 참조와 테이블의 외래 키를 매핑ㅊ

## 단방향 연관관계

- 테이블과 객체의 차이
  - 테이블은 외래키로 join을 사용해서 연관된 테이블을 찾는다
  - 객체는 참조를 사용해서 연관된 객체를 찾는다
- teamId 대신 team 참조값을 그대로 사용

## 양방향 연관관계와 **연관관계의 주인** (기본)

- 테이블은 변함 없다. 문제는 객체.
- 연관관계의 주인과 **mappedBy**
- 양방향 관계는 사실 서로 다른 단방향 관계 2개다
- 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리한다
  - 딜레마: `Member`와 `Team` 둘 중 하나로 외래키를 관리해야 한다
  - `Member`의 `team`을 변경했을때 `MEMBER`의 `TEAM_ID`가 update 되어야 하나?
  - 아니면 `Team`의 `members`가 변경되었을 때 `MEMBER`의 `TEAM_ID`가 update 되어야 하나?
  - **연관관계의 주인만이 외래 키를 등록, 수정할 수 있다**
  - **주인이 아닌 쪽은 읽기만 가능**
  - 주인은 `mappedBy` X
  - 주인이 아니면 `mappedBy`로 주인 지정 O
```java
// Member
@ManyToOne
@JoinColumn(name = "TEAM_ID") // 연관관계의 주인, FK를 관리.
private Team team;

// Team
@OneToMany(mappedBy = "team") // 조회만 가능, 변경은 불가. (애초에 참조를 안함)
private List<Member> members = new ArrayList<>();
```
- **누구를 주인으로? -> 외래키가 있는곳을 주인으로 정해야 혼란이 적다 + 성능향상**
  - 소수:다수의 경우 보통 다수쪽에 외래키가 있게 된다.

## 양방향 연관관계와 **연관관계의 주인** (주의점, 정리)



## (예제) 연관관계 매핑 시작

