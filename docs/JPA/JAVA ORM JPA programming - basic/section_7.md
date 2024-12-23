---
layout: default
title: 7. 다양한 연관관계 매핑
parent: 자바 ORM 표준 JPA 프로그래밍 - 기본편
nav_order: 7
---

# 다양한 연관관계 매핑
{: .no_toc }

1. TOC
{:toc}

## 다대일 (N : 1)

![대다일1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FEsfbO%2FbtrvocxzxhH%2FThAbRPkXL2hRkll0TXGUQk%2Fimg.png)

![다대일2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb8XtgK%2FbtrvoPISnED%2FmNZJu182VGy8EV7nf9h681%2Fimg.png)

## 일대다 (1 : N)

![일대다](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcmt7jL%2Fbtrvxjveywa%2Fz2NrFisfYFV0WPgyUylYVk%2Fimg.png)

- **권장하지 않음**
- N이 아닌 1이 연관관계의 주인
  - 소수:다수 상황에서는 다수(MEMBERS) 테이블에 외래키가 들어가야만 한다
  - 엔티티가 관리하는 **외래 키가 다른 테이블에 있음**
  - 대신 **양방향 다대일 사용을 권장**(insert 관련 혼란 방지)
  - @JoinColumn을 꼭 사용해야 함.
    - 그렇지 않으면 **JoinTable(중간테이블 자동 생성)** 방식을 사용함
- 양방향도 가능(야매로)
  - `@JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)`
  - 읽기 전용 매핑

## 일대일 (1 : 1)

- **외래 키에 DB unique 제약조건 추가**

![일대일1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlXUfm%2FbtrvylsPjPa%2FzitYlHoRzlJiJkb8dZoV2K%2Fimg.png)

![일대일2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcsxCT8%2FbtrvvDOwfru%2FDG6nZTtYHN1y0A7Cn5DnG0%2Fimg.png)

![일대일3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbz0ht1%2FbtrvxlmjTHP%2FmlBvlaFfj5dVaOeOMci3Xk%2Fimg.png)

- 대상 테이블에 외래키 + 단방향 = **불가능**

![일대일4](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdA0Cka%2FbtrvwjhYlUg%2FMPTApiSUJeS7MUrImk8kZ1%2Fimg.png)

- **(주 테이블에 외래키 + 양방향)을 뒤집은 것과 같다**

- DBA의 입장
  - 테이블은 한번 만들면 나중에 바뀌기 힘들다
  - 비즈니스 룰이 바뀌면? (ex. 하나의 member가 여러개의 locker를 가지게 된다면)
  - [대상 테이블에 외래 키 양방향]에서 unique 제약조건만 빼면 된다 -> 여러개 insert 가능
  - [주 테이블에 외래키 단방향]에서는 테이블에 컬럼 추가하고 기능 변경도 해야함
- 개발자의 입장
  - MEMBER에 LOCKER_ID 외래키가 있는게 성능적으로 유리
- 너무 먼 미래는 생각하지 않는것이 좋다

- 정리
  - 주 테이블에 외래 키
    - 주로 객체지향 개발자 선호
    - JPA 매핑 편리
    - 장점: 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능 (join 불필요)
    - 단점: 값이 없으면 외래 키에 null 허용
  - 대상 테이블에 외래 키
    - 주로 DBA 선호
    - **양방향 필수**
    - 장점: 일대일에서 일대다 변경시 테이블 구조 유지
    - 단점: **프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨**
      - 값이 없으면 null을 넣어줘야 하는데...(프록시)
      - Member 로딩 - locker 유무 확인 필요 - MEMBER 테이블에 외래키 없음 - LOCKER 테이블 조회 필요(where)

## 다대다 (N : M)

- **실무에서 거의 사용하지 않음**

![다대다1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fn6tI3%2Fbtrvyll5oHz%2FrMwvPyFbKtY9AwZTKKyf20%2Fimg.png)

![다대다2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FeoAXx1%2Fbtrvyk1KUuP%2FLnDaYwzRqvniX0lOfW6rwk%2Fimg.png)

![다대다3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FXHy7q%2Fbtrvt2ncgis%2F6I4ReoituYtNxuR3wSg8Gk%2Fimg.png)

- **PK는** 의미없는 값을 **독립적으로** 만들어서 사용하는게 좋다

- **객체는 컬렉션을 사용해서 다대다 관계 가능**
- **중간 테이블 생성됨**
  - 단순히 연결만 하고 끝나지 않는다
  - 주문시간, 수량 같은 데이터가 들어올 수 있음
    - 하지만 **컬럼 추가 불가능**
  - 예상치 못한 쿼리가 나간다
- 연결 테이블용 엔티티 추가(연결 테이블을 엔티티로 승격)
  - **@ManyToMany -> @OneToMany + @ManyToOne**
    - 원하는 **컬럼 추가 가능**


## (예제) 다양한 연관관계 매핑

JPA는 셀프매핑이 가능하다!

```java
@Entity
public class Category {
    
    // id, name, ...

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Category parent;

    @OneToMany(mappedby = "parent")
    private List<Category> child = new ArrayList<>();
}
```

