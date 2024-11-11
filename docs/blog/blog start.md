---
layout: default
title: 깃허브 블로그 만들기
parent: 블로그 관리
nav_order: 0
---

# 깃허브 블로그 만들기
{: .no_toc }

1. TOC
{:toc}

## Github Pages

**Github Pages**는 Github에서 제공하는 **정적 웹사이트 호스팅 서비스**이다. 즉, HTML, CSS, JavaScript 등의 파일들을 개인 리포지토리에 업로드하는것 만으로 간단한 웹사이트를 만들 수 있다.

프로젝트 문서화나 포트폴리오 목적으로 출발한 기능이지만, 이를 이용하여 개발 블로그를 운영하기도 한다.  
다른 블로그 서비스들과는 달리, 깃허브 블로그의 경우 나만의 블로그를 스스로 가꾸며 애정도 생기고  
동시에 깃허브 잔디도 심을 수 있으니 일석이조라고 생각했다!


Github Pages 블로그를 구축하는 경우 다양한 기능을 가진 페이지를 직접 구현하기는 번거로우므로, 주로 **정적 웹사이트 생성기**를 이용한다.  
정적 웹사이트 생성기는 **Jekyll**, **Hugo**, **Gatsby**, **Docusaurus** 등 여러 종류가 있지만, 다양한 테마와 활발한 정보교류, 쉽고 편한 강좌가 많아 따라하기가 쉬워 필자는 Jekyll을 선택하였다. 다른 생성기로 전환도 어렵지 않은 편이라 크게 고민되지 않았다.

## jekyll

위에서 설명했듯 [jekyll](https://jekyllrb.com/)은 정적 웹사이트 생성기이다. jekyll은 루비로 만들어진 프로그램이기 때문에 먼저 ruby 설치가 필요하다.  
맥은 ruby가 기본적으로 설치되어있지만, 그대로 jekyll을 설치하면 오류가 발생하므로 ruby 버전 관리자인 rbenv를 통해 최신 ruby 버전을 설치한다.  
```
brew update
brew install rbenv ruby-build
rbenv versions
```
이후 설치 가능한 루비 버전 목록을 확인하고, 최신 버전을 설치한 뒤
```
rbenv install -l
rbenv install 3.3.5
```
기존 시스템 ruby가 아닌 새로 설치한 버전을 사용하도록 global 설정을 변경해준다.
```
rbenv global 3.3.5
```
이후 jekyll과 bundler를 설치한다.
```
gem install jekyll bundler
```

## repository 생성과 theme 적용

구글에 jekyll theme를 검색하면 테마를 모아둔 다양한 사이트들을 확인할 수 있다.  
필자는 [Just-The-Docs](https://just-the-docs.com/)라는 테마를 사용하였다.  
Just-The-Docs의 github repo를 fork해도 되지만, 그러면 블로그 업데이트 시 나의 깃허브에 잔디가 심어지지 않는다.

비어있는 레포지토리에서 jekyll을 초기화 생성하여 동작을 확인할 수도 있지만, 이후 테마 적용을 위한 재 초기화 과정이 번거로우므로 처음부터 바로 [Just-The-Docs Template](https://just-the-docs.github.io/just-the-docs-template/)을 사용하여 본인의 깃허브 닉네임이 들어간 `username.github.io` 가 이름인 레포지토리를 생성한다.  
이후 settings - pages - build and deployment - source 를 GitHub Actions 로 변경해준다.

원하는 디렉토리에 clone하고 해당 디렉토리에 이동하여
```
bundle install
bundle update
bundle install
bundle exec jekyll serve
```
을 실행하고 `http://127.0.0.1:4000/` 에 접속하면 jekyll 페이지가 생성되었음을 확인할 수 있다.
`http://127.0.0.1:4000/` 에서 게시글 작성 과정을 실시간으로 확인 가능하고, repository에 push하면 그대로 `username.github.io` 사이트에 반영된다.

## 게시물 작성법

docs폴더 안에 마크다운(.md) 파일을 작성하면 블로그 페이지 좌측에 포스트가 뜨게 된다. 마크다운 작성 시 최상단에 아래 내용이 항상 들어가야한다.
```
---
layout: default (default: 좌측 네비바 표시, minimal: 좌측 네비바 사라지고 최대화면)
title: this is title (제목)
nav_order: 5 (표시할 순서)
parent: how to jekyll (부모의 title) (생략가능)
---

마크다운 작성
```
docs 내부에 폴더를 만들어서 관리할 경우, 폴더 내부에 index.md를 만들고 마찬가지로 위와 같이 작성하여 카테고리 대표 페이지를 만들 수 있다.

## 블로그 꾸미기

사실 위 방식대로 just-the-docs template을 사용하여 블로그를 개설한 경우, 최소한의 파일들만 생성되기 때문에 커스터마이징이 쉽지 않았다. 필자는 just-the-docs의 github repo를 통째로 가져와서 직접 css를 편집하여 취향껏 꾸몄다. [MIT 라이선스](https://ko.wikipedia.org/wiki/MIT_%ED%97%88%EA%B0%80%EC%84%9C)가 적용되어 있으므로, 마음껏 사용은 가능하지만 라이선스 및 저작권 고지는 항상 유지되어야 한다. 디렉토리 내 `LICENSE` 파일을 지우지 않도록 유의한다.