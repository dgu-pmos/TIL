# Docker

## 이론

### Container

- 운영체제 수준의 가상화 기법을 사용
- 운영체제 내의 응용 프로그램을 대상으로 폐쇄되고 제한되어 있는 분리된 환경을 제공
- 프로세스 수준의 가상화이므로 하드웨어 사용량 제한 및 격리된 가시성 제공
  - 가상 하드웨어 에뮬레이션을 하는 오버헤드가 없음
- 응용프로그램을 그 안에서 독립적으로 실행
- Guest OS, 응용프로그램은 호스트 운영체제가 지원하는 동일한 커널 상에서 실행

### Container 역사

1. chroot jail
   - chroot jail이란 chroot로 특정 디렉토리를 루트 디렉토리로 설정하면 발생
   - chroot jail 안에서는 바깥 파일과 디렉토리 접근 불가
2. LXC(LinuX Container)
   - cgroup과 namespace isolation을 이용해 가상 공간 제공

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

### Docker

#### Docker 특징

- Immutable Infrastructure라는 패러다임이 생김
  - 호스트 OS와 서비스 운영 환경(서버 프로그램, 소스 코드, 컴파일된 바이너리 등)을 분리
  - 한 번 설정한 운영 환경은 변경하지 않음
  - 서비스 운영 환경을 이미지로 생성하고 서버에 배포 및 실행
  - 서비스 업데이트 시, 운영 환경을 변경하지 않고, 이미지를 새로 생성해서 배포
- Immutable Infrastructure 장점
  - 편리한 관리
    - 이미지로 관리해서 버전 관리에 유용
  - 확장
    - 이미지 하나로 서버를 계속 찍어낼 수 있음
  - 테스트
    - 개발자 PC 및 테스트 서버에서 이미지를 실행해 서비스 운영 환경과 동일한 환경에서 테스트 가능
  - 가벼움
    - 운영체제와 서비스 운영 환경을 분리해 가볍고 어디서든 실행 가능
- 이미지 버전 관리 기능 제공
- Docker 이미지를 공유할 수 있는 Docker Hub 제공

#### Image

- 베이스 이미지

  - 리눅스 배포판의 유저랜드만 설치된 파일
    - 유저랜드는 운영체제 내 유저 공간에서 실행되는 실행 파일과 라이브러리를 의미

- Docker 이미지

  - 베이스 이미지에 필요한 프로그램과 라이브러리, 소스를 설치한 뒤 파일 하나로 만든 것
  - 베이스 이미지에서 바뀐 부분만 이미지로 생성하고, 실행 시에는 베이스 이미지와 결합

- 이미지 의존관계

  ![image-20200612121231073](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200612121231073.png)

  - Docker 이미지는 16진수 Hash value로 ID를 구성
  - centos:centos7은 511136ea3c5a, 34e94e67e63a, b157b66b1a65가 조합된 것
  - Docker는 이미지를 통째로 생성하지 않고, 바뀐 부분만 생성한 뒤 부모 이미지를 계속 참조
  - 이를 Layer라고 부름

- Docker Container 는 이미지를 실행한 상태

#### 명령문

| command                                                | description                                                  |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| docker container run [IMAGE _NAME]                     | 새 컨테이너 생성 및 실행<br /><br />options<br />-i : 컨테이너의 표준 출력을 연결<br />-t : tty를 확보(터미널 오픈을 위해 주로 -it를 함께 사용)<br />--name : 컨테이너 이름 지정<br />--hostname : 컨테이너 내 호스트명 지정<br />-rm : 지정된 명령을 실행 한 뒤 컨테이너 자동 삭제<br />-d : 백그라운드에서 서비스 제공<br />-e : 환경변수 설정<br />-p : 포트포워딩<br />-v : NFS<br />-link : 타 Container 연결 |
| docker container ls -a                                 | 컨테이너 리스트 확인                                         |
| docker container stop [CONTAINER_NAME]                 | 컨테이너 중지                                                |
| docker container start [CONTAINER_NAME]                | 컨테이너 시작                                                |
| docker container rm [CONTAINER_NAME]                   | 컨테이너 삭제                                                |
| docker image ls -a                                     | 로컬 저장소에 보관중인 이미지 리스트 확인                    |
| docker search                                          | 도커 허브에 공개된 이미지 탐색                               |
| docker iamge tag [IMAGE _NAME] [NEW_IMAGE_NAME]        | 다운로드된 이미지를 복사해 별도의 태그 작성                  |
| docker image inspect [IMAGE _NAME]                     | 이미지에서 포함하고 있는 정보 확인                           |
| docker image pull [IMAGE_NAME]                         | 도커허브를 통한 이미지 다운로드                              |
| docker image rm [IMAGE _NAME]                          | 이미지를 삭제<br /><br />options<br />-f : 이미지 강제 삭제<br />-a : 사용하지 않는 모든 이미지 삭제 |
| docker exec [CONTAINER_NAME] [COMMAND] [PRAMMETERS]    | 호스트에서 컨테이너 안의 명령 실행                           |
| docker container commit [CONTAINER_NAME] [IMAGE _NAME] | 컨테이너로부터 이미지 작성<br /><br />options<br />-a : author 지정<br />-m : message 작성<br />-c : commit 시 Dockerfile 명령 지정<br />-p : 컨테이너를 일시정지하고 commit 진행 |

