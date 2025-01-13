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

