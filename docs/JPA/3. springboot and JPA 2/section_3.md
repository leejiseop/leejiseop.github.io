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

## 페이징과 한계 돌파

- 

## 주문 조회 V4: JPA에서 DTO 직접 조회

- 

## 주문 조회 V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화

- 

## 주문 조회 V6: JPA에서 DTO로 직접 조회, 플랫 데이터 최적화

- 

## API 개발 고급 정리

- 