### Dockerfile

- text 형식의 파일이며 Editor를 이용해 원하는 내용을 기술
- 베이스 이미지를 지정한 뒤, 필요한 미들웨어 및 명령어를 추가해 원하는 형태의 이미지 생성 
- Dockerfile은 Docker의 build 명령을 통해 Image로 작성

| command    | description                        | command     | description                                                  |
| ---------- | ---------------------------------- | ----------- | ------------------------------------------------------------ |
| FROM       | 베이스 이미지 지정                 | VOLUME      | 이미지에 볼륨을 할당하고자 할 때 사용 <br />호스트나 다른 컨테이너에서 마운트를 수행 |
| RUN        | 명령 실행                          | USER        | 사용자 지정                                                  |
| CMD        | 컨테이너 명령 실행                 | WORKDIR     | 작업 디렉토리 지정                                           |
| LABEL      | 라벨 설정                          | ARG         | Dockerfile 안의 변수                                         |
| EXPOSE     | 외부에서 접근 가능하도록 포트 공개 | ONBUILD     | 빌드 완료 후 명령 실행                                       |
| ENV        | 환경변수 제어                      | STOPSIGNAL  | 시스템 콜 시그널 설정                                        |
| ADD        | 파일/디렉토리 추가                 | HEALTHCHECK | 컨테이너 내 프로세스가 정상적으로 동작하는지 체크            |
| COPY       | 파일 복사                          | SHELL       | Shell 지정                                                   |
| ENTRYPOINT | 명령 실행                          |             |                                                              |

#### RUN

- 지정한 베이스 이미지에 대해 애플리케이션 및 미들웨어 등을 설치하거나 환경 구축을 위한 명령

#### CMD & ENTRYPOINT

- CMD와 ENTRYPOINT는 공통적으로 컨테이너 안에서 명령을 실행
- CMD
  - CMD는 Dockerfile 내에서 한 번만 기술
  - 여러번 기술 되었다면 마지막 CMD만 유효
  - docker container run 명령어에서 CMD와 중복되는게 있다면 run 명령어의 새로운 명령을 우선 실행
- ENTRYPOINT
  - docker container run 명령을 실행했을 때 ENTRYPOINT가 실행
  - run 명령어와 중복된다면, 무조건 ENTRYPOINT를 우선 실행

#### ONBUILD

- ONBUILD ADD index.html /var/www/html/index.html
  - Build를 진행한 후, 빌드를 진행한 경로에 index.html이 존재 해야함
  - 현재 경로에 존재하는 index.html이 Container의 /var/www/html 디렉토리에 추가

#### ADD

- 호스트 상의 파일이나 디렉토리를 추가할 때 사용
- ADD [HOST_PATH] [DOCKER_PATH]
- 만약 호스트 파일이 압축포맷이라면 자동으로 풀어서 옮겨줌
- URL로부터 다운로드 한 압축파일은 풀어주지 않음

#### VOLUME

- 이미지에 볼륨을 마운트할 때 사용
- 호스트나 다른 컨테이너에서 마운트를 수행

### Private registry (Docker Registry)

- 프로젝트를 위해 작성한 이미지를 인터넷 상에 공개하지 않고 팀 내에서 사용하는 용도
- 개발자 각자가 자신의 로컬 저장소에서 관리하던 이미지를 한 곳에 일원화하여 효율적인 개발이 가능
- Docker store에 공개된 'registry'를 사용해 구축

### Docker Compose

- 한장의 파일로 원하는 서비스 기술
- docker-compose.yml 파일에 환경 정보를 모아서 설정
- yaml 형식
  - 구조화된 데이터를 표현하기 위한 데이터 포맷
  - Python과 같이 들여쓰기로 데이터의 계층 구조를 나타냄
  - 들여쓰기는 스페이스 사용
  - 데이터 맨 앞에 '-'를 붙이면 배열을 의미

```yaml
# docker-compose.yml
version: "1.0"

services:
  webengine:
    image: rapa/newnginx100:1.0
    ports:
      - "80:80"
```

#### options

