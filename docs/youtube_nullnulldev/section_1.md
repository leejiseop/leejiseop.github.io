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

---

- Web = HTTP + HTML (티모시 버너스 리)
  - 첫 시작은 '문서 공유' 였다
  - 단방향 (요청받으면 전송한다)
- 좋은 개발은 유지보수 하기 좋은 것
  - UI + Control + Data 분리
- HTTP는 Stateless: 상태가 없다, 연결을 하지 않는다
  - TCP는 연결이란걸 한다
  - TCP 연결 위에 상태 개념이 없는 문서가 오간다(?) -> HTTP 1.0
- 움직임을 주자 -> 어떻게 움직일지 스트립트 같은게 있으면 되겠네 -> MochaScript -> LiveScript -> JavaScript
  - **스크립트 위치는 Server, 실행은 Client**
    - **CORS와 밀접한 연관**
  - HTML CSS image ... 등 정적 리소스에 이어서 이젠 JS까지 추가
  - HTML을 태그 구문분석해서 렌더링 해야한다 -> Parser(1), Rendering(2)
  - 그리고 렌더링에 JS가 반영되어야 한다 (일종의 인터프리터) -> JS engine(3)
    - **1 2 3 순서로 전송되고 구동된다**
- 단방향 -> 양방향 상호작용 -> **'상태' 라는 개념이 생김** -> 상태는 전이하기 마련 -> 그걸 기억해야함 -> 그런데 HTTP는 Stateless -> 프로토콜만으로 해결 안됨
  - server는 DataBase로 구현
  - client는 Cookie로 구현
    - key-value
  - POST로 데이터가 함께 넘어간다
    - = **사용자 입력** = **무조건 신뢰하면 안된다** = **검증 대상**
    - Server에 Application 탑재 = Web Application Server = WAS
    - Client - Web Server - WAS - DB
      - View <- WAS(Control) -> Model = **MVC Architecture**
    - 데이터를 받아서 상황에 알맞게 생성된 문서 -> 동적 문서