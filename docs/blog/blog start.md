---
layout: default
title: 깃허브 블로그 만들기
parent: 블로그 관리
---

# 깃허브 블로그 만들기
{: .no_toc }

1. TOC
{:toc}

## Github Pages

**Github Pages**는 Github에서 제공하는 **정적 웹사이트 호스팅 서비스**이다. 즉, HTML, CSS, JavaScript 등의 파일들을 개인 리포지토리에 업로드하는것 만으로 간단한 웹사이트를 만들 수 있다.

프로젝트 문서화나 포트폴리오 목적으로 출발한 기능이지만, 이를 이용하여 개발 블로그를 운영하기도 한다.  
티스토리나 벨로그 등의 블로그 서비스들과는 달리, 깃허브 블로그의 경우 나만의 블로그를 스스로 가꾸며 애정도 생기고 동시에 깃허브 잔디도 심을 수 있으니 일석이조라고 생각했다!


Github Pages 블로그를 구축하는 경우 다양한 기능을 가진 페이지를 직접 구현하기는 번거로우므로, 주로 **정적 웹사이트 생성기**를 이용한다.  
필자는 **jekyll**이라는 생성기를 이용했지만, **hugo**나 **gatsby** 등 다양한 생성기가 있으니 취향에 맞춰 사용하면 된다.

## jekyll

위에서 설명했듯 [jekyll](https://jekyllrb.com/)은 정적 웹사이트 생성기이다. jekyll은 루비로 만들어진 프로그램이기 때문에 먼저 ruby 설치가 필요하다. 맥은 ruby가 기본적으로 설치되어있지만, 그대로 jekyll을 설치하면 오류가 발생하므로 ruby 버전 관리자인 rbenv를 통해 최신 ruby 버전을 설치한다.  
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

## username.github.io

본인의 깃허브 닉네임이 들어간 `username.github.io` 가 이름인 레포지토리를 생성하고, 원하는 디렉토리에 clone한다.  
해당 디렉토리에 이동하여
```

```




https://2dowon.github.io/docs/algorithm/python-book/dynamic-programming/
1. 블로그 만들기
   1. https://jeesub.github.io/blog/jekyll-blog-%EB%A7%8C%EB%93%A4%EA%B8%B0/
   2. 
   3. jekyll이란? 다른 정적사이트 생성기랑도 비교하고, 왜 jekyll 선택했나
      1. 테마 많고 정보 교류 활발하고 강좌 많아서 따라하기 좋고 다른 정적페이지 생성기들끼리 서로 옮기기도 무난하니
   4. JTD 설명하고 적용방법