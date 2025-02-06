---
layout: default
title: 9. 프록시와 연관관계 관리
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 9
---

# 프록시와 연관관계 관리
{: .no_toc }

1. TOC
{:toc}

## 프록시

- em.find : DB를 통해서 실제 엔티티 객체 조회
- em.getReference() : DB 조회를 미루는 **가짜(프록시)** 엔티티 객체 조회  
  ex) hellojpa.Member**\$HibernateProxy\$ttLMHZRq** (실제 사용될 때 쿼리가 나간다)

실제 엔티티를 상속받아서 껍데기는 같은데 속은 비어있는 프록시 객체  
실제 객체의 참조(target)를 보관한다  
`조회 -> target 비어있음 -> 영속성 컨텍스트로 초기화 요청`  
`-> DB조회 -> 실제 엔티티 생성 -> 프록시의 target과 연결 -> 값 반환`  
초기화? DB를 통해서 진짜 값을 가져와서 진짜 엔티티를 만들어내는 과정. 한 번만 한다  
프록시 객체가 실제 엔티티로 대체되는게 아니라, **프록시 객체를 통해서 실제 엔티티에 접근**하는것이다.  
프록시 객체는 원본 엔티티를 상속받는다. **타입 체크시 ==이 아닌 instance of를 사용해야함**  

조회 시 영속성 컨텍스트에 찾는 엔티티가 이미 있으면  
getReference 호출해도 프록시 객체 대신 실제 엔티티 반환  
애초에 프록시든 실제 객체든 상관없이 한 영속성 컨텍스트 안에서 가져온거고,  
**PK가 같으면 == 비교해도 true를 보장한다. 그게 JPA의 기본 정책**  
(한 트랙잭션 안에서는 같은걸 보장해준다)  
같은 이유로 영속성 컨텍스트에 이미 프록시 객체가 있으면, 이후에 em.find로 조회해도 프록시 객체를 반환해준다. ==을 보장해야하기 때문.  
- 영속성 컨텍스트의 도움을 받을 수 없는 **준영속 상태**일 때 프록시를 초기화하면 문제가 발생한다.  
프록시 인스턴스 초기화 여부 확인 `emf.getPersistenceUnitUtil().isLoaded(refMember)`  
프록시 강제 초기화 `Hibernate.initialize(refMember)` (JPA 표준은 강제 초기화 제공 X)

## 즉시 로딩과 지연 로딩

```java
public class Member extends BaseEntity{
    // 중략
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID") // db 입장에서 join 할때 필요한 column 명
    private Team team;
    // 중략
}
```
`@ManyToOne(fetch = FetchType.LAZY)` 가 붙은 `Team` 이 바로 프록시 객체 조회  
`Team` 을 **실제로 사용할 때** `Team` 이 조회가 된다.  
`EAGER` 는 **JOIN** 해서 **즉시로딩**으로 가져온다.  
**항상 LAZY loading을 사용할 것**  
즉시로딩은 **전혀 예상치 못한 쿼리**를 날릴 수 있고, **N+1 문제**를 야기할 수 있다.  
- **N+1 문제 해결법** : LAZY loading, **JPQL fetch join**, entity graph annotation, batch size  
fetch join은 왜 n+1이 안생기나? 이거랑 eager의 차이

@ManyToOne, @OneToOne = **EAGER** default  
@OneToMany, @ManyToMany = **LAZY** default

## 영속성 전이: CASCADE

즉시로딩, 지연로딩, 연관관계 세팅과 전혀 상관 없다!  
`@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)`  
하나를 persist하면 줄줄이 이어서  
**Life cycle이 유사하고, 명확하게 하나의 소유자와 종속 관계가 있을때** 사용 (게시판과 첨부파일)

## 고아 객체와 생명주기

부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제  
`@OneToMany(mappedBy = "parent", orphanRemoval = true)`  
```java
Parent parent1 = em.find(Parent.class, id);
parent1.getChildren().remove(0);
// 자식 엔티티를 컬렉션에서 제거
// DELETE FROM CHILD WHERE ID = ?
```
**참조하는 곳이 하나일 때** 사용해야함! (게시판과 첨부파일)  
**특정 엔티티가 개인 소유할 때** 사용
**부모를 제거할때도 자식들까지 제거된다**

영속성 전이 + 고아객체 모두 활성화하면  
부모 엔티티를 통해서 자식의 **생명주기**를 관리할 수 있음 - DDD의 Aggregate Root 개념 구혈할 때 유용

