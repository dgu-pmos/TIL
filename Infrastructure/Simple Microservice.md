# Simple Microservice

1. Architecture

   

2. making Kubernetes Engine(GCP)

   - 영역 : asia-east2-b
   - 마스터 버전(정적 버전) : 1.14.10-gke.36
   - 노드 운영체제 : cos
   - 머신 구성 : N1
   - 노드 수 : 3

3. making Storage(GCP)

   - 데이터 저장 위치 : multi-region (aisa)

   - 스토리지 클래스 : Standard

   - 엑세스 제어 방식 : 균일한 엑세스 제어

     <클러스터 연동까지 완료>

4. making & applying PV

   ```
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
       storage: 5Gi
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
       storage: 5Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: "/data001/pv0002"
   '''
   kubectl create -f pv2.yaml
   ```

   

5. making & applying PVC

   ```
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
         storage: 2Gi
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
         storage: 2Gi
   '''
   kubectl create -f mysql-volumeclaim.yaml
   ```

   

6. making & applying Services

   ```
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

   

7. making & applying Pods(Deployments)

   ```
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

   

8. developing Web Service

   ```
   
   ```

   

9. backup

   