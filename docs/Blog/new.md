---
layout: default
title: Just The Docs 올릴것들
parent: 블로그 관리
---

1. 블로그 만들기
   1. 깃헙블로그 장단점
   2. jekyll이란? 다른 정적사이트 생성기랑도 비교하고, 왜 jekyll 선택했나
      1. 테마 많고 정보 교류 활발하고 강좌 많아서 따라하기 좋고 다른 정적페이지 생성기들끼리 서로 옮기기도 무난하니
   3. JTD 설명하고 적용방법
2. 한글검색 적용
   1. lunr 설명
   2. vendor에 js 파일들 넣었는데 왜 깃헙에 안뜨지
      1. 그냥 그거만 gitignore 제외하자
   3. this.use(lunr.multiLanguage('en', 'ko')); 어디다가 추가하나
3. 테마 컬러와 폰트 변경
   1. 커스텀 scheme 만들기
4. 구글 애널리틱스
5. 명언과 cors
   1. 에러메세지 - 경로문제였네
   2. {{ site.url }} 하니까 또 cors 뜨더라 - 그거 해결법은 gpt
   3. 그냥 귀찮아서 깃허브 주소 그대로 넣었다