---
layout: default
title: 4. 회원 도메인 개발
parent: 실전! 스프링부트와 JPA 활용1
---

# 4. 회원 도메인 개발
{: .no_toc }

1. TOC
{:toc}

## 회원 리포지토리 개발

- `@PersistenceContext`
  - 엔티티 매니저 주입
- `@PersistenceUnit`
  - 엔티티 메니저 팩토리 주입

## 회원 서비스 개발

- 검증 로직이 있어도 `회원` 테이블의 `회원명` 컬럼에 `UNIQUE` 제약 조건을 추가하는 것이 안전
  - 멀티 쓰레드 상황을 고려
- 생성자 주입시 `final`키워드를 추가하여 **컴파일 시점에** memberRepository를 설정하지 않는 **오류를 체크**할 수 있다

## 회원 기능 테스트

- `@Transactional`
  - 테스트 실행때마다 트랜잭션 시작, 끝나면 다시 롤백