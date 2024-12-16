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
