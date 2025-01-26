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

![heap](https://user-images.githubusercontent.com/14002238/113507707-1deb1000-9587-11eb-9ab6-2bbe0091be87.jpg)

- Heap 메모리 영역 자동 정리
  - 단점
    - **메모리 해제 시점을 정확하게 알기 힘들다** -> 제어 힘듬
    - GC 동작 시 **다른 동작들이 멈춤** -> 오버헤드 발생 **(Stop-The-World)**
      - 실시간 성이 강조되는 프로그램일 경우, GC에 맡기는 것이 좋지 않을 수도 있다.

## GC 대상

- 특정 개체가 garbage인지 아닌지 판단해야한다
  - 도달성(Reachability)
    - 객체에 레퍼런스가 있다면(참조중이라면) reachable, 없다면 unreachable

## GC 방식

![GC](https://velog.velcdn.com/images/ddangle/post/94e97f8c-7114-47e6-b8f2-05f9741fb911/image.png)
![GC](https://velog.velcdn.com/images/ddangle/post/1e907432-c96c-4b46-bfd5-d13495fdbfae/image.png)

- Mark and Sweep
  - Mark: Root Space부터 그래프 순회를 통해 어떤 객체들을 참조중인지 마킹
    - Root Space: Heap을 참조하는 method area, static 변수, stack, native method stack ...
  - Sweep: 마킹되지 않은 unreachable 객체들을 Heap에서 제거
  - Compaction: 분산된 위치의 객체들을 한 곳으로 모아 압축(GC 종류에 따라 생략하는 경우도 있음)

## GC 동작 과정

![GC memory](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FtgPPC%2FbtrI2Z4pXMo%2FJEIwKoiT0J5I3gyQSYRzl1%2Fimg.png)

## heap 메모리의 구조

- heap 영역은 2가지를 전제로 설계되었다
  - **대부분의 객체는 금방 unreachable이 된다**
  - **오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다**
- 즉, **객체는 대부분 일회성**이며, 메모리에 오래 남아있는 경우는 드물다
- heap 물리영역을 Young과 Old로 나눔
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

## Minor GC 과정 (Copy & Scavenge)

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

## Major GC 과정 (Full GC, Mark & Compact)

- age 임계값에 도달한 객체들이 계속 Promotion되어 Old 영역의 메모리가 부족해지면 실행
- Old 영역을 Mark and Sweep하는 단순한 방식
  - 하지만 Young 영역에 비해 크기가 커서 대략적으로 10배 이상 걸린다
  - Stop-the-World
    - thread가 멈추고 mark and sweep 작업 실행 -> cpu 부하 -> 애플리케이션이 멈추거나 버벅임
- Compact 실행

## GC 알고리즘 종류

- Non Concurrent Collectors
  - Serial Collector
  - Parallel Collector
  - Parallel Old Collector
  - Incremental GC

- Concurrent Collectors
  - Concurrent Mark Sweep Collector (CMS)
  - Garbage-First Collector (G1GC)
  - Z Collector (ZGC)

**JVM이 메모리 관리**해주는것은 상당한 이점  
-> 하지만 **그 시기를 예측하기 어려워** 애플리케이션이 갑자기 중지되는 문제 발생  
-> 또한 자바가 발전됨에 따라 **Heap 사이즈도 커짐**  
-> 애플리케이션이 중지되는 **문제가 더욱 빈번해짐** -> **최적화 알고리즘** 개발 필요  

**상황에 따라 필요한 GC 방식을 설정**해서 사용이 가능하다!

- Serial GC
  - **CPU 코어가 1개일 경우**, 가장 단순한 GC
  - **GC를 처리하는 쓰레드가 1개**이기 때문에 **Stop-the-World 시간이 가장 길다**
  - Minor GC: Mark-Sweep
  - Major GC: Mark-Sweep-Compact
  - 실무에서는 잘 사용하지 않는다
    ![Serial GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkBL3D%2FbtrIT91k2n7%2FOmhFna09F6duWkBh1FUFd0%2Fimg.png)
  - 실행 명령어
    - 자바 프로그램을 실행할때 -XX:+UseSerialGC GC 옵션을 지정 -> 해당 가비지 컬렉션 알고리즘으로 힙 메모리를 관리하도록 실행
    ```bash
    java -XX:+UseSerialGC -jar Application.java
    ```

- Parallel GC (Throughput GC)
  - **Java 8 버전의 기본 GC**
  - Serial GC와 기본 알고리즘은 같지만 **Minor GC를 멀티 쓰레드로 수행**
  -  -> **Stop-the-World 시간 감소**
    ![Parallel GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FckjP6E%2FbtrIVf74R58%2FLdPm5Nmd3tCLFLuknpOc2K%2Fimg.png)
  - 실행 명령어
    - GC 스레드는 기본적으로 cpu 개수만큼 할당 -> 옵션을 통해 설정 가능
    ```bash
    java -XX:+UseParallelGC -jar Application.java 
    # -XX:ParallelGCThreads=N : 사용할 쓰레드의 갯수
    ```

- Parallel Old GC (Parallel Compacting Collector)
  - Parallel GC를 개선한 버전
  - Old 영역에서도 멀티쓰레드 GC 수행
  - Mark-Summary-Compact 방식
  - Serial GC와 기본 알고리즘은 같지만 **Minor GC를 멀티 쓰레드로 수행**
  -  -> **Stop-the-World 시간 감소**
    ![Parallel Old GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcs71MN%2FbtrI0ePXZqr%2F6nI0EkNdQ7fBzMYlwewk8K%2Fimg.png)
  - 실행 명령어
    ```bash
    java -XX:+UseParallelOldGC -jar Application.java
    # -XX:ParallelGCThreads=N : 사용할 쓰레드의 갯수
    ```

- CMS GC (Concurrent Mark Sweep, Low Latency GC)
  - Stop-the-World 시간을 최대한 줄이기 위해 **어플리케이션 쓰레드와 GC 쓰레드가 동시 실행**
  - 대신 **GC 과정이 복잡**해짐 -> GC 대상을 파악하는 과정이 복잡 -> CPU 사용량이 높다
  - Compaction 단계 없음 -> **메모리 단편화** 문제 (외부)
    - Compaction을 진행하면 다른 GC보다 더 오래 걸린다
  - Java 9부터 deprecated 되었고, Java 14에서는 **사용 중지**
    ![CMS GC](https://velog.velcdn.com/images/ddangle/post/5d25731f-7681-4cf6-9e98-61dc8ac4e565/image.png)
    ![CMS GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbtq9xn%2FbtrIUalHp5a%2Fuoqi42gxuTywMfm5Uhcs9k%2Fimg.png)
  - `Initial Mark`: 클래스 로더에서 가장 가까운 객체 중 살아있는 객체를 찾는다
  - `Concurrent Mark`: 앞에서 마크한 객체들을 따라가면서 확인
  - `ReMark`: 앞에서 새로 추가되거나 참조가 끊긴 객체들을 호가인
  - `Concurrent Sewwp`: 쓰레기 정리 작업
  - 실행 명령어
    ```bash
    # deprecated in java9 and finally dropped in java14
    java -XX:+UseConcMarkSweepGC -jar Application.java
    ```

- G1 GC (Garbage First)
  - **CMS GC를 대체**하기 위해 jdk 7 버전에서 release
  - **Java 9+ 버전의 기본 GC**
  - 4GB 이상의 heap 메모리, Stop-the-World 시간이 0.5초 정도 필요한 상황에 사용
    - **heap 메모리가 작을 경우에는 미사용 권장**
  - 기존의 Young, Old와는 달리 **Region**이라는 새로운 개념 도임
    - 전체 heap 영역을 체스같이 분할하여 상황에 따라 Eden, Survivor, Old 등 역할을 **고정이 아닌 동적으로 부여**
    - Garbage로 가득찬 영역을 **빠르게 회수하여 빈 공간 확보** -> **GC 빈도가 줄어듬**
  - 효율성
    - 일일이 메모리를 탐색헤 객체들을 제거하지 않는다
    - 대신 메모리가 많이 차있는 영역(Region)을 우선적으로 GC한다
    - 즉, **메모리 전체가 아닌 영역별로 GC**가 일어난다
    - 또한, Eden -> Survivor 0 -> Survivor 1 -> ... 순으로 이동되던 기존 GC와는 달리, G1 GC에서는 더욱 효율적이라고 생각되는 위치로 객체를 재할당(Reallocate) 시킨다
      - ex. Survivor 1 영역의 객체를 Eden으로 옮기는 것이 더 효율적이라고 판단하고 재할당
    ![Serial GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb3VAZL%2FbtrIVglH89c%2F80uC2DsGKDc1HXG57ByKak%2Fimg.png)
  - 실행 명령어
    ```bash
    java -XX:+UseG1GC -jar Application.java
    ```

- Shenandoah GC
  - Java 12에 release, RedHat에서 개발한 GC
  - **기존 CMS의 메모리 단편화, G1의 pause 이슈 해결**
  - 강력한 Concurrency와 가벼운 GC 로직 -> **heap 사이즈에 영향을 받지 않고 일정한 pause 시간 소요**
  - ZGC와 비슷하게 대량의 메모리 처리에 우수한 퍼포먼스를 내지만 좀 더 많은 옵션을 제공
    ![Serial GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlHh4s%2FbtrISNkkpMV%2FcuFxgmAT0DuafPhABn0L00%2Fimg.png)
  - 실행 명령어
    ```bash
    java -XX:+UseShenandoahGC -jar Application.java
    ```

- ZGC
  - Java 15에 release
  - 대용량 메모리(8MB ~ 16TB)를 low-latency로 처리하기 위해 디자인 된 GC
  - 어플리케이션과 Concurrently하게 동작 -> Heap Reference를 위해 Load barrier를 사용
  - G1의 region처럼 **ZPage**라는 영역을 사용
  - G1의 region은 크기가 고정인 반면 ZPage는 **2MB 배수로 동적으로 운영**
  - **heap 크기가 증가하더라도 Stop-the-World 시간이 절대 10ms를 넘지 않는다**
    ![Serial GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FIKx8x%2FbtrIT9f9qjM%2Ff1g0NHSGJI7ce9rfxPZyuk%2Fimg.png)
  - 실행 명령어
    ```bash
    java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -jar Application.java
    ```

- Incremental GC (Train GC)
  - Full GC에서 의해 애플리케이션이 멈추는 시간을 줄이기 위함 (응답성이 중요한 경우)
  - Minor GC가 일어날 때마다 Old 영역을 조금씩 GC -> Full GC가 발생하는 횟수나 시간을 줄임

- Epsilon GC
  - 수학에서 '아주 작은 값'을 나타내는 ε (epsilon)에서 유래
    - 최소한의 기능만 가진 GC, 즉 '아무것도 하지 않는 GC'라는 의미
  - Java Heap 영역을 모두 소진하게 되면 JVM이 Shut down (`OutOfMemoryError`)
  - Epsilon의 목적은 제한된 영역의 메모리 할당을 허용 -> 최대한 latency overhead를 줄인다
  - 외부 환경으로부터 고립된 채로 실행 -> 메모리 사용량과 퍼포먼스 등을 보다 정확하게 측정할
  - 주 사용 사례
    - GC의 성능 영향을 제외하고 애플리케이션 성능을 테스트할 때
    - 메모리 수명이 짧아 GC가 필요 없는 상황

## GC 선정 기준

- **성능상 이슈가 없다면 JVM이 선정해주는 Collector를 사용한다**
- 적은 데이터, 싱글 코어 -> Serial GC
- 전체 처리량이 중요, Stop-the-World 시간이 1초 이상이어도 괜찮다 -> Parallel GC
- 응답 시간이 중요, Stop-the-World 시간이 1초 이하여야 한다 -> Concurrent Collectors 중 택 1
  - Java 15 미만: ZGC는 실험적 GC(옵션 활성화 필요)이기 때문에 G1GC와 CMS를 비교하여 적절한 것을 채택
  - Java 11: G1GC (default)
  - Java 8: CMS (default)
- Collector를 변경해가며 벤치마크 진행하며 성능과 리소스 사용량 확인

## GC 로그 수집 방법

- JVM에서 GC 로그 수집 옵션 제공
  - 자바 실행용 `.jar` 파일 생성 -> 실행 옵션으로 `-verbosegc` -> `>` 리다이렉션 명령어로 파일로 저장 후 분석

## JVM GC 튜닝

- 목표값 설정
  - 메모리 사용량? GC 횟수? GC 시간? 성능 향상? -> 목표치에 근접하도록 JVM 파라미터 조정
- heap 크기와 permanent(Java 7 이하) 크기 설정
  - `java -Xms2048m -Xmx2048m -jar application.jar`
  - `-Xms`, `-Xmx` -> heap 메모리 초기 크기, 최대 크기
  - 급격한 변화가 있는 경우가 아닌 대부분의 경우에는 동일한 것이 좋다
    - Growing, Shrinking 등 불필요한 heap 추가요청을 없앤다
- 테스트와 로그 분석
  - GC 로그, 부하 테스트(nGrinder, JMeter) 로그 수집
- permanent 크기 조정
- GC 수행시간 분석
  - Full GC가 많이 일어남 -> Old 영역 늘림 -> Full GC 횟수 감소, 시간 증가
  - 서버 어플리케이션은 Full GC시 JVM 자체가 멈춰버리기 때문에 일정 시간동안 응답을 못하는 상태가 된다
  - Full GC 횟수와 시간을 모두 줄이려면
    - Old 영역 메모리 줄이기 -> Full GC 시간 감소
    - 여러 개의 인스턴스를 동시에 띄워서 부하 분산 -> Full GC 횟수 감소
      - 하나가 멈춰있는 동안 다른쪽이 응답 -> 애플리케이션이 멈출 때 받는 영향 최소화
- 파라미터
  - Old와 New(Eden, Survivor) 영역 배분


출처 [인파](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC) [땡글로그](https://velog.io/@ddangle/Java-GC-Garbage-Collector%EC%97%90-%EB%8C%80%ED%95%B4) [RoyOps](https://imprint.tistory.com/35) [준비된개발자](https://readystory.tistory.com/48) [dongwookiee96](https://dongwooklee96.github.io/post/2021/04/04/gcgarbage-collector-%EC%A2%85%EB%A5%98-%EB%B0%8F-%EB%82%B4%EB%B6%80-%EC%9B%90%EB%A6%AC.html)