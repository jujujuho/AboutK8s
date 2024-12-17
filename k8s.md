# K8s

## k8s란?

쿠버네티스(k8s)는 컨테이너화된 애플리케이션의 배포, 관리 및 확장을 예약, 자동화하기 위한 컨테이너 오케스트레이션 플랫폼이다.  
오늘날 많은 도커 컨테이너를 관리하기 위해 만들어짐.

### 기능

1. 배포  
2. 롤아웃  
3. 서비스 검색  
4. 스토리지 프로비저닝  
5. 로드 밸런싱  
6. 오토 스케일링  
7. 자가 치료  

---

## 워크로드 
- 클러스터에서 실행되는 작업 단위.
- 클러스터 안에 있는 파드의 집합내에서 실행됨. 
- 워크로드 리소스를 통해 관리가 편함.

### Pod
- Kubernetes에서 배포 가능한 가장 작은 컴퓨팅 단위.  
- 하나 이상의 컨테이너 그룹으로 구성되며, **스토리지와 네트워크를 공유**.  
- 파드는 항상 함께 배치되고 스케줄되며, 공유된 **컨텍스트**에서 실행.
- 파드는 삭제될 때가지 유지됨(Final Status).

#### 주요 특징

- **공유 리소스 제공**: Linux 네임스페이스와 컨트롤 그룹 등으로 격리.  
- 파드 내에서 **개별 애플리케이션의 하위 격리** 가능.  
- 보통 파드를 직접 생성하기보다는 **리소스를 통해 관리**.

### 워크로드 리소스

#### 레플리카셋(ReplicaSet)
- 레플리카 파드 집합의 실행을 안정적으로 유지시키는 것이 목표.
- 명시된 동일 파드 개수에 대한 가용성을 보증하는데 사용.
- 셀렉터, 레클리카의 개수, 파드 템플릿으로 구성됨
- 셀렉터는 획득 가능한 파드를 식별하는 방법이 명시됨
- 레플리카의 개수는 유지해야 하는 파드의 개수를 명시
- 파드 템플릿은 레플리카 수 유지를 위해 새로 생성되는 신규 파드의 데이터를 명시

##### 레플리카셋 매니페스트
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # 케이스에 따라 레플리카를 수정한다.
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```
- ApiVersion, kind, metadata 필드가 필요.
- kind값은 항상 ReplicaSet이다.
- 오브젝트 이름은 유효한 DNS 서브도메인 이름이어야함
- .spec 섹션 필요
- 템플릿은 .spec.template을 붙이고, 다른 컨트롤러의 셀렉터와 겹치지 않도록 주의.
- .spec.replicas를 설정해서 동시 동작 파드 수를 지정.(수를 맞추기 위해 파드 삭제, 생성함)
- .spec.replicas의 기본 값은 1이다.
##### 셀렉터
```
matchLabels:
  tier: frontend
