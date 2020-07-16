# Openstack

## 이론

### 주요 컴포넌트

<img src="https://i.ibb.co/xSWhhJY/image-20200709141641495.png" style="zoom:70%;" />

#### Nova

- 하이퍼바이저, 메세지 큐, 인스턴스 접속 등을 지원해 가상 서버를 생성할 수 있는 시스템 구성

- 실행 순서

  1. nova-api(CLI or Dashboard)로부터 요청을 받음
  2. 메세지 큐를 이용해 nova-compute에 인스턴스 생성 명령 전달
  3. nova-compute는 하이퍼바이저에게 명령 전달
  4. 하이퍼바이저가 인스턴스 생성
  5. 생성된 인스턴스는 nova-api로 접근 가능하며, nova의 모든 기능은 메세지 큐로 처리

- 기본 하이퍼바이저는 KVM/QEMU

#### Swift

- 오브젝트 스토리지
- swift-proxy-server가 swift-(object, account, container) server 관리
- account는 사용자 계정 및 사용자의 컨테이너를 메타데이터로 관리
- container는 컨테이너 안의 오브젝트 정보를 메타데이터로 관리
- object에 실질적인 데이터 저장
- REST API로 통신
- 여러 스토리지를 하나로 묶어 가상화하고, 그 안에 사용자만의 별도 스토리지 공간이 있는 것처럼 또다시 가상화

#### Glance

- 가상머신 이미지 및 운영체제 관리

#### Keystone

- 서비스를 사용하기 위해 통과해야 하는 인증 서비스
- LDAP, SQL 서버와 같은 외부 사용자 관리 시스템과 연계해서 사용
- 구성요소
  - 사용자 : 사람 또는 서비스를 의미
  - 인증 : 사용자의 신분을 확인하는 절차, 이름과 비밀번호 키스톤에게 제출
  - 토큰 : 신분을 증명하는 임의의 텍스트 데이터
  - 프로젝트 : 자원에 대한 권리를 가진 보안 그룹(버전 2 까지는 테넌트)
  - 엔드포인트 : 서비스를 이용하기 위해 제공하는 네트워크 주소

#### Cinder

- 인스턴스에게 블록 스토리지 제공
- LVM의 LV 단위로 iSCSI 통신을 이용해 데이터 통신

#### Neutron

- 네트워크 제공

### multiple node configuration

