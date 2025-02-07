---
layout: default
title: 블로그 한글 검색 적용
parent: 📚 블로그 관리
---

# 블로그 한글 검색 적용
{: .no_toc }

1. TOC
{:toc}

## Just-The-Docs와 lunr

Just-The-Docs에는 [lunr](https://lunrjs.com/)라는 검색 라이브러리가 적용되어있다.  
하지만 기본적으로 영어검색만 지원되므로, 한국어 검색기능을 추가하려면 약간의 수정이 필요하다.

## 적용 방법

- 먼저, [lunr github repo](https://github.com/MihaiValentin/lunr-languages?tab=readme-ov-file) 에서 아래 파일을 다운로드하여 `/assets/js/vendor/` 안에 추가해준다
```
lunr.ko.js
lunr.multi.js
lunr.stemmer.support.js
```
- `/_includes/head.html` 에서 추가한 js 파일들을 import 해준다.
```html
<script src="{{ '/assets/js/vendor/lunr.min.js' | relative_url }}"></script>
<!-- 내용 추가 -->
<script src="{{ '/assets/js/vendor/lunr.multi.js' | relative_url }}"></script>
<script src="{{ '/assets/js/vendor/lunr.stemmer.support.js' | relative_url }}"></script>
<script src="{{ '/assets/js/vendor/lunr.ko.js' | relative_url }}"></script>
<!-- 내용 추가 -->
```

{: .warning }
min - multi - stemmer.support - ko 순서가 바뀌지 않도록 주의한다.

- `.prettierignore` 에도 아래 내용을 추가해준다.
```
assets/js/vendor/lunr.min.js
assets/js/vendor/lunr.ko.js
assets/js/vendor/lunr.multi.js
assets/js/vendor/lunr.stemmer.support.js
```
- `/assets/js/just-the-docs.js` 에서 index 처리 부분을 찾아 최상단에 한글검색 적용 코드를 추가해준다.
```js
var index = lunr(function(){
        this.use(lunr.multiLanguage('en', 'ko')); // 한국어 검색

        this.ref('id');
        this.field('title', { boost: 200 });
// ... 생략
```

## 오류

이렇게 다 적용하고 로컬 서버에서도 정상 동작을 확인했는데... 이상하게 배포만 하면 오류가 생겼다.  
알고보니 리포지토리의 `/assets/js/vendor/`에 **gitignore**가 적용되어 lunr 한글검색 관련 js 파일들이 업로드 되지 않아 생기는 오류였다.  
`.gitignore` 파일에 `!/assets/js/vendor/` 구문을 임시로 추가하여 정상적으로 업로드 되도록 조치하여 해결하였다.
