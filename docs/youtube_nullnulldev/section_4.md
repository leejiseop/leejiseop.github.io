---
layout: default
title: 4. WAS와 JVM
parent: 널널한개발자 면접 대비
---

# WAS와 JVM
{: .no_toc }

1. TOC
{:toc}

- CPU가 인식할 수 있는 코드: 머신 코드
  - 프로세서마다 구체적인 값이 다르다
- JVM: (가상의) OS + CPU + MEM

## JVM 기본 구조

- Class Loader, Runtime Data Area, Execution Engine, Native Method Interface, Native Method Library
- 클래스 로더에서 클래스 로딩, 링킹, 초기화
- Execution Engine에서 인터프리터, JIT 컴파일러, **GC**
- 널널한개발자 JVM 유튜브 참고
- HTTP는 문서 프로토콜 -> 문자열이 많을 수 밖에 없다 -> Heap 영역 많이 쓰인다 -> 거기다 멀티스레딩 부하까지..
  - GC가 알아서 해준다  -> **드러나지 않는 문제를 파악할 수 있어야 한다**
    - 운영을 감안한 개발이 중요 -> 서비스 장애 경험
    - **JVM 구조를 알아야 한다**

## GC 원리

- Heap 영역은 유한 자원이다
- Runtime Data Area (total size) = Heap space + Metaspace + Native area
- **Full GC / OVER / stop-the-world -> JVM 다운**
  - 큰 지연, 장애 상황
- JVM heap 영역
  - MetaSpave (Java 8)
    - 로드되는 클래스, 메소드 등에 관한 메타정보 저장 (자동확장 가능)
    - Java heap이 아닌 **Native 메모리 영역 사용**
    - 리플렉션 클래스 로드 시 사용 -> **Spring**
  - 크기 계산 기준: request(TPS)
- Full GC가 일어날 확률이 적은 코드를 만들어야 한다
  - 어떤 객체가 old generation 까지 가게될까?
  - **JVM heap 덤프**
- 살아남는 객체를 선정하는 방법
  - GC는 객체 참조의 유효(Reachable)함과 그렇지 않음(Unreachable)을 근거로 대상 식별
  - GC Roots, 의존 연결
    - Stack 데이터 (Stack -> heap)
    - 메서드 static 데이터
    - JNI로 만들어진 데이터
    - 강의 10:30

## GC와 JVM에 대해 물은 의도는 따로 있다

- GC 종류
  - Serial GC
    - 단일 스레드 환경 및 소규모 응용프로그램을 위한 간단한 GC
    - Minor GC에서 Copy & Scavenge 알고리즘 적용
    - Full GC에서 Mark & Compact 알고리즘 적용
  - Parallel GC
    - JVM 기본 옵션(Java 8)
    - 멀티스레드 기반(개수 지정 가능)으로 작동해 효율을 높임
    - Low-pause(응용 프로그램 중단 최소화)
    - Throughput(Mark & Compact 알고리즘을 기반으로 신속성 최대화)
  - Concurrent GC
    - Low-pause와 유사하며 응용 프로그램 실행 중 GC 실시
    - 동작 중지 최소화
  - Incremental GC (Train GC)
    - Concurrent GC와 유사하나 Minor GC 발생 시 Full GC를 일부 병행
    - 경우에 따라 오히려 더 느려지는 부작용
  - G1(Garbage First) GC
    - Young Generation, Eden ... 등이 아니라 regions 영역으로 구분
    - **4GB 이상 대용량 Heap 메모리**를 사용하는 멀티스레드 기반 응용 프로그램에 특화된 GC
    - **Heap을 영역(1~32MB)단위로 분할**한 후 멀티스레드로 스캔
    - 가비지가 가장 많 은 영역부터 수집 실시
  - CMS(Concurrent Mark Sweep) GC
    - Java 9부터 사용하지 않다가, Java 14에서 G1 GC를 지원하고자 완전히 제거
  - **각각 장단점 공부 필요**

- 부하테스트
  - APM과 NPM
    - Application Performance Management
      - 제니퍼, **Scouter**(무료)
      - 어플리메이션 성능 모니터링
    - Network Performance Management
      - 여기까지 해보면 더 좋다
  - Client - 이 지점을 부하테스트 - [Web server - WAS] - [DB]
    - PC #1: Client
    - PC #2: Web server + WAS
    - PC #3: DB
      - PC #2 #3에 APM 설치하여 모니터링
      - 그걸 GUI 모니터링할 콘솔이 필요
    - 부하테스트 -> 당연히 문제가 생길 수 밖에 없다 -> 그래프가 튄다
      - **어떠한 문제인지 인지, 개선**
        - 힌트는 어떻게 얻었는지?
        - 어디를 어떻게 개선?
        - 그 결과?
          - **정량적인 평가**
          - 응답속도, TPS ...
      - **문서화**