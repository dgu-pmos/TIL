# Container

## 이론

### Container

- 운영체제 수준의 가상화 기법을 사용
- 운영체제 내의 응용 프로그램을 대상으로 폐쇄되고 제한되어 있는 분리된 환경을 제공
- 프로세스 수준의 가상화이므로 하드웨어 사용량 제한 및 격리된 가시성 제공
  - 가상 하드웨어 에뮬레이션을 하는 오버헤드가 없음
- 응용프로그램을 그 안에서 독립적으로 실행
- Guest OS, 응용프로그램은 호스트 운영체제가 지원하는 동일한 커널 상에서 실행

### Container의 이해

#### Namespace Isolation

- 프로세스 상에서 사용하는 특정 자원에 대한 가시성을 제한
- 별도의 네임스페이스를 사용하는 프로세스 간에는 서로의 자원을 볼 수 없음

| 네임스페이스 유형           | 목적                                                         |
| --------------------------- | ------------------------------------------------------------ |
| User Namespace              | 특정 프로세스를 위해 별도의 사용자 및 그룹 ID를 생성         |
| UNIX Time-sharing Namespace | 응용프로그램 입장에서 호스트명과 도메인에 대한 분리된 가시성 제공 |
| IPC Namespace               | Pipe, Message Queue 와 같은 IPC는 프로세스가 상호 통신하는데 사용됨. 프로세스는 독립된 IPC Namespace를 이용해 IPC를 위한 자체적인 Namespce 생성 |
| Mount Namespace             | 프로세스에서 마운트된 파일시스템에 대한 가시성을 위해 자신만의 Mount Namespace를 사용함. 이 Namespace 내에서 마운트/언마운트 하는 것은 동일한 호스트에서 구동하는 다른 응용프로그램에서는 보이지 않음. |
| PID Namespace               | PID Namespace를 이용해 호스트상의 다른 응용프로그램으로부터 격리할 수 있음. |
| Network Namespace           | 네트워크 인터페이스, 라우팅 테이블 등에 대한 분리된 가시성 제공 |

#### Control-group

- 네임스페이스는 시스템 자원에 대해 프로세스마다 다른 가시성을 제공
- 그러나, 자원들을 사용하는 것은 막을 수 없음
- Cgroup은 자원을 제어하며 우선순위를 조정

### OS Container

![](https://blog.risingstack.com/content/images/2019/08/os-containers.jpg)

- Container 내에서 게스트 운영체제를 실행하는 이유는 다음과 같음
  - 특정 응용프로그램에서 특수한 라이브러리, 시스템 바이너리를 필요로 하는 경우
  - 게스트 운영체제 위에 있는 응용프로그램은 게스트 운영체제의 라이브러리와 시스템 바이너리 사용
  - 이 경우에 호스트 운영체제와의 바이너리에 대한 의존성이 사라짐
  - 하지만 커널은 여전히 공유
- Docker는 동일한 버전이나 다른 배포판을 동작시킬 수 있음
- 하이퍼바이저가 아닌 컨테이너 API 사용

### Application Container

- 일반적인 컨테이너
- 라이브러리, 시스템 바이너리, 커널이 호스트와 동일
- 네트워킹과 디스크 마운드 지점에 대한 자체적인 네임스페이스 갖음
- 한 번에 하나의 서비스 실행하도록 설계
- 단위 서비스를 제공하는 응용프로그램 컨테이너들로 그룹을 만들 수 있음
  - Node.js + Nginx + Postgres
  - 이를 마이크로 서비스라고 부름

<추가 예정>



## 실습

1. Ubuntu Desktop 설치 및 기본 셋팅

   ```
   // root 비밀번호 설정
   sudo passwd root
   // root 접속 확인
   su root
   ```

   VMware workstation ->  VM -> install VMware Tools.. 클릭

   Ubuntu -> Setting -> Power -> blank screen.. Never 선택

   ```
   cp /media/user1/VMware\ Tools/VMwareTools-10.3.10-13959562.tar.gz .// vmware tool 마운트 확인
   df
   // root 홈 디렉토리 이동
   cd
   // vmware tools 설치 파일 복사
   cp /media/user1/VMware\ Tools/VMwareTools-10.3.10-13959562.tar.gz .
   // 압축 해제
   tar xfz VMwareTools-10.3.10-13959562.tar.gz
   // 설치
   cd vmware-tools-distrib 
   ./vmware-install.pl
   // firewall 중지 및 부팅 시 disable
   ufw disable
   // 일반 계정인 'user1'을 임시로 root 권한을 갖도록 변경
   vi /etc/sudoers
   '''
   #21	user1 ALL=(ALL:ALL) ALL
   '''
   usermod -aG sudo user1
   groups user1
   // 그래픽 문제 해결
   sudo vi /etc/default/grub
   '''
   ~~~
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"
   ~~~
   '''
   sudo update-grub
   // 업데이트 및 필수 패키지 설치
   sudo apt-get update && sudo apt-get -y install \
   apt-transport-https ca-certificates curl software-properties-common net-tools openssh-server vim
   ```

2. Ubuntu 일지정지 -> Snapshot 등록 -> 컴퓨터 재부팅

3. Docket 설치

   ```
   // Docker 공식 GPG Key 추가
   curl -fssL https://download.docker.com/linux/ubuntu/gpg | sudo apt-get add -
   // Docker 저장소 경로 추가 후 업데이트
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && sudo apt-get update
   // Docker 설치
   sudo apt-get -y install docker-ce
   // Docker 버전 확인
   docker version
   ```

4. Docker 실행 및 제어 명령어

   ```
   // 로컬 저장소에 보관 중인 이미지 목록 확인
   docker image ls
   // 새 컨테이너 실행
   docker container run
   // 컨테이너 리스트 확인
   docker container ls
   // Docker hub에 공개된 이미지 탐색
   docker search
   // Docker hub로부터 이미지 다운로드
   docker image pull
   ```