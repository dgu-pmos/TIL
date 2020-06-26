# Kubernetes

## 이론

### Kubernetes

- 가장 많이 사용되는 컨테이너 오케스트레이션 도구
- 분산 환경에서 '마치 한 대의 컴퓨터'처럼 투과적으로 컨테이너에 엑세스 할 수 있음
- 유연하게 스케일하거나 여러 개의 컨테이너를 효율적으로 통합 관리가 가능
- 모든 리소스는 오브젝트 형태로 관리

#### Object

- Pod

  - 하나 이상의 컨테이너 캡슐화해서 배포하는 단위

  - 파드는 반드시 동일한 노드 상에 동시에 전개
  
    - 파드 안의 여러 컨테이너가 가상 NIC(가상 IP)를 공유하기 때문
    
  - 생성 과정
  
    <img src="https://i.ibb.co/k3rDtHs/image-20200617134450981.png" alt="image-20200617134450981" style="zoom:80%;" />

    1. kubectl은 API 서버에 파드 생성 요청
  2. API 서버는 etcd에 노드에 할당되지 않은 파드가 있음을 업데이트
    3. Scheduler는 etcd의 변경사항을 API 서버를 통해 감지하고 파드를 실행할 노드를 선택해 API 서버에 해당 노드에 파드를 배정하도록 업데이트
    4. 해당 노드의 kubelet은 생성할 파드의 정보를 감지해서 도커 컨테이너를 실행하고 결과를 API 서버에게 지속적으로 업데이트
    5. API 서버는 전달받은 파드 상태를 etcd에 업데이트
  
- ReplicaSet

  - 클러스터 상에서 미리 지정된 파드를 작성해 실행시켜 두는 장치
    - 클러스터 상에 정해진 수의 파드를 반드시 실행시킨다는 뜻
  - 파드 템플릿을 이용해 사용자를 대신해 자동 파드 관리
    - 파드 템플릿은 파드 생성을 위한 명세
    - 파드 템플릿을 수정하면 이미 존재하는 파드에는 영향을 미치지 않음
  - 실행 중인 파드를 감지해 장애와 같은 이유로 정지된 경우, 해당 파드를 삭제하고 새로 생성해 정해진 갯수를 유지

- Deployment

  - ReplicaSet의 이력을 관리
    - 파드 안의 컨테이너의  버전을 업데이트 하고 싶다면 무정지 롤링 업데이트 수행
    - 이력을 바탕으로 이전 세대로 롤백
  - 일반적으로 파드, ReplicaSet이 아닌 Deployment를 가장 많이 사용

- Service

  - 클러스터 안에서 실행되는 파드에 대해 외부로부터 엑세스 하는 방법을 추상화
  - 유동적으로 클러스터 내에서 변경되는 파드의 IP를 고정된 주소 할당
  - 서비스에 의해 할당되는 주소는 Cluster IP, External IP가 존재

    - Cluster IP는 클러스터 안의 파드들끼지 연결하기 위한 사설IP
    - 외부 클라이언트가 연결하기 위한 공인 IP
  - 서비스를 사용할 오브젝트는 selector로 지정
  - 모든 서비스는 기본적으로 Round-Robin으로 통신

- Namespace

  ```
  apiVersion: v1
  kind: Namespace
  metadata:
    name: production
  ```

  - 쿠버네티스 리소스들(파드, Service, Deployment ..)이 묶여 있는 하나의 가상 공간
  - 기본 값으로 default가 설정되어 있어 아무 처리를 하지 않았다면 default 소속
  - 사용한다면 metadata 안에 namespace로 설정
  - 노드, PV는 namespace로 구분할 수 없음

- Configmap

  ```
  apiVersion: v1
  data:
    LOG_LEVEL: DEBUG
  kind: ConfigMap
  ```

  - yaml 파일에 직접 환경 변수를 직접 설정하는 하드 코딩 방식을 대체
  - 설정 값을 따로 분리해서 저장
  - configMapKeyRef를 이용해 참조

