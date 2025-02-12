---
layout: default
title: Spring 간단 역사
nav_order: 0
parent: etc
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