# KVM

## 이론

### What is KVM?

![](https://t1.daumcdn.net/cfile/tistory/998C37335A064EC534)

- Kernel-based Virtual Machine의 줄임말
- Full Hypervisor, Type 1 Hypervisor
- 리눅스 커널의 Mainline에 포함된 정식 커널 모듈 중 하나
- CPU에 VT 기능 지원 해야 사용 가능
- 리눅스를 Hypervisor로 변환해 독립된 가상 머신 여러 개 실행
- 가상머신은 각자의 가상화 하드웨어 사용 



#### Qemu

- Quick Emulator의 줄임말
- 에뮬레이터 기능을 제공하는 가상화 솔루션
- Guest CPU를 Host CPU용 명령으로 실행 중에 동적으로 변환
- 거의 모든 Hardware 에뮬레이팅



#### /dev/kvm

- 가상화된 Hardware를 각각의 VM에 노출
- 이 과정에서 KVM모듈과 Qemu 프로세스가 인터페이싱



#### virbr

- virtual bridge의 줄임말
- 주로 NAT로 사용
- libvirt 라이브러리에 의해 제공됨



### Migration

##### 특징

- KVM은 실행 중인 VM을 물리적 호스트 사이에 서비스 중단 없이 이동할 수 있음(live migration)
- AMD와 Intel 간 migation 가능
- 64bit는 64bit끼리 가능(32bit는 제한 없음)
- 호스트 간 image directory는 같은 경로 추천
- 같은 네트워크 환경이여야 Migrate 가능



##### 알고리즘

1. setup
2. transfer memory
   - Guest는 계속 실행
   - 대역폭 제한(유저가 제어)
   - 첫 전송 시, 메모리 전체 전송
   - 반복적으로 모든 dirty page 전송
3. stop the guest
   - 종료 후, VM image 동기화
4. transfer state
   - 최대한 빠르게 전송(대역폭 제한 없음)
   - 가상머신의 모든 state, dirty page는 아직 전송 중
5. continue the guest
   - 성공 시 도착지
     - 새 장소의 NIC을 알리기 위해 "I'm over here" 이더넷 패킷 브로드캐스팅
   - 실패 시 출발지
     - exception 1개 발생



### Overlay Network

- 물리적 네트워크를 바탕으로 그 위에 논리적인 topology를 재구성
- 서로 다른 대역의 네트워크라도 이 기술을 이용해 사설 네트워크끼리 통신 가능
- 반대로 공인 IP를 이용한 통신을 underlay network라고 함
- 아래의 그림과 같이 일종의 Tunneling이라고 볼 수 있음

![](https://i.ibb.co/98pJN4m/1.png)



#### 용어정리

##### Dirty page

읽은 파일이 디스크에 업데이트 되지 않고 page cache 내 특정 공간에 업데이트 된 경우

##### state
운영체제 프로세스의 상태를 의미



## 실습

### 방화벽 live migrate

![](https://i.ibb.co/mz5V0Gx/2020-06-02-204454.png)

1. KVM1, KVM2, Storage 생성
   - VMWare Workstation에서 3대의 가상머신 생성(CentOS7)
   - NIC : NAT, VMnet1(Host-only)
   - 내부 도메인 인식을 위해 /etc/hosts 수정
   - SELinux 및 방화벽 해제
   
3. Storage 설정
   - /etc/exports 수정
   - systemctl restart nfs-server
   
3. KVM 설정

   - KVM Package 설치
     - qemu-kvm 

       - KVM 핵심 설치 패키지

     - libvirt
     
       ![](https://i.ibb.co/4f1GVWV/image.png)
       
       - KVM 데몬(다양한 하이퍼바이저 통합 관리 API)
       - virsh 명령을 사용하거나 API를 사용하는 도구를 만들어서 사용

     - virt-install
       - CLI 상 가상머신 설치 도구
       - 하드웨어 사양, 설치 미디어 지정, 디스크 크기 등 지정
       - libvirt를 통해서 KVM container 생성
     - virt-manager 
       - GUI Tool
       - libvirt를 통해서 가상머신을 관리하기 위한 데스크탑 유저 인터페이스
     - virt-viewer 
     
       - 가상머신 화면 전시

- qemu.conf 수정
  
```
   vi /etc/libvirt/qemu.conf
   
   '''
   # qemu로 생성된 프로세스들의 user ID 설정
   #442	user = "root"
   #443	group = "root"
   '''
```

- 데몬 및 프로세스 실행
  
```
   # libvirtd 실행
   systemctl start libvirtd
   systemctl enable libvirtd
   # virt-manager 열고 터미널 계속 사용
   virt-manager &
```

- KVM 연결
  

![image-20200602164518207](https://i.ibb.co/bRjYpW6/image-20200602164518207.png)

![image-20200602164550894](https://i.ibb.co/BwsVqGV/image-20200602164550894.png)

add connection 에서 hostname을 이용해 KVM 끼리 연결한다.

4. 기존 네트워크 이름으로 변경(ens -> eth)

   ```
   vi /etc/default/grub
   '''
   # net.ifnames=0 biosdevname=0 을 추가하면 바이오스에서 사용하는 장치 드라이버가 아닌, 설정하고자 하는 인터페이스 명으로 변경
   #6	~~~ "~~~ quite net.ifnames=0 biosdevname=0"
   '''
   # grub.cfg 설정 적용
   grub2-mkconfig -o /boot/grub2/grub.cfg
   cd /etc/sysconfig/network-scripts
   # 기존 네트워크 설정 백업
   cp ifcfg-ens32 ifcfg-ens32.bak
   cp ifcfg-ens33 ifcfg-ens33.bak
   # eth0,1 생성
   mv ifcfg-ens32 ifcfg-eth0
   mv ifcfg-ens33 ifcfg-eth1
   # eth0,1 설정
   vi ifcfg-eth0
   '''
   TYPE=Ethernet
   BOOTPROTO=none
   NAME=eth0
   DEVICE=eth0
   ONBOOT=yes
   IPADDR=211.183.3.101
   PREFIX=24
   GATEWAY=211.183.3.2
   DNS1=8.8.8.8
   '''
   v1 ifcfg-eth1
   '''
   TYPE=Ethernet
   BOOTPROTO=none
   NAME=eth1
   DEVICE=eth1
   ONBOOT=yes
   IPADDR=192.168.1.101
   PREFIX=24
   '''
   # NM_CONTROLLED=no 은 GUI 모드에서 편리한 네트워크 설정 허용
   echo "NM_CONTROLLED=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
   echo "NM_CONTROLLED=no" >> /etc/sysconfig/network-scripts/ifcfg-eth1
   # 포트 off -> on
   ifdown eth0
   ifup eth0
   ifdown eth1
   ifup eth1
   ```

5. Ubuntu, vyos 설치

6. 가상머신 생성

   ```
   [kvm1]
   # 이름이 fw1인 가상머신 생성
   # cpu : 1, ram : 512mb, size: 5G, os : vyos
   virt-install --name fw1 --vcpus 1 --ram 512 --disk \
   path=/storage/fw1.img, size=5 --cdrom /storage/vyos.iso
   ```

7. 방화벽 IP 설정

   ```
   [fw1]
   # eth0의 IP dhcp로 할당
   config
   set int eth eth0 address dhcp
   # 변경사항 적용
   commit
   ```

8. 네트워크 인터페이스 추가 및 적용

   ```
   [kvm1]
   # 새로운 가상 네트워크 생성
   # network name : virbr1, ip : 127.16.10.0, mask : 24, dhcp = yes
   vi /storage/private1.xml
   '''
   <network>
   	<name>private1</name>
   	<bridge name='virbr1'/>
   	<ip address='127.16.10.1' netmask='255.255.255.0'>
   		<dhcp>
   			<range start='172.16.10.101' end='172.16.10.199'/>
   		</dhcp>
   	</ip>
   </network>
   '''
   # virsh를 이용해 kvm1에 virbr1 적용
   virsh net-define /storage/private1.xml
   # 네트워크 시작
   vrish net-start private1
   # 자동 시작 설정
   vrish net-autostart private1
   # 가상머신 fw1에 private1 네트워크 인터페이스 부착
   virsh attach-interface --domain fw1 --source private1 --type network --model virtio --config --live
   
   [kvm2]
   # kvm2에도 추가
   virsh net-define /storage/private1.xml
   vrish net-start private1
   vrish net-autostart private1
   
   [fw1]
   # eth1의 IP dhcp 할당
   config
   set int eth eth1 address dhcp
   commit
   ```

9. 결과

   ![image-20200602171641922](https://i.ibb.co/rfBmWvq/image-20200602171641922.png)

   ![image-20200602171810786](https://i.ibb.co/YyM1qr3/image-20200602171810786.png)

KVM 끼리 연결하고, 같은 네트워크 환경을 적용하니 Live Migration 이 성공적으로 동작했다.