- Secret

  - 민감한 정보를 저장하기 위한 용도
  - 평문이 아닌 base64로 인코딩하는 방식
  - 사용방법은 configmap과 거의 유사
  - secretKeyRef를 이용해 참조
  
- ServiceAccount

- Role

  - 네임스페이스에 속하는 오브젝트들에 대한 권한을 정의

- ClusterRole

  - 네임스페이스에 속하지 않는 오브젝트들에 대한 권한을 정의

- RoleBinding

  - Role과 계정을 묶어주는 기능

- Cronjobs

  - 주기적으로 실행하는 오브젝트
  - 리눅스에서 쓰이는 cron과 동일


#### Service

- Cluster IP

  - 기본 서비스 타입
  - 클러스터 내부의 노드나 파드에게 클러스터 IP 할당
  - 순수하게 내부 통신용으로 사용

- NodePort

  <img src="https://i.ibb.co/Wz84KFN/image-20200617154933419.png" alt="image-20200617154933419" style="zoom:67%;" />

  - 모든 노드의 지정된 포트를 할당
    - 만약 한 노드에만 파드가 떠있어도 다른 노드들도 동일한 포트를 열어야 함
  - 외부 IP와 연결해 클러스터 외부에서도 클러스터에 접속 가능하도록 함
  - 30000-32767 사이의 포트만 사용 가능

- Load Balancer

<img src="https://i.ibb.co/jJ7N5zs/image-20200617155108342.png" alt="image-20200617155108342" style="zoom:67%;" />

- 클라우드 플랫폼으로부터 도메인 이름과 IP를 할당받아 쉽게 파드에 접근

- 서비스를 인터넷에 노출하는 일반적인 방식

  - 모든 트래픽을 서비스로 포워딩하는 단 하나의 IP 주소를 제공

- Ingress

  <img src="https://i.ibb.co/0tQJqmS/image-20200618152558760.png" alt="image-20200618152558760" style="zoom:67%;" />

  - 외부에서 서버로 유입되는 트래픽을 처리하기 위한 네트워크
  - 외부 요청을 어떻게 처리할 것인지를 정의하는 오브젝트
    - 외부 요청의 라우팅(fruits/apple, fruits/orange)
    - 가상 호스트 기반 요청 처리 
      - 같은 IP(서버)에 대해 다른 도메인 이름으로 요청이 도착했을 때, 처리 방법 정의
    - SSL/TLS 보안 연결 처리

#### Label

- Key-Value 형식으로 된 임의의 문자열
  - ex. color: blue
- 라벨이 붙은 리소스를 참조하려면 selector로 지정
- 관련 있는 파드별로 모아서 유연하게 관리하고 싶을 때 임의의 라벨 설정
- 쿠버네티스의 정의 파일인 매니페스트 파일을 참조할 때도 사용

#### Persistent Volume

- 데이터베이스와 같이 상태가 있는(stateful) 애플리케이션은 데이터 보관 필요

- 어느 노드에서도 접근해 사용할 수 있는 퍼시스펀트 볼륨이 적함

- 워커 노드들이 네트워크 상에서 스토리지를 마운트해 영속적으로 데이터 저장 가능

- 파드에 장애가 생겨 다른 노드로 옮겨갔을 때에도 데이터 유지 가능

- PV에게 마운트 요청은 PVC가 함

  - PV와 PVC는 볼륨의 타입을 몰라도 사용할 수 있도록 추상화
  - PVC 요청에 부합하는 PV를 바인드해 컨테이너에 마운트

- accessModes

  | 이름          | 목록에 출력되는 이름 | 설명                  |
  | ------------- | -------------------- | --------------------- |
  | ReadWriteOnce | RWO                  | 1:1 마운트, 읽기 쓰기 |
  | ReadOnlyMany  | ROX                  | 1:N 마운트, 읽기      |
  | ReadWriteMany | RWX                  | 1:N 마운트, 읽기 쓰기 |

##### PV 과정

<img src="https://i.ibb.co/Kjk14bB/image-20200619112250615.png" alt="image-20200619112250615" style="zoom:67%;" />

