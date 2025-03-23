---
layout: default
title: 2. API 개발 고급 - 준비
parent: 실전! 스프링부트와 JPA 활용2
---

# 2. API 개발 고급 - 준비
{: .no_toc }

1. TOC
{:toc}

- **xToOne** 관계 최적화
- 양방향 연관관계의 한쪽은 `@JsonIgnore` 처리
- `order`의 `member`와 `delivery` 는 지연 로딩이다.
  - (fetch = LAZY)
  - 따라서 실제 엔티티 대신에 프록시 존재
  - **bytebuddy**
    - `org.hibernate.proxy.pojo.bytebuddy`
    - 프록시 객체를 생성할 때 사실은 `private Member member = new ByteBuddyInterceptor()`이 실행된다
  - json 처리시 오류발생(일반적인 객체가 아니다)
  - 지연로딩의 경우, hibernate에게 bytebuddy는 json 뿌리지 말라고 명령가능
    - hibernate 5 module [(링크)](https://mvnrepository.com/artifact/com.fasterxml.jackson.datatype/jackson-datatype-hibernate5)

```java
// build.gradle
// implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'
// 보통 version은 빼도 된다 -> spring boot가 최적화된 버전 관리해줌

@Bean
Hibernate5Module hibernate5Module() {
    Hibernate5Module hibernate5Module = new Hibernate5Module();
    //강제 지연 로딩 당겨오기 설정
    // 예시로 보여주는거지, 이런거 비추
    hibernate5Module.configure(Hibernate5Module.Feature.FORCE_LAZY_LOADING, true);
    return hibernate5Module;
}
```

- 불필요한 쿼리까지 다 날린다
- 이렇게 하지말고 애초에 DTO 생성해서 반환하면 된다
- 웬만하면 항상 **지연로딩** 사용 + 성능 최적화 필요한 경우 **fetch join**