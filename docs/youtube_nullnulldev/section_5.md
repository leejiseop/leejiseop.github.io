---
layout: default
title: 5. 데이터베이스
parent: 널널한개발자 면접 대비
---

# 데이터베이스
{: .no_toc }

1. TOC
{:toc}

## 인덱싱을 하는 이유는 무엇일까?

- 데이터베이스 = **자료구조**
  - B-Tree: index node활용 -> 추가 삭제 용이
    - 모든 노드에 데이터와 키가 저장 -> 검색 시 내부 노드에서도 데이터를 확인 -> 탐색이 조금 더 느릴 수 있음
  - B+Tree: B-Tree보다는 약간 느리다, 검색은 빠름
    - 데이터는 리프 노드에만 저장, 내부 노드는 키만 저장 -> 데이터 접근은 리프에서만 이루어져 탐색 경로가 단순해지고 범위 검색이 더 빠름
- 데이터는 결국 선형구조의 2차 메모리에 저장되게 된다 (hdd, ssd)
  - **검색 속도**를 높이기 위해 인덱싱
  - **없는 정보 검색 시 빠른 판단 가능**
- 대규모 테이블(10만건 ~ 1억건 ...)에 대해 적용, **삽입 수정 삭제가 자주 발생하지 않는 경우**
  - **데이터 10건, 10만건, 1억건 비교해보기**
- 오버헤드
  - 인덱스를 위한 추가 메모리 소모
  - 데이터 수정시 인덱스까지 수정


## 트랜잭션에 대해 물어본다면

- 읽기 중 쓰기 방지
- 질의 수행 시 일련의 작업이 모두 수행 or 하나라도 실패 시 모두 취소
  - **작업의 논리적 단위**
  - **원자성** 보장
  - **TPS**
- ACID 특성
  - 원자성 Atomicity
  - 일관성 Consistency
  - 격리성 Isolation
  - 지속성 Durability
- RAM에 올라온걸 2차 메모리에 실질 반영 = commit
- **동시성 관련 이슈**
  - 커밋 완료 전 read (dirty read)
  - 커밋 완료 전 overwrite
  - read 중 데이터가 변경되는 경우
  - 변경한 데이터 유실 (lost update)
- 트랜잭션 isolation level
  - READ UNCOMMITED
  - READ-COMMITED
  - REPEATABLE READ
    - 트랜잭션 동안 데이터를 읽게 함
    - 특정 버전에 해당하는 데이터를 읽음
  - SERIALIZABLE

## DB 커넥션 풀의 용도와 필요를 묻는 숨은 의도

- DBCP Connection Pool 연결 대기 지연 현상 [SK C&C TECH BLOG](https://engineering-skcc.github.io/cloud/tomcat/apache/DB-Pool-For-Event/) -> **꼭 읽어보기!**
  - 한정판 굿즈 판매 이벤트, 대학교 수강신청, 명절 KTX 예매 ...
  - connection pool이 부족하지면 다른 자원이 여유있어도 장애가 발생한다
  - 경우에 따라서는 pool이 아닌 설정 문제일수도
- request가 오고나서 WAS-DB 연결하고 DB 질의하면 너무 오래걸림
  - DB 연결도 시간이 꽤 걸리는 작업
  - 그래서 **미리 연결**시켜둔다 - **connection pool** - 응답성을 높인다
    - **모자르면 -> 새 connection 생성** -> 연결하느라 느려짐
    - 어느정도가 적당...? -> **부하테스트**
  - 미들웨어(Tomcat ...)가 관리
- **결국은 운영 이슈**
  - 운영을 고려하는 개발