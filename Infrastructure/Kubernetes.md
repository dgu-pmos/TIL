# Kubernetes

## 이론

### Kubenetes

- 가장 많이 사용되는 컨테이너 오케스트레이션 도구
- 분산 환경에서 '마치 한 대의 컴퓨터'처럼 투과적으로 컨테이너에 엑세스 할 수 있음
- 유연하게 스케일하거나 여러 개의 컨테이너를 효율적으로 통합 관리가 가능
- 모든 리소스는 오브젝트 형태로 관리

#### 서버 구성

- 마스터 서버
  - 쿠버네티스 클러스터 안의 컨테이너를 조작하기 위한 서버
  - kubectl 명령을 사용해 조작
- 백엔드 데이터베이스
  - etcd라 불리는 분산 key-value store(KVS)를 사용하여 클러스터 구성 정보를 관리
  - 클러스터를 구축하기 위한 설정 정보가 들어있음
- 노드
  - 실제로 도커 컨테이너를 작동시키는 서버
  - 노드의 관리는 마스터가 진행

#### 어플리케이션 구성

![image-20200617134450981](https://i.ibb.co/k3rDtHs/image-20200617134450981.png)

- Pod

  - 컨테이너, 저장소 리소스, 특정 네트워크 정보, 옵션 등을 캡슐화해서 배포하는 단위
  - 이 단위로 컨테이너 작성, 시작, 삭제 등과 같은 조작 수행
  - Pod는 반드시 동일한 노드 상에 동시에 전개
    - Pod 안의 여러 컨테이너가 가상 NIC(가상IP)를 공유하기 때문
  - pod template를 이용해 사용자를 대신해 자동 pod 관리
    - pod template는 pod 생성을 위한 명세
    - pod template를 수정하면 이미 존재하는 pod에는 영향을 미치지 않음

- Replica Set

  - 클러스터 상에서 미리 지정된 포드를 작성해 실행시켜 두는 장치
    - 클러스터 상에 정해진 수의 포드를 반드시 실행시킨다는 뜻
  - 실행 중인 포드를 감지해 장애와 같은 이유로 정지된 경우, 해당 포드를 삭제하고 새로 생성해 정해진 갯수를 유지

- Deployment

  - Pod과 Replica Set을 모은 것
  - Replica set의 이력을 관리
    - 포드 안의 컨테이너의  버전업을 하고 싶다면 무정지 롤링 업데이트 수행
    - 이력을 바탕으로 이전 세대로 롤백
  - Replica set의 작성이나 갱신을 정의하는 것

- Service

  - 클러스터 안에서 실행되는 포드에 대해 외부로부터 엑세스 하는 것을 정의

  - 쿠버네티스 네트워크 관리

    - 대표적으로 로드 밸런서

  - 서비스에 의해 할당되는 주소는 Cluster IP, External IP가 존재

    - Cluster IP는 클러스터 안의 포드끼리 통신을 위한 IP
    - 외부 클라이언트가 연결하기 위한 공인 IP

  - NodePort

    ![image-20200617154933419](https://i.ibb.co/Wz84KFN/image-20200617154933419.png)

    - 클러스터 외부에서도 클러스터에 접속 가능하도록 함
    - 모든 노드의 특정 포트를 개방해 서비스에 접근
    - 각 노드의 IP를 알아야만 pod에 접근 가능
    - 포트 당 한 서비스만 할당할 수 있음
    - 30000-32767 사이의 포트만 사용 가능
    - Node의 IP 주소가 바뀌면 이를 반영해야 함
    - 보안적인 측면과 각 노드의 IP를 알아야 함 

  - Load Balancer

    ![image-20200617155108342](https://i.ibb.co/jJ7N5zs/image-20200617155108342.png)

    - pod과 연결해서 사용
    - 클라우드 플랫폼으로부터 도메인 이름과 IP를 할당받아 쉽게 pod에 접근
    - 서비스를 인터넷에 노출하는 일반적인 방식
    - 모든 트래픽을 서비스로 포워딩하는 단 하나의 IP 주소를 제공
    - 지정한 포트의 모든 트래픽은 서비스로 포워딩(거의 모든 종류의 트래픽을 보낼 수 있음)

#### Label

- 쿠버네티스는 리소스를 식별하기 위해 내부에서 작동하는 랜덤한 이름이 부여
  - 이것만으로는 리소스 관리 힘들어서 label 붙여서 관리
- Key-Value 형식으로 된 임의의 문자열
  - ex. color: blue
- label이 붙은 리소스를 참조하려면 selector로 지정
- 관련 있는 포드별로 모아서 유연하게 관리하고 싶을 때 임의의 label 설정
- 쿠버네티스의 정의 파일인 매니페스트 파일을 참조할 때도 사용

### 쿠버네티스 구조

![image-20200617104233952](https://i.ibb.co/sRwKtjZ/image-20200617104233952.png)

- API Server
  - 쿠버네티스의 리소스 정보를 관리하기 위한 프론트엔드 REST API
  - 각 컴포넌트로부터 리소스 정보를 받아 etcd에 저장
  - 다른 컴포넌트는 API server를 통해 etcd에 엑세스
  - 개발자가 엑세스하려면 web GUI 툴이나 kubectl 명령 사용
- Scheduler
  - 포드를 어떤 노드에서 작동시킬지를 제어하는 백엔드 컴포넌트
- controller manager
  - 쿠버네티스 클러스터의 상태를 모니터링하는 백엔드 컴포넌트
  - 정의 파일에서 정의한 것과 실제 상태를 모아서 관리
- etcd
  - 쿠버네티스 클러스터 구성을 유지, 관리하는 KVS 형으로 데이터 관리
  - API Server가 참조
- node
  - kubelet 이라는 에이전트가 작동
    - 이것은 포드의 정의 파일에 따라 도커 컨테이너를 실행하거나 스토리지를 마운트하는 등의 기능 제공
    - 노드의 상태를 정기적으로 감시하고 상태가 바뀌면 API Server에게 통지

#### YAML

- apiVersion
  - yaml 파일에서 정의한 오브젝트의 API 버전
- kind
  - 오브젝트의 종류
- metadata
  - 오브젝트 식별 데이터
- label
  - 외부에서 pod 탐색 시 사용. 복수 label 입력 가능
- spec
  - 오브젝트의 리소스 생성을 위한 자세한 정보
- selector
  - 리소스를 선택하기 위해 사용
- contaniers
  - pod 내부에서 생성할 컨테이너 목록
- ports
  - 포트 개방
  - containerPort : 컨테이너에서 공개할 포트
  - targetPort : pod에 설정된 contanierPort에 접근할 때 사용할 포트

###### 출처

https://blog.leocat.kr/notes/2019/08/22/translation-kubernetes-nodeport-vs-loadbalancer-vs-ingress

## 실습

### GCP를 이용한 Private Registry 구축

1. Google Container Registry API 등록

   ![](https://i.ibb.co/HtJY3DZ/image-20200616155808893.png)

2. 로컬 호스트에 패키지 소스로 Cloud SDK 배포 URL 추가

   ```
   echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
   ```

3. Google Cloud 공개 키 가져오기

   ```
   curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
   ```

4. 업데이트 후 설치

   ```
   sudo apt-get update && sudo apt-get install google-cloud-sdk
   ```

5. Gloud SDK 초기화 (Gloud SDK 설정 완료)

   ```
   gcloud init
   ```

6. 프로젝트명 전역변수 설정

   ```
   export PROJECT_ID=$(gcloud config list project --format "value(core.project)")
   ```

7. 이미지 태그

   ```
   docker tag rapa/xe:1.0 asia.gcr.io/rapa-phm/gcp-test
   ```

8. Google Container Registry에 이미지 업로드

   ```
   gcloud docker -- push asia.gcr.io/rapa-phm/gcp-test
   ```

   ![image-20200616165651900](https://i.ibb.co/Drsp5Dh/image-20200616165651900.png)

9. 결과

   ![image-20200616170256300](https://i.ibb.co/0F1qgWz/image-20200616170256300.png)

10. 로컬 저장소에 이미지 삭제 후 pull

   ```
   docker image rm -f asia.gcr.io/rapa-phm/gcp-test
   gcloud docker -- pull asia.gcr.io/rapa-phm/gcp-test
   docker image ls
   ```

11. 결과

    ![image-20200616170632486](https://i.ibb.co/cL4wxpn/image-20200616170632486.png)

    

### Google Kubenetes Engine을 이용한 클러스터 구축

1. 클러스터 생성

   ![image-20200616174107689](https://i.ibb.co/rb0V2Cr/image-20200616174107689.png)

2. 결과

   ![image-20200616174457953](https://i.ibb.co/RH9vYDc/image-20200616174457953.png)

3. 로컬 호스트와 클러스터 연결

   ![image-20200616174514814](https://i.ibb.co/LNKjP8C/image-20200616174514814.png)

   ![image-20200616174609496](https://i.ibb.co/gFCX2Yn/image-20200616174609496.png)

4. kubectl 설치 및 클러스터 상태 확인

   ```
   snap install kubectl --classic
   kubectl get nodes
   ```

   ![image-20200617092121157](https://i.ibb.co/D1jmtGF/image-20200617092121157.png)

### replicaset을 이용한 서비스 구축 및 replica 수 변경 적용 실습

1. yaml으로 replicaset 생성

   ```yaml
   vim replicaset-nginx.yaml
   '''
   apiVersion: apps/v1
   # 오브젝트 타입은 replicaset
   kind: ReplicaSet
   metadata:
     # 오브젝트 이름은 replicaset-nginx
     name: replicaset-nginx
   # 오브젝트 명세
   spec:
     # 레플리카 수는 3개
     replicas: 3
     # 적용할 pod 라벨 기준으로 선택
     selector:
       matchLabels:
         app: my-nginx-pods-label
     # pod 템플릿 정의
     template:
       # 이름은 my-nginx-pod, 라벨은 my-nginx-pods-label
       metadata:
         name: my-nginx-pod
         labels:
           app: my-nginx-pods-label
       # nginx 컨테이너 1개 생성 (80번 포트 open)   
       spec:
         containers:
         - name: nginx
           image: nginx:latest
           ports:
           - containerPort: 80
   '''
   kubectl apply -f replicaset-nginx.yaml
   kubectl get pods
   ```

2. 결과

   ![image-20200617115922031](https://i.ibb.co/7RFP4NH/image-20200617115922031.png)

3. 다른 yaml으로 replica 설정 변경

   ```yaml
   vim replicaset-nginx-4pods.yaml
   '''
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
     name: replicaset-nginx
   spec:
     # 이전과 동일하지만 replica 수만 3에서 4로 바뀜
     # Deployment에서 matchLabels는 기본적으로 deployment에서 관리하는 pod을 지정할 때 사용
     replicas: 4
     selector:
       matchLabels:
         app: my-nginx-pods-label
     template:
       metadata:
         name: my-nginx-pod
         labels:
           app: my-nginx-pods-label
       spec:
         containers:
         - name: nginx
           image: nginx:latest
           ports:
           - containerPort: 80
   
   '''
   kubectl apply -f replicaset-nginx-4pods.yaml
   kubectl get pods
   ```

4. 결과

   ![image-20200617115953998](https://i.ibb.co/xYgKG7w/image-20200617115953998.png)

   같은 label을 selector로 지정했으므로 pod이 1개만 추가됨.

### Deployment를 활용한 rollback 실습

1. deployment 생성

   ```yaml
   vim deployment-nginx.yaml
   '''
   apiVersion: apps/v1
   # 타입은 deployment
   kind: Deployment
   metadata:
     # deployment 이름
     name: my-nginx-deployment
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: my-nginx
     template:
       metadata:
         name: my-nginx-pod
         labels:
           app: my-nginx
       spec:
         containers:
         - name: nginx
           # 컨테이너가 사용하는 이미지는 nginx:1.0
           image: nginx:1.10
           ports:
           - containerPort: 80
   '''
   # revision : 1
   # my-nginx-deployment-77449ffd95-d4kv8
   # my-nginx-deployment-77449ffd95-dg5gp
   # my-nginx-deployment-77449ffd95-zxkx8
   kubectl apply -f deployment-nginx.yaml --record
   kubectl get pods
   kubectl rollout history deployment my-nginx-deployment
   ```

   ![image-20200617141235171](https://i.ibb.co/zNNSrPS/image-20200617141229755.png)

   ![image-20200617141214863](https://i.ibb.co/hF1920R/image-20200617141214863.png)

2. record

   ```
   # revision : 2
   # 새로운 nginx:1.11 로 만든 pod 들
   # my-nginx-deployment-64b9979c9d-k5ccj
   # my-nginx-deployment-64b9979c9d-dg5gp
   # my-nginx-deployment-64b9979c9d-zxkx8
   # 실제 동작은 새로 업데이트 된 pod이고 이전 pod은 중지
   kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record
   kubectl get pods
   kubectl rollout history deployment my-nginx-deployment
   ```

3. 결과

   ![image-20200617141452209](https://i.ibb.co/8jF6Yq1/image-20200617141452209.png)

   ![image-20200617141440252](https://i.ibb.co/n6hyVC8/image-20200617141440252.png)

4. rollback

   ```yaml
   # 기록했던 revision 1 로 rollback
   # my-nginx-deployment-77449ffd95-fd8rs
   # my-nginx-deployment-77449ffd95-fsn9c
   # my-nginx-deployment-77449ffd95-nxd5d
   kubectl rollout undo deployment my-nginx-deployment --to-revision=1
   kubectl get pods
   ```

5. 결과

   ![image-20200617141745359](https://i.ibb.co/jgx5HBz/image-20200617141745359.png)

### GCP를 활용해 서비스 배포

1. google private repositry에 이미지 배포

   ```yaml
   vim cloudbuild.yaml
   '''
   steps:
       # docker 명령을 실행하기 위한 이름 지정(컨테이너 이미지)
       # gcr.io/cloud-builders/docker : GCP에서 제공하는 도커 클라우드 빌더
     - name: 'gcr.io/cloud-builders/docker'
       # 하위 디렉토리 'blue'의 Dockerfile을 빌드
       args: ['build', '-t', 'gcr.io/$PROJECT_ID/imageview:blue', './blue']
     - name: 'gcr.io/cloud-builders/docker'
       # 하위 디렉토리 'green'의 Dockerfile을 빌드
       args: ['build', '-t', 'gcr.io/$PROJECT_ID/imageview:green', './green']
   # 빌드된 이미지의 이미지명 정의
   images: ['gcr.io/$PROJECT_ID/imageview:blue', 'gcr.io/$PROJECT_ID/imageview:green']
   '''
   gcloud builds submit --config config/cloudbuild.yaml
   ```

   ![image-20200617093434834](https://i.ibb.co/sHMH9KH/image-20200617093434834.png)

2. 기타 설정 적용(프로젝트 이름, 암호화)

   ```yaml
   # 프로젝트 이름 설정
   vim configmap.yaml
   '''
   apiVersion: v1
   kind: ConfigMap
   metadata:
       name: projectid
   data:
       project.id: "hello-docker"
   '''
   kubectl create -f config/configmap.yaml
   # 암호화 설정
   vim secret.yaml
   '''
   apiVersion: v1
   kind: Secret
   metadata:
     name: apikey
   type: Opaque
   data:
     id: YXNh
     key: YUJjRDEyMw==
   '''
   kubectl create -f config/secrets.yaml
   ```

   ![image-20200617153206266](https://i.ibb.co/bBwM71f/image-20200617153206266.png)

3.  Deployment 적용

   ```yaml
   # webserver-blue 라는 Deployment 설정
   vim deployment-blue.yaml
   '''
   apiVersion: extensions/v1beta1
   kind: Deployment
   metadata:
     name: webserver-blue
   spec:
     replicas: 3
     template:
       metadata:
         labels:
           type: webserver
           color: blue
       spec:
         containers:
         - image: gcr.io/rapa-phm/imageview:blue
           name: webserver-container
           env:
           - name: PROJECT_ID
             valueFrom:
               configMapKeyRef:
                 name: projectid
                 key: project.id
           - name: SECRET_ID
             valueFrom:
               secretKeyRef:
                 name: apikey
                 key: id
           - name: SECRET_KEY
             valueFrom:
               secretKeyRef:
                 name: apikey
                 key: key
           ports:
           - containerPort: 80
             name: http-server
   '''
   kubectl apply -f config/deployment-blue.yaml
   # 그린도 동일
   vim deployment-green.yaml
   kubectl apply -f config/deployment-green.yaml
   ```

   ![image-20200617153154327](https://i.ibb.co/0rcFTS9/image-20200617153154327.png)

4. 로드밸런서 서비스 설정

   ```yaml
   # 로드밸런서 서비스 설정
   vim service.yaml
   '''
   apiVersion: v1
   kind: Service
   metadata:
     name: webserver
   spec:
     type: LoadBalancer
     ports:
       - port: 80
         targetPort: 80
         protocol: TCP
     selector:
       type: webserver
       color: green
   '''
   ```

   ![image-20200617153803046](https://i.ibb.co/tq7z0KK/image-20200617153803046.png)

5. 결과

   ![image-20200617153753143](https://i.ibb.co/RCMD2DS/image-20200617153753143.png)