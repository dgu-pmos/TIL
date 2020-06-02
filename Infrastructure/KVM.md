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

![](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200602153953489.png)



#### 용어정리

##### Dirty page

읽은 파일이 디스크에 업데이트 되지 않고 page cache 내 특정 공간에 업데이트 된 경우

##### state

![](data:image/png;base64,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)

운영체제 프로세스의 상태를 의미



## 실습

### 방화벽 live migrate

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
     - qemu-kvm : KVM 핵심 설치 패키지
     - libvirt : KVM 데몬
     - virt-install : CLI 상 가상머신 설치 도구
     - virt-manager : GUI Tool
     - virt-viewer : 가상머신 화면 전시
   - qemu.conf 수정

   ```
   vi /etc/libvirt/qemu.conf
   
   '''
   #442	user = "root"
   #443	group = "root"
   '''
   ```

   - 데몬 및 프로세스 실행

   ```
   systemctl start libvirtd
   systemctl enable libvirtd
   virt-manager &
   ```

   - KVM 연결

   ![image-20200602164518207](https://i.ibb.co/bRjYpW6/image-20200602164518207.png)

   ![image-20200602164550894](https://i.ibb.co/BwsVqGV/image-20200602164550894.png)

4. 네트워크 이름 변경

   ```
   vi /etc/default/grub
   '''
   #6	~~~ "~~~ quite net.ifnames=0 biosdevname=0"
   '''
   cd /etc/sysconfig/network-scripts
   cp ifcfg-ens32 ifcfg-ens32.bak
   cp ifcfg-ens33 ifcfg-ens33.bak
   mv ifcfg-ens32 ifcfg-eth0
   mv ifcfg-ens33 ifcfg-eth1
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
   grub2-mkconfig -o /boot/grub2/grub.cfg
   echo "NM_CONTROLLED=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
   echo "NM_CONTROLLED=no" >> /etc/sysconfig/network-scripts/ifcfg-eth1
   ifdown eth0
   ifup eth0
   ifdown eth1
   ifup eth1
   ```

5. Ubuntu, vyos 설치

6. 가상머신 생성

   ```
   [kvm1]
   virt-install --name fw1 --vcpus 1 --ram 512 --disk \
   path=/storage/fw1.img, size=5 --cdrom /storage/vyos.iso
   ```

7. 방화벽 IP 설정

   ```
   [fw1]
   config
   set int eth eth0 address dhcp
   commit
   run show int
   commit
   ```

8. 네트워크 인터페이스 추가 및 적용

   ```
   [kvm1]
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
   virsh net-define /storage/private1.xml
   vrish net-start private1
   vrish net-autostart private1
   virsh attach-interface --domain fw1 --source private1 --type network --model virtio --config --live
   
   [kvm2]
   virsh net-define /storage/private1.xml
   vrish net-start private1
   vrish net-autostart private1
   
   [fw1]
   config
   set int eth eth1 address dhcp
   commit
   ```

9. 결과

   ![image-20200602171641922](https://i.ibb.co/rfBmWvq/image-20200602171641922.png)

   ![image-20200602171810786](https://i.ibb.co/YyM1qr3/image-20200602171810786.png)
