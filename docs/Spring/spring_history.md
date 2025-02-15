---
layout: default
title: Spring 간단 역사
nav_order: 0
parent: Spring
---

# Spring 간단 역사
{: .no_toc }

1. TOC
{:toc}

- Web HTML
  - 정적 페이지
  - 데이터 입출력의 필요성이 생김
- Servlet
  - 웹 페이지 동적 생성
  - 자바 안에 HTML을 넣는다 -> 복잡
- JSP
  - HTML에 Java를 넣는다 -> 편하지만 스파게티
- MVC
  - 복잡하니 분리하자 -> Front + Back
  - Java 내부에서도 분리하자 -> MVC
  - 아직 표준이 없다
- MVC Framework
  - MVC 개발 프레임웍을 만들어 배포
  - 개발자들에게 봄을 선사하겠다 -> Spring -> Springboot
- 파일 저장의 한계
  - Web을 통해 DB를 조회할 수 있도록 중간에서 WAS가 돕게 됨
- 어떤 환경으로 개발하든 `.war`로 묶이면 결국 비슷하다
  - 톰캣에서 실행하기 위해
  - Springboot는 톰캣이 내장되어있기 때문에, `.jar`를 실행하기만 하면 된다
- 서블릿의 init, destroy, service 메서드
  - service 메서드
    - 요청을 처리할때 호출
    - 큰 흐름: GET 요청이 들어오면 doGET 호출, POST 요청이 들어오면 doPOST 호출, ...
    - 결국 개발자가 할 일은 doXXX 메서드를 찾아서 재정의 하는것
- 서블릿 컨테이너
  - **서블릿 객체는 싱글톤**
  - 요청이 들어오면 해당 요청과 매핑된 서블릿을 찾는다 -> 설정 파일에 정의
    - 무슨 요청이 들어오면 -> 무슨 서블릿으로 처리하겠다 -> res, req를 인자로 넘겨줘서 service 실행
- 동시에 여러 요청이 들어오면
  - 멀티스레딩
    - 컨텍스트 스위칭 -> 속도 저하
    - 스레드 제한없이 생성 -> 하드웨어 한계 넘기면 서버 터짐
  - 핸들러의 공통 로딩 중복의 문제
    - 매니저(프론트 컨트롤러 = **Dispatcher Servlet**)가 주문, 계산, 포장을 담당하고
    - 각 메뉴 전문 알바생이 음료를 제작한다
      - 이전까지는 요청마다 서블릿을 지정하고 수행시마다 스레드를 생성했다면
      - 이제는 하나의 서블릿만 정의하고 모든 요청 수행
    - 계산담당, 포장담당도 따로 분리하는것이 좋다
      - Handler Mapping, ViewResolver, HandlerAdapter ...
        - 다 인터페이스들이고, 다양한 구현체들이 있다 -> **SpringContainer**로부터 주입
        - 개발자는 요청 처리 로직에만 집중한다

출처: [링크](https://youtu.be/PH8-V6ah0XQ?si=b6DhXw6zBucbKt0u)  
[Servlet vs Spring](https://youtu.be/calGCwG_B4Y?si=18ju66rn_MqiFjzr)