1. 인프라 관리자는 네트워크 볼륨의 정보를 이용해 PV 리소스를 미리 생성(정보 안에는 마운트하기 위한 엔드포인트 포함)
2. 사용자는 파드를 정의하는 yaml 파일에 '이 파드는 영속적으로 데이터를 저장하기 위한 볼륨과 마운트 해야한다.' 라는 의미의 PVC를 명시하고, 해당 PVC 생성
3. 쿠버네티스는 생성해 둔 볼륨의 속성과 사용자의 PVC 요구사항이 일치한다면 두 개의 리소스를 매칭 시켜 바인드하고 마운트 실행

#### ServiceAccount

- 여러 개발자와 애플리케이션이 쿠버네티스를 동시에 사용할 수 있도록 보안 제공
- 체계적으로 권한을 관리하기 위한 오브젝트
- RBAC(Role Based Access Control) 기반 기능
- 계정 혹은 클러스터마다 role을 정해서 기능을 제한
- kubectl을 사용하면 자동으로 관리자 권한으로 동작
  - ~./kube/config 파일 권한 설정이 존재
  - kubectl은 ~/.kube/config 파일 설정을 읽어서 클러스터를 제어

##### Role 예시

![image-20200619142107251](https://i.ibb.co/NsdPyxb/image-20200619142107251.png)

서비스 조회할 수 있는 권한 없음

```yaml
vim service-reader-role.yaml
'''
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: service-reader
rules:
- apiGroups: [""]                 # 1. 대상이 될 오브젝트의 API 그룹
  resources: ["services"]          # 2. 대상이 될 오브젝트의 이름
  verbs: ["get", "list"]             # 3. 어떠한 동작을 허용할 것인지 명시
'''
kubectl apply -f service-reader-role.yaml
vim rolebinding-service-reader.yaml
'''
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: service-reader-rolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: parkhm
  namespace: default
roleRef:
  kind: Role
  name: service-reader
  apiGroup: rbac.authorization.k8s.io
'''
kubectl apply -f rolebindinf-service-reader.yaml
```

권한 부여 후, 명령어 동작되는 것을 확인

![image-20200619143149113](https://i.ibb.co/ZJTpFyw/image-20200619143149113.png)

##### Cluster Role 예시

```yaml
vim pv-read.yaml
'''
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  namespace: default
  name: pv-read
rules:
- apiGroups: [""]                    # 1. 대상이 될 오브젝트의 API 그룹
  resources: ["persistentvolumes"]   # 2. 대상이 될 오브젝트의 이름
  verbs: ["get", "list"]             # 3. 어떠한 동작을 허용할 것인지 명시
'''
kubectl apply -f pv-read.yaml
vim rolebinding-pv-read.yaml
'''
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: rolebinding-pv-read
  namespace: default
subjects:
- kind: ServiceAccount
  name: parkhm
  namespace: default
roleRef:
  kind: ClusterRole
  name: pv-read
  apiGroup: rbac.authorization.k8s.io
'''
kubectl apply -f rolebinding-pv-read.yaml
```

![image-20200619153801798](https://i.ibb.co/0tTWfqj/image-20200619153801798.png)

### 쿠버네티스 구조

![image-20200617104233952](https://i.ibb.co/sRwKtjZ/image-20200617104233952.png)

- API Server
  - 쿠버네티스의 리소스 정보를 관리하기 위한 프론트엔드 REST API
  - 쿠버네티스 컨트롤 플레인의 프론트엔드
  - 각 컴포넌트로부터 리소스 정보를 받아 etcd에 저장
  - 다른 컴포넌트는 API server를 통해 etcd에 엑세스
  - 개발자가 엑세스하려면 web GUI 툴이나 kubectl 명령 사용
- Scheduler
  - 할당되지 않은 파드를 어떤 노드에서 작동시킬지를 제어하는 백엔드 컴포넌트
- controller manager
  - 클러스터 내 지정된 컨테이너 개수를 유지할 수 있도록 컴트롤러 관리
  - 쿠버네티스 클러스터의 상태를 모니터링하는 백엔드 컴포넌트
  - 정의 파일에서 정의한 것과 실제 상태를 모아서 관리
- cloud-controller-manager
  - 클라우드 제공 업체(GCP)와 연동하는 컨트롤러를 실행
  - 노드 구성 요소는 클러스터 내 모든 노드에서 실행하는 데이터 플레인 

- etcd
  - 클러스터의 모든 데이터를 저장하는 Key-Value 데이터베이스
  - 쿠버네티스 클러스터 구성을 유지, 관리
  - API Server가 참조
- node
  - kubelet 이라는 에이전트가 작동
    - 클러스터 내 각 노드에서 실행됨
    - 노드와 마스터를 연결하기 위해 필요(마스터의 명령 수신)
    - 파드 명세에 맞게 파드 내 컨테이너를 실행하는 역할
    - 노드의 상태를 정기적으로 감시하고 상태가 바뀌면 API Server에게 통지

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

### ReplicaSet을 이용한 서비스 구축 및 replica 수 변경 적용 실습

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
     # 적용할 파드 라벨 기준으로 선택
     selector:
       matchLabels:
         app: my-nginx-파드s-label
     # 파드 템플릿 정의
     template:
       # 이름은 my-nginx-파드, 라벨은 my-nginx-파드s-label
       metadata:
         name: my-nginx-파드
         labels:
           app: my-nginx-파드s-label
       # nginx 컨테이너 1개 생성 (80번 포트 open)   
       spec:
         containers:
         - name: nginx
           image: nginx:latest
           ports:
           - containerPort: 80
   '''
   kubectl apply -f replicaset-nginx.yaml
   kubectl get 파드s
   ```

2. 결과

   ![image-20200617115922031](https://i.ibb.co/7RFP4NH/image-20200617115922031.png)

3. 다른 yaml으로 replica 설정 변경

   ```yaml
   vim replicaset-nginx-4파드s.yaml
   '''
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
     name: replicaset-nginx
   spec:
     # 이전과 동일하지만 replica 수만 3에서 4로 바뀜
     # Deployment에서 matchLabels는 기본적으로 deployment에서 관리하는 파드을 지정할 때 사용
     replicas: 4
     selector:
       matchLabels:
         app: my-nginx-파드s-label
     template:
       metadata:
         name: my-nginx-파드
         labels:
           app: my-nginx-파드s-label
       spec:
         containers:
         - name: nginx
           image: nginx:latest
           ports:
           - containerPort: 80
   
   '''
   kubectl apply -f replicaset-nginx-4파드s.yaml
   kubectl get 파드s
   ```

4. 결과

   ![image-20200617115953998](https://i.ibb.co/xYgKG7w/image-20200617115953998.png)

   같은 label을 selector로 지정했으므로 파드이 1개만 추가됨.

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
         name: my-nginx-파드
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
   kubectl get 파드s
   kubectl rollout history deployment my-nginx-deployment
   ```

   ![image-20200617141235171](https://i.ibb.co/zNNSrPS/image-20200617141229755.png)

   ![image-20200617141214863](https://i.ibb.co/hF1920R/image-20200617141214863.png)

2. record

   ```
   # revision : 2
   # 새로운 nginx:1.11 로 만든 파드 들
   # my-nginx-deployment-64b9979c9d-k5ccj
   # my-nginx-deployment-64b9979c9d-dg5gp
   # my-nginx-deployment-64b9979c9d-zxkx8
   # 실제 동작은 새로 업데이트 된 파드이고 이전 파드은 중지
   kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record
   kubectl get 파드s
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
   kubectl get 파드s
   ```

5. 결과

   ![image-20200617141745359](https://i.ibb.co/jgx5HBz/image-20200617141745359.png)

### Kubernetes를 이용한 웹 서비스 배포

1. google private registry에 이미지 배포

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

### Ingress를 이용한 서비스 배포

![image-20200619111303334](https://i.ibb.co/g4dfwnh/image-20200619111303334.png)

1. 인그레스 컨트롤러 서버 생성

   ```
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/2de5a893aa15f14102d714e918b0045b960ad1a5/deploy/static/mandatory.yaml
   ```

   - 인그레스는 인그레스 컨트롤러 서버라고 하는 특수한 서버가 필요
   - 이 서버에 규칙을 적용해서 동작하는 방식
   - nginx 인그레스 컨트롤러는 쿠버네티스에서 공식적으로 개발
   - 실제로 외부 요청을 받아들이는 것이 이 서버이기 때문에 외부와 연결할 서비스 필요

2. 인그레스 규칙 생성 및 적용

   ```yaml
   vim ingress-example.yaml
   '''
   apiVersion: networking.k8s.io/v1beta1
   kind: Ingress
   metadata:
     name: ingress-example
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
       kubernetes.io/ingress.class: "nginx"
   spec:
     # 동작 방식 명세
     rules:
       # 시작 호스트 주소
     - host: alicek106.example.com                  
       http:
         paths:
           # http 경로가 [base_url]/echo-hostname 라면
         - path: /echo-hostname                     
           # service 이름이 hostname-service인 곳에 포트 80으로 포워드ㄴ
           backend:
             serviceName: hostname-service          
             servicePort: 80
   
   '''
   ```

3. 인그레스 컨트롤러 서버에 LB service 적용

   ```yaml
   vim ingress-service-lb.yaml
   '''
   kind: Service
   apiVersion: v1
   metadata:
     name: ingress-nginx
     namespace: ingress-nginx
   spec:
     # 디플로이먼트는 1개이지만 편의상 LB로 적용(NodePort도 무관)  
     type: LoadBalancer
     selector:
       # 서비스 적용 대상은 ingress-nginx라는 파드(디플로이먼트)
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
     ports:
         # 80, 443 포트를 오픈
       - name: http
         port: 80
         targetPort: http
       - name: https
         port: 443
         targetPort: https
   '''
   kubectl apply -f ingress-service-lb.yaml
   ```

4. 실제 웹 서비스를 담당할 deployment

   ```yaml
   vim hostname-deployment.yaml
   '''
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hostname-deployment
   spec:
     # 레플리카 수 : 3
     replicas: 3
     # 적용 대상은 app: webserver
     selector:
       matchLabels:
         app: webserver
     # 레플리카 생성 시 쿠버네티스가 참고하는 템플릿    
     template:
       metadata:
         name: my-webserver
         labels:
           app: webserver
       spec:
         containers:
         - name: my-webserver
           image: alicek106/ingress-annotation-test:0.0
           # 웹 서버를 위해 오픈하는 포트 : 5000
           ports:
           - containerPort: 5000
             name: flask-port
   '''
   kubectl apply -f hostname-deployment.yaml
   ```

5.  my-webserver에 적용할 서비스

   ```yaml
   vim hostname-service.yaml
   '''
   apiVersion: v1
   kind: Service
   metadata:
     name: hostname-service
   spec:
     # 80번 포트로 요청하면 파드의 5000 포트로 포워딩
     ports:
       - name: web-port
         port: 80
         targetPort: flask-port
     selector:
       app: webserver
     # 내부 통신만 필요하므로 CluterIP 사용
     type: ClusterIP
   '''
   kubectl apply -f hostname-service.yaml
   ```

6. 도메인 네임 연결 및 로컬 호스트에 추가

   ```
   # Load Balancer에 도메인 네임 설정
   curl --resolve alicek106.example.com:80:34.96.233.80 alicek106.example.com/echo-hostname
   # 로컬 호스트에 도메인 네임 추가
   vim /etc/hosts
   '''
   ~~~
   <ip> <domain name>
   '''
   ```

7. 결과

   ![image-20200618173917251](https://i.ibb.co/BcHdKG2/image-20200618173917251.png)

---



### Kubernetes를 이용해 Wordpress, Mysql 구축

1. Wordpress, MYSQL을 위한 Persistent Volume 생성

   ```yaml
   vim pv1.yaml
   '''
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv0001
     labels:
       type: local
   spec:
     capacity:
       storage: 25Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: "/data001/pv0001"
   '''
   kubectl create -f pv1.yaml
   vim pv2.yaml
   '''
   kind: PersistentVolume
   apiVersion: v1
   metadata:
     name: pv0002
     labels:
       type: local
   spec:
     capacity:
       storage: 25Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: "/data001/pv0002"
   '''
   kubectl create -f pv2.yaml
   ```

2. 연결을 위한 Persistent Volume Claim 생성

   ```yaml
   vim wordpress-volumeclaim.yaml
   '''
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: wordpress-volumeclaim
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 10Gi
   '''
   kubectl create -f wordpress-volumeclaim.yaml
   vim mysql-volumeclaim.yaml
   '''
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: mysql-volumeclaim
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 10Gi
   '''
   kubectl create -f mysql-volumeclaim.yaml
   ```

3. MYSQL을 위한 Service 생성(ClusterIP)

   ```yaml
   vim mysql-service.yaml
   '''
   apiVersion: v1
   kind: Service
   metadata:
     name: mysql
     labels:
       app: mysql
   spec:
     type: ClusterIP
     ports:
       - port: 3306
     selector:
       app: mysql
   '''
   kubectl create -f mysql-service.yaml
   ```

4. Wordpress를 위한 Service 생성(LoadBalancer)

   ```yaml
   vim wordpress-service.yaml
   '''
   apiVersion: v1
   kind: Service
   metadata:
     name: my-lb-service
     labels:
       app: wordpress
       name: wordpress
   spec:
     type: LoadBalancer
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
     selector:
       app: wordpress
   '''
   kubectl create -f wordpress-service.yaml
   ```

5. MYSQL 생성(Deployment)

   ```yaml
   vim mysql.yaml
   '''
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mysql
     labels:
       app: mysql
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mysql
     template:
       metadata:
         labels:
           app: mysql
       spec:
         containers:
           - image: mysql:5.6
             name: mysql
             env:
               - name: MYSQL_ROOT_PASSWORD
                 valueFrom:
                   secretKeyRef:
                     name: mysql-password
                     key: password
               - name: MYSQL_DATABASE # 구성할 database 명
                 value: k8sdb
               - name: MYSQL_USER # database 에 권한이 있는 user
                 value: k8suser
               - name: MYSQL_ROOT_HOST # 접근 호스트
                 value: '%'
               - name: MYSQL_PASSWORD # database 에 권한이 있는 user 의 패스워드
                 value: P@ssw0rd!!
             ports:
               - containerPort: 3306
                 name: mysql
             volumeMounts:
               - name: mysql-persistent-storage
                 mountPath: /var/lib/mysql
         volumes:
           - name: mysql-persistent-storage
             persistentVolumeClaim:
               claimName: mysql-volumeclaim
   '''
   kubectl create -f mysql.yaml
   ```

6. Wordpress 생성(Deployment)

   ```yaml
   vim wordpress.yaml
   '''
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: wordpress
     labels:
       app: wordpress
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: wordpress
     template:
       metadata:
         labels:
           app: wordpress
       spec:
         containers:
           - image: wordpress
             name: wordpress
             env:
               - name: WORDPRESS_DB_HOST
                 value: mysql:3306
               - name: WORDPRESS_DB_NAME
                 value: k8sdb
               - name: WORDPRESS_DB_USER
                 value: k8suser
               - name: WORDPRESS_DB_PASSWORD
                 value: P@ssw0rd!!
             ports:
               - containerPort: 80
                 name: wordpress
             volumeMounts:
               - name: wordpress-persistent-storage
                 mountPath: /var/www/html
         volumes:
           - name: wordpress-persistent-storage
             persistentVolumeClaim:
               claimName: wordpress-volumeclaim
   '''
   kubectl create -f wordpress.yaml
   ```

7. 결과

   ![image-20200623175721763](https://i.ibb.co/Jxy5dBR/image-20200623175721763.png)