| option       | description                            |
| ------------ | -------------------------------------- |
| build        | Dockerfile을 이용해 이미지 빌드를 진행 |
| expose       | 포트 공개                              |
| volumes      | 컨테이너에 볼륨을 마운트               |
| volumes_from | 다른 컨테이너로부터 모든 볼륨을 마운트 |
| command      | 컨테이너 안에서 작동하는 명령 지정     |
| links        | 다른 컨테이너와 연결                   |
| depends_on   | 서비스 의존관계 정의                   |
| environment  | 컨테이너 환경변수 지정                 |

##### depends_on

- 두 개 이상의 컨테이너가 link 되어있을 경우, 재시작 시에 어떤 컨테이너를 먼저 준비해야 할지 의존 관계를 정의

#### compose 기본 명령

| command | description                                                  |
| ------- | ------------------------------------------------------------ |
| up      | 컨테이너 생성 후 시작<br />options<br />-d : 백그라운드 실행<br />-scale SERVICE=서비스 수 : 서비스 수 지정 |
| ps      | 컨테이너 목록 표시                                           |
| logs    | 컨테이너 로그 출력                                           |
| restart | 컨테이너 재시작                                              |
| pause   | 컨테이너 일시정지                                            |
| unpause | 컨테이너 재개                                                |
| port    | 공개 포트 번호 표시                                          |
| config  | 구성 확인                                                    |
| kill    | 컨테이너 강제 중지                                           |
| rm      | 컨테이너 삭제                                                |
| down    | 리소스 삭제                                                  |

### 컨테이너 생성 방법

1. CLI
   - 동일한 명령어에서 포트번호, 이름 등을 변경해 반복 수동 작업
   - 이미지는 제공된 이미지만 사용
2. Dockerfile
   - 베이스 이미지를 기반으로 자신이 원하는 형태의 서비스를 위한 수정된 이미지 사용
3. Compose
   - 이미지를 이용해 환경 구성을 한 번에 수정
   - 두 개 이상의 컨테이너를 연결해서 서비스를 생성할 경우 유용
   - 동일 서비스에서 여러 개 만들어야 한다면, 포트 설정에 유의
     - 이를 해결하기 위해 도커가 설치된 호스트를 클러스터로 묶어 컨테이너들을 오케스트레이션 할 수 있음
     - 대표적으로 Docker Swarm, Kubernetes

## 실습

### 도커 실습 환경 구축

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

### Docker, Mysql, Wordpress 를 이용한 웹 사이트 구축

```
// MYSQL Container 생성 및 실행
// EXPOSE를 하지 않은 이유는 기본적으로 3306이 열려있기 때문
docker container run -d --name mysql -v mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=wordpress -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=wordpress mysql:5.7
// Wordpress Container 생성 및 실행
// 환경 변수를 이용해 MYSQL Container와 연결
docker container run -d --name wordpress -v wordpress:/var/www/html --link mysql:mysql -e WORDPRESS_DB_HOST=mysql:3306 -e WORDPRESS_DB_PASSWORD=wordpress -p 8080:80 wordpress:latest
```

