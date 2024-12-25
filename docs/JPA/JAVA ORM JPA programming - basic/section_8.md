---
layout: default
title: 8. 고급 매핑
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 8
---

# 고급 매핑
{: .no_toc }

1. TOC
{:toc}

## 상속관계 매핑

- Item
  - Album
  - Movie
  - Book

- DB에는 상속관계가 없다
- 슈퍼타입, 서브타입 관계라는 모델링 기법이 객체 상속과 유사
- 상속관계 매핑: 객체의 상속과 구조와 DB의 슈퍼타입 서브타입 관계를 매핑

- 슈퍼타입, 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법
- **ITEM은 추상테이블**로 생성하는게 맞다
  - 각각 테이블로 변환 -> **JOIN 전략 (중요할 때)**
    - ITEM 테이블을 만들고 **ALBUM, MOVIE, BOOK 테이블과 JOIN**
    - ITEM 테이블 안에 **DTYPE** 컬럼을 둬서 구분
    - 장점: 정규화, 외래키 참조 뮤결성 제약조건, 저장공간 효율
    - 단점: 조회시 JOIN 다수 사용 -> 성능 저하, 쿼리 복잡, 저장시 insert query 2회 호출
  - 통합 테이블로 변환 -> **단일 테이블 전략 (단순할 때)**
    - ITEM **한 테이블**에 각종 컬럼들 싹다 모아놓는다
    - ITEM 테이블 안에 **DTYPE** 컬럼을 둬서 구분 
    - 장점: JOIN 필요없음 -> 성능 빠름, 단순
    - 단점: null 허용 -> 무결성 애매해짐, 테이블 거대화 -> 오히려 느려질 수도 있음
  - 서브타입 테이블로 변환 -> **구현 클래스마다 테이블 전략 (비추)**
    - ITEM 테이블을 만들지 않고 ALBUM, MOVIE, BOOK **독립적인 테이블을 각각 구현**
    - 사용할땐 단순해서 좋지만 **조회할때 union all**로 테이블 다 찾아봐야함 -> 비효율적
    - 장점: 명확한 구분, not null 가능
    - 단점: 여러 테이블 조회시 성능 느려짐(JOIN), 자식 테이블들에 통합 쿼리 어려움
- 어떻게 구현하든 객체 입장에선 다 똑같다

### 주요 어노테이션

- `@Inheritance(strategy = InheritanceType.XXX)`
  - `JOINED`
  - `SINGLE_TABLE`
  - `TABLE_PER_CLASS`
- `@DiscriminatorColumn(name = "DTYPE")`
- `@DiscriminatorValue("XXX")`

## Mapped Superclass(매핑 정보 상속)

**상속관계 매핑과는 다르다**

- (객체 입장에서) Member, Seller, Team ... 계속 나오는 공통 속성
- 공통 매핑 `BaseEntity`로 묶어서 관리하고싶다
- **매핑 정보만 받는 부모 클래스**
  - 조회, 검색 불가
  - 직접 생성해서 사용할 일 없다
    - **추상 클래스 권장**
    - extends: `@Entity` or `@MappedSuperClass` 가능

```java
// BaseEntity
@MappedSuperClass // Entity가 아니다
public abstract class BaseEntity {

  @Column(name = "INSERT_MEMBER")
  private String createdBy;

  private LocalDateTime createdDate;

  @Column(name = "UPDATE_MEMBER")
  private String lastModifiedBy;
  
  private LocalDateTime lastModifiedDate;

  // ... getter setter
}

// Member
@Entity
public class Member extends BaseEntity {

}

// Seller
@Entity
public class Seller extends BaseEntity {

}

// Team
@Entity
public class Team extends BaseEntity {

}
```

## (예제) 상속관계 매핑

