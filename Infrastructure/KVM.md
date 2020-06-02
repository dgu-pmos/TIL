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

![](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200602153953489.png)



#### 용어정리

##### Dirty page

읽은 파일이 디스크에 업데이트 되지 않고 page cache 내 특정 공간에 업데이트 된 경우

##### state
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
     - qemu-kvm 

       - KVM 핵심 설치 패키지

     - libvirt

       ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAQsAAAC9CAMAAACTb6i8AAABLFBMVEX////h8/3N58r/+8zp6ez/4L/P6szB2r7Z2dm9y9Xj9v9gbGHn+v/f8vpcYGP5+fq40bYuPDX/5sTozK51hHNOTUr29MWxrYxRW0/dwqWwrrBhX2FUYlOYrpXa9tfw8POVlZdZU07//9Z4eHlfaG6otbtgXl3//9lPT1FvblNoZk28uZf29tPw/v+RkJHMysy5ubxreGmqwalMT0SfiXSJmodfXkwAAACTppGvxax2dF7i37XHxJ+joINSUUF+jnw9RTxsYFHP4OmDgGjRzaeUkncwMDF7e32zs7XExMdDQ0QuMy1odWe7pIyrl4F8bl1/iY9hX0RIRznOzK///+fZ2LAhKyAWFhgqKivp/+VHQjlfUUXs4b/KsphEOzMVFRCkj3qQemcaHxqVoaltzlYUAAAMXklEQVR4nO2cC0PaTBaGbYoJGC+jZGGEJLCAlfUCZNLQBJDEKwhRWbt+bW2x+9X//x/2TFCiq1WElkaYl5hOJpd583CSMzFjZ2aYmJiYmJiYmJiYxqylYTV5Xo+P/jGcWtnfe96PqNga1mtjoOMnNmJDKbOT/81n/lClf28N5XU9nhwoMhL/yrwdRuvxP8BiY2s4r5u/l0WGsWAsGAvGgrFgLGYYi7tiLHwxFr4YC1+MhS/Gwhdj4Yux8PWAxYBoJp1FbaNS2a89erhqvBosFrUKeD2IDeR1GBY75c3azvlm5u1BLZaJVau1WLUGh63WarEYTDGoDgyLnXqtFq/HM28vwNQzXodisb+VydTK1Z3zcv3iY71Sr2zUN2IH5Up5/+Dw4rAMFcFhUVl/m9ksxw7r5fLFxTlYq5Qr1VuvO+XKeSU2Iou3mdj5Tn0LivGr2MX5x2+Vzc34xU59p3JQ2c8clA8ygWEBTqjXauZwf/OyWr36GNvYrMUv4uC1tn8IXmsjszioH9b/k6lVdurr1bLHorJfoSz245mLjQCxgLg4KO+U19drlXg5U61/jPW91qjXymgsKluxanmnVr/Y2j+Mn69f1IHFzsbhVvycxsVO5iJAcbFx4/UgUzncrPdY7JQPY9Srx2K0uIifl8v1ffi3fl7Z+lhZr+5fbB3WNsvlciUO94vNTPXwIigsNuE+UT+MZajX6kElA15jhwf3vB6MwgJux9VYr3DzA1PsLVSCaMn7CQaLJ73G7ntl/U5fjAVjwVgwFowFY0H1OlmESonh1Cw+4fpVslhKtArpoZRMpiaORTa0lBpCS+nsBLIIp0JDKMVYMBaMBWPBWDAWjMVksBgKxpPjfl8pi+Z5fChtVo5KzVslmuHfxCLlN9JsNob0Gt8fkMVf0i+Qnr7X2CMsvMXUc4AesAg77ujubDzgNSLzv0BS4RkW4RJUhI8fREv4GRZFjEZ3pyqDs+BG1PMsUukPiaVU4UNhqRcivSmUKmVTz7MY1R4XOBaFUCh/VJgpJZPZRDidzRcK+UI4+SmRmjAW3LMsCsnkcfEoWwi3EsV8snTULH1KF1vNdOv/mU0Di0K20Mg20s1keCadLCXD4XwxlUw0kxN3jQzAotHMHzUbBaCwVKAsisDiXSIxlSyyqdZRqFBYyiYbn3OlbDichLholo5K03eNHB+nSqXQcTGUSCeyyVCJJtkQTInS1MUFTaEpL5Fm3yXzTS+f9jPr1LHoq/iwwzW1LJ7sh08Ri9Sdp5JHkUwRC7g+6LwIzyHhYviRDaaFRSqchf53MZFL5tOhbDb5rvQQ19SwKGahG55otIrH+USrESokH9lmSliEio1c9lOiUEjNJAv58Mxx8mFgBIcF35/99KRHYVF4d1z8nCgkQ2GIi/RSGnqeL2IxrL0AxkWqeZR8Byw+51vwuNq66XL9yrgQfhkLHrkCz1uawNOz834h5M25/pwo95dvXQ4YF6nwcTgcbjSOi0uh1nHxsUzzBAtekCxo0UWqb4/rm4Syiuy7znx7Q7BQUZtTVeIgIIIEAZkE2iSmxXNIURCUkUVUy1SgpJjg0FLIy1h4JxsqpGkHI19cenT1EyyQTlSVc4gAWMCeYlKf1CQCk/AFQhUvmLTWMqhpogj8CCwIQoauyAISCXZcx+YkrOkmcrDrmqoi2jZxRGwIri46FmlDiX8pi1s91rl4loVpIeIQV1Fd026LWBQUXcMSrzuaaKtINmUElSJngHXDcrCNRmDxXRZFvY1EwxCRY6sEm21bEbHpINWWOWzYkuLYJjIdjjc0w3npNXI/Pl7MQtB1UZR/WLZLMJFkjgNjriI5pK2oJhYkTdEtrCkKrFUtx9TJSNfIMsQXcQQTuwavK0BacjRbMhSMgIuBLUPjFIgWQ/+iKrKtqyOw+Bmip+ICE7DqIAuLmiDZvCoabdGWJAhnVRAhnonOWZIuAxdVWDbu3HSHYGG1PRYcchyLd2TBhugwkC0RHfG89sHgbMnULNMxsYJkw3BUbrwsdMpi2eK076aqOZaCiesi06UseOOHzMH1oxHFMUUJ2aKij8LiJo9InCpijncwFi24R2CRIIkyki3VNAQNlnlFxhJHtDGz6OURDalGG/GSg2WFRxqWTd6FeiSaqiUJBpYN3hKxi5A0Coub/CxwCjbhfm0KNHVxNIEItyvp3Rq4wH38tnZ8LPr2LNFVOU0CI9Qe2LmpB0fUGc2l9+2N0O9EIuRpzia9ei9V91f2lm9m42ZxY0PSoMo0uYf2+MftjdTv7DcwoMbE4p69l/gbgcWL++TjZvFSf8F6h/hLWbxYY2Dx8/fsM4nsz3pTT2sp3XjqPXtgWdzdjbK4+39EQlwMg4LGRejugWaOsRX8uIBM067rt7q8/Oc9/f/ywHqw41W/jfrlsFx+Pwvt7+jqOBXt3nnECBqL9ysrK7OzK95ndvZmdlPzsLBCp15N/3O/4G06++jO3hQNNIvZwbXSP9dhtbI6KSxOVle7qye3NFZWT7zvuzuFLFaiuyfR6MnXb94CfKKrtNTdjU4ni5PT7vfLy93V9ycrq2en71fPrr5efX/JMSaIxenf3atudPf09Ou392enu92vp7OnX1enlcXJ7reV/56tfu3unnR3u2fdle7ut4EPMWEsum0aFyunH86iwGIXWHyNTl9czEbPTk7fd692d8+iK92r05Xu2SrERfRyCu8XAGM2SicvfUCJLtPS6uBHmBwWs14u7fcsb+Yv6X2Nh8VQ6vXBx6mobqlDWR2YBRaGExL/jo5XXUcZzipnDsai9B0PKd1ZHrOcIZ3Ky8lBUEBcoImXYIzzb2mCrXH+7jfoYix8MRa+GAtfY2Phv3Z9ZN1PN+7d1H5yxLvbv/R95mMaFwseeaO2iMVzD06OVwT/Xa83F3oD4yyBGIZBBP4eHa537kjxqwXTRLyFRjHIjY+Faoj0hAlRvfEJdwbXwdQmdJgCL9AVdHgA55oq7GJKAsaShDFSe4ME6Lw/hgCZFIjHScCiqxNLFEaLjPGyEExCJFF3BZuoyCaSxdv0jJaxjOlgKt21eDh1RflBh/gImsJjWC2IBmfr2OYUDdCIusETGTaybM52MWzA8SYWeE3iNfNVsXANs00sbGJTtXTLwJJM34O2Nc6WiW5ymkscU7GRbHhnyHOUBW+4to4s3ZQczm5b5C/BsJGETV1YtgV7GZ7DJAmO73KKPqLH8bLQDNMVOMmgLLClyh8sFa4Lh3xRRNsVvhDZkhxsCiJcI6qBv9ywEEVbVelfnKumzKO2QDQX6xAMEEykTVlonGqLvPXjVbGAuBAFTjOwQUfLEuwY9LaxTFRTNIGNgYlpGcvkHgsBG7bMc7Jtaz0WBNuWdJeF4l0jqvXXK2FhOqKoKZJpajQuYAljAhHgffE6URVXkBxZVwRdFl1Bo0MIiCzwcG8RHZcT6GMksiEuRB45yMWugxVZEIGFA4HFwcEoyVE9ji2nKiYkPgsJFsdZiCcKsizC8RZNtZA1oZonkBghPSiwMR2iDOGjWrATrOEFBfIuQjyCvQldQgSKFuxHvERCV3Oy8jrunXdGpNzrRN2OH7s/wN6rVYmG/M1u829/xDtd4m5HpFGZ0oi9rSD3wcfdYJBZvBDGyAYDzWLMYix8MRa+fvv7kdejwVkM+37kFWnQ9yPNH+LEy3UGjIt8ZOK1d/1uMBa5+YnX9trALObeTLgYC1+MhS/Gwhdj4Yux8MVY+GIsfDEWvhgLX4yFL8bCF2PhK2gs5mgrd1sa4zcQNBZ7C3NvFvYWbhfn3uyNo9WegsVibq/RWZjf66xtz89De3Pz23vQrlceg4LGIvdpDc5/bT5yvbY3v7d2fX20DfPIwvP7jq7AsWjkIsDiOt/JZRc6i51WPtTJpfPX87+/8eCxWFtbjGSvc2vbkVwnF9ley6+1Ote51jhaDx6L7c5i7nqRXikQItsRj0WnM41xQSG0Pkc6ubXOYjjXWVts7S12ItPJ4joyPx9pRPauc50IJJQcUAAwncg4EkmwWPQ0R7Ppwtw8Lcx709wU5tQbeX3PuV5p7s2YSASUxR8SY+GLsfDFWPhiLHwxFr4YC1+MhS/Gwhdj4Yux8MVY+GIsfDEWvhgLX4yFL8bCF2Phi7Hw9QIW83MTrhewWJh0Dcri+PPi5KuVHQTFzFJ4GhQaiAUTk6fw8Z92EBiFs4nin/YQFIWyifCf9hAYhVhYMDExMQ2j/wH4MVI68huIDAAAAABJRU5ErkJggg==)

       - KVM 데몬(다양한 하이퍼바이저 통합 관리 API)
       - virsh 명령을 사용하거나 API를 사용하는 도구를 만들어서 사용

     - virt-install
       - CLI 상 가상머신 설치 도구
       - 하드웨어 사양, 설치 미디어 지정, 디스크 크기 등 지정
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