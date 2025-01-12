---
layout: default
title: 4. WAS와 JVM
parent: 널널한개발자 면접 대비
---

# WAS와 JVM

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

- 

## GC와 JVM에 대해 물은 의도는 따로 있다