```
##### 배포된 레플리카셋 확인
```
kubectl get rs
```
```
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       6s
```
##### 레플리카셋의 상태 확인
```
kubectl describe rs/frontend
```
```
Name:         frontend
Namespace:    default
Selector:     tier=frontend
Labels:       app=guestbook
              tier=frontend
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"annotations":{},"labels":{"app":"guestbook","tier":"frontend"},"name":"frontend",...
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   php-redis:
    Image:        gcr.io/google_samples/gb-frontend:v3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  117s  replicaset-controller  Created pod: frontend-wtsmm
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-b2zdv
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-vcmts
```
##### 유사한 파드 정보 확인
```
kubectl get pods
```
```
NAME             READY   STATUS    RESTARTS   AGE
frontend-b2zdv   1/1     Running   0          6m36s
frontend-vcmts   1/1     Running   0          6m36s
frontend-wtsmm   1/1     Running   0          6m36s
```
##### 실행중인 Pods Yaml 파일 확인
```
kubectl get pods (Pod's name) -o yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-02-12T07:06:16Z"
  generateName: frontend-
  labels:
    tier: frontend
  name: frontend-b2zdv
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: frontend
    uid: f391f6db-bb9b-4c09-ae74-6a1f77f3d5cf
...
```
##### 레플리카 셋을 참조하지 않는 파드를 가지는 법
- 레플리카를 배치하고 파드를 가져올 시, 레플리카 파드 3개와 참조하지 않는 파드 2개 생성됨
- 이렇게 되면, 레플리카 파드 3개가 Running이 되고, 참조하지 않는 파드는 terminating 상태가 됨
- 파드를 먼저 가져오고 레플리카를 배치할 시 참조하지않는 파드 2개와, 레플리카 파드 1개가 생성됨(Running)
##### 레플리카셋의 스케일링
- 스케일 다운할 때, 컨트롤러에서 스케일 다운하는 우선순위를 지정하기 위해 다음 기준으로 삭제할 파드를 지정
  1. Pending Status(스케줄링 X)
  2. controller.kubernetes.io/pod-deletion-cost 어노테이션이 설정 되어져 있는 파드에 대한, 설정 값이 낮은 파드가 먼저 다운
  3. 상대적으로 많은 래플리카가 있는 노드의 파드가 더 적은 레플리카가 있는 노드의 파드보다 먼저 스케일 다운됨
  4. 최근에 생성된 파드가 스케일 다운됨
  5. 만약 모든 조건이 같으면 임의로 선택됨
#### 디플로이먼트(Deployment)
- 파드와 레플리카셋에 대한 선언적 업데이트를 제공.
- 새 레플리카셋을 생성하거나, 기존 디플로이먼트를 제거하고, 모든 리소스에 새 디플로이먼트를 지정할 수 있음.

##### 디플로이먼트 생성
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
- .metadata.name 필드를 통해 nginx-deployment 이름을 가진 디플로이먼트가 생성
- .spec.replicas 필드에 따라 디플로이먼트는 3개의 레플리카 파드를 생성하는 래플리카셋을 만듬
- .spec.selector 필드는 생성된 레플리카셋이 관리할 파드를 찾아내는 법을 정의.
##### 디플로이먼트 확인 
```
kubectl get deployments
```
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     0            0           1s
```
- ``` name ```은 네임스페이스에 있는 디플로이먼트 이름의 목록이다
- ``` Ready ```는 사용자가 사용할 수 있는 애플리케이션의 레플리카의 수를 표시
- ``` UP-To-DATE ``` 의도한 상태를 얻기 위해 업데이트된 레플리카의 수를 표시
- ``` Available ```은 사용자가 사용할 수 있는 애플리케이션 레플리카의 수를 표시
- ``` Age ``` 는 애플리케이션의 실행된 시간을 표시

- ```kubectl get rs```를 통해, 디플로이먼트로 생성된 레플리카셋을 볼 수 있음
```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-75675f5897   3         3         3       18s
```
- ```kubectl describe deployment (deployment-name)``` 디플로이먼트 설명 명령어
```
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sun, 02 Sep 2018 18:17:55 -0500
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=4
                        kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.16.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-c4747d96c (3/3 replicas created)
Events:
  Type    Reason              Age   From                   Message
  ----    ------              ----  ----                   -------
  Normal  ScalingReplicaSet   12m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 1
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 2
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 2
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 1
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 3
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 0
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
  Normal  DeploymentRollback  15s   deployment-controller  Rolled back deployment "nginx-deployment" to revision 2
  Normal  ScalingReplicaSet   15s   deployment-controller  Scaled down replica set nginx-deployment-595696685f to 0
```
##### 디플로이먼트 업데이트
- ```kubectl set```을 통해, 디플로이먼트의 내용을 수정할 수 있음.
- ```kubectl rollout status```를 톨해 롤아웃 상태를 확인할 수 있음.
```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
```
- 롤아웃이 완료되면, ```kubectl get deployments```로 확인 가능
- ``` kubectl get rs ```, ``` kubectl get pods ```로 업데이트된 디플로이먼트의 레플리카셋과 새로 생성된 pods를 확인 할 수 있음

  ##### 디플로이먼트 롤오버
- 디플로이먼트 업데이트하는 중에, 또 다른 업데이트를 한다면, 생성된과 생성되고있는 노드를 다 죽이고, 새로 만듬

 ##### 디플로이먼트 롤백
- ```kubectl rollout history ?```을 통해 롤아웃 기록을 볼 수 있음.
```  
  
  deployments
  "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml
2           kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
3           kubectl set image deployment/nginx-deployment nginx=nginx:1.161
```
- ```kubectl rollout undo (Deployment name)```이전 수정 버전으로 롤백
- ```kubectl rollout undo (Deployment name) --to-revision=(revision num)``` 롤아웃 버전으로 롤백
---
## Cluster

클러스터는 컨테이너화된 애플리케이션을 실행하는 노드들의 집합임.

1. **컨트롤 플레인 (Master Node)**  
2. **노드 (Worker Node)**  

![image](https://github.com/user-attachments/assets/a6f9665a-b424-4650-a351-c433e0739f61)

# 컨트롤 플레인

컨트롤 플레인은 보통 1개에서 n개로 존재 (홀수개).  
컨트롤 플레인은 클러스터의 상태 관리와 명령어를 처리하는 역할을 함.  
`etcd`, `controller-manager`, `scheduler`, `kube api server`라는 여러 컴포넌트로 이루어짐.  

`kubectl`은 `kube api server`와 커뮤니케이션하는 역할을 함.  

마치 전통적인 3-tier 아키텍처 관계처럼 클라이언트 - 백엔드 서버 - 데이터 서버의 역할을 하는 방식으로  
마스터 노드가 돌아간다고 볼 수 있다.  

- 클라이언트의 요청을 처리하는 **API 서버**가 있고,  
- 그 API가 클러스터의 상태 데이터를 저장하는 데 활용하는 **DB**가 있음.  

(예: `scheduler`, `controller-manager`가 클라이언트, `kube api server`가 백엔드, `etcd`가 데이터 서버에 해당)

---

## 컴포넌트 분석

### **Scheduler**

- API 서버와 통신하는 컴포넌트로, 각각의 노드(Worker Node)의 자원 (CPU, Memory, GPU 등) 사용 상태를 관리.  
- 아직 노드가 배정되지 않은 새로 생성된 Pod를 감지하고, 새로운 워크로드를 띄우는 역할.  
- 클러스터 내 자원 할당이 가능한 노드 중 알맞은 노드를 선택하여 해당 노드에 워크로드를 배포 (Pod을 띄움).  

### **Controller-manager**

- 여러 컨트롤러 프로세스를 관리.  

#### 두 가지로 나뉨
1. **Kube 컨트롤러 매니저**  
2. **Cloud 컨트롤러 매니저**

- **Cloud 컨트롤러 매니저**:  
  대표적인 클라우드 공급자들 (AWS, GCP, AZURE)과 통신하며 각각의 클라우드 환경에 맞춘 컨트롤러를 관리.  
  공급자 종속적인 기능을 클러스터에서 수행.  

- **Kube 컨트롤러 매니저**:  
  다양한 API 리소스들 (Pod, Deployment, Service, Secret 등)을 관리하는 컨트롤러들로 구성.  
  API 서버를 주기적으로 확인하며, 현재 클러스터 상태와 etcd에 저장된 리소스의 상태를 비교.  
  만약 상태 변화가 발생하면 이를 클러스터 리소스에 반영하여 동기화.  
  이러한 반복적인 동기화 과정을 **Reconcile**이라고 부름.  

### **Kube API Server**

- 쿠버네티스 리소스와 클러스터의 상태 관리 및 동기화를 위한 API를 제공.  
- `etcd`를 데이터 저장소로 사용.  

### **etcd**

- 분산 Key-Value 저장소로 클러스터의 상태를 저장.  
- 클러스터 상태 백업 및 복구 시 사용.  
- 일반적으로 컨트롤 플레인 내부에 포함되지만, 컨트롤 플레인 밖에 따로 관리되기도 함.  

---

## 워커노드

애플리케이션이 실질적으로 실행되는 공간 (컨테이너가 띄워지는 공간)에 위치.  
노드는 최소 1개에서 n개로 구성됨.  
모든 노드 내부 구성은 동일.  

기본적으로 **Container Runtime** 위에서 Pod가 실행됨.  
그 외 System 컴포넌트로 **Kubelet**, **Kube-proxy**, **Network Add-On** 등이 돌아감.  

### **Kubelet**

- 모든 노드에 기본적으로 설치되는 컴포넌트.  
- API 서버와 통신하며 노드의 리소스 상태를 보고하고 관리.  
- 컨테이너 런타임과도 통신하며, 해당 노드 내에서 컨테이너 라이프사이클을 관리.  

### **CRI (Container Runtime Interface)**

- `Kubelet`이 다양한 컨테이너 런타임과 통신할 수 있도록 쿠버네티스에서 제공하는 인터페이스.  
- Docker뿐만 아니라 다양한 컨테이너 런타임을 지원하며 Docker 의존성을 줄이기 위한 목적.  

### **kube-proxy**

- 클러스터 상에서 오버레이 네트워크를 구성.  
- 네트워크 프록시 및 내부 로드밸런서 역할을 수행.

## Service

- Pod를 Ip주소와 파드 집합에게 단일 DNS를 부여해, 네트워크 서비스에 노출시켜줌.
### Service 
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
- my-service라는 새로운 서비스 오브젝트를 생성하고, app.kubernetes.io/name=MyApp 레이블을 가진 파드의 TCP 9376 포트들 대상으로 함
- 쿠버네티스는 이 서비스에 서비스 프록시가 사용하는 IP주소 (Cluster IP)를 할당함
- 서비스 셀렉터의 컨트롤러는 셀렉터와 일치하는 파드를 지속적으로 검색하고, "my-service"라는 엔드포인트 오브젝트에 대한 모든 업데이트를 POST함.
- 서비스는 모든 수신 Port를 targetport에 매핑할 수 있음. 기본적으로, targetport는 port필드와 같은 값으로 설정됨
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```
- 포트에 이름을 주어 service에 포트 이름으로 타겟 포트로 설정 가능(기본 TCP)
- 셀렉터 없이 정의를 하기 위해서는 엔드포인트슬라이스를 수동으로 추가하여 매핑해야한다.
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
```
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: my-service-1 # 관행적으로, 서비스의 이름을
                     # 엔드포인트슬라이스 이름의 접두어로 사용한다.
  labels:
    # "kubernetes.io/service-name" 레이블을 설정해야 한다.
    # 이 레이블의 값은 서비스의 이름과 일치하도록 지정한다.
    kubernetes.io/service-name: my-service
addressType: IPv4
ports:
  - name: '' # 9376 포트는 (IANA에 의해) 잘 알려진 포트로 할당되어 있지 않으므로
             # 이 칸은 비워 둔다.
    appProtocol: http
    protocol: TCP
    port: 9376
endpoints:
  - addresses:
      - "10.4.5.6" # 이 목록에 IP 주소를 기재할 때 순서는 상관하지 않는다.
      - "10.1.2.3"
```
- kubernetes.io/service-name을 통해 엔드포인트슬라이스를 서비스와 연결할 수 있음
- 엔드포인트는 최대 1000개의 엔드포인트를 가질 수 있다
- 만약 갯 수를 넘는다면, 1000개만 선정하여 어노테이션한다. 만약 떨어지면 어노테이션은 없어진다.
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```
- 쿠버네티스는 서비스에서 멀티포트를 제공한다.
- 그리고 만약 사용 시, 모든 포트 이름은 명확하게 지정해야한다.
- 포트 이름은 영숫자로 끝나야하고, 특수 기호는 - 밖에 사용이 불가하다.
- CIDR 범위 내의 유효한 IPv4또는 IPv6 주소로 .spec.clusterIP필드에 설정할 수 있다
- 이를 통해, 서비스 생성 요청시 고유한 클러스터 Ip주소를 지정할 수 있다.
- 만약 유효하지 않으면 422에러가 발생.
### 환경 변수
```
REDIS_PRIMARY_SERVICE_HOST=10.0.0.11
REDIS_PRIMARY_SERVICE_PORT=6379
REDIS_PRIMARY_PORT=tcp://10.0.0.11:6379
REDIS_PRIMARY_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_PRIMARY_PORT_6379_TCP_PROTO=tcp
REDIS_PRIMARY_PORT_6379_TCP_PORT=6379
REDIS_PRIMARY_PORT_6379_TCP_ADDR=10.0.0.11
```
- TCP 6379 개방, 클러스터 IP: 10.0.0.11 할당된 서비스 redis-primary를 생성 시, 위의 환경 변수를 만듬
- 서비스 접근이 필요한 파드나, 환경 변수를 사용하는 파드가 있을 경우, 서비스를 먼저 만들어야한다.

### DNS 
- 애드-온을 사용하여 쿠버네티스 클러스터의 DNS 서비스를 설정할 수 있다.
- CoreDNS와 같은, 클러스터-인식 DNS 서버는 새로운 서비스를 위해 쿠버네티스 API를 감시하고 각각에 대한 DNS 레코드 세트를 생성한다
- 클러스터 전체에서 DNS가 활성화된 경우 모든 파드는 DNS 이름으로 서비스를 자동으로 확인할 수 있어야 한다.

- 또한, DNS SRV 레코드를 지원
- TCP로 설정된 http라는 포트가 있는 경우, IP 주소와 http에 대한 포트 번호를 검색하기 위해
- ```_http._tcp.my-service.my-ns```에 대한 DNS SRV 쿼리를 수행할 수 있다.

- 쿠버네티스 DNS서버는 ExternalName 서비스에 접근할 수 있는 유일한 방법.\

### 서비스 퍼블리싱 Type 유형
- Cluster Ip: 서비스를 클러스터-내부 IP에 노출시킨다. 이 값을 선택하면 클러스터내에서만 서비스에 도달할 수 있다.
- NodePort : 고정 포트로 각 노드의 IP에 서비스를 노출시킨다. type: ClusterIp인 서비스를 요청했을 때와 마찬가지로 클러스터 Ip를 구성함
- LoadBalancer: 클라우드 공급자의 로드밸런서를 이용해, 서비스를 외부에 노출시킴
### NodePort 유형
-Type필드를 NodePort로 지정하게 되면 컨트롤 플레인에서 ```--service-node-port-range``` 플래그로 지정된 범위에서 포트를 할당한다.
-해제하기 위해서는 모든 노드 서비스에서 빼야한다.

### ExternalName
-spec.externalName에 도메인 이름을 설정하게 되면, 해당 서비스와 도메인이 연결된다.
- Origin서버가 인식 못하는 헤더가 있기 때문에, TLS서버는 클라이언트가 연결된 호스트 이름과 일치하는 인증서를 제공할 수 없음
## Ingress
- 클러스터 외부에서 클러스터 내부 서비스로 HTTP와 HTTPS 경로를 노출한다. 트래픽 라우팅은 인그레스 리소스에 정의된 규칙에 의해 컨트롤된다.
- 인그레스는 외부에서 서비스로 접속이 가능한 URL, 로드 밸런스 트래픽, SSL / TLS종료 그리고 이름-기반의 가상 호스팅을 제공하도록 구성된다.
- 인그레스 컨트롤러는 일반적으로 로드 밸런서를 사용해서 인그레스를 수행할 책임이 있으며, 트래픽을 처리하는데 도움이 되도록 에지 라우터, 프런트 엔드를 구성할 수 있다.
- 임의의 포트 또는 프로토콜을 노출시키지 않음. HTTP, HTTPS이외의 서비스를 인터넷에 노출시키려면 Service.Type=NodePort 또는 Service.Type=Loadbalancer유형의 서비스를 사용
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```
- 인그레스 컨트롤러가 있어야 인그레스를 충족할 수 있음. 리소스만 있으면 소용 X
- apiVersion, kind, metadata 및 spec 필드가 명시되어야 한다.
- 인그레스 오브젝트 이름은 유효한 DNS 서브도메인 이름이어야 한다.
- 어노테이션을 통해 다양한 컨트롤러를 제작할 수 있음
- ingressClassNamem을 생략하기 위해서는 ingressClass로 기본적인 인그레스 클래스가 정의 되어야 한다.
### 인그레스 규칙 
- 선택적 호스트: 지정된 IP주소를 통해 모든 인바운드 HTTP 트래픽에 규칙이 적용 된다. 
- 경로 목록: 각각 service.name과 service.port.name 또는 service.port.number가 정의되어 있는 관련 백엔드를 가지고 있다. 로드밸런서가 트래픽을 참조된 서비스로 보내기 전에 호스트와 경로가 모두 수신 요청의 내용과 일치해야함
- 백엔드는  서비스와 포트 이름의 조합. 규칙의 호스트와 경로가 일치하는 인그레스에 대한 HTTP(S) 요청은 백엔드 목록으로 전송
- 규칙이 없는 인그레스는 DefaultBackend를 통해 처리되고, rules에 명시되지 않으면, 반드시 DefaultBackend에는 명시되어야하는 것이다.
### 리소스 백엔드
- 인그레스 오브젝트와 동일한 네임스페이스 내에 있는 다른 쿠버네티스 리소스에 대한 ObjectRef이다. 리소스는 서비스와 상호 배타적인 설정이며, 둘 다 지정하면 유효성 검사에 실패한다.
- 리소스 백엔드의 일반적인 용도는 정적 자산이 있는 오브젝트 스토리지 백엔드로 데이터를 수신하는 것이다.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-backend
spec:
  defaultBackend:
    resource:
      apiGroup: k8s.example.com
      kind: StorageBucket
      name: static-assets
  rules:
    - http:
        paths:
          - path: /icons
            pathType: ImplementationSpecific
            backend:
              resource:
                apiGroup: k8s.example.com
                kind: StorageBucket
                name: icon-assets
```

- ```kubectl describe ingress *ingress-name)``` 인그레스 확인 명령어
### 경로 유형
- ```ImplementationSpecific```: 경로 유형의 일치 여부는 IngressClass에 따라 달라진다. 이를 구현할 때 별도 pathType으로 처리하거나, Prefix또는 Exact 경로 유형과 같이 동일하게 처리할 수 있다.
- ```Exact```: URL 경로의 대소문자를 엄격하게 일치시킨다.
- ```Prefix```: URL 경로의 접두사를 /를 기준으로 분리한 ㄱ밧과 일치시킨다. 일치는 대소문자 구분하고, 요소 별로 경로 요소에 대해 수행.

- 여러 경로가 요청과 일치할 때, 긴 일치하는 경로가 우선하게 되고, 여전히 동일하다면, Exact경로 유형을 가진 경로가 사용 된다.

## 호스트네임 와일드카드
- 호스는 정확한 일치 또는 와일드카드일 수 있다. 정확한 일치를 위해서는 HTTP host헤더가 host필드와 일치해야한다.
- 와일드카드 일치를 위해서는 HTTP host헤더가 와일드 카드 규칙의 접미사와 동일해야 한다.

## 인그레스 클래스
- 인그레스는 서로 다른 컨트롤러에 의해 구현될 수 있으며. 종종 다른 구성으로 구현될 수 있다. 각 인그레스에서는 클래스를 구현해야하는 컨트롤러 이름을 포함하여 추가 구성이 포함된 ingressclass 리소스에 대한 참조 클래스를 지정해야 한다.
```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: external-lb
spec:
  controller: example.com/ingress-controller
  parameters:
    apiGroup: k8s.example.com
    kind: IngressParameters
    name: external-lb

```
- .spec.parameters 필드를 사용해 해당 인그래스클래스와 연관있는 환경 설정을 제공하는 다른 리소스를 참조할 수 있음.
- 사용 가능한 파라미터의 상세한 타입은 인그레스클래스의 .spec.parameters 필드에 명시한 인그레스 컨트롤러의 종류에 따라 다르다.
- 특징 ingressClass를 클러스터의 기본값으로 표시할 수 있다.
- Ingressclass.kubernetes.io/is-default-class를 true로 설정하면 ingressclassname 필드가 저장되지 않은 채 기본 ingressClass로 할당된다.
### 범위
- 인그레스 클래스 파라미터의 기본 범위는 클러스터 범위이다.
- ```.spec.parameters``` 필드만 설정하고 ```.spec.parameters.scope```필드는 지정하지 않거나, ```.spec.parameters.scope```필드를 ```Cluster```로 지정하면 인그레스 클래스는 클러스터 범위의 리소스를 참조한다.
- 파라미터의 kind는 클러스터 범위의 APi를 참조하며, 파라미터의 name은 해당 API에 대한 특정 클러스터 범위 리소스를 가리킨다.


  ![image](https://github.com/user-attachments/assets/867d6763-2d06-41cd-a9d2-4f41bce3a9db)



## NameSpace
- 클러스터 내의 리소스 그룹을 격리하기 위한 매커니즘, 논리적 그룹
- 쿠버네티스 리소스는 하나의 네임스페이스에서만 있을 수 있다. 
- 리소스 할당량을 통해 클러스터 리소스를 여러 사용자간에 나눌 수 있다.
### 초기 네임스페이스
- Default: 이 네임스페이스가 기본적으로 포함되어있어, 네임스페이스를 만들지 않고 새 클러스터에 쓸 수 있다.
- kube-node-lease: 각 노드와 관련된 임대 객체를 보관, 노드 임대는 kubelet이 하드비트를 보내 컨트롤 플레인이 노드 장애를 감지할 수 있도록 한다.
- kube-public: 모든 클라이언트가 읽을 수 있으며, 대부분 클러스터 사용을 위해 에약되어있다.
- kube-system: 시스템이 만든 네임스페이스
### 명령어
```kubectl get namespace``` 클러스터의 현재 네임스페이스 보기
```
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```
- 요청에 대한 네임스페이스 설정. 현재 요청에 대한 네임스페이스 설정은 ```-namespace```플래그를 사용함.
```
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
```
- 네임스페이스 환경설정. 컨텍스트에서 모든 후속 kubectl 명령에 대한 네임스페이스를 영구적으로 저장.
```
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# Validate it
kubectl config view --minify | grep namespace:
```
## StatefulSet
- 스테이트풀을 관리하기 위해 사용되는 워크로드 API 오브젝트이다.
- 디플로이먼트와 스케일링을 관리, 파드들의 순서와 고유성을 보장함.
- 디플로이먼트와 유사하게 컨테이너 스펙을 기반으로 한 파드들을 관리함. 교체는 불가.
- > 재스케줄링 간에도 지속적으로 유지되는 식별자를 가짐

- 파드에 지정된 스토리지는 관리자에 의해 PVP를 기반으로 하는 스토리지 클래스를 요청해서 프로비전이 되어야 한다.
- 스테이트풀셋을 삭제, 스케일 다운해도 관련된 불륨은 삭제되지 않는다.
- 현재 파드의 네트워크 신원을 책임지고 있는 헤드리스 서비스가 필요하다
- 스테이트풀셋을 삭제할 시, 스케일을 0으로 지정하고 삭제해야한다. 이 후에 파드의 종료는 스테이트풀셋이 책임지지 않음.
- 롤링 업데이트, 파드 매니지먼트 플래시를 함께 사용시 복구를 위한 수동 개입이 필요한 파손 상태로 빠질 수 있음.

### 구성요소
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # .spec.template.metadata.labels 와 일치해야 한다
  serviceName: "nginx"
  replicas: 3 # 기본값은 1
  minReadySeconds: 10 # 기본값은 0
  template:
    metadata:
      labels:
        app: nginx # .spec.selector.matchLabels 와 일치해야 한다
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```
- 이름이 nginx라는 헤드리스 서비스는 네트워크 도메인을 컨트롤하는데 사용된다.
- 이름이 web인 스테이트풀셋이 3개의
- 파드 .spec.selector 필드는 .spec.template.metadata.labels 레이블과 일치하도록 설정해야함
  > 해당되는 파드 셀렉터를 찾지 못하면 스테이트풀셋 생성 과정에서 검증 오류가 발생한다.
- volumeClaimTemplates은 퍼시스턴트 불륨 프로비저너에서 프로비전한 퍼시스턴트 불륨을 사용해서 안정적인 스토리지를 제공함.
- .spec.volumeClaimTemplates를 설정하여, PVP에 의해 프로비전된 PV을 이용하는 안정적인 스토리지를 제공할 수 있다.
- .spec.minReadySeconds는 파드가 사용가능이라고 간주될 수 있도록 파드의 모든 컨테이너가 문제 없이 실행되고 준비되는 최소 시간을 나타내는 선택적인 필드이다. ( 기본 값: 0 ) 

### 순서 색인 
- N개의 레플리카가 있는 스테이트풀셋은 각 파드에 0에서 N-1 까지의 정수가 순서대로 할당 되며, 해당 스테이트풀셋에서 고유하다.
### 시작 순서
- .spec.ordinals은 각 파드에 할당할 순서에 대한 정수값을 설정할 수 있게 해주는 선택적인 필드이다.
- 기본 값은 nil이며, StateFulsetStartOrdinal 기능 게이트를 활성화 해야함
- 활성화시: .spec.ordinals.start -> .spec.ordinals.start + .spec.replicas - 1 순서대로 할당된다.
### 네트워크 신원 
- 스테이트풀셋의 각 파드는 이름을 가진다. $(statefulset.name)-$(ordinal)
- 파드의 도메인을 제어하기 워해, 헤드리스 서비스를 사용할 수 있다. $(service name).$(namespace).svc.cluster.local
- cluster.local은 클러스터 도메인.
- 각 파드는 생성되면 $(podname).$(governing service domain) 형식을 가지고 일치되는 DNS 서브 도메인을 가진다
  -> 여기서 거비닝 서비스는 스테이트풀셋의 serviceName 필드에 의해 정의된다.
- 즉시 검색을 하기 위해서는 DNS 조회에 의존하지 않고 쿠버네티스 API를 직접 쿼리하거나
- 쿠버네티스 DNS공급자의 캐싱시간을 줄여야 한다.

### 안정된 스토리지
- Request를 통해 위의 구성요소는 my-storage-class라는 이름의 1 Gib 프로비전 스토리지를 가지는 단일 퍼시스턴트 불륨을 받게 된다.
- 스케줄 혹은 재스케출이 되면 파드의 volumeMounts는 PVC와 관련된 PV가 마운트 된다. 파드나 스테이트풀셋이 삭제되도 PV와 PVC는 삭제 되지 않는다.
- 스테이트풀셋 컨트롤러가 파드를 생성할 때 파드 이름으로 ```statefulset.kubernetes.io/pod-name```레이블이 추가된다.
## 디플로이먼트와 스케일링 보증 
- 파드가 삭제될 때는 역순으로 종료된다.
- 스케줄링 작업을 적용하기 이전에 모든 선행 파드가 Running혹은 Ready 상태여야함
- 파드가 종료되기 전에 모든 후속 파드가 완전히 종료되어야 한다.
## 업데이트 전략
- 최대 사용 불가능 파드 수를 정해, 업데이트 과정에서 사용 불가한 파드를 몇 개까지 허용할 것인지 설정할 수 있다. 절대값 또는 퍼센티지로 설정할 수 있다 ( 기본 값: 1 )
### OnDelete(삭제)
- 스테이트풀셋의 ```.spec.updateStrategy.type```은 OnDelete를 설정하며, 스테이트풀셋 컨트롤러는 스테이트풀셋의 파드를 자동으로 업데이트 하지 않는다.
- 수동으로 파드를 삭제해야한다.
### RollingUpdate
- 스테이트풀셋의 파드에 대한 롤링 업데이트를 구현함
- 각 파드를 순차적으로 종료가 한다.
### 강제 롤백
- 파드 템플릿을 Running이나 Ready 상태가 되지 않는 구성으로 업데이트하면, 스테이트풀셋은 롤아웃을 중지하고 기다린다.
- 템플릿을 되둘린 후에는 잘못된 구성으로 시도된 파드들을 모두 삭제해야한다.

## PVC 유보
- 선택 필드 ```.spec.persistentVolumeClaimRetentionPolicy```는 스테이트풀셋의 생애주기동안 PVC를 삭제할 것인지, 어떻게 삭제할 것인지를 관리
- API서버와 컨트롤러 매니저에 StatefulSetAutoDeletePVC 기능 게이트 를 활성화해야한다.
### whenDeleted
- 스테이트풀셋이 삭제될 때 적용될 불륨 유보 동작을 설정
### whenScaled
- 스테이트풀셋의 레플리카 수가 줄어들 때
### Delete
- PVC는 정책에 영향을 받는 각 파드에 대해 삭제
- whenDeleted가 이 값으로 설정되어 있으면 volumeClaimTemplated으로부터 생성된 모든 PVC는 파드가 삭제된 뒤에 삭제된다.
- whenScaled가 이 값으로 설정되어 있으면 스케일 다운된 파드 레플리카가 삭제된 뒤, 삭제된 파드에 해당하는 PVC만 삭제된다.
### Retain(Default)
- 파드가 삭제되어도 volumeClaimTemplate으로부터 생성된 PVC는 영향을 받지 않는다.
```
apiVersion: apps/v1
kind: StatefulSet
...
spec:
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain
    whenScaled: Delete
...
```
- HorizontalPodAutoScaler가 디플로이먼트 크기를 관리하고 있다면, 레플리카의 수를 지정해서는 안된다.
- 컨트롤 플레인이 자동으로 관리하기 때문

## 데몬셋
- 모든(일부) 노드가 파드의 사본을 실행하도록 함.
