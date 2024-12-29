---
layout: default
title: 쉬운코드 - 소켓 통신
parent: Network
---

# 쉬운코드 - 소켓 통신

## OSI 7 Layer

**[ Application layer ]**  
**[ Presentation layer ]**  
**[ Session layer ]**  
**[ Transport layer ]**  
**[ Network layer ]**  
**[ Data link layer ]**  
**[ Physical layer ]**  
- 네트워크 시스템 구성을 위한 범용적이고 **개념적인 모델**

## TCP/IP stack

**[ Application layer ]**  
**[ Transport layer ]**  
**[ Internet layer ]**  
**[ Link layer ]**  
- 인터넷이 발명되면서 함께 개발된 프로토콜 스택
- 인터넷 구조에 특화된 계층 구조

## 통신 과정 (데이터) (TCP/IP stack)

- Client 에서 Server 로

- **[ Application layer ]**
  - Application layer 프로토콜을 통해 송/수신되는 실제 데이터
    - `header` + [`data`]
      - `payload` = 헤더 를 제외한 나머지 부분 (대괄호 안)
      - 프로토콜 동작을 위한 헤더가 추가된다
  - Transport layer 로 이동
- **[ Transport layer ]**
  - Application layer 프로토콜을 통해 송/수신되는 실제 데이터
    - `header` + [`header` + `data`]
      - TCP: `TCP segment`
      - UDP: `UDP datagram`
      - 서로 헤더 구조가 다른데, 보내는 쪽 받는 쪽 포트번호 있는건 동일
- **[ Internet layer ]**  
  - 호스트와 호스트 간의 경로를 따라 전달하는 네트워크를 담당 (라우팅)
  - IP 통신을 위한 헤더 추가
  - Internet layer 프로토콜을 통해 송/수신되는 실제 데이터
    - `header` + [`header` + [`header` + `data`] ]
      - `IP datagram` or `IP packet`
      - 보내는 쪽 받는 쪽 IP address가 들어간다
      - TCP인지 UDP인지 정보도 담겨있다 (TCP: 6, UDP: 17)
        - IPv4: Protocol
        - IPv6: Next header
- **[ Link layer ]**  
  - 두 노드 사이에서 데이터를 주고받는 역할 (`IP datagram` or `IP packet` 전송)
  - 이를 위해 앞뒤로 `header` 와 `trailer` 가 붙는다
  - Link layer 프로토콜을 통해 송/수신되는 실제 데이터
    - `header` + [ `header` + [`header` + [`header` + `data`] ] ] + `trailer`
      - `frame`

## 라우팅과 소켓 통신

- **현재 Link layer**
- **Internet layer로 올라간다**
  - 헤더와 트레일러 뗌
  - 목적지 ip 주소 확인
  - 라우팅 테이블 참고하여 방향 확인
- **Link layer로 내린다**
  - 다음 노드로 이동 위해 헤더와 트레일러 붙여서 프레임 만들어서 전송
- **라우팅을 거쳐 목적지에 도착**
- **Internet layer로 올라간다**
  - 헤더와 트레일러 뗌
  - 목적지와 호스트 ip 주소 같은지 확인
  - TCP인지 UDP인지 확인
    - 나중에 Transport layer로 넘겨줄 때 필요
  - 출발지와 목적지 ip 주소, TCP or UDP 확인정보 따로 저장
    - protocol, s.ip_address, d.ip_address
  - ip datagram의 헤더 뗌
- **Transport layer로 올라간다**
  - 기능
    - Internet layer로부터 받은 segment나 datagram에 있는 payload를 적절한 socket으로 전달
      - Demultiplexing
    - 여러 socket들로부터 데이터를 수집해서 각각 segment나 datagram으로 만든 후 Internet layer로 내려보내는 것
      - Multiplexing
  - 호스트에서는 다양한 프로세스들이 동시에 실행중이다 - 어떤 어플리케이션 프로세스에게 보내야하나?
  - 특정 소켓으로 보내야 한다
    - UDP냐 TCP냐에 따라 다르다
      - UDP: UDP소켓들 중 목적지 ip주소와 목적지 포트번호 찾아가면 됨
      - TCP: 커넥션 맺는 단계라면 listening 소켓으로, 이미 맺어져 있다면 일반 TCP 소켓으로
        - TCP segment 헤더의 syn flag 확인 (1이면 listening 소켓으로, 아니면 0)
  - 출발지와 목적지 포트번호 따로 저장
    - s.ip_address, d.ip_address, s.port, d.port
  - segment 헤더를 떼어내고 Application layer로 전달
- **Application layer로 올라간다**
  - application header 따로 저장
    - s.ip_address, d.ip_address, s.port, d.port, application protocol header
  - payload와 header가 비즈니스 로직으로 전달