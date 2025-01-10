---
layout: default
title: 2. 운영체제 기술
parent: 널널한개발자 면접 대비
---

# 운영체제 기술

- 플랫폼: OS + hardware
- 플랫폼에 의존적인 코드: Native code -> C, C++
  - Java: Managed code
- JVM: vCPU + vMem + vOS ...

## 브라우저에 URL을 입력하면 일어나는 일

- 뭘 알고있는지 포괄적으로 보겠다
- 1. url 입력
  - protocol://host(www).domain(abc.com)
    - abc.com 도메인에 속한 이름이 www인 컴퓨터에 protocol 연결
  - 인터넷은 ip 주소 기반으로 접속하게 되어있다
    - http에 앞서 tcp/ip 통신이 가능해야한다
- 2. ip주소를 알아내야 한다
  - hosts 파일
    - www.a.com -> 123.123.123.123 단순 기술
    - 가장 우선 참고하는 곳이라 **보안 중요** -> 임의 변조 못하도록
  - DNS Cache
  - 없다면 DNS query
    - ex. 인터넷이 KT -> KT의 DNS 서버 (설정에 따름)
- 3. DNS query
  - 응답 오면 DNS Cache 업데이트
- 4. TCP/IP 연결
- 5. HTTP request
  - HTTP response
- 6. HTML 파싱, 렌더링
  - 이후 데이터 가져올때 CORS 정핵
- 7. 6번 웹 리소스들을 디스크에 캐싱

- DNS 작동 원리 www.abc.com
  - ROOT DNS
    - 전 세계에 13대
  - com 관리하는 DNS 서버 알려줌
  - abc 관리하는 DNS 서버 알려줌
  - www 주소 알려줌

- 이 외에는 프론트.. 로그인.. WAS.. DB.. 보안..

