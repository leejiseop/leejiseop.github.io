---
layout: default
title: 8. 입출력 장치
parent: OS
---

# 입출력 장치
{: .no_toc }

1. TOC
{:toc}

### 섹션8 입출력 장치

- 주변장치 - 그래픽카드, HDD, SSD, 키보드, 마우스 등
  - 캐릭터 디바이스
    - 캐릭터 단위의 적은 양의 데이터 전송
    - 마우스, 키보드, 사운드카드, 직렬 병렬 포트
  - 블록 디바이스
    - 블록 단위의 큰 데이터 전송
    - HDD, SSD, 그래픽카드
  - 버스 인터페이스를 통해 I/O 버스에 연결되어있다.
- I/O 버스
  - Address 버스, Data 버스, Control 버스
- 예전에는 하나의 버스에 모든 장치들을 연결했었다
  - 효율이 좋지 않아 입출력 제어기와 여러개의 버스 탄생
    - CPU가 I/O입력이 필요하면 입출력 제어기에 역할을 넘기고 그 동안 다른 프로세스 실행
    - 입출력 제어기는 다시 고속 입출력과 저속 입출력 채널로 나뉨
      - 속도 차이로 인한 병목 해결
  - 그래픽 카드는 대용량이라 시스템 버스에 바로 연결
- 입출력 제어기는 입출력 버스에 들어온 데이터를 메모리로 옮기는데, 메모리는 CPU의 명령으로 움직이기 때문에, 해당 역할을 대신 해주는 DMA 제어기가 추가됨
  - Memory Mapped I/O
    - CPU가 사용하는 메모리 영역과 DMA가 사용하는 메모리 영역을 나눈다
    - Direct Memory Access
- 하드디스크의 구조
  - 플래터, 섹터, 트랙, 실린더 등