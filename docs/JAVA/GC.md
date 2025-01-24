---
layout: default
title: GC 동작 원리와 종류
parent: JAVA
---

# GC 동작 원리와 종류
{: .no_toc }

1. TOC
{:toc}

## GC

- Heap 메모리 영역 자동 정리
  - 단점
    - **메모리 해제 시점을 정확하게 알기 힘들다** -> 제어 힘듬
    - GC 동작 시 **다른 동작들이 멈춤** -> 오버헤드 발생 **(Stop-The-World)**
      - 실시간 성이 강조되는 프로그램일 경우, GC에 맡기는 것이 좋지 않을 수도 있다.

## GC 대상 1:15

- 특정 개체가 garbage인지 아닌지 판단해야한다
  - 도달성(Reachability)
    - 객체에 레퍼런스가 있다면(참조중이라면) reachable, 없다면 unreachable

## GC 방식

- Mark and Sweep
  - Mark: Root Space부터 그래프 순회를 통해 어떤 객체들을 참조중인지 마킹
    - Root Space: Heap을 참조하는 method area, static 변수, stack, native method stack ...
  - Sweep: 마킹되지 않은 unreachable 객체들을 Heap에서 제거
  - Compaction: 분산된 위치의 객체들을 한 곳으로 모아 압축(GC 종류에 따라 생략하는 경우도 있음)

## GC 동작 과정

![GC memory](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FtgPPC%2FbtrI2Z4pXMo%2FJEIwKoiT0J5I3gyQSYRzl1%2Fimg.png)

출처: [인파 블로그](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)

## heap 메모리의 구조

- heap 영역은 2가지를 전제로 설계되었다
  - 대부분의 객체는 금방 unreachable이 된다
  - 오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다
- 즉, 객체는 대부분 일회성이며, 메모리에 오래 남아있는 경우는 드물다
- 물리 영역을 Young과 Old로 나눔
  - Young
    - 새로운 객체 할당
    - 수명이 짧은 객체들 -> 큰 영역 필요 없음
    - Eden, survivor 0, survivor 1
    - minor GC
    - 세부 분류
      - Eden
        - 새로 생성된 객체(new)
        - 살아남은 객체는 survivor로
      - Survivor 0 / Survivor 1
        - 최소 1번 이상의 GC에서 살아남은 객체들이 존재
        - 둘 중 하나는 비어있어야 한다
  - Old
    - Young에서 살아남은 객체가 복사됨
    - Young보다 크게 할당되고, 영역이 큰 만큼 가비지는 적게 발생
    - major GC, full GC
  - (Permanent)
    - 생성된 객체들의 정보와 주소값을 저장
    - heap에 존재(Java 7) -> Native Method Stack에 존재(Java 8)

## Minor GC 과정

- Young Generation 공간이 작다 -> 메모리 상의 객체를 빨리 찾아서 제거하기 수월하다
1. Eden 영역이 꽉 차면 minor GC 실행
2. Mark -> reachable 객체 탐색
3. 살아남은 객체는 survivor 영역으로 이동
4. 사용되지 않는 Eden 객체들 Sweep
5. survivor 객체들의 age 값이 1씩 증가
6. Eden 영역이 꽉 차면 minor GC 실행 -> Mark
7. Mark된 객체들과 기존 survivor 영역의 객체들을 다른 survivor 영역으로 옮기고 Eden 영역 Sweep
8. survivor 객체들의 age 값이 1씩 증가
9. 반복

- age 값
  - 객체가 살아남은 횟수, Object Header에 기록
  - 임계값에 다다르면 Promotion(Old 영역으로 이동) 여부 결정
  - HotSpot JVM -> age 기본 임계값 31 (6 bit)

## Major GC 과정

- 

## GC 알고리즘 종류

- 
