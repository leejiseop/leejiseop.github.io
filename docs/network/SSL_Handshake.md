---
layout: default
title: SSL Handshake와 인증서
parent: Network
---

# SSL Handshake와 인증서
{: .no_toc }

1. TOC
{:toc}

출처: https://nuritech.tistory.com/25

- HTTPS 프로토콜을 사용하기 위해서는 서버가 인증기관(CA)으로부터 SSL 인증서를 발급받아야 한다

## 인증서 발급 과정

1. 서버가 공개키와 비밀키를 생성
2. 서버가 CA에게 공개키와 각종 정보 전달
3. CA가 서버로부터 받은 정보를 담아 SSL 인증서 발급
4. **CA가 공개키와 비밀키를 생성하고 인증서를 비밀키로 암호화**
5. 암호화된 SSL 인증서를 서버로 전송하여 완료

![SSL Handshake](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FblusyN%2FbtraoOJE4fj%2FTBe4rGKq1fbTVONcSHine0%2Fimg.png)

## 클라이언트와 서버 통신 과정

- **Client -> Server (Client Hello)**
  - 클라이언트가 사용 가능한 cipher suite 목록
    - ex. TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
    - 어떤 프로토콜(TLS/SSL)을 사용할 지, 어떤 방식으로 암호화할 지
  - session id
  - ssl protocol version
  - ...
- **Client <- Server (Server Hello)**
  - 선택한 cipher suite
  - ssl protocol version
  - ...
- **Client <- Server (Certificate, Server Key Exchange, Server Hello done)**
  - Certificate 라는 내용의 패킷을 보낸다 (SSL 인증서)
    - signature algorithm
    - public key info
    - ...
  - Server key Exchange
    - SSL 인증서에 서버의 공개키가 없는 경우, 서버가 직접 전달한다
- **Client : Server의 SSL 인증서 검증**
  - CA가 CA의 비밀키로 인증서를 암호화한 상태
  - 그래서 SSL 인증서는 해당 CA의 공개키를 이용해서만 복호화 할 수 있다
    - CA의 공개키는 클라이언트가 가지고있음
    - 리스트에 없다면 외부 인터넷을 통해 인증기관의 정보와 공개키를 확보
      - 인터넷 연결이 필요한 이유
- **Client -> Server 키 전달 (Client Key Exchange, Change Ciper Spec)**
  - 클라이언트가 대칭키를 생성
  - Client Key Exchange
    - 대칭키를 서버의 공개키로 암호화하여 전달한다
    - 서버의 공개키 획득 경로 : Certificate
      - SSL 인증서를 복호화하여 획득
      - 인증서 내부에 없을 경우 Server key Exchange
  - SSL Handshake의 완료를 알ㄹ
- **Client / Server SSL Handshake Finished**
  - 서버와 클라이언트 모두 동일한 대칭키를 가지고 통신 시작
  - 통신할 준비가 완료되었다는 Change Ciper Spec을 서버로 보낸다

## 요약

서버가 CA에게 공개키 전달  
CA가 서버의 공개키를 담은 인증서 발급  
CA가 인증서를 CA의 비밀키로 암호화하여 서버로 전달  

클라이언트가 연결 요청  
서버가 클라이언트에게 인증서 전달  
클라이언트가 CA의 공개키로 인증서 복호화하여 검증  
인증서 안에 들어있던 서버의 공개키로 클라이언트와 서버의 대칭키 암호화  
서버가 클라이언트의 대칭키를 자신의 비밀키로 복호화하여 사용  

## 3줄 요약
1. CA의 비밀키로 인증서를 암호화하여 서버로 전달
2. 서버에게서 전달받은 인증서를 클라이언트가 CA의 공개키로 복호화
3. 인증서에 들어있던 서버의 공개키로 통신에 사용할 대칭키를 암호화

## 추가 - X.509 인증서?

- ㅁㄴㅇㄹ