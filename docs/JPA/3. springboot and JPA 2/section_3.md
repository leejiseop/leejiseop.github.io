---
layout: default
title: 3. 컬렉션 조회 최적화
parent: 실전! 스프링부트와 JPA 활용2
---

# 3. 컬렉션 조회 최적화
{: .no_toc }

1. TOC
{:toc}

## 주문 조회 V1: 엔티티 직접 노출

- 생략

## 주문 조회 V2: 엔티티를 DTO로 변환

- DTO를 만들 때 컬렉션도 각각 별도의 DTO로 감싼다(stream 이용)

## 주문 조회 V3: 엔티티를 DTO로 변환 - fetch join 최적화

```java
public List<Order> findAllWithItem() {
    return em.createQuery(
            "select distinct o from Order o" +
                    " join fetch o.member m" +
                    " join fetch o.delivery d" +
                    " join fetch o.orderItems oi" +
                    " join fetch oi.item i", Order.class)
            .getResultList();
}
```

```bash
# distinct를 쓰지 않으면 같은 객체가 두개씩 총 4개 출력된다
orderDto = OrderApiController.OrderDto(orderId=4, name=userA, orderDate=2025-03-31T22:52:31.211014, orderStatus=ORDER, address=jpabook.jpashop.domain.Address@91a317f, orderItems=[OrderApiController.OrderItemDto(itemName=JPA1 BOOK, orderPrice=10000, count=1), OrderApiController.OrderItemDto(itemName=JPA2 BOOK, orderPrice=20000, count=2)])
orderDto = OrderApiController.OrderDto(orderId=4, name=userA, orderDate=2025-03-31T22:52:31.211014, orderStatus=ORDER, address=jpabook.jpashop.domain.Address@91a317f, orderItems=[OrderApiController.OrderItemDto(itemName=JPA1 BOOK, orderPrice=10000, count=1), OrderApiController.OrderItemDto(itemName=JPA2 BOOK, orderPrice=20000, count=2)])
orderDto = OrderApiController.OrderDto(orderId=11, name=userB, orderDate=2025-03-31T22:52:31.225743, orderStatus=ORDER, address=jpabook.jpashop.domain.Address@281525e, orderItems=[OrderApiController.OrderItemDto(itemName=SPRING1 BOOK, orderPrice=20000, count=3), OrderApiController.OrderItemDto(itemName=SPRING2 BOOK, orderPrice=40000, count=4)])
orderDto = OrderApiController.OrderDto(orderId=11, name=userB, orderDate=2025-03-31T22:52:31.225743, orderStatus=ORDER, address=jpabook.jpashop.domain.Address@281525e, orderItems=[OrderApiController.OrderItemDto(itemName=SPRING1 BOOK, orderPrice=20000, count=3), OrderApiController.OrderItemDto(itemName=SPRING2 BOOK, orderPrice=40000, count=4)])
```

- `@OneToMany` 매핑의 경우 데이터가 뻥튀기됨
  - **객체 중복 매핑**
- 쿼리 날릴때 `distinct` 키워드 추가하여 중복 객체 제거
  - DB의 `distinct`는 row가 정말 똑같아야 중복 제거
  - **DB 쿼리 결과는 차이없고** JPA 자체적으로 값을 매핑할때 **같은 객체 id값이면 중복 제거**(컬렉션 담을 때)
  - **단점: 페이징 불가**
    - **페이징 쿼리가 아예 안나감**
    - `firstResult/maxResults specified with collection fetch; applying in memory!`
      - **데이터를 전부 가져와서 메모리에서 페이징 처리**하겠다
      - 데이터가 많은 경우 `out of memory` 문제 생길수도
    - 우리가 기대하는 order: **2개**
    - 실제 데이터(쿼리 결과): **4개**
    - 여기에서 오는 불일치..
    - 일대다 fetch join에서는 페이징을 하면 안된다
      - **대안이 있다**
- 컬렉션 fetch join은 1개만 사용 가능
  - 여러개 쓰면 데이터가 1 * N * N * ... 으로 뻥튀기된다

## 주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파

- ~ToOne은 일단 다 가져와놓고
- application.yaml
  - jpa
    - properties
      - default_batch_fetch_size: 100
- order 내부의 orderItems 컬렉션: @OneToMany
- where ~ in ~ 쿼리가 날라간다
  - select ... from order_item where order_id in (4, 11);
- default_batch_fetch_size: 몇 개의 in 쿼리를 날릴건지
  - 설정한 값만큼 루프를 돌면서 가져온다

- 이정도만 해도 웬만한 성능이 나온다
- 이 이상의 최적화는 레디스 캐시 등 활용

- v3 `fetch join`
  - 데이터가 N만큼 한번에 뻥튀기됨
  - 용량 이슈 생길수도
  - 페이징 불가
- v3.1 `defaultBatchSize` = `@Batchsize(size = 100)`
  - 쿼리는 3번 나가지만(여러번 나눠서 가져온다)
  - 한방에 가져오는 데이터 양은 줄어들고
  - 중복 없이 정확한 데이터들만 가져온다(정규화와 비슷)
  - 페이징 가능
- 상황에 따라 다르다

## 주문 조회 V4 V5 V6 ...

- 핵심: ToOne 관계를 먼저 조회하고 + 거기서 얻은 id들로 ToMany 관계를 한번에 가져옴

## **API 개발 고급 정리**

- 엔티티 조회
  - 엔티티를 조회해서 그대로 반환: **V1**
    - 엔티티 스펙이 변하면 api 스펙도 변해버린다(위험)
  - 엔티티 조회 후 DTO로 변환: **V2**
    - V1 문제 예방
    - 지연 로딩시 여러 테이블 join으로 성능이 잘 안나옴
  - 페치 조인으로 쿼리 수 최적화: **V3**
    - 한번에 다 가져와서 N+1 문제 해결
    - 한계: 컬렉션 페이징 불가
  - 컬렉션 페이징과 한계 돌파: **V3.1**
    - 컬렉션은 페치 조인시 페이징이 불가능
    - **ToOne 관계는 fetch join으로 쿼리 수 최적화**
    - **컬렉션은 fetch join 대신에 지연 로딩을 유지**하고, `hibernate.default_batch_fetch_size` or **`@BatchSize`로 최적화**
- DTO 직접 조회
  - JPA에서 DTO를 직접 조회: **V4**
  - 컬렉션 조회 최적화 - 일대다 관계인 컬렉션은 IN 절을 활용해서 메모리에 미리 조회해서 최적화: **V5**
    - 
  - 플랫 데이터 최적화 - JOIN 결과를 그대로 조회 후 애플리케이션에서 원하는 모양으로 직접 변환: **V6**
    - 전부 램에 올린다
- redis, 로컬 메모리 캐싱 등... 엔티티는 캐싱하면 안된다!
  - 영속성 컨텍스트가 꼬일수도
  - **캐싱하려면 DTO로**