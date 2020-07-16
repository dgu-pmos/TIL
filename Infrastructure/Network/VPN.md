# VPN

## 이론

### VPN

<img src="https://www.atriainnovation.com/wp-content/uploads/2020/03/1_VPN_ATRIA.jpg" style="zoom:67%;" />

- 네트워크를 바깥 사람에게 드러내지 않고 통신할 목적으로 사용하는 사설 통신망

- VPN 커널을 적절히 설계하면 안전하지 않은 네트워크를 통해 전송되는 데이터를 외부에 노출하지 않으면서 서버와 연결
- 터널이 연결된 후에는 원격 네트워크를 마치 로컬 네트워크처럼 사용할 수 있음
- 터널링은 GRE, VXLAN 등을 사용하며, 암호화는 IPsec을 주로 사용 

### VPN 수립단계

1. IKE Phase 1
   - ISAKMP(Internet Security Associastion Key Management Protocol)
   - 두 end-point를 연결하는 가상의 터널을 수립하는 단계(세션 수립)
   - 순서
     1. SA파라미터 값 교환(알고리즘 방식 등)
     2. Diffie-Hellman 에 의한 키 교환
     3. 접속처 인증
2. IKE Phase 2
   - IPsec(Internet Protocol Security)
   - IKE Phase 1 단계에서 수립된 세션을 통해 전송, 수신되는 데이터에 대한 보호단계
   - 순서
     1. IPsec SA 생성

## 실습

1. 기본 IP 설정 및 물리적 연결

   ![image-20200629083222675](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200629083222675.png)

   ```
   MYDC
   '''
   conf t
   # IP 설정
   int fa0/1
   ip add 192.168.1.1 255.255.255.0
   no sh
   int fa0/0
   ip add 12.12.12.1 255.255.255.0 
   no sh
   # 스태틱 라우팅
   ip route 0.0.0.0 0.0.0.0 12.12.12.2
   '''
   INT
   '''
   conf t
   # IP 설정
   int fa0/1
   ip add 12.12.12.2 255.255.255.0
   no sh
   int fa0/0
   ip add 23.23.23.2 255.255.255.0
   no sh
   '''
   AWS
   '''
   conf t
   # IP 설정
   int fa0/0
   ip add 23.23.23.3 255.255.255.0
   no sh
   int fa0/1
   ip add 192.168.2.1 255.255.255.0
   no sh
   # 스태틱 라우팅
   ip route 0.0.0.0 0.0.0.0 23.23.23.2
   '''
   ```

2. IKE Phase 1

   ```
   MYDC
   '''
   conf t
   # isakmp 정책 생성 및 설정(정책 식별 번호 1)
   crypto isakmp policy 1
   # 인증 방식 : pre-shared key
   auth pre
   # 암호 알고리즘 : 3des
   enc 3des
   # Diffie-hellman 그룹 번호 1
   group 1
   exit
   # isakmp 적용(key 비밀번호 : test123, 타겟 IP : 23.23.23.3)
   crypto isakmp key test123 add 23.23.23.3
   '''
   AWS
   '''
   conf t
   # isakmp 정책 생성 및 설정(정책 식별 번호 1)
   crypto isakmp policy 1
   # 인증 방식 : pre-shared key
   auth pre
   # 암호 알고리즘 : 3des
   enc 3des
   # Diffie-hellman 그룹 번호 1
   group 1
   exit
   # isakmp 적용(key 비밀번호 : test123, 타겟 IP : 12.12.12.1)
   crypto isakmp key test123 add 12.12.12.1
   '''
   ```

3. IKE Phase 2

   ```
   MYDC
   '''
   # ipsec 설정(정책 이름 : test, 암호화 알고리즘 : esp-aes 128, 인증 알고리즘 : esp-md5-hmac)
   crypto ipsec transform-set test esp-aes 128 esp-md5-hmac
   conf t
   # ACL 정책 설정(식별번호 111, permit ip 192.168.1.0, 192.168.2.0)
   access-list 111 per ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
   # crypto map 정책 설정(이름 : test, 식별 번호 : 10)
   crypto map test 10 ipsec-isakmp
   # isakmp 정책 111과 매칭
   match add 111 
   set peer 23.23.23.3
   set transform-set test
   # fastethernet 0/0에 crypto map "test" 적용
   int fa0/0
   crypto map test
   '''
   AWS
   '''
   crypto ipsec transform-set test esp-aes 128 esp-md5-hmac
   access-list 111 per ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
   crypto map test 10 ipsec-isakmp
   match add 111
   set peer 12.12.12.1
   set transform-set test
   int fa0/0
   crypto map test
   '''
   ```

4. 결과

   ![image-20200626160750320](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200626160750320.png)

   ![image-20200626160916562](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200626160916562.png)

   ![image-20200626160925880](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200626160925880.png)