![image-20200709154534352](https://i.ibb.co/vQpCyPZ/image-20200709154534352.png)

#### controller node

- 오픈스택 환경의 control plane
- nova-api, nova-scheduler, keystone, cinder-api, ...

#### compute node

- 하이퍼바이저를 동작하는 노드
- nova-compute, neutron-openvswitch-agent

#### network node

- 테넌트, 라우팅, 다른 노드 간의 모든 네트워크 처리
- neutron-server, neutron-openvswitch-agent, neutron-l3-agent,neutron-dhcp-agent, ...

### 네트워크

#### nova-network

  <img src="https://i.ibb.co/NFH7t0x/image-20200709152401352.png" alt="image-20200709152401352" style="zoom:70%;" />

  - nova-network는 nova 프로젝트 내에서 네트워크를 담당하는 프로세스
  - openstack 초기에는 nova 프로젝트에서 직접 인스턴스와 관련된 네트워크 관리

##### flat network

  <img src="https://i.ibb.co/xYVzJHC/image-20200709152605335.png" alt="image-20200709152605335" style="zoom:70%;" />

  - 가장 기본적인 네트워크 구성모델
  - IP pool에서 fixed IP를 할당받음
  - 문제점
    - 수동적인 할당 : 사용자가 직접 nova api를 통해 ip pool을 생성한 후에 fixed ip 할당이 가능
    - bridge 연결 문제 : 모든 인스턴스가 하나의 bridge에 연결해서 네트워크 오류에 취약

##### vlan network

  <img src="https://i.ibb.co/6FW6B4T/image-20200709153119146.png" alt="image-20200709153119146" style="zoom:70%;" />

  - 하나의 물리적 NIC에 여러 개의 그룹별로 단독 VLAN 망을 구성
  - 각 VLAN은 dnsmasq와 하나의 bridge를 구성

#### Neutron

<img src="https://i.ibb.co/WnhDrvg/image-20200709153400118.png" alt="image-20200709153400118" style="zoom:70%;" />

- 네트워크의 생성/변경/삭제에 대한 API만 제공할 뿐 실제로는 plugin을 통해서 네트워크 구성
- IP 할당 동작 원리
  1. 사용자는 neutron API를 이용해 neutron-server로 IP 할당 요청
  2. neutron-server는 queue로 전달
  3. queue는 neutron-dhcp-agent, neutron 3rd party plugin으로 IP 할당 지시
  4. 작업 내용 수시로 neutron database에 저장
- openvswitch, linuxbridge, ML2 지원

##### Provider network

- 2 계층 서비스로 VLAN 세그멘테이션이 가능한 단순한 네트워크 서비스
- 가상 네트워크를 물리 네트워크에 연결

##### ML2

- flat : 모든 인스턴스는 동일한 네트워크에 위치하며 분리될 수 없음
- vlan : 같은 스위치에서 트래픽을 분할하기 위해 사용
- vxlan : L3 기반에서 사용되는 L2 overlay 기술

###### answer.txt를 통해 외부 네트워크로의 연결은 br-ex 브리지를 사용하도록 매핑되어있음

###### 기본값으로 메커니즘 드라이버는 openvswitch 사용

<img src="https://i.ibb.co/XtH5QWb/image-20200709164245329.png" alt="image-20200709164245329" style="zoom: 80%;" />

#### Fixed IP, Floating IP

##### Fixed IP

- 클라우드 플랫폼 내에서 생성한 가상머신에 할당되는 내부 IP
- 외부 통신은 불가능

##### Floating IP

- 외부망에 접근하기 위해 배정받는 IP

## 실습

### 오픈스택 설치

#### 가상머신 사양

![image-20200709105726316](https://i.ibb.co/D7FCvcJ/image-20200709105726316.png)

#### 기본 보안 및 네트워크 설정

```shell
# 방화벽 해제
systemctl stop firewalld
systemctl disable firewalld
# NetworkManager : 네트워크의 변경 사항을 탐지하고 설정해주는 역할
# 네트워크매니저 해제
systemctl stop NetworkManager
systemctl disable NetworkManager
# SELINUX 해제
setenforce 0
vi /etc/selinux/config
'''
selinux=disabled
# vi 대신 vim 사용
'''
yum -y install vim
vi ~/.bashrc
'''
alias vi='vim'
'''
# 네트워크 인터페이스 변경
cd /etc/sysconfig/network-scripts/
cp ifcfg-ens32 ifcfg-ens32.bak
mv ifcfg-ens32 ifcfg-eth0
vi ifcfg-eth0
'''
TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=211.183.3.100
PREFIX=24
GATEWAY=211.183.3.2
DNS1=8.8.8.8
NM_CONTROLLED=no
'''
vi /etc/default/grub
'''
~~ quiet net.ifnames=0 biosdevname=0"
'''
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

#### 오픈스택 설치

```shell
yum -y update
# openstack rocky 배포판의 레포지토리 설치
yum -y install centos-release-openstack-rocky
# packstack : openstack 자동화 설치 툴
yum -y install openstack-packstack
yum -y update
# answer file 생성 및 수정
packstack --gen-answer-file=answer.txt
'''
64	CONFIG_MAGNUM_INSTALL=y
326	CONFIG_KEYSTONE_ADMIN_PW=test123
329	CONFIG_KEYSTONE_DEMO_PW=test123
782	CONFIG_LBAAS_INSTALL=y
'''
# answer file 적용해서 openstack 설치
packstack --answer-file=answer.txt
```

### 기본 조작

#### flavor 생성

<img src="https://i.ibb.co/NNynpjf/image-20200709095012491.png" alt="image-20200709095012491" style="zoom:70%;" />

#### 네트워크 생성

##### 네트워크 및 서브넷 생성

<img src="https://i.ibb.co/F6tDdLJ/image-20200709102120064.png" alt="image-20200709102120064" style="zoom:70%;" />

<img src="https://i.ibb.co/JkshkpD/image-20200709102131042.png" alt="image-20200709102131042" style="zoom:70%;" />

<img src="https://i.ibb.co/H2hS4qL/image-20200709102159101.png" alt="image-20200709102159101" style="zoom:70%;" />

##### 라우터 생성

<img src="https://i.ibb.co/MC0KWCN/image-20200709102306658.png" alt="image-20200709102306658" style="zoom:70%;" />

<img src="https://i.ibb.co/hsyJDGr/image-20200709102348369.png" alt="image-20200709102348369" style="zoom:70%;" />

##### 결과

<img src="https://i.ibb.co/ygVvgD3/image-20200709102405746.png" alt="image-20200709102405746" style="zoom:70%;" />

#### 이미지 생성

<img src="https://i.ibb.co/r4GNSZQ/image-20200709102706471.png" alt="image-20200709102706471" style="zoom:70%;" />



#### 프로젝트 생성

<img src="https://i.ibb.co/kJR0WRB/image-20200709103640508.png" alt="image-20200709103640508" style="zoom:70%;" />

##### 할당량 편집

<img src="https://i.ibb.co/VM8xwJm/image-20200709103451455.png" alt="image-20200709103451455" style="zoom:70%;" />

##### 사용자 생성

<img src="https://i.ibb.co/g7jT8ft/image-20200709103611255.png" alt="image-20200709103611255" style="zoom:70%;" />

#### 인스턴스 생성

<img src="https://i.ibb.co/DQrJ2nz/image-20200709105537054.png" alt="image-20200709105537054" style="zoom:70%;" />

##### 결과

<img src="https://i.ibb.co/DDRFgMs/image-20200709105642490.png" alt="image-20200709105642490" style="zoom:70%;" />

#### floating IP 지정

```shell
openstack floating ip create --project admin --subnet ext_sub ext_net
openstack server add floating ip centos_www_test1 211.183.3.X
openstack server list
```

![image-20200709165835422](https://i.ibb.co/ngSwQTh/image-20200709165835422.png)

### 로드밸런서 구성

1. demo 계정 로그인

   ```shell
   source keystonerc_demo
   ```

2. 이미지 생성

   ```shell
   wget http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-20150628_01.qcow2.xz
   xz -d CentOS-7-x86_64-GenericCloud-20150628_01.qcow2.xz
   openstack image create "CentOS7" --file CentOS-7-x86_64-GenericCloud-20150628_01.qcow2 --disk-format qcow2
   openstack image list
   ```

   <img src="https://i.ibb.co/M8rkd0p/image-20200709115024206.png" alt="image-20200709115024206" style="zoom:80%;" />

3. 네트워크, 서브넷, 라우터 생성

   ```shell
   openstack network create demo_private01
   openstack subnet create demo_private01_subnet --subnet-range 172.16.10.0/24 --network demo_private01 --dns-nameserver 8.8.8.8 --dhcp
   openstack router create DEMO_R1
   openstack router add subnet DEMO_R1 demo_private01_subnet
   ```

   <img src="https://i.ibb.co/W0DB0X2/image-20200709115124510.png" alt="image-20200709115124510" style="zoom: 80%;" />

4. 보안 그룹 생성

   <img src="https://i.ibb.co/cvwnkR9/image-20200709115216945.png" alt="image-20200709115216945" style="zoom:80%;" />

   <img src="https://i.ibb.co/V26SfNb/image-20200709115249506.png" alt="image-20200709115249506" style="zoom:80%;" />

   <img src="https://i.ibb.co/cQgCkC0/image-20200709115323528.png" alt="image-20200709115323528" style="zoom:80%;" />

   <img src="https://i.ibb.co/FwtXpq1/image-20200709115344657.png" alt="image-20200709115344657" style="zoom:80%;" />

5. 키 페어 생성

   <img src="https://i.ibb.co/X2tdxCw/image-20200709115418597.png" alt="image-20200709115418597" style="zoom:80%;" />

   <img src="https://i.ibb.co/Gt11PMg/image-20200709115428933.png" alt="image-20200709115428933" style="zoom: 80%;" />

   <img src="https://i.ibb.co/RcVzKbw/image-20200709115731476.png" alt="image-20200709115731476" style="zoom:70%;" />

6. 인스턴스 생성

   ```shell
   openstack server create --flavor m1.xsmall --image CentOS7 --security-group www_SSH_PERMIT --key-name demoweb --nic net-id=08e862e5-b9bd-46f9-b400-2b6a12d1250a centos_www1
   ```

   <img src="https://i.ibb.co/C54SknR/image-20200709115948409.png" alt="image-20200709115948409" style="zoom:80%;" />

7. open v switch로 NAT 연결

   ```shell
   cd /etc/sysconfig/network-scripts
   cp ifcfg-eth0 ifcfg-br-ex
   vim ifcfg-eth0
   '''
   TYPE=OVSPort
   BOOTPROTO=none
   NAME=eth0
   DEVICE=eth0
   DEVICETYPE=ovs
   OVS_BRIDGE=br-ex
   ONBOOT=yes
   NM_CONTROLLED=no
   '''
   vim ifcfg-br-ex
   '''
   TYPE=OVSBridge
   BOOTPROTO=none
   NAME=br-ex
   DEVICE=br-ex
   DEVICETYPE=ovs
   ONBOOT=yes
   IPADDR=211.183.3.100
   PREFIX=24
   GATEWAY=211.183.3.2
   DNS1=8.8.8.8
   NM_CONTROLLED=no
   '''
   ifdown ifcfg-eth0
   ifup ifcfg-br-ex
   ifup ifcfg-eth0
   reboot
   ifconfig
   ovs-vsctl show
   ```

   <img src="https://i.ibb.co/tLwVWf7/image-20200709122459953.png" alt="image-20200709122459953" style="zoom:80%;" />

   ![image-20200709122436805](https://i.ibb.co/5TyM2NG/image-20200709122436805.png)
   
8. openvswitch br-ex와 openstack의 라우터 연결

   ```shell
   source keystonerc_admin
   # --share							모든 프로젝트와 공유
   # --provider-physical-network		외부 네트워크 장비에 맵핑(answer.txt에 설정되어있음)
   # --provider-network-type flat		flat형으로 네트워크 사용
   # --external						가상 네트워크가 외부에 보이도록 정의
   openstack network create --share --provider-physical-network extnet --provider-network-type flat --external ext_net
   openstack subnet create ext_sub --network ext_net --subnet-range 211.183.3.0/24 --allocation-pool start=211.183.3.111,end=211.183.3.199 --gateway 211.183.3.2 --dns-nameserver 8.8.8.8 --no-dhcp
   source keystonerc_demo
   # neutron router-gateway-set : 라우터의 게이트웨이 ext_net으로 설정
   neutron router-gateway-set DEMO_R1 ext_net
   ```

9. 인스턴스 추가(GUI)

10. 로드밸런서 추가

    ![image-20200709182146505](https://i.ibb.co/MfHGHm5/image-20200709182146505.png)

    ![image-20200709182152124](https://i.ibb.co/17RV47D/image-20200709182152124.png)

    ![image-20200709182356594](https://i.ibb.co/99yqrzt/image-20200709182356594.png)

    ![image-20200709182601609](https://i.ibb.co/WHz66cH/image-20200709182601609.png)

11. 네트워크 토폴로지 확인

    <img src="https://i.ibb.co/8gCY70B/image-20200709181954766.png" alt="image-20200709181954766" style="zoom:80%;" />

### heat 실습

```shell
vim heat-stack.yml
'''
heat_template_version: ocata

description: first Heat Template

parameters:
  NetID:
    type: string
    description: Network ID for the server
    
resources:
  server:
    type: OS::Nova::Server
    properties:
      name: "Heat_Ubuntu"
      image: "cirros"
      flavor: "m1.tiny"
      networks:
        - network: { get_param: NetID }
        
outputs:
  server_ip:
    description: IP address of instance from pricate network
    value: { get_attr: { server, first_address } }
'''
export NET_ID=$(openstack network list | awk '/ private1 / { print $2 }')
openstack stack create -t heat-stack.yml --parameter "NetID=$NET_ID" stack1
```

