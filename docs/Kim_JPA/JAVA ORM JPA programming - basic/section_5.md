---
layout: default
title: 5. 엔티티 매핑
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 5
---

# 엔티티 매핑
{: .no_toc }

1. TOC
{:toc}

- 객체와 테이블 매핑: @Entity, @Table
- 필드와 컬럼 매핑: @Column
- 기본 키 매핑: @Id
- 연관관계 매핑: @ManyToOne, @JoinColumn

## 객체와 테이블 매핑

- @Entity
  - 기본 생성자 필수(파라미터가 없는 public or protected 생성자)
  - final class, enum, interface, inner class 는 매핑 불가
  - 저장할 필드에 final 사용 불가

## 데이터베이스 스키마 자동 생성

운영보다는 개발 단계일때 사용  
애플리케이션 실행 시점에 DDL 자동 생성  
생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용  
`<property name="hibernate.hbm2ddl.auto" value="create"/>`  
 -> 애플리케이션 로딩 시점에 기존 테이블 다 지우고 다시 만듬  
 -> 개발하고 또 DB가서 테이블 만들고 왔다갔다 할 필요가 없음  

- `create`: 기존 테이블 삭제 후 다시 생성(drop + create) **(운영DB에 사용하면 안됨)**
- `create-drop`: create와 같으나 종료시점에 drop **(운영DB에 사용하면 안됨)**
- `update`: alter table로 변경분만 반영(변경분 추가만 가능) **(운영DB에 사용하면 안됨)**
- `validate`: 엔티티-테이블 정상 매핑 되었는지만 확인
- `none`: 사용하지 않음(아무렇게나 적은거와 같은데 관례상 none이라고 써둠)

{: .highlight}
**개발 초기단계**는 `create` or `update`  
**테스트 서버**는 `update` or `validate`  
**스테이징**과 **운영 서버**는 `validate` or `none`  
(그래도 validate 외 다른 옵션 사용은 가급적 비추.. DB lock 걸리는 실수 방지)
ddl 직접 짜서 DB에 직접 적용해서 테스트후 문제없으면 검수 후 운영에 적용하는걸 권장
개인 로컬 PC에서만 자유롭게 사용

{: .warning }
운영 장비에는 절대 `create`, `create-drop`, `update` 사용하지 말 것

- 제약조건 추가 (unique 제약조건, `@Column(nullable = false, length = 10)` 등)

DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고  
JPA의 살행 로직(성능)에는 영향을 주지 않는다  

## 필드와 컬럼 매핑

`@Column`, `@Enumerated`, `@Temporal`. `@Lob`, `@Transient`
- `@Column`
  - name: 필드와 매핑할 테이블의 컬럼명
  - insertable, updatable: 등록 변경 가능 여부
  - nullable(DDL): null or not null
  - unique(DDL): 한 칼럼에 unique 제약조건 -> 이름이 복잡하게 떠서 운영에서는 잘 안씀
    - 대신 `@Table(uniqueConstraints = )`를 사용하면 이름까지 직접 부여 가능
  - columnDefinition(DDL): DB의 컬럼 정보 직접 설정
  - length(DDL): 문자 길이 제약조건(String)
  - precision, scale(DDL): 아주 큰 숫자나 소수점

- `@Enumerated`
  - 자바 enum 매핑시 사용

    {: .warning}
    ORDINAL 사용하면 안됨!! String으로 사용해야 유연하게 변경 가능하다

- `@Temporal`
  - 최근엔 잘 안쓴다
    - (Java 8 이후 -> 어노테이션 따로 없어도 LocalDateTime -> 최신 hibernate가 지원)

- `@Lob`
  - 속성 없음
  - 문자면 `clob`, 나머지는 `blob` 매핑

- `@Transient`
  - 필드 매핑 X, DB 저장 및 조회 X
  - 메모리 상에서만 임시로 사용하고 싶은 필드

## 기본 키 매핑

- `@Id`
  - 넉넉하게 Long 권장 -> 이후 변경 어렵기 때문에 미리 Long으로
- `@GeneratedVallue`
  - `GenerationType.AUTO`
    - DB 방언에 맞춰서 아래 **3가지 중 하나가 자동 선택**됨
  - `GenerationType.IDENTITY`
    - 기본 키 생성을 **DB에 위임** (mysql -> auto_increment)
    - JPA는 보통 트랜잭션 commit 시점에 insert sql 실행
    - auto_increment는 **insert sql 실행 이후에 id 값을 알 수 있음**
    - 그런데, 영속성 컨텍스트를 사용하려면 id(PK)값이 필요하다(?)
      - **그래서 em.persist 호출시 -> 바로 insert 날리고 id값을 가져와서 영속성 컨텍스트 세팅**
        - 내부 로직 상 return 으로 id값을 받아오기 때문에, 추가 select 쿼리는 날리지 않는다
  - `GenerationType.SEQUENCE`
    - oracle에서 자주 사용
    - 기본적으로는 hibernate가 만든 **hibernate_sequence**를 사용하여 관리
    - **Table마다 시퀀스를 따로 관리**하고싶으면 `@SequenceGenerator` 사용
        ```java
        @Entity
        @SequenceGenerator(
            name = "MEMBER_SEQ_GENERATOR", // 사용할 식별자 생성기 이름
            sequenceName = "MEMBER_SEQ", // db에 등록되는 시퀀스 이름
            initialValue = 1, // ddl 생성시에만 사용됨
            allocationSize = 1 // 시퀀스 한번 호출에 증가하는 수 -> 최적화에 사용됨, 기본값 50
            // db의 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 allocationSize를 반드시 1로 설정해야한다
            )
        public class Member {
            @Id
            @GeneratedValue(
                strategy = GenerationType.SEQUENCE,
                generator = "MEMBER_SEQ_GENERATOR")
            private Long id;
        }
        ```
    - **em.persist 호출시 db에게서 값을 얻어온다 -> `call next value for MEMBER_SEQ`**
      - IDENTITY와는 달리 insert query는 날아가지 않는다
      - 버퍼링 가능
      - allocationSize로 한번에 DB에서 가져와서 이후부턴 메모리에서 호출 -> 동시성 이슈 없음
        - `create sequence MEMBER_SEQ start with 1 increment by 50`
        - 너무 크게 잡으면 사용하지 않는 범위 손해볼 수도 있음
        - 50 ~ 100 추천
  - `GenerationType.TABLE`
    - 키 생성 전용 **테이블을 만들어서 시퀀스를 흉내**
    - 장점: 모든 DB에 사용 가능
    - 단점: 성능, 락 위험, 운영에서는 부담 -> 비추.. 잘 안씀
        ```java
        @Entity
        @TableGenerator(
            name = "MEMBER_SEQ_GENERATOR", // 사용할 식별자 생성기 이름
            table = "MY_SEQUENCES", // 생성 테이블명
            pkColumnValue = "MEMBER_SEQ", // 키로 사용할 값 이름
            allocationSize = 1
            )
        public class Member {
            @Id
            @GeneratedValue(
                strategy = GenerationType.TABLE,
                generator = "MEMBER_SEQ_GENERATOR")
            private Long id;
        }
        ```
  - **기본 키 제약 조건:** not null, 유일, **불변**
    - 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다 -> 대리키(대체키) 사용
      - ex. 주민번호는 기본 키로 적절하지 않다 -> 향후 정책 변경 가능성
    - 권장: Long형 + 대체키 + 키 생성전략 사용
    - **비즈니스를 key로 끌고오지 말 것**

