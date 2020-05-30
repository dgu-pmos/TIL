## SDN 아키텍쳐

### 개요

 현재 네트워크는 네트워크 장비 간 프로토콜을 맞추어 서로 통신은 하지만 트래픽에 대한 연산은 개별 장비에서 이루어지기 때문에 네트워크 회선 사용률이 매우 낮다.

 전 세계 인터넷 트래픽의 7%가 google에서 사용된다는 보고서가 있다. 이러한 비효율적 회선 사용으로 큰 타격을 입은 google은 WAN Fabrics를 도입해 전체 트래픽을 균등하게 분산해 회성 사용률을 40%에서 95%로 올렸다. 이 WAN Fabrics 의 핵심 기술이 SDN/Openflow이다. SDN은 쉽게 설명하면 네트워크의 모든 교통 수단(네트워크 장비)를 지능화된 중앙 관리 시스템(Controller)에 의해서 관리하는 기술이다.



### SDN 아키텍쳐

![image-20200529204531263](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200529204531263.png)

네트워크 장비에서 HW 기능과 SW 기능을 분리해 해는 것이다.

#### 기존 네트워크 구조

- 데이터 전송을 담당하는 Data plane(하드웨어 영역)
- 운영체제 기능을 담당하는 Control plane(소프트웨어 영역)
- 네트워크 지능화 기능을 담당하는 Application(소프트웨어 영역)

#### 기존 네트워크 문제점

초기 인터넷에서는 안정성의 문제로 개별 네트워크 장비 안에 이 3가지 구성 요소가 동시에 필요했다. 그러나, 인터넷의 규모가 커지면서 부가적인 기능들이 더욱 증가하였고 결국에 복잡함이 커지는 결과를 초래했다.



#### SDN 구조

![image-20200529204615220](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200529204615220.png)

SDN은 네트워크 구조를 위와 같이 3개의 layer로 분리하는데 각 layer는 Open interface를 통해 서로 통신한다. 

- Infrastructure Layer(Data Plane) <-> Network Control Layer(Control Plane)
  - Southbound API (Openflow)
- Network Control Layer(Control Plane) <-> Application Layer(Application Plane)
  - Northbound API



#### Openflow

Infrastructure Layer는 수많은 네트워크 장비가 논리적으로 마치 한 대의 네트워크 장비가 동작하는 것처럼 운용되는 것을 목적으로 한다. 이를 위해 모든 개별 스위치들은 Control Layer와 긴밀하게 통일된 정보를 주고 받아야 한다. 여기서 널리 사용되는 것이 Openflow이다. 

![image-20200529204704257](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200529204704257.png)

Openflow를 통한 통신을 위해 Controller와 스위치 간에는 Secure channel을 통해 연결된다. Secure channel을 통해 다양한 정보들이 통신되는데 패킷 처리를 위한 Flow Table도 Secure Channel을 통해 전달된다.



#### Flow table

Flow는 패킷을 처리해주는 규칙이 단위이다. 이러한 Flow를 관리하는 테이블으 Flow Table 이라고 한다.

##### Rule

어떠한 패킷을 처리할지를 정의하는 영역이다. Layer 1부터 Layer 4까지 12개의 구분자를 가지고 패킷을 처리한다.

##### Action

Rule에 의해 정의된 패킷을 어떻게 처리할지를 정의한다. Forward 명령어를 사용하면 패킷은 지정한 포트를 통해서 전송되지만 Drop 명령어를 사용하면 해당 패킷은 폐기된다.

##### Stats

해당 Flow table에 얼마나 많은 Packet이 매칭되었고, 얼마나 큰 Byte가 전송되었는지를 보여준다.



### SDN 계층

#### Infrastructure Layer

- SDN에서 데이터 전송 담당 영역인 Data Plane 을 의미
- SDN을 지지하는 여러 기업들이 기존 벤더의 힘을 빌리지 않고 구현 중에 있음
- SDN 시장의 리더인 Big switch에서 발벗고 나서 Opensource 기반으로 둔 Switch 프로젝트 진행 중(Switch Light)



#### Controller Layer

- 공통부
  - Openflow와 직접적으로 관련된 영역
    - topology, path, link 관리 등

- Application

  - 공통부를 이용해 네트워크를 지능화 하는 영역
    - routing, security, monitoring 등

- Contorller design

  - centralized controller

    ![image-20200530171342572](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200530171342572.png)

  가장 많이 사용되는 방식이다. controller를 Active, Standby로 배치해 이중화했다.

  - distributed controller

  ![image-20200530171508239](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200530171508239.png)

  매우 큰 데이터의 잦은 통신이 일어나는 환경에 좋다. 구글의 G-scale에서 사용된다.

  - multilayer controller

  ![image-20200530171526934](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200530171526934.png)

  본사와 다수의 지사가 연결된 은행이나 체인점 등에 유용하다.



#### Application Layer

- SDN에서 이 영역은 실제로 매우 광범위
- 기존 Application 영역 기업들은 Northbound API 제공만 한다면 문제 없음
- 그러나, 아직 Northbound API가 표준화되지 않았음