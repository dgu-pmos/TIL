# Hypervisor

## Hypervisor

### What is Hypervisor?

- 호스트 컴퓨터에서 다수의 운영체제를 동시에 실행하기 위한 논리적 플랫폼
- 호스트 OS는 가상머신을 Application으로 처리(Img 파일로 관리)
- 가상머신을 모니터링하기 때문에  VMM(Virtual Machine Monitor)라고 불림



### 종류

#### Type 1

- 물리 자원 위 OS 위치에 직접 설치되는 방식으로 커널에 위치
- 물리 자원을 직접 접근 및 제어
- 성능이 우수하며 Bare Metal Hypervisor라고 불림
- KVM, VMWare ESXi, XenServer ..

#### Type 2

- Host OS 위에 소프트웨어적으로 설치되는 방식
- Guest OS의 요청을 Hypervisor가 모니터링하고 커널에 요청
- 상대적으로 성능이 떨어짐
- VMware Workstation, Virtual Box ..



#### Full virtualization

##### Software Assisted Full Virtualization

![](https://www.unixarena.com/wp-content/uploads/2017/12/Binaray-Transalation-Full-Virtualization.jpg)

- VM이 Guest OS에게 Instruction을 전달
- Guest OS는 Hypervisor에게 Binary Translation 요청
- Hyervisor가 변환 후 하드웨어에게 전달
- Binary Translation 때문에 성능 저하 발생
- VMware workstation, VirtualBox ..

##### Hardware Assisted Full Virtualization

![](https://www.unixarena.com/wp-content/uploads/2017/12/Hardware-assisted-virtualization-Hypervisor2.jpg)

- Binary Translation을 제거함
- CPU의 가상화 기능(Intel VT-x, AMD-V) 사용해 직접적으로 하드웨어에 인터럽트 요청
- VM의 Instruction에서도 Processor에게 직접 권한을 요하는 명령 실행 가능
- KVM, VMware ESXi, Xen ..



#### Paravirtualization

- 하드웨어 가상화가 필요 없음
- 하이퍼바이저는 물리적 서버 위에서 설치됨
- Guest는 자신이 가상화된 사실은 인지
  - 드라이버를 사용해 Host(Hypervisor)와 직접 통신

- Guest는 Hypervisor에게 API call을 하기 위해 확장이 필요(Hyper Call)
  - 이를 위해서 Guest OS는 조금 수정되어야 함(그래서 윈도우는 불가)
- Xen, IBM LPAR ..

##### Xen의 Paravirtualization 방법

![](https://wiki.xen.org/images/thumb/7/73/XenPV.png/750px-XenPV.png)

- DOM 0는 부팅 시 생성되는 초기 VM
  - 본질적으로 Host OS
  - 하드웨어에 직접 접근할 수 있는 권한 갖고있음
- Guest OS와 DOM0은 PV front-end, back-end 드라이버로 통신
- 요청을 받으면 DOM0가 Driver를 통해 하드웨어에 접근



## Cloud

### Cloud의 6가지 특징

1. 기민한 탄력성
   - 탄력성이란 필요에 따라 자원의 확장과 감소를 처리할 수 있는 능력
   - 이런 작업을 수분 ~ 수십분 이내로 가능
2. 측정 가능한 서비스
   - 자원의 사용량을 실시간으로 수집하고 모니터링
   - 정보를 활용해 과금 처리 및 자원 추가 요청 등의 작업 수행
3. on-demand self service
   - 중간 개입자 없이 원하는 시점에 서비스를 바로 사용
4. 유비쿼터스 네트워크 접근
   - 클라우드 서비스 제공자는 네트워크 기반으로 서비스에 접속할 수 있도록 함
5. 자원 풀
   - 물리적 및 가상화 자원은 풀로 관리
   - 사용자는 자원의 물리적인 위치, 크기 등은 모르고 자원을 추상화시켜 제공
6. 지역 간 무중단 이동
   - 서로 다른 가상 머신 사이에 App의 중단 없이 시스템 이동이 가능



### 그리드 컴퓨팅, 유틸리티 컴퓨팅

#### Grid Computing

- 인터넷에 분산된 다양한 시스템과 잉여 자원 공유
- 분산 컴퓨팅 구조
- 클러스터 컴퓨팅의 확장 개념
- 클라우드 서비스 제공 방식



#### Utility Computing

- 가스, 전기 등과 같이 필요할 때마다 사용하는 방식
- 클라우드의 과금 모형





### The NIST Cloud Definition Framework

![](https://www.hostway.co.kr/sites/default/files/styles/large/public/20110131_1.jpg?itok=NHG0vWdp)

#### Software as a Service

- Cloud 환경에서 동작하는 응용 프로그램을 서비스 형태로 제공
- Google Cloud



#### Platform as a Service

- Cloud 환경에서 서비스를 개발할 수 있는 안정적인 환경을 제공
- Docker, Google App Engine ..



#### Infrastructure as a Service

- Cloud 환경에서 서버를 운영하기 위한 기본적인 인프라를 제공
- GCP, AWS EC2 ..



###### 출처

###### - https://www.unixarena.com/2017/12/para-virtualization-full-virtualization-hardware-assisted-virtualization.html/