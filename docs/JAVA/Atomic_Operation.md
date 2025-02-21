---
layout: default
title: Atomic Operation 
parent: JAVA
---

# Atomic Operation
{: .no_toc }

1. TOC
{:toc}

## Atomic Operation

- 공유 자원 경쟁 상태을 막기 위한 방법 (동기화)
- 가능 조건
  - 모든 조작이 완료될 때까지 어떤 프로세스도 변경을 알 수 없어야 한다.(비가시적)
  - 어느 하나라도 실패한다면 조작 이전의 상태로 복구한다
- 원자성이 보장되지 않는다면
  - 프로세스가 하나인 경우
    - 프로세스가 메모리에서 값을 읽어온다(캐싱)
    - 프로세스가 값에 1을 더한다
    - 계산한 값을 메모리의 원래 위치에 써넣는다
  - 프로세스가 둘인 경우
    - 1번: 메모리에서 값을 읽어오고 1을 더한다
    - 실행권 박탈, 2번 프로세스 실행
    - 2번: 메모리에서 값을 읽어오고 1을 더한다
    - 2번: 계산한 값을 메모리의 원래 위치에 써넣는다
    - 실행권 박탈, 1번 프로세스 실행
    - 1번: 이 사실을 모르고 잘못된 값을 메모리에 써넣는다

## AtomicInteger

AtomicInteger 란 **원자성을 보장하는 Interger**를 의미한다. 멀티 쓰레드 환경에서 동기화 문제를 **별도의 synchronized 키워드 없이** 해결하기 위해서 고안된 방법이다.

```java
// 1. 메서드 전체를 임계 영역으로 지정
public class Counter {
    private int count = 0;

    // 메서드 전체를 임계 영역으로 지정
    public synchronized void increment() {
        count++;  // 이 코드는 한 번에 하나의 스레드만 접근 가능
    }

    public int getCount() {
        return count;
    }
}


// 2.특정 영역을 임계 영역으로 지정
public class Counter {
    private int count = 0;

    public void increment() {
        synchronized(this) {  // 특정 코드 블록을 임계 영역으로 지정
            count++;  // 이 코드는 한 번에 하나의 스레드만 접근 가능
        }
    }

    public int getCount() {
        return count;
    }
}
```

synchronized은 **특정 Thread가 해당 블락 전체를 lock** 하기 때문에 다른 Thread는 아무작업을 못하고 기다리는 상황이 되어 낭비가 심하다. 그래서 **NonBlocking하면서 동기화 문제를 해결**하기 위한 방법이 Atomic이다.

## CAS 알고리즘

![CAS](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FSnIds%2FbtrdmAJBT68%2Fn9Y4KyFk2BrLZ9a9ZEzBd0%2Fimg.png)

- 멀티 코어(쓰레드) 환경에서 각 CPU는 메모리 값이 아닌 **캐시 영역에서 값을 참조**한다. 이 때, 메인 메모리의 원래 값과 캐시 영역에서의 값이 다른 경우가 있다.(가시성 문제)
- 현재 쓰레드에 저장된 값과 **메인 메모리에 저장된 값을 비교**하여 일치하는 경우 새로운 값으로 갱신하여 가시성 문제를 해결한다






뮤텍스는 세마포어 기법의 일부분으로 포함되는 개념


출처: [truehong](https://truehong.tistory.com/71) [vaert](https://vaert.tistory.com/39) [javaplant](https://javaplant.tistory.com/23)