---
layout: default
title: 2. API 개발 고급 - 준비
parent: 실전! 스프링부트와 JPA 활용2
---

# 2. API 개발 고급 - 준비
{: .no_toc }

1. TOC
{:toc}

- **xToOne** 관계 최적화
- 양방향 연관관계의 한쪽은 `@JsonIgnore` 처리
- `order`의 `member`와 `delivery` 는 지연 로딩이다.
  - 따라서 실제 엔티티 대신에 프록시 존재
  - **bytebuddy**
    - `org.hibernate.proxy.pojo.bytebuddy`