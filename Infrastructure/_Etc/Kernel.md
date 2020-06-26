# Kernel (System Software 공부 시 추가 예정)

## Computer System

### Operating System

- 사용자와 Computer Hardware 사이에서 자원을 관리하고 Program들을 지원
  - Program 실행
  - User problem 쉽게 해결
  - User에게 편리성 제공
  - Hardware Resource 효율적 관리

### Program

- main 함수를 포함한 다른 다양한 함수들이 모인 존재

### Kernel

- 본질적으로는 프로그램이지만 Memory Resident(메모리에 상주) 한다는 점이 특수함
- Kernel을 제외한 프로그램들은 Utility(Command)라고 함

### Shell

- 많은 Program들의 메모리를 정리해주는 역할(Job Control)
- Keyboard 입력을 읽어서 Command를 실행(Interpreter)

### File

- sequence of bytes
- Linux에는 I/O Device도 File로 취급

## Kernel,  Shell, Utility의 관계

![](https://miro.medium.com/max/1400/1*Kp6TSWr2zjL2Qgp6TO1xew.png)

- 시스템을 부팅하면 Main Memory에 Kernel이 올라감

![](https://miro.medium.com/max/1400/1*zeV2GLulxXQ9bEY492FCOw.png)

- User가 Terminal에 전원을 키면 Shell이 Main Memory에 올라감
- User가 PPT를 실행하면 Shell은 Child Process 생성



## Linux Kernel 파고들기

### 핵심 요소

- Resource Management
- Process Management
- Device Management
- I/O Device Management
- Process cooperation
- Memory Control
- System Calls
- Resource Allocation

### 커널 종류

![](https://graphviz.gitlab.io/_pages/Gallery/directed/Linux_kernel_diagram.png)

### Library

- 함께 사용하는 모듈의 집합
- 프로그램이나 다른 라이브러리에서 사용될 수 있음

### Module

- 공통 네임스페이스로 묶인 함수의 집합

### Package

- 실행 파일 또는 라이브러리를 포함하는 배포 단위
- 코드를 공유하는 방법

### OS Binary

<??>

###### 출처

###### - [리눅스 커널(운영체제) 강의노트 [1]](https://medium.com/pocs/리눅스-커널-운영체제-강의노트-1-d36d6c961566)

