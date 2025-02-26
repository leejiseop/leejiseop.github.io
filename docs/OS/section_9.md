---
layout: default
title: 9. 파일시스템
parent: OS
---

# 파일시스템
{: .no_toc }

1. TOC
{:toc}

### 섹션9 파일시스템

- 운영체제가 사용자의 요청을 받아 파일을 저장
- 파일 관리자 - 메모리 관리자가 페이지 테이블을 이용해서 가상주소를 물리주소로 변환하듯, **파일 테이블을 이용해서 파일을 관리**
  - 파일과 디렉토리 생성
  - 파일과 디렉토리 수정, 삭제
  - 파일 권한 관리
  - 무결성 보장
  - 백업과 복구
  - 암호화
- 저장장치의 전송 단위는 블록이지만, 사용자는 바이트 단위로 파일에 접근 가능해야 함
  - 파일 관리자가 중간에서 관리
- 파일의 구조
  - 헤더
    - 속성, 이름, 식별자, 유형, 크기 ...
  - 데이터
- 파일 디스크립터 - 운영체제가 파일을 관리하기 위해 정보를 보관하는 FCB (File Control Block)
  - 파일마다 독립적으로 존재
  - 저장장치에 존재하다가 파일이 오픈되면 메모리로 이동
  - 파일시스템이 관리, 사용자가 직접 참조 불가능
  - 사용자는 파일시스템이 건내준 FCB로 파일에 접근 가능
- 파일 = 데이터의 집합
  - 순차 파일구조
    - 파일의 내용이 연속적으로 이어짐 (카세트테이프)
      - lseek
      - 공간의 낭비가 없고 구조가 단순
      - 삽입, 삭제 - 탐색 비효율적
  - 직접 파일구조 (해시함수를 통해 저장 위치를 결정)
    - 해시 테이블 자료구조
    - json
    - 해시 사용 - 접근이 빠르다
    - 해시함수를 잘 골라야함, 저장공간 낭비 주의
  - 인텍스 파일구조
    - 순차접근과 직접접근의 장점을 가져옴
    - 음악 재생 프로그램의 재생 목록

- **디렉토리도 파일이다**
  - 파일에는 데이터가 저장, 디렉토리에는 파일 정보가 저장
- 일정한 크기로 나눈 공간을
  - 메모리에선 페이지라 부르고
  - 파일시스템에선 블록이라고 부른다 (1~8KB)
    - 하나의 파일은 여러개의 블록으로 나뉘어있다.
      - 연속 할당
        - 외부단편화가 발생하여 실제로는 사용 X
      - 불연속 할당
        - 디스크의 비어있는 공간에 데이터를 분산하여 저장 - 파일 시스템이 관리
        - 연결 할당
          - FCB가 연결 리스트의 헤드에 연결
        - 인덱스 할당
          - FCB가 데이터들의 인덱스를 가지고 있는 인덱스 블록에 연결
          - i-node - 이중 간접 포인터, 삼중 간접 포인터

- 파일 시스템은 효율적인 관리를 위해 빈 공간을 모아둔 free block list를 가지고있다.
  - 파일을 삭제하는 것은 파일 테이블의 헤더를 삭제하고 free block list에 추가는 것일 뿐이다.
  - 디지털 포렌식을 통한 데이터 복구 가능