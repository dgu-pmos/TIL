# SDN

  ## 현재 네트워크 구조의 한계

  1. 한 장비에 여러 Plane 구성

     한 장비에 data plane, control plane, management plane이 모두 구성되어 패킷이 장비를 거쳐갈 때마다 각 장비에서 동일한 패킷 처리 절차를 수행하게 된다. 이러한 Black Box 구조는 출고 시부터 너무 많은 기능들이 탑재되기 때문에 사용자가 원하는 기능이 10개이더라도 이미 5,400개의 RFC가 탑재되어 자원을 낭비하게 된다.

  2. 운영 자동화와 중앙 관리의 어려움

     현재 네트워크 구성에서는 변경이나 확장이 필요한 경우 관리자가 모든 라우터, 스위치, 방화벽 등을 일일이 변경된 정책과 설정을 입력해야 한다. 이럴 경우에 휴먼 에러 발생률 및 작업 시간도 매우 크게 증가한다. 또한, 다양한 벤더의 장비가 도입된 경우에는 장비별 구현 방식에 따른 특징들도 정확히 확인해야 한다.

  3. 효율과 비용 문제

     네트워크 디자인 시 안정성 확보를 위해 장비를 이중화하고 STP를 적용한다. 그러나, STP는 장애 복구 시간이 오래 걸리고 한쪽 링크를 사용할 수 없어 비효율적 회선 사용이 문제가 된다.

  4. 개별 처리로 인한 네트워크 복잡성 증가

     Dijkstra 알고리즘을 중앙 집중형 동작 방식 원리로 하면 4 페이지의 문서로 설명이 끝나는데, 분산 시스템으로 구현하려면 100 페이지가 넘어가 버린다.

  


  ## NFV

  ### 개념

  ![NFV](https://i.ibb.co/QNnFqZ4/image-20200529175146006.png)

  -   네트워크 서비스를 가상화하고, 이를 전용 하드웨어에서 추출
  -   각 요소를 하드웨어 단위로 구분하던 기존 네트워크와는 달리 물리적 자원 최소화
  -   전체 효율성을 향상시켜 시스템 복잡성을 감소



  ### SDN과 NFV 비교표

| Category        | SDN                                                          | NFV                                                   |
| --------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| 출범 동기       | control plane과 data plane 분리, 중앙화된 관리 네트워크 프로그래밍 | 특정 장비에 귀속된 네트워크 기능을 일반 서버에 재배치 |
| 적용 위치       | campus, data center/cloud                                    | service provider network                              |
| 사용 장비       | commodity server, switches                                   | commodity server, switches                            |
| applications    | cloud orchestration, networking                              | router, firewall, gateway, CDN..                      |
| 대표적 프로토콜 | openflow                                                     | 아직 제정 안됨                                        |
| 규격 제정       | open networking forum(ONF)                                   | ETSI NFV Working Group                                |

  

  ## SDN 아키텍쳐

  ### 개요

   현재 네트워크는 네트워크 장비 간 프로토콜을 맞추어 서로 통신은 하지만 트래픽에 대한 연산은 개별 장비에서 이루어지기 때문에 네트워크 회선 사용률이 매우 낮다.

   전 세계 인터넷 트래픽의 7%가 google에서 사용된다는 보고서가 있다. 이러한 비효율적 회선 사용으로 큰 타격을 입은 google은 WAN Fabrics를 도입해 전체 트래픽을 균등하게 분산해 회성 사용률을 40%에서 95%로 올렸다. 이 WAN Fabrics 의 핵심 기술이 SDN/Openflow이다. SDN은 쉽게 설명하면 네트워크의 모든 교통 수단(네트워크 장비)를 지능화된 중앙 관리 시스템(Controller)에 의해서 관리하는 기술이다.

  

  ### SDN 아키텍쳐

  ![image-20200529204531263](https://i.ibb.co/BVshxJ8/image-20200529204531263.png)

  네트워크 장비에서 HW 기능과 SW 기능을 분리해 해는 것이다.

  #### 기존 네트워크 구조

  - 데이터 전송을 담당하는 Data plane(하드웨어 영역)
  - 운영체제 기능을 담당하는 Control plane(소프트웨어 영역)
  - 네트워크 지능화 기능을 담당하는 Application(소프트웨어 영역)

  #### 기존 네트워크 문제점

  초기 인터넷에서는 안정성의 문제로 개별 네트워크 장비 안에 이 3가지 구성 요소가 동시에 필요했다. 그러나, 인터넷의 규모가 커지면서 부가적인 기능들이 더욱 증가하였고 결국에 복잡함이 커지는 결과를 초래했다.

  

  #### SDN 구조

  ![image-20200529204615220](https://i.ibb.co/4jNrZx7/image-20200529204615220.png)

  SDN은 네트워크 구조를 위와 같이 3개의 layer로 분리하는데 각 layer는 Open interface를 통해 서로 통신한다. 

  - Infrastructure Layer(Data Plane) <-> Network Control Layer(Control Plane)
    - Southbound API (Openflow)
  - Network Control Layer(Control Plane) <-> Application Layer(Application Plane)
    - Northbound API

  

  #### Openflow

  Infrastructure Layer는 수많은 네트워크 장비가 논리적으로 마치 한 대의 네트워크 장비가 동작하는 것처럼 운용되는 것을 목적으로 한다. 이를 위해 모든 개별 스위치들은 Control Layer와 긴밀하게 통일된 정보를 주고 받아야 한다. 여기서 널리 사용되는 것이 Openflow이다. 

  ![image-20200529204704257](https://i.ibb.co/PwvgtLb/image-20200529204704257.png)

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

      ![image-20200530171342572](https://i.ibb.co/w7r1JVc/image-20200530171342572.png)

    가장 많이 사용되는 방식이다. controller를 Active, Standby로 배치해 이중화했다.

    - distributed controller

    ![image-20200530171508239](https://i.ibb.co/N3W06Vn/image-20200530171508239.png)

    매우 큰 데이터의 잦은 통신이 일어나는 환경에 좋다. 구글의 G-scale에서 사용된다.

    - multilayer controller

    ![image-20200530171526934](https://i.ibb.co/RS1fKnB/image-20200530171526934.png)

    본사와 다수의 지사가 연결된 은행이나 체인점 등에 유용하다.

  

  #### Application Layer

  - SDN에서 이 영역은 실제로 매우 광범위
  - 기존 Application 영역 기업들은 Northbound API 제공만 한다면 문제 없음
  - 그러나, 아직 Northbound API가 표준화되지 않았음

  

  ## Overlay SDN

  ### Overlay

  - 사전 상 의미는 '덮어씌우다.'
  - 서버 가상화 영역에 SDN을 구현해 물리적 스위치 교체 없이 네트워크 지능화 구현
  - Hypervisor 영역
  - VXLAN, NVGRE가 대표적

  ![image-20200530174306697](https://i.ibb.co/ZJghZ11/image-20200530174306697.png)

  

  

  

  ## Network Virtualization

  ### Network Virtualization

  - 사용자의 위치와 상관없이 업무의 목적에 맞게 네트워크를 분리할 수 있음

  ![image-20200530174736272](https://i.ibb.co/YLGZBhP/image-20200530174736272.png)

  - 네트워크를 필요에 따라 동적으로 쉽게 생성하거나 제거
  - QoS를 통해 트래픽 제어
  - 하나의 공통된 물리적 인프라에서 자신이 원하는 가상 네트워크 생성

  

  ## 용어 정리

  - Black Box
    - 운영체제 및 각종 RFC가 탑재된 스위치
  - White Box
    - 운영체제가 없는 깡통 스위치
  - North-South 트래픽
    - 데이터 센터와 클라이언트, 네트워크 상의 데이터 센터 외부와 통신되는 트래픽
    - 내부 트래픽을 제외한 나머지 트래픽
  - East-West 트래픽
    - 데이터 센터 내부에서 발생되는 트래픽
    - 서버 간의 트래픽
  - 3 Tier 구조
    - core switch
      - 고성능 라우터 장비들에 해당하는 레이어
      - 포트 수는 작더라도 대량의 패킷 처리
      - 소규모 네트워크에서는 생략
    - distributed switch
      - 라우터와 스위치의 적절한 조합으로 기능 수행
    - access switch
      - 포트 수가 많고 저렴한 스위치 장비 위주
  - QoS
      - 데이터의 종류에 따른 우선 순위에 따라 트래픽 제어
  - Hypervisor
        - 호스트 컴퓨터에서 다수의 운영체제를 동시에 실행하기 위한 논리적 플랫폼
  - VXLAN
      - 기존 VLAN에서 더 큰 확장성을 제공(L4 UDP에서 동작)
      - 16,000,000개 이상의 VLAN을 단일 도메인에서 제공