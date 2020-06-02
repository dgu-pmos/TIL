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

![](data:image/jpeg;base64,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)

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

   ![image-20200602164518207](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200602164518207.png)

   ![image-20200602164550894](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200602164550894.png)

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

   ![image-20200602171641922](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200602171641922.png)

   ![image-20200602171810786](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200602171810786.png)