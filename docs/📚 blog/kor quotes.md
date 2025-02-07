---
layout: default
title: 블로그 한글 명언 적용
parent: 📚 블로그 관리
---

# 한글 명언 적용
{: .no_toc }

1. TOC
{:toc}

## 본문 하단에 명언 적용

블로그도 꾸미고 개인적으로 자극도 받을 겸, 본문 하단에 랜덤 명언 기능을 추가하였다.  
구글에 '인생 명언', 'developer quotes' 등을 검색하여 다양한 명언 JSON 리스트들을 구할 수 있었다.  
랜덤으로 명언을 띄우는 소스코드는 [링크](https://csj000714.tistory.com/507) 를 참조하여 수정하였다.  
JSON 파일을 프로젝트 안에 두고 fetch 명령어를 통해 불러와서 화면에 뿌리는 형식으로 적용했다.

fetch 주소를 localhost로 하니 Github pages 배포 이후 **CORS 문제**가 발생해서 주소를 나의 Github repository로 설정해두었다. **Github는 자체 서비스 간 CORS 요청을 기본적으로 허용**한다.

```js
<script>
   (async function () {
      try {
         const kor_quotes_response = await fetch('https://leejiseop.github.io/assets/js/kor_quotes.json');
         const phrase_ = await kor_quotes_response.json();
         const $q_message = document.getElementById('q_message');
         const $q_author = document.getElementById('q_author');
         const $q_author_detail = document.getElementById('q_author_detail');
         function ranQuote() {
            const ran1 = Math.floor(Math.random() * phrase_.length);
            $q_author.innerHTML = phrase_[ran1]['author'];
            $q_author_detail.innerHTML = phrase_[ran1]['author_detail'];
            $q_message.innerHTML = phrase_[ran1]['message'];
         }
         ranQuote();
         } catch (err) {
            console.log(err);
         }
   })();
</script>
```
해당 코드를 `footer_custom.html`에 추가하고 CSS를 적절히 다듬어 마무리하였다.
