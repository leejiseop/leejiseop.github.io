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

- HTTPS 프로토콜을 사용하기 위해서는 인증기관(CA)로부터 SSL 인증서를 발급받아야 한다

## 인증서 발급 과정

1. 서버가 공개키와 비밀키를 생성
2. 서버가 CA에게 공개키와 각종 정보 전달
3. CA가 서버로부터 받은 정보를 담아 SSL 인증서 발급
4. **CA가 공개키와 비밀키를 생성하고 인증서를 비밀키로 암호화**
5. 암호화된 SSL 인증서를 서버로 전송하여 완료

![SSL Handshake](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FblusyN%2FbtraoOJE4fj%2FTBe4rGKq1fbTVONcSHine0%2Fimg.png)

wfe