![image-20200611181543501](https://i.ibb.co/X2H1nJ9/image-20200611181543501.png)

![image-20200611181736210](https://i.ibb.co/z733fqr/image-20200611181736210.png)

### Dockerfile을 이용한 Nginx Container 생성

1. docker 디렉토리 생성 후, index.html과 a.jpg, b.jpg를 담은 image 디렉토리 생성

   ```html
   // 홈 디렉토리 이동 후, docker 디렉토리 생성 성공 시 이동
   cd ; mkdir docker ; cd docker
   mkdir image
   vim index.html
   '''
   <!DOCTYPE html>
           <html>
                   <head>
                           <title>test page</title>
                   </head>
           <body>
                   <img src="image/a.jpg"><br>
                   <img src="image/b.jpg"><br>
           </body>
       </html>
   '''
   // a.jpg, b.jpg는 구글링해서 다운받기
   ```

2. Dockerfile 작성

   ```
   FROM nginx:latest
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   WORKDIR /usr/share/nginx/html
   ADD index.html /usr/share/nginx/html
   ADD image.tar /usr/share/nginx/html
   ```

3. Dockerfile을 이용해 Image 생성

   ```
   sudo docker build -t rapa/newnginx100:1.0 .
   ```

4. Image를 이용해 Container 생성

   ```
   sudo docker container run -d -p 8089:80 --name rapanginx100 rapa/newnginx100:1.0
   ```

5. 결과

   ![image-20200611183955433](https://i.ibb.co/yXWsTFz/image-20200611183955433.png)

![image-20200611184015966](https://i.ibb.co/Jd1rkgc/image-20200611184015966.png)

![image-20200611184115572](https://i.ibb.co/rdS36TW/image-20200611184115572.png)

### Private Registry 생성 및 사용

1. 레지스트리 이미지 설치

   ```
   docker image pull registry
   ```

2. 사설 레지스트리 컨테이너 생성

   ```
   docker container run -d -p 5000:5000 --name preregistry registry
   ```

3. 업로드할 이미지 생성

   ```
   vim Dockerfile
   '''
   FROM centos
   
   RUN ["echo", "HELLO"]
   '''
   docker build -t hello .
   ```

4. 사설 레지스트리에 업로드하기 위해 Image 태깅

   ```
   docker image tag hello localhost:5000/helloimg
   docker image push localhost:5000/helloimg
   ```

5. 이미지 업로드 -> 로컬 저장소에 위치한 이미지 삭제 -> 업로드된 이미지 다운로드 

   ```
   docker image rm hello
   docker image rm localhost:5000/helloimg
   docker image pull localhost:5000/helloimg
   ```

### Docker Compose를 활용한 Wordpress 구축

1. compose를 이용해 시스템 환경 정의(wordpress, db)

   ```
   vim docker-compose.yml 
   '''
   version: "3"
   services:
     web:
       image: wordpress
       ports:
         - "8888:80"
       depends_on:
         - "db"
       links:
         - "db:mariadb"
         
     db:
       image: mariadb
       volumes:
         - ./mysql:/var/lib/mysql
       environment:
         - MYSQL_ROOT_PASSWORD=wordpress
         - MYSQL_DATABASE=wordpress
         - MYSQL_USER=wordpress
         - MYSQL_PASSWORD=wordpress
   '''
   ```

2. compose 실행

   ```
   docker-compose up -d
   ```

3. 결과

   ![image-20200612163302075](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200612163302075.png)

   ![image-20200612163234660](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200612163234660.png)

### 개발 팀장이 개발자들에게 이미지 배포

1. 디렉토리 생성 및 개발 팀장이 이미지 배포

   ```
   cd /home/user1/docker
   mkdir 0615 ; cd 0615 ; mkdir manager ; mkdir member
   cd manager
   vim Dockerfile
   '''
   FROM centos
   
   RUN yum -y install httpd
   EXPOSE 80
   ONBUILD ADD web.tar /var/www/html/
   
   CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
   '''
   docker build -t web:1.0 .
   ```

2. 개발자는 ONBUILD에 필요한 파일 준비 및 이미지 빌드

   ```
   cd ../member
   tar cf web.tar web/*
   vim Dockerfile
   '''
   FROM web:1.0
   '''
   docker build -t member .
   ```

3. 컨테이너 생성

   ```
   docker container run -d -p 8001:80 member
   ```

4. 결과

   ![image-20200615100149179](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200615100149179.png)

### Xpressengine, MariaDB, Docker를 이용한 서비스 배포

1. xe-core/index.php로 redirect 할 index.html 생성

   ```html
   mkdir bbs ; cd bbs
   vim index.html
   '''
   <html>
   	<head>
   		<title>TEST PAGE</title>
   	</head>
   	<meta http-equiv="refresh" content="0; url=xe-core/index.php"/>
   </html>
   '''
   ```

2. xe 서버용 디스크 생성

   ```
   vim Dockerfile
   '''
   FROM ubuntu:18.04
   ENV DEBIAN_FRONTEND=noninteractive
   RUN apt-get update
   RUN apt-get -y install apache2
   RUN apt-get install -y git software-properties-common
   RUN add-apt-repository ppa:ondrej/php#
   RUN apt-get update
   RUN apt-get -y install php5.6 php5.6-common php5.6-mysql php5.6-gd php5.6-fpm php5.6-xml libapache2-mod-php5.6
   WORKDIR /var/www/html
   RUN git clone https://github.com/xpressengine/xe-core.git
   ADD index.html /var/www/html
   RUN chmod 777 -R /var/www/html/xe-core
   EXPOSE 80
   CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
   '''
   docker build -t rapa/xe:1.0
   ```

3. xe 서버, db 서버를 구축할 compose file 생성 및 실행

   ```yml
   vim docker-compose.yml
   '''
   version: "3"
   
   services:
     xe3:
       image: rapa/xe:1.0
       ports:
         - "8001:80"
       links:
         - "db3"
       depends_on:
         - "db3"
         
   
     db3:
       image: mariadb
       volumes:
         - ./mariadb:/var/lib/mysql
       environment:
         - MYSQL_ROOT_PASSWORD=xe
         - MYSQL_DATEBASE=xe
         - MYSQL_USER=xe
         - MYSQL_PASSWORD=xe
   '''
   docker-compose up -d
   ```

4. 결과

   ![image-20200615140952545](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200615140952545.png)

#### Docker Swamp, HAProxy, Nginx를 활용한 웹서비스 로드밸런싱





