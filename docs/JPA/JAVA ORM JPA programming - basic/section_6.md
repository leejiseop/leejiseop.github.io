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

- **테이블은 변함 없다.** 문제는 객체.
  - 일단 단방향 설계하고 나중에 필요하면 양방향 추가해도 테이블은 변화 없다
- 연관관계의 주인과 **mappedBy**
- **양방향 관계**는 사실 **서로 다른 단방향 관계 2개다**
  - **사실 단방향 매핑 만으로도 이미 연관관계 매핑은 완료**
- 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리한다
  - 딜레마: `Member`와 `Team` 둘 중 하나로 외래키를 관리해야 한다
  - `Member`의 `team`을 변경했을때 `MEMBER`의 `TEAM_ID`가 update 되어야 하나?
  - 아니면 `Team`의 `members`가 변경되었을 때 `MEMBER`의 `TEAM_ID`가 update 되어야 하나?
  - **연관관계의 주인만이 외래 키를 등록, 수정할 수 있다**
  - **주인이 아닌 쪽은 읽기만 가능** (객체 그래프 탐색용)
    - JPQL에서 역방향으로 탐색할 일이 많다
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
- **누구를 주인으로?**
  - **외래키가 있는곳을 주인으로 정해야 혼란이 적다 + 성능향상**
  - 소수:다수의 경우 보통 다수쪽에 외래키가 있게 된다.

## 양방향 연관관계와 **연관관계의 주인** (주의점, 정리)

- **연관관계의 주인에게 값을 넣어야 DB에 반영**이 된다!
- 하지만 한쪽에만 값을 넣으면 조회시 헷갈릴 수 있으므로 **양쪽 다 넣어주는게 좋다**
  - 양방향 매핑 시 **무한루프 조심**
    - **toString(), lombok, JSON 생성 라이브러리**
      - JSON: 컨트롤러에서는 **entity 반환하지 말 것**
        - 이후 api 스펙 변경 문제도 있고 등등..
        - **DTO**로 변환해서 반환
  - **연관관계 편의 메소드 작성**
    - **둘 중에 하나**를 구현하여 사용
    - 양쪽 다 만들면 나중에 **무한루프** 걸릴 수도 있다

    ```java
    // Member
    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }

    // Team
    public void addMember(Member member) {
        this.members.add(member);
        member.setTeam(this);
    }
    ```
