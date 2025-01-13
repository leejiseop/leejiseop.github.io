---
layout: default
title: 1. 웹 서비스 구조
parent: 널널한개발자 면접 대비
---

# 웹 서비스 구조
{: .no_toc }

1. TOC
{:toc}

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

## 웹의 흐름

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
    - = **사용자 입력** = **무조건 신뢰하면 안된다** = **검증 대상** = 보안
      - SQL injection, XSS attack
        - 보안장치가 해줄수도 있고
        - WAS에서 구현할 수도 있고 -> **시큐어 코딩**
    - Server에 Application 탑재 = Web Application Server = WAS
    - Client - Web Server - WAS - DB
      - View <- WAS(Control) -> Model = **MVC Architecture**
    - 데이터를 받아서 상황에 알맞게 생성된 문서 -> 동적 문서

- 동적 환경 변화
  - **view 의존성**: 기기, 브라우저에 의존적 (브라우저 종류, 해상도, ...)
  - 프론트와 백이 긴밀해짐 -> 유지보수하기 힘들어졌다
    - 분리 시도
    - 동적 문서의 핵심은 결국 데이터, 출력은 프론트 단에서 알아서...
      - 데이터만 보내자
      - XML, JSON
    - 각 기기에 맞춰서 HTML 생성 -> 클라이언트가 스스로
      - 클라이언트에겐 JS가 있다
      - 프론트엔드 라이브러리 탄생
        - React, Vue.js, ...

- 전통적 의미의 web이 아닌 다른 개념이 성립하기 시작
  - Request: 문서 달라는 의미에서 입출력을 해달라는 의미로 변화(I/O Request)
    - 읽던지 쓰던지
    - **CRUD**
      - **RESTful API**

- WAS와 DB 연결
  - ODBC, JDBC, ...
  - DB가 에러나면 모든것이 막힌다
    - 장애 대응
    - DB 질의응답 - **응답시간 모니터링**
      - JVM에서 무슨 일이 나는지 안나는지
      - **JVM**
        - JSP, PHP, ASP, node.js ... 결국은 자바로 변환되어 JVM 위에서 실행
          - Spring: Java reflextion 활용한 프레임워크 = Spring Container(객체)
      - **APM**
        - DB 응답시간 모니터링
        - JVM 상태 모니터링
        - 제니퍼, Scouter, Xlog
          - 운영업무
          - **운영을 고민하며 개발해야한다**

- 웹서버 - WAS - DB
  - 3-tier 구조
  - 보안
    - IPS - SSL - WAF - 웹서버 - (여기까지가 DMZ) - (WAF) - WAS - DB
      - **ISMS-P 인증**

