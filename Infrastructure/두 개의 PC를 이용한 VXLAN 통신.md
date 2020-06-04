# 두 개의 PC를 이용한 VXLAN 통신

## 이론

### 네트워크 구성

![](https://i.ibb.co/XWFnGbt/zzz.png)

- 처음에는 다음과 같이 물리적 호스트들과 가상머신(1st VM)이 모두 같은 공유기에 연결되어 있는 상태
- 가상머신(2nd VM)은 가상 브릿지를 사용하고 있으므로 생략

![](https://i.ibb.co/5T6NXKz/zzz2.png)

- 2nd VM을 내부 네트워크처럼 사용하기 위해 다음과 같이 네트워크 구성
- Open vSwtich로 가상 스위치를 생성하고, 1st VM와 2nd VM을 이 스위치에 연결
- VXLAN을 이용해 OVerlay Network 구성



### 실습 목표

- 두 컴퓨터에서 각자 2nd VM 까지 구축
- Open vSwitch를 이용해 VXLAN 통신
- VM2는 MYSQL Server를 제공하고, VM1은 php를 이용해 DB를 연결



## 실습

1. VM 설정

   - 네트워크 종류 : Bridged(같은 공유기를 사용하는 내부 네트워크 환경)
   - IP : 192.168.0.2/24
   - 상대방 IP : 192.168.0.25/24

2. 상대방 리눅스와 통신 연결 확인 및 보안 제거

   ```
   [KVM_HM]
   setenforce 0
   sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
   systemctl stop firewalld
   systemctl disable firewalld
   ping 192.168.0.25
   reboot
   ```

3. 인터페이스 이름 변경 -> eth0, 불필요한 사항 삭제

   ```
   [KVM_HM]
   vi /etc/default/grub
   '''
   # net.ifnames=0 biosdevname=0 을 추가하면 바이오스에서 사용하는 장치 드라이버가 아닌, 설정하고자 하는 인터페이스 명으로 변경
   #6	~~~ "~~~ quite net.ifnames=0 biosdevname=0"
   '''
   # grub.cfg 설정 적용
   grub2-mkconfig -o /boot/grub2/grub.cfg
   # 기존 네트워크 설정 백업
   cp ifcfg-ens32 ifcfg-ens32.bak
   # eth0로 변경
   mv ifcfg-ens32 ifcfg-eth0
   '''
   TYPE=Ethernet
   BOOTPROTO=none
   NAME=eth0
   DEVICE=eth0
   ONBOOT=yes
   IPADDR=192.168.0.2
   PREFIX=24
   GATEWAY=192.168.0.1
   DNS1=8.8.8.8
   '''
   reboot
   ```

4. kvm 관련 패키지 설치, virt-manager와 virt-viewer는 GUI 상에서 동작해 설치할 필요 없음

   대신 html로 확인이 가능하도록 kimchi(wok)를 설치

   ```
   [KVM_HM]
   yum install -y libvirt virt-install
   vi /etc/libvirt/qemu.conf
   '''
   # qemu로 생성된 프로세스들의 user ID 설정
   #442 user = "root"
   #443 group = "root"
   '''
   systemctl start libvirtd
   systemctl enable libvirtd
   yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
   yum install https://github.com/kimchi-project/kimchi/releases/download/2.5.0/wok-2.5.0-0.el7.centos.noarch.rpm
   yum install https://github.com/kimchi-project/kimchi/releases/download/2.5.0/kimchi-2.5.0-0.el7.centos.noarch.rpm
   systemctl start wokd
   systemctl enable wokd
   ```

5. 각자 서버를 설치할 iso 파일 다운로드(/root/Download/xxx.iso)

   ```
   [KVM_HM]
   yum install -y wget
   mkdir /root/Download
   cd /root/Download
   wget http://mirror.kakao.com/centos/7.8.2003/isos/x86_64/CentOS-7-x86_64-DVD-2003.iso
   mv CentOS-7-x86_64-DVD-2003.iso CentOS.iso
   ```

6. ovs 설치

   ```
   [KVM_HM]
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

7. vswitch002 생성(eth0을 ovsport로 포함)

   ```
   [KVM_HM]
   vi /etc/sysconfig/network-scripts/ifcfg-vswitch002
   '''
   TYPE=OVSBridge
   DEVICETYPE=ovs
   BOOTPROTO=static
   NAME=vswitch002
   DEVICE=vswitch002
   ONBOOT=yes
   IPADDR=192.168.0.2
   PREFIX=24
   GATEWAY=192.168.0.1
   DNS1=8.8.8.8
   NM_CONTROLLED=no
   '''
   vi /etc/sysconfig/network-scripts/ifcfg-eth0
   '''
   TYPE=OVSPort
   BOOTPROTO=static
   NAME=eth0
   DEVICE=eth0
   ONBOOT=yes
   NM_CONTROLLED=no
   OVS_BRIDGE=vswitch002
   '''
   ```

8. 가상머신 생성 -> 기본 네트워크로 설치

   ```
   [KVM_HM]
   virt-install --name vm1 --vcpus 1 --ram 2048 --disk path=/root/Download/vm1.img,size=20 --cdrom /root/Download/CentOS.iso
   ```

9. 가상머신에 php 설치

   ```
   [VM1]
   yum -y install wget
   wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
   wget http://rpms.remirepo.net/enterprise/remi-release-7.rpm
   rpm -Uvh remi-release-7.rpm epel-release-latest-7.noarch.rpm
   yum install yum-utils
   yum-config-manager --enable remi-php72
   yum install -y php php-common php-opcache php-mcrypt php-cli php-gd php-curl php-mysql httpd
   systemctl start httpd
   systemctl enable httpd
   vi /etc/httpd/conf/httpd.conf
   '''
   ~~~
   <IfModule dir_module>
       DirectoryIndex index.html index.php
   <IfModule>
   ~~~
       AddType text/html .shtml
       AddOutputFilter INCLUDES .shtml
    
       AddType application/x-httpd-php .html .htm .php .inc
       AddType application/x-httpd-php-source .phps
   </IfModule>
   ~~~
   '''
   systemctl restart httpd
   ```

10. 가상머신 종료 후, virsh edit 가상머신 이름 -> 네트워크 인터페이스 수정

    ```
    [KVM_HM]
    virsh edit vm1
    '''
    68     <interface type='bridge'>
    69       <source bridge='vswitch002'/>
    70       <virtualport type='openvswitch'/>
    71       <target dev='vm1_vp01'/>
    72       <model type='virtio'/>
    73     </interface>
    '''
    virsh reboot vm1
    ```

11. VXLAN 설정(Tunneling)

    ```
    [KVM_HM]
    ovs-vsctl add-port vswitch002 vxlan10 -- set interface vxlan10 type=vxlan options:remote_ip=192.168.0.25
    ```

12. 10.10.10.10으로 IP 등록하고 20과 통신

    ```
    [VM1]
    ping 10.10.10.20
    ```

    ![](https://i.ibb.co/TBwLqq4/2020-06-04-150452.png)

13. php 작성 -> host : 10.10.10.20, user : root, password : 1234

    ```
    [VM1]
    vi /var/www/html/phpinfo.php
    '''
    <?php
        $conn = mysqli_connect(
        	'10.10.10.20',
        	'root',
        	'1234',
        	'mysql'
        );
        echo '<h1>DB Connection Success</h1>';
    ?>
    '''
    ```
    
    브라우저에서 localhost:80/phpinfo.php를 열면 다음과 같은 화면이 출력된다.
    
      ![](https://i.ibb.co/HF8CNXG/2020-06-04-150343.png)