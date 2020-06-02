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

![](data:image/png;base64,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)



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
