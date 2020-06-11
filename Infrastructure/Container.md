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

### 명령문 모음

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
| docker container commit [CONTAINER_NAME] [IMAGE _NAME] | 컨테이너로부터 이미지 작성<br /><br />options<br />-a : author 지정<br />-m : message 작성<br />-c : commit 시 Dockerfile 명령 지정<br />-p : 컨테이너를 일시정지하고 commit 진행 |



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



