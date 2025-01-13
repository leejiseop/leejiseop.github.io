---
layout: default
title: 3. Web server와 HTTP
parent: 널널한개발자 면접 대비
---

# Web server와 HTTP
{: .no_toc }

1. TOC
{:toc}

HTTP/2가 압도적으로 빠르다
- 이유와 특징

## HTTP 버전별 차이 - HTTP/2가 빠른 이유

![http2 vs http3](https://images.ctfassets.net/ee3ypdtck0rk/4du4aqnKuOLU4YbbHWYSfv/c88d774f278090e6ef3a5435b46bbfea/Screen_Shot_2021-01-28_at_6.54.47.png?w=1558&h=636&q=80&fm=webp)
https://ably.com/topic/http-2-vs-http-3

- HTTP 1.1 vs HTTP/2
  - HTTP 1.0 1.1
    - 리소스 3개면 TCP 연결도 3번
  - HTTP 1.1
    - 세션 수는 하나로 줄이고 직렬 요청
  - HTTP/2
    - 한 세션에 **병렬 요청 (Binary Framing)**
    - TLS(구 SSL) 적용 (option) -> HTTPS
    - 서버 푸시
    - 헤더 압축
      - 중복 요소들이 많았다 - 2번째 요청부터는 중복요소 안보냄
  - HTTP/3
    - **TLS (Must)**
    - TCP -> **UDP, QUIC**


## IPS와 Inline 구조

- OSI 7 layer의 L3~L4
  - Inline과 Out of path 구조
    - Inline - 공기청정기 필터
    - Out of path - sensor
  - 패킷 필터링과 IPS(침입 방지 시스템)
    - Inline 장치
    - HTTP 통신 감시 보안장치
    - HTTPS는 WAF(Application Proxy)
      - nginx, apache가 이런 역할을 해줄 수 있다
- Inline 구조, IPS, DMZ
- **SSL 인증서**
  - IPS에 설치해야할 수도 있다
  - 또는 로브 밸런서 L4 스위치에 설치해야 할 수도

## WAF와 Proxy 구조

- Inline 보안구조: 마치 없는 것 처럼
- Proxy는 소켓수준 작동
  - 1. client(3.3.3.3) - server(10.10.10.20) 접속
  - 2. clisnt - proxy(50.50.50.50) - server
    - proxy가 client 대신 접속해서 정보를 알려준다
    - server 입장에서는 client가 아니라 proxy가 접속한것으로 인지
- WAF도 proxy 방식
  - WAF(5.5.5.5)
  - client가 server 주소 알려달라고 DNS 질의하면 5.5.5.5(WAF) 알려줌
  - **모든 입력은 신뢰하면 안된다 - 검증**
    - 파라미터, 파일 업로드 ... 검사
  - SSL 인증서 배치

## SSL 인증서는 어디에 설치되나?

- "내가 접속한게 진짜 Google이 맞나" client에서 검증, **공증**
  - **PKI, X.509**
    - PAA(과학기술정보통신부, 미래창조과학부), PCA(정책인증기관, KISA), CA(인증기관), RA(등록기관)
  - client가 생성한 **session key로** HTTP -> **HTTPS 암호화**
  - CA -> RA -> 인증서 -> 서버 설치
- PKI 인증체계
  - **CA의 PK로 Hash 결과가 암호화** 되어 있으며 검증 과정에서는 PC에 사전 배포된 CA Public key로 암호를 풀어 **검증**할 수 있다