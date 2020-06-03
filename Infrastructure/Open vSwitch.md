# Open vSwitch

## 이론

### Overlay Network

- 물리적 네트워크를 바탕으로 그 위에 논리적인 topology를 재구성
- 서로 다른 대역의 네트워크라도 이 기술을 이용해 사설 네트워크끼리 통신 가능
- 반대로 공인 IP를 이용한 통신을 underlay network라고 함
- 아래의 그림과 같이 일종의 Tunneling이라고 볼 수 있음
- VXLAN, VPNs 등과 같이 다양한 프로토콜로 구현 가능

![](https://i.ibb.co/98pJN4m/1.png)



### Open vSwitch

![](http://chonnom.com/data/mw.cheditor/1105/YuMQfjFLqvKoerHVR23cU5I6NCHK.gif)

- Apache2 라이센스의 오픈소스 가상 스위치
- Multi Layer Network Switch 기능을 수행하는 소프트웨어
- LXVAN, Openflow, QoS 등 제공 
- 그림과 같이, 물리적 NIC를 연결하거나 독자적으로 사용할 수 있음



### VXLAN

#### 특징

- VXLAN의 'X'의 의미는 eXtensible을 뜻함
- VXLAN header는 24bit VXLAN network ID로 구성
  - 이로 인해, 무려 16,000,000개 이상의 VLAN을 단일 도메인에서 제공
- Tunneling 기반 기법
  - Packet의 Encapsulation 발생 지점이 VTEP
    - VTEP는 VXLAN Tunnel End Point의 약자
    - VTEP는 가상 Software가 될 수도 있고, VXLAN을 지원하는 물리 장치가 될 수도 있음
- VM은 Hypervisor가 제공하는 Software VTEP, 물리 장치는 물리 VTEP 사용

#### 동작 방식

![](https://ssup2.github.io/images/theory_analysis/Overlay_Network_VXLAN/VXLAN_Overview.PNG)

1. encapsulation된 packet은 VXLAN header에 있는 VNI(VXLAN ID)를 통해 어느 가상 network인지 구분

   ![](https://ssup2.github.io/images/theory_analysis/Overlay_Network_VXLAN/VXLAN_Packet.PNG)

   - IP/UDP를 이용해 Tunneling 수행
     - packet 밖에는 Host가 이용하는 물리 network의 Ethernet header, IP header, UDP header 존재
     - packer 안에는 VXLAN header, 가상 network 안에서 발생한 VM의 L2 packer이 위치

2. VXLAN은 가상 network 안에서 발생한 Broadcast Packet을 물리 Network 안에서의 IP Multicast로 처리

   - IP/UDP로 Tunneling 하는 이유도 이 때문
   - 가상 network 안에서 발생한 broadcast packet이 물리 network 안에서 IP multicast로 처리된다는 의미는 특정 VNI와 multicast group이 mapping되어 있다는 의미
   - 이러한 mapping 정보는 VTEP에 설정
   - VNI는 16,000,000개, Multicast Group은 1000개 이므로 N : 1 관계
   - VTEP는 자신에게 온 가상 Network packet을 encapsulation 하므로 VNI와 VLAN의 mapping 정보를 VTEP에 설정

3.  가상 network 안에서 발생한 ARP packet 에 따른 VTEP의 MAC address learning 과정

   ![](https://ssup2.github.io/images/theory_analysis/Overlay_Network_VXLAN/VXLAN_Address_Learning.PNG)

   1. Machine A에서 IP B의 MAC address를 알기 위해 ARP request packet을 VLAN ID 1과 함께 전송
   2. VTEP 1은 packet이 Virtual network VLAN ID 1 인 것을 확인
   3. VTEP에서 VLAN 1과 VNI 10이 mapping 되어 있고, VNI 10는 Multicast 239.1.1.1에 mapping 되어 있어 VNI 10으로 encapsulation 된 후 239.1.1.1 Multicast group에 전송
   4. encapsulated packet은 239.1.1.1  Multicast group에 참여한 VTEP 2, VTEP 3에 전송
   5. VTEP 2, VTEP 3은 encapsulated packet의 정보를 바탕으로 source MAC/VNI/outer source IP mapping table을 생성
   6. decapsulated packet으로 변환해 원래의 ARP packet을 확인하고 Machine B와 C에게 전송
   7. IP B가 machine B의 IP 이기 때문에 ARP response를 Machine A에게 Unicast 통신
   8. VTEP 2에 저장된 Mapping table을 바탕으로 ARP response packet을 전송
   9. VTEP 1은 ARP response packet을 machine A에게 전송



### QoS

- packet loss, latency 등을 줄이기 위해 트래픽을 관리하는 기술
- 네트워크 상에서 데이터 타입에 따라 우선순위를 선정해 네트워크 자원 관리



#### 용어정리

##### Multicast

- Multicast group에 소속된 특정 다수에게 데이터를 전송하는 기법

- 다수에게 보낼 때 1 개의 소켓으로 N번 전송

- UDP를 사용해 신뢰성을 보장받지 못함

- 하나의 Client에서 여러 Multicast 주소를 수용 가능



#### 출처

https://ssup2.github.io/theory_analysis/Overlay_Network_VXLAN/

https://searchunifiedcommunications.techtarget.com/definition/QoS-Quality-of-Service



## 실습

### bridge를 이용한 네트워크

![](https://i.ibb.co/cCTQMRV/2020-06-03-195245.png)

1. 기존 설정 확인(NFS, SELinux, Firewall, Libvirt, ping..)

2. ip_forward 활성화 확인

   ```
   cat /proc/sys/net/ipv4/ip_forward
   ```

   - enable : 1, disable : 0

   - 해당 시스템 안에서 커널이 처리하는 packet을 외부로 forwarding 가능

   - 가상화 환경에서 서로 다른 네트워크를 사용하는 가상머신 간 라우팅 시 사용

3. br0, eth0 설정

   ```
   [br0]
   vi /etc/sysconfig/network-scripts/ifcfg-br0
   '''
   # 네트워크 인터페이스 타입 설정
   TYPE=Bridge
   # BOOTPROTO : IP 할당 타입
   # none(없음) | static(수동) | dhcp(동적)
   BOOTPROTO=none
   NAME=br0
   DEVICE=br0
   ONBOOT=yes
   IPADDR=211.183.3.101
   PREFIX=24
   GATEWAY=211.183.3.2
   DNS1=8.8.8.8
   NM_CONTROLLED=no
   '''
   
   [eth0]
   vi /etc/sysconfig/network-scripts/ifcfg-eth0
   '''
   TYPE=Ethernet
   BOOTPROTO=none
   NAME=eth0
   DEVICE=eth0
   ONBOOT=yes
   NM_CONTROLLED=no
   # 부착할 Bridge 설정
   BRIDGE=br0
   '''
   ```

4. fw1, fw2 설치 및 셋팅(ram=512, cpu=1, network=bridge:br0, OS=vyos)

   ```
   [kvm1]
   # 방화벽 fw1 생성(네트워크는 br0)
   virt-install --name fw1 --vcpus 1 --ram 512 --disk path=/storage/fw1.img,size=5 --network bridge:br0 --cdrom /storage/vyos.iso 
   # 방화벽 fw2 생성(네트워크는 br0)
   virt-install --name fw2 --vcpus 1 --ram 512 --disk path=/storage/fw2.img,size=5 --network bridge:br0 --cdrom /storage/vyos.iso 
   
   [fw1, fw2]
   config
   # ip 동적 할당
   set int eth eth0 address dhcp
   # 활성화
   commit
   # 변경사항 저장
   save
   ```

5. live migration

   ![](https://i.ibb.co/RjpBR2d/2020-06-03-202005.png)

   bridge가 같은 네트워크 대역이라서 통신이 가능했다.

   

### Open vSwitch를 이용한 네트워크

![](https://i.ibb.co/hRprmpf/2020-06-03-195256.png)

bridge 대신 가상 스위치 Open vSwitch로 변경하고, 각 방화벽에 VXLAN을 적용해 Ping 통신을 시도할 것이다.

1. opencvswitch 설치

   ```
   # EPEL은 Fedora Project에서 제공하는 저장소
   # CentOS는 해당 저장소 URL 추가 해야함
   yum install -y epel-release https://rdoproject.org/repos/rdo-release.rpm
   # 패키지 설치
   yum install -y openvswitch bridge-utils
   # yum 업데이트
   yum update -y
   # openvswitch 시작
   systemctl start openvswitch
   systemctl enable openvswitch
   ```

2. vswitch002, eth0 설정

   ```
   [kvm1]
   vi /etc/sysconfig/network-scripts/ifcfg-vswitch002
   '''
   TYPE=OVSBridge
   DEVICETYPE=ovs
   BOOTPROTO=static
   NAME=vswitch002
   DEVICE=vswitch002
   ONBOOT=yes
   IPADDR=211.183.3.101
   PREFIX=24
   GATEWAY=211.183.3.2
   DNS1=8.8.8.8
   NM_CONTROLLED=no
   '''
   vi /etc/sysconfig/network-scripts/ifcfg-eth0
   '''
   OTPROTO=static
   NAME=eth0
   DEVICE=eth0
   ONBOOT=yes
   NM_CONTROLLED=no
   OVS_BRIDGE=vswitch002
   '''
   
   [kvm2]
   vi /etc/sysconfig/network-scripts/ifcfg-vswitch002
   '''
   TYPE=OVSBridge
   DEVICETYPE=ovs
   BOOTPROTO=static
   NAME=vswitch002
   DEVICE=vswitch002
   ONBOOT=yes
   IPADDR=211.183.3.102
   PREFIX=24
   GATEWAY=211.183.3.2
   DNS1=8.8.8.8
   NM_CONTROLLED=no
   '''
   vi /etc/sysconfig/network-scripts/ifcfg-eth0
   '''
   OTPROTO=static
   NAME=eth0
   DEVICE=eth0
   ONBOOT=yes
   NM_CONTROLLED=no
   OVS_BRIDGE=vswitch002
   '''
   ```

3. fw1.xml 수정(vswitch002 적용)

   ```
   [kvm1]
   virsh edit fw1
   '''
   68     <interface type='bridge'>
   69       <source bridge='vswitch002'/>
   70       <virtualport type='openvswitch'/>
   71       <target dev='fw1_vp01'/>
   72       <model type='virtio'/>
   73     </interface>
   '''
   virsh reboot fw1
   
   [kvm2]
   virsh edit fw2
   '''
   68     <interface type='bridge'>
   69       <source bridge='vswitch002'/>
   70       <virtualport type='openvswitch'/>
   71       <target dev='fw2_vp01'/>
   72       <model type='virtio'/>
   73     </interface>
   '''
   virsh reboot fw2
   ```

4. VXLAN 적용

   ```
   ovs-vsctl add-port vswitch002 vxlan10 -- set interface vxlan10 type=vxlan options:remote_ip=211.183.3.101
   ovs-vsctl add-port vswitch002 vxlan10 -- set interface vxlan10 type=vxlan options:remote_ip=211.183.3.102
   ```

