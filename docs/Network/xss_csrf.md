---
layout: default
title: XSS와 CSRF
parent: Network
---

# XSS와 CSRF

{: .no_toc }

1. TOC
{:toc}

## XSS

- Cross-site Scripting  
- 사이트에 스크립트를 삽입(ex. 게시글 내용에 자바스크립트 코드) -> 이후 접속자들이 해당 코드를 실행하게 됨
- 일반적으로 쿠키나 세션 토큰 등 민감정보 탈취
  - Stored XSS
    - 게시판, 댓글, 닉네임 등 스크립트가 서버에 저장되어 실행
  - Reflected XSS
    - URL 파라미터에 스크립트를 넣어 즉시 실행 -> 브라우저에서 차단되는 경우가 많다
- 해결법
  - javascript 접근 옵션
  - **CORS**
  - character escaping -> plain text
  - 네이버: Lucy XSS Filter

## CSRF

- Cross-Site Request Forgery(위조)
- 자신의 의지와 무관하게 공격자가 의도한 행동을 하게 만듬(낚시)
- 서버에서는 POST와 GET방식을 처리하는데 따로 구분을 두지 않는다

만일, A라는 사이트의 사용자 개인 비밀번호 변경을 하는 주소 패턴이 `http://example.com/user.do?cmd=user_passwd_change&user=admin&newPwd=1234`라고 한다면 이러한 링크를 사용자의 메일로 보내는데, 만약 사용자가 메일을 읽게 되면 해당 사용자의 패스워드가 `1234`로 초기화된다.  
이를 관리자에게 보내서 일반 계정을 관리자 계정으로 바꾸도록 하거나, 관리자 계정 패스워드를 바꾸는 데 이용한다면 해당 사이트의 모든 정보가 해킹당하는 데는 오랜 시간이 걸리지 않는다.  
출처: [나무위키](https://namu.wiki/w/CSRF)  

- 해결법
  - GET이 아닌 POST 요청으로 -> 하이퍼링크 공격 원천 차단
  - Referrer 검증
    - 같은 도메인 상에서 요청이 들어오지 않는다면 차단
  - CSRF 토큰
    - 랜덤한 수를 사용자의 세션에 저장하여 사용자의 모든 요청에 대해 서버단에서 검증
      - 로그인시 or 작업요청시 CSRF 토큰을 생성하여 **세션에 저장**
      - 요청 페이지에 CSRF 토큰을 hidden으로 보이지않게 세팅하여 전송
  - CAPTCHA 사용

## Drive-by Download

- 악성 SW를 다운로드 시키도록 유도
