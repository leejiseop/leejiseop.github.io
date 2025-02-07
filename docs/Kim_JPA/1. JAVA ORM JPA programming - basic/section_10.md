---
layout: default
title: 10. 값 타입
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 10
---

# 값 타입
{: .no_toc }

1. TOC
{:toc}

## JPA의 데이터 분류

JPA는 데이터를 두 가지 타입으로 분류한다
- **엔티티 타입**
  - @Entity
  - 데이터가 변해도 식별자로 **지속 추적 가능**
    - 회원 엔티티의 키나 나이 값을 변경해도 식별자로 엔티티 인식
- **값타입**
  - int Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입 또는 객체
  - 식별자 없고 값만 있음 -> **변경시 추적 불가**
  - 3가지 타입 종류
    - **기본값 타입**
      - 자바 기본 타입(int, double)
      - wrapper class(Integer, Long)
      - String
    - **임베디드 타입**(복합 값 타입)
      - ex) x, y 좌표를 묶은 position이라는 class, 커스텀 포지션. 기본 생성자 필수
    - **컬렉션 값 타입**(자바 Collection)


## 기본값 타입

ex) String name, int age  
- **생명주기를 엔티티에 의존**
  - 회원을 삭제하면 이름, 나이 필드도 함께 삭제
- 값 타입은 공유되면 안된다
  - 회원 이름 변경시 다른 회원 이름도 함께 변경되면 안됨
- 자바의 기본 타입은 절대 공유되지 않는다
  - 기본 타입은 항상 값을 복사함
  - Integer같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체이지만 변경 X
    - Integer class는 초기화 이후 새로운 값으로 set 안됨

## 임베디드 타입(복합 값 타입)

- **@Embeddable** 값 타입 **정의**하는 곳에 표시
- **@Embedded** 값 타입 **사용**하는 곳에 표시
- 재사용 가능, 응집도 높음, 해당 값 타입만 사용하는 메소드 생성 가능(객체지향적)
- **매핑 전후 테이블 변화는 없음**
- 잘 설계한 ORM 어플리케이션들은 테이블 수보다 매핑 수가 더 많다
- 중복 사용시에는 @AttributeOverride


## 값 타입과 불변 객체

### 값 타입 공유 참조

임베디드 타입 같은 값 타입을 **여러 엔티티에서 공유할 수도 있다** (위험, 부작용 발생)  
회원 1과 회원 2가 하나의 임베디드 타입을 참조 -> **하나를 바꿨는데 UPDATE 쿼리가 여러개**  
같이 뭔가를 **공유해서 쓸 의도라면** 임베디드 타입을 만들게 아니라 **엔티티를 하나 만들어야 한다**  
임베디드 값은 새로 만들고 내용을 복사해서 사용한다  
하지만 결국 실수는 나오기 마련이고 값 타입(객체) 공유 참조는 피할 수 없다  
그래서 객체 타입을 **수정할 수 없게 만들면 부작용이 원천 차단**된다.  
**불변 객체(Immutable object)**: 생성 시점 이후 절대 **값을 변경할 수 없는 객체**  
**setter를 만들지 않으면 됨** (또는 private final ... 등)  
Integer, String: 자바가 제공하는 대표적인 불변 객체  
**그리고 변경이 필요하면 아예 새로 만들어서 새로 할당하라**
```java
// home_city -> new_city
// X
findMember.getHomeAddress().setCity("new_city");
// O - HomeAddress를 아예 갈아끼운다
Address a = findMember.getHomeAddress();
findMember.setHomeAddress(new Address("new_city", a.getStreet(), a.getZipcode()));

// 치킨 -> 한식
// 통째로 뺘고 갈아끼운다?
findMember.getFavoriteFoods().remove("치킨");
findMember.getFavoriteFoods().add("한식");

// equals 를 사용한다. 컬렉션에 equals 메서드 구현 잘 되어있나 확인 필요
findMember.getAddressHistory().remove(new Address("old1", "strees", "10000"));

```

## 값 타입의 비교

값 타입 : 인스턴스가 달라도 그 안의 값이 같으면 true로 나온다  
객체 타입 : 단순 비교하면 false  
**동일성 비교** (identity): 인스턴스의 참조값을 비교 **==**  
**동등성 비교** (equivalence): 인스턴스의 값을 비교, **equals()**  
**equals() 메소드는 기본적으로 == 비교**라서, **오버라이드 후 재정의가 필요**하다  

## 값 타입 컬렉션

하나 이상의 값 타입을 컬렉션에 담아서 쓴다 -> 테이블이 필요, 엔티티는 아님  
```java
@ElementCollection
@CollectionTable(name = "ADDRESS", joinColumns =
    @JoinColumn(name = "MEMBER_ID")
)
private List<Address> addressHistory = new ArrayList<>();
```
값 타입 컬렉션은 **엔티티에 종속되어 생명주기를 공유**한다 (Cascade와 비슷)  
컬렉션은 지연 로딩이 default  

값 타입은 엔티티와 다르게 **식별자 개념이 없어서 변경시 추적이 어렵다**  
변경이 발생하면 주인 엔티티와 연관된 **모든 데이터를 삭제**하고,  
값 타입 컬렉션에 있는 현재 값을 **모두 다시 저장**한다.  

값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본키를 구성해야한다.
- null 입력 x
- 중복 저장 x

복잡하니 대안을 쓴다  
실무에서는 상황에 따라 값 타입 컬렉션 대신 **일대다 관계 사용을 고려**한다.
자체적인 ID 값이 있으니 값 타입이 아닌 엔티티이다.  
"값 타입을 엔티티로 승극한다. 올려서 쓴다." 실무에서 이 방법을 많이 쓴다.

그럼 값 타입 컬렉션은 언제 쓰나?
- 진짜 단순할 때. 추척할 필요도 없고, 값이 바뀌어도 update 할 필요가 없을 때
- 이럴때가 아니면 웬만하면 엔티티로 승격하여 사용하는게 좋다.

## 정리

- 엔티티
  - 식별자 O
  - 생명 주기 관리
  - 공유
- 값 타입
  - 식별자 X
  - 생명 주기가 엔티티에 의존
  - 공유하지 않는 것이 안전(복사해서 사용)
  - 혹시 모를 공유에 대비해 불변 객체로 만드는 것이 안전
- 값 타입은 정말 값 타입을 써야한다고 판단될 때만 사용



## 값 타입 매핑

**equals와 hashCode 생성**할 때 `Use getters when available` 선택  
그래야 **프록시 객체**에서도 정상 작동함  
**메서드를 통하는 과정**이 있어야 **값을 불러온다**  
평소에 이런 습관을 들어두는게 안전한 편  
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Address address = (Address) o;
    return Objects.equals(getCity(), address.getCity()) && Objects.equals(getStreet(), address.getStreet()) && Objects.equals(getZipcode(), address.getZipcode());
}

@Override
public int hashCode() {
    return Objects.hash(getCity(), getStreet(), getZipcode());
}
```

값 타입을 UML에서는 스테레오 타입 이라고도 부른다?  
\<<Value Type\>> << >>