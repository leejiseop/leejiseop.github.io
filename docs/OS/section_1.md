---
layout: default
title: 1. 프로세스와 쓰레드
parent: OS
---

# 프로세스와 쓰레드
{: .no_toc }

1. TOC
{:toc}

### 섹션1 프로세스와 쓰레드

**프로그램**이란? 저장장치에 **저장된** 명령문의 집합체  
**프로세스**란? RAM에 올라와서 **실행중인** 프로그램  

**프로세스의 구조**  

- code - 자신을 실행하는 코드
- data - 전역변수, static 변수
- heap - 프로그래머가 런타임시 메모리를 동적할당
- stack - 지역변수, 함수 호출시 필요한 정보들(매개변수, 돌아갈 주소 등…)

컴파일 과정  

- test.c
  - **전처리기** - 치환, 파일불러오기
- test.i
  - **컴파일러** - 어셈블리어로 변환
- test.s
  - **어셈블러** - 기계어로 변환
- test.o
  - **링커** - 라이브러리, 다른 소스코드 연결
- test.exe

멀티프로그래밍과 멀티프로세싱  

- 유니프로그래밍 - 메모리에 오직 하나의 프로세스가 올라옴 **(메모리 관점)**
- **멀티프로그래밍** - 메모리에 여러 프로세스가 올라옴 **(메모리 관점)**
- **멀티프로세싱** - CPU가 여러 프로세스를 시분할 처리함 **(CPU 관점)**
- 스와핑 - 메모리와 저장장치의 프로세스 교환 (유니프로그래밍에서의 멀티프로세싱)

PCB  

- 프로세스의 상태를 저장하고있는 **Process Control Block**
- 프로세스가 만들어지면 PCB도 생성
- PCB끼리 **연결리스트로 연결**되어있다
  - 프로세스가 종료되면, 연결리스트에서 해당 PCB 제거
- 구성
  - 포인터
    - 부모와 자식 프로세스
    - 프로세스 간 전환에 이용
  - 프로세스 상태
    - **프로세스의 5가지 상태** - **생성, 준비, 실행, 대기, 완료**
  - 프로세스 ID
    - 프로세스 식별용 숫자
  - 프로그램 카운터
    - 다음에 실행될 명령어의 주소
    - 시분할 처리에서 중요
  - 레지스터 정보
    - 가장 최근 레지스터 값들 저장
    - CPU를 뺏기고 다시 시작할 때 이전 사용값 복구 용도
  - 메모리 관련 정보
    - 프로세스 위치 정보
    - 메모리 침범을 막기 위한 경계레지스터 값
  - CPU 스케줄링 정보
    - 우선순위, 최종 실행시간, CPU 점유시간
  - 등…
- **프로세스 상태** - 시분할 처리를 위한
  - 생성 - PCB를 생성하고 메모리 적재를 요청한 상태
    - 생성 → 준비
  - 준비 - 메모리 적재 후 CPU 사용을 위해 기다리는 상태 (**CPU 스케줄링**)
    - 대부분 프로세스들의 상태
    - 차례가 오면 실행 상태로 전환 (CPU 스케줄러에 의해 할당)
  - 대기 - 입출력 대기중 (그동안 CPU는 다른 프로세스 실행)
    - 입출력 완료되면 준비상태로 전환하고 순서 기다림
  - 실행 - 준비 상태의 프로세스가 **CPU를 할당받아 실행됨** (실행개수: CPU의 개수만큼)
    - 시간이 지나면 강제로 CPU 스케줄러에 의해 준비로 전환
    - 입출력 필요하면 대기상태로 전환
  - 완료 - 프로세스가 종료된 상태
    - 메모리에서 제거, PCB 제거
    - 실행 → 완료

**컨텍스트 스위칭**  

- 다른 프로세스로 전환을 위해 현재 실행중인 프로세스의 상태를 PCB에 저장하고, 다른 프로세스의 상태값으로 교체하는 작업
- 발생 이유
  - CPU 점유시간이 다됨
  - IO 요청
  - 다른 종류의 인터럽트

프로세스 생성 과정  

- 컴퓨터가 부팅되고, 0번 프로세스 최초 실행
- 나머지 모든 프로세스들은 새로 생성하지 않고, **0번 프로세스를 복사하여 사용**
  - 생성보다 **복사가 속도가 더 빠르다**
  - fork()로 복사
  - 이후 exec()을 사용하여 코드와 데이터 영역을 원하는 값으로 덮어 씌워 사용
  - wait()로 자식프로세스가 끝나길 기다릴 수도 있다
    - 자식 프로세스가 끝남을 알리면, 부모 프로세스가 자식 프로세스를 완전히 없애고 본인이 마저 실행
    - 만약 부모 프로세스가 자식 프로세스보다 먼저 종료되거나,
    자식 프로세스가 비정상적으로 종료돼 exit() 신호를 주지 못해서 ExitStatus를 읽지 못해
    자식 프로세스가 메모리에 계속 살아있는 경우, **좀비 프로세스**라고 한다.
      - 이런 메모리 차지는 재부팅으로 메모리 초기화
  - 복사된 프로세스들과는 부모-자식 관계

쓰레드  

- **프로세스** - **운영체제가 작업을 처리하는 단위**, 각각의 은행 지점
  - **프로세스 수만큼** PCB, 코드, 데이터,  스택, 힙영역 **할당 필요**
  - 비슷한 프로세스가 너무 많아지면 메모리 차지가 많아짐 (웹브라우저의 탭)
  - 프로세스 간 통신 IPC는 속도가 느리다
  - 프로세스 내에 여러 쓰레드를 만들어서 해결
- **쓰레드** - **프로세스 내에서의 실행 단위**, 은행 지점 하나에 속한 고객 창구 여러 개
  - **한 프로세스 내의** PCB, 코드, 데이터, 힙 영역 등의 **자원들을 공유함**
  - **스택은 쓰레드마다 하나씩**
  - 쓰레드 구분을 위해 쓰레드ID 부여
  - Thread Control Block, TCB 부여

- **프로세스는 서로 독립적**이므로, 프로데드에 문제가 생겨도 **다른 프로세스들은 영향을 받지 않음**
- **쓰레드는 프로세스 내에 존재**하기 때문에, 프로세스에 문제가 생기면 **내부 쓰레드들도 같이 영향을 받음**
- 각각의 프로세스들은 고유한 자원을 가지고있다.
  - 프로세스 간 통신 IPC는 속도가 느리다
    - 공유메모리, 파이프, 소켓
    https://dar0m.tistory.com/233
- 반면 쓰레드는 한 프로세스 내에서 스택 제외하고 모두 공유하기 때문에 속도가 빠름
  - 쓰레드 간 통신은 쉽지만, 쓰레드 동기화 문제가 생길 수 있음
