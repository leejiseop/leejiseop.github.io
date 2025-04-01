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
    - 페이징 쿼리가 아예 안나감
    - `firstResult/maxResults specified with collection fetch; applying in memory!`

## 주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파

- 

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
