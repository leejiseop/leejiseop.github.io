---
layout: default
title: 1. API 개발 기본
parent: 실전! 스프링부트와 JPA 활용2
---

# 1. API 개발 기본
{: .no_toc }

1. TOC
{:toc}

JPA를 사용하면서 API 만들기 -> 어떻게 해야 좋은 방향인가?

## 회원 등록 API

- (템플릿 엔진) 렌더링 컨트롤러와 api 컨트롤러 패키지를 분리하면 편하다
  - 서로 공통 처리해야할 요소가 다르다
  - 애러발생시 HTML or JSON api spec...
- 엔티티 api는 여러 곳에서 쓰인다
  - 예고 없이 엔티티 속성명이 바뀔 확률도 높다 (name -> username)
    - api spec도 바뀌어버림!
    - 기존에 잘 사용하던 곳들에서 에러 발생
  - 결국 컨트롤러에서 엔티티 json을 파라미터로 그대로 받아서 생기는 문제
    - **별도의 DTO**를 만들어서 파라미터로 받아 해결 -> v2
    - **엔티티를 외부에 노출하지 않는게 안전 (중요)**
  - 기존: json을 받아서 memberService.join 후 return id
  - 변경: dto를 받아서 member를 생성 후 join 후 return id
    - name -> username 바뀌어도 바로바로 컴파일 에러가 뜬다

## 회원 수정 API

- 멱등성: 여러번 호출해도 결과는 변함없다
- 커맨드워 쿼리는 분리해야한다, **CQS** [SheryLog(링크)](https://velog.io/@yena1025/CQS-Command-Query-Separation)
- 엔티티를 통째로 return하지 말고 꼭 DTO 생성하여 return
- 배열을 그대로 JSON으로 반환하기보다 한단계 껍데기를 씌워서 반환하는것이 향후에 유연하게 대처하기 좋다
- **REST 스타일**
  - 전체 업데이트: `PUT`
  - 부분 업데이트: `PATCH` or `POST`

```json
{
    {
        "name": "1"
    }, 
    {
        "name": "2"
    },
    {
        "name": "3"
    }
}
```

```json
{
    "count": 4,
    "data": [
        {
            "name": "1"
        },
        {
            "name": "2"
        },
        {
            "name": "3"
        }
    ]
}
```

- asdf

## 회원 조회 API

- 마찬가지로 **API 응답 스펙**에 맞추어 **별도의 DTO를 반환**

{: .highlight}
실무에서는 `member` 엔티티의 데이터가 필요한 API가 계속 증가하게 된다.
어떤 API는 `name` 필드가 필요하지만, 어떤 API는 `name` 필드가 필요없을 수 있다.
결론적으로 엔티티 대신에 API 스펙에 맞는 **별도의 DTO를 노출**해야 한다.

- 엔티티가 변해도 API 스펙이 변경되지 않는다.
- `Result` 클래스로 컬렉션을 감싸서 향후 필요한 필드를 추가할 수 있다.3