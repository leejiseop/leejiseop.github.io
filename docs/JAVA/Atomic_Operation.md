---
layout: default
title: Atomic Operation 
parent: JAVA
---

# Atomic Operation
{: .no_toc }

1. TOC
{:toc}

## 원자성이 보장되지 않는다면
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

## Atomic Operation

- 공유 자원 경쟁 상태를 막기 위한 방법 (동기화)
- 가능 조건
  - 모든 조작이 완료될 때까지 어떤 프로세스도 변경을 알 수 없어야 한다.(비가시적)
  - 어느 하나라도 실패한다면 조작 이전의 상태로 복구한다

## volatile

- 일반 변수는 CPU 캐시를 이용할 수 있지만, `volatile` 변수는 항상 **메인 메모리에서** 읽고 쓴다
- 한 스레드가 `volatile` 변수를 변경하면 다른 스레드도 **즉시 변경된 값을 볼 수 있음**
  - **flag 변수** 등에 사용
  - 스레드 종료, 설정값 변경.. 등
- 원자성은 보장되지 않음 -> synchronized나 Atomic 클래스 필요

## synchronized

```java
// 1. 메서드 전체를 임계 영역으로 지정
public class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}


// 2.특정 영역을 임계 영역으로 지정
public class Counter {
    private int count = 0;

    public void increment() {
        synchronized(this) {
            count++;
        }
    }

    public int getCount() {
        return count;
    }
}
```

- synchronized는 **특정 Thread가 해당 블락 전체를 lock** 하기 때문에 다른 Thread는 아무작업을 못하고 기다리는 상황이 되어 낭비가 심하다. -> 컨텍스트 스위칭 비용
  - 대신 여러개의 연산을 연속적으로 하는 경우에도 확실하게 원자성을 보장한다
- 그래서 **NonBlocking하면서 동기화 문제를 해결**하기 위한 방법이 Atomic이다.

## AtomicInteger

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    AtomicInteger count = new AtomicInteger(0);

    void increment() {
        count.incrementAndGet();
    }
}
```

- AtomicInteger 란 **원자성을 보장하는 Interger**를 의미한다.
- 동기화 문제를 **별도의 synchronized 키워드 없이** 해결하기 위해서 고안된 방법이다.
  - 대신 단일 연산만 가능
  - synchronized는 여러 연산을 덩어리로 묶기 가능
- CAS 연산 실행 후 값 적용

### CAS 알고리즘

![CAS](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FSnIds%2FbtrdmAJBT68%2Fn9Y4KyFk2BrLZ9a9ZEzBd0%2Fimg.png)

- 멀티 코어(쓰레드) 환경에서 각 CPU는 메모리 값이 아닌 **캐시 영역에서 값을 참조**
  - 메인 메모리의 원래 값과 캐시 영역에서의 값이 다른 경우가 있다 (가시성 문제)
- 현재 쓰레드에 저장된 값과 **메인 메모리에 저장된 값을 비교**
  - **일치하는 경우 새로운 값으로 갱신**하여 가시성 문제를 해결한다
  - 중간에 다른 스레드가 변경하면 다시 시도하므로 값이 안전하게 증가됨
- volatile 키워드가 붙어 있는 객체는 CPU 캐시가 아닌 메인 메모리에서 값을 참조해온다

## 세마포어와 뮤텍스

```java
import java.util.concurrent.Semaphore;

// 2개 초과하는 스레드는 semaphore.acquire()에서 대기
// 작업이 끝나면 semaphore.release()를 호출하여 세마포어 반환
public class SemaphoreExample {
    private static final Semaphore semaphore = new Semaphore(2); // 최대 2개 스레드 허용

    public static void main(String[] args) {
        Runnable task = () -> {
            try {
                semaphore.acquire(); // 🔒 세마포어 획득
                System.out.println(Thread.currentThread().getName() + " 가 임계 영역에 접근");
                Thread.sleep(1000);  // 작업 수행
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release(); // 🔓 세마포어 반환
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        Thread t3 = new Thread(task);

        t1.start();
        t2.start();
        t3.start();
    }
}
```

- 세마포어(Semaphore)
  - 카운팅 락으로 **제한된 수의 스레드만 접근을 허용**
  - **동시에 실행될 수 있는 스레드의 수를 제한**할 때 사용
  - 락을 여러 번 걸 수 있어서 동시에 여러 스레드가 접근 가능

```java
import java.util.concurrent.locks.ReentrantLock;

// 한 스레드가 lock.lock()을 호출하면 다른 스레드는 대기
// 작업이 끝나면 lock.unlock()을 호출하여 락 해제
public class MutexExample {
    private static final ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Runnable task = () -> {
            lock.lock();  // 🔒 락 획득
            try {
                System.out.println(Thread.currentThread().getName() + " 가 임계 영역에 접근");
                Thread.sleep(1000);  // 작업 수행
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();  // 🔓 락 해제
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);

        t1.start();
        t2.start();
    }
}
```

- 뮤텍스(Mutex)
  - **상호 배제(Mutual Exclusion)**를 위한 락으로, **하나의 스레드만 접근 허용**
  - **하나의 스레드만 특정 코드 블록에 접근**할 수 있도록 할 때 사용
  - 락을 걸 때마다 반드시 해제해야 함 -> 세마포어와 달리 하나의 스레드만 접근 가능

- 뮤텍스는 세마포어 기법의 일부분으로 포함되는 개념
  - 즉, 뮤텍스는 1개만 허용하는 세마포어( `Semaphore(1)` )와 비슷하지만
  - 개념적으로 **한 스레드가 락을 반드시 해제해야 하는 특징**이 있음

출처: [truehong](https://truehong.tistory.com/71) [vaert](https://vaert.tistory.com/39) [javaplant](https://javaplant.tistory.com/23)