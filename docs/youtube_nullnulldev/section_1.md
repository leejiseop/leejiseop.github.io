---
layout: default
title: 1. 웹 서비스 구조와 OS (1)
parent: 널널한개발자 면접 대비
---

# 웹 서비스 구조와 OS (1)

- 웹을 이루는 핵심 기술
  - HTTP + HTML(CSS) (http 1.0)
  - WAS(CGI)
  - DB

- URL과 URI
  - URI에 URL이 포함된다
  - https://host.domain
  - Protocol://Address:PortNumber/Path(or filename)**?Parameter=value**
    - URI
  - 웹: 요구와 응답 구조
    - 보통 ASCII 코드 문자열 형태 (바이너리일수도)
    - method
      - GET, POST
      - **PUT, DELETE -> 보안**
    - http 응답 코드
  - 웹 서비스 기본 구조
    - Web client -> TCP/IP 연결, HTTP 세션 생성, HTML GET ->  Web server
    - Web client <- HTML 수신하고 연결 종료 <-  Web server
      - Web server -> 사용자 요청 전달 -> WAS
        - WAS -> 정보조회 -> DB
        - WAS <- 정보제공 <- DB
      - Web server <- 회신 응답 <- WAS
    - 프론트, 네트워크 구조, HTTP 버전별 특성, DB 장애
      - **전체적인 구조와 흐름 파악**이 중요하다
      - 왜 이렇게 되었나? 어떻게 변해왔나? **역사를 알면 흐름이 보인다**