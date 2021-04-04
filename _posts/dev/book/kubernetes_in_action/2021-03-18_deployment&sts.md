---
title: "[Kubernetes in action] chapter 9 & 10"

categories:
  - dev
  - book
tags:
  - dev
  - book
  - kubernetes in action
---

---
# chapter 9. Deployment
---

# 서론
- 파드를 최신 버전으로 교체
- 관리되는 파드 업데이트
- 이플로이먼트 리소스로 파드의 선언적 업데이트
- 롤링 업데이트 수행
- 잘못된 버전의 롤아웃 자동 차단
- 롤아웃 속도 제어
- 이전 버전으로 파드 되돌리기

---

애플리케이션은 언제나 변경될 수 있다. 

이번 장에서는 쿠버네티스트에서 실행되는 애플리케이션을 업데이트하는 방법과 무중단 업데이트를 어떻게 하는지 설명할 것이다.

---

# 파드에서 실행중인 애플리케이션 업데이트

기본 버전 v1을 태그로 가지는 애플리케이션을 외부에서 접근할 수 있다. 그런 상태에서 우리는 새로운 버전인 v2를 배포하고 싶다. 그러기 위해서는 우리는 기존 파드를 삭제하고 새로운 파드를 실행해야한다. 두가지 방법이 있다고 할 수 있다.

1. 기존 파드를 한번에 모두 삭제한 다음 새 파드를 실행
   - 일시적으로 서비스 접근이 불가능하다. 
1. 새로운 파드를 실행하고 기존 파드를 삭제. 새로운 파드를 모두 띄우고 삭제하거나 그 중간 중간에 삭제와 배포를 같이 한다.
   - 두가지 버전이 동시에 실행되어야 하기 때문에 서비스가 같은 기능을 제공하지 못할수도 있으며 데이터 저장소 같은 경우 새 버전이 이전 버전을 손상 시킬 수 있어서 데이터 스키마나 데이터 수정을 해서는 안된다.

---

# 롤링 업데이트

---

## 과정
![rolling](https://drek4537l1klr.cloudfront.net/luksa/Figures/09fig04_alt.jpg)

---

## 레플리케이션컨트롤러와 kubectl을 이용한 rolling update

이전에 hoon:v1이라는 이미지를 활용해서 컨테이너를 만들다고 가정할 때.

새로운 버전인 hoon:v2를 배포하고 싶다.

``` bash
$ k rolling-update <이전버전 이름> <새로운버전 이름> <업데이트 내용>

$ k rolling-update hoon-v1 hoon-v2 --image=hoon:v2
```

---

## 지금은 사용하지 않는 이유
1. 운영자가 아닌 쿠버네티스가 자원에 대해서 수정을 진행. (쿠버네티스 내부적으로 라벨을 변경함.)
2. k8s client가 이 과정을 진행. 네트워크 이슈로 클라이언트와 연결이 끊어지면 큰일.
3. 명령을 나타난다. 쿠너베티스는 상태를 선언하고 그것을 달성하는 것이 베스트라고 생각하기에.

---

# deployment

애플리케이션을 배포하고 선언적으로 업데이트하기 위한 높은 수준의 리소스 <-> 레플리케이션컨트롤러 또는 레플리카셋과 같은 낮은 수준의 개념

![deployment](https://drek4537l1klr.cloudfront.net/luksa/Figures/09fig08.jpg)

```
deployment -> 레플리카셋 -> pods
```

---

## 형식

``` yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: kubia
spec:
  replicas: 3
  template:
    metadata:
      name: kubia
      labels:
        app: kubia
    spec:
      containers:
      - image: luksa/kubia:v1
        name: nodejs
```
레플리케이션컨트롤러를 만드는 것과 다르지 않다. 

---

## 파드 생성 방식 이해
이름에 숫자가 들어감. 그 값은 디플로이먼트와 파드 텟플릿의 해시값. 그것을 통해서 해당 리소스 이름과 합쳐져서 레플리카셋 이름이 만들어짐.

추후 여러 리플리카셋이 나오게 되는데 이런 것을 통해서 어떤 레플리카셋의 파드인지 알 수 있음. 그리고 관리됨. -> 지정한 버전에 대한 해시값을 통해서 특정 버전 실행 가능

---

``` bash
$ kubectl get po
NAME                     READY     STATUS    RESTARTS   AGE
kubia-1506449474-otnnh   1/1       Running   0          14s
kubia-1506449474-vmn7s   1/1       Running   0          14s
kubia-1506449474-xis6m   1/1       Running   0          14s

---
$ kubectl get replicasets
NAME               DESIRED   CURRENT   AGE
kubia-1506449474   3         3         10s
```

---

# deployment를 통한 업데이트

## 왜 사용?
레플리카셋, 컨트롤러로도 충분히 배포는 가능하다고 할 수 있다.

---

## 레플리케이션 컨트롤러 업데이트 
- 레플리케이션컨트롤러은 명시적으로 하라면 **명령도 내리고** 새로운 레컨 이름도 지정해줘야했음. 
- 업데이트가 끝나면 **이전 레플리케이션컨트롤러을 삭제함**. 
- 터미널을 열어두고 끝날 때까지 기달려야 했음.

---

## 업데이트 전략
- (default) RollingUpdate: 이전에 보여준 것처럼 하나씩 하나씩. 이전 버전과 현재 버전이 동시에 동작하는 것이 가능한 경우.
- Recreate: 한번에 모두 삭제한 다음 새로운 파드를 만듬. 이전 버전을 완전히 싸그리 없애버리고 새로 만듬. **짧은 다운타임이 생김**

---

### Pod deployment trigger

- 파드 템플릿을 변경하지 않은 경우에 디플로이먼트는 배포를 하지 않는다.

Pod에 대한 정보가 바뀐 것이 아니기 때문에 image나 다른 정보가 바뀌는 경우에는 배포가 진행됨.

- 파드 템플릿이 컨피그맵(또는 시크릿)을 참고하고 있고 해당 컨피그맵이 업데이트된다고해서 배포가 되지는 않는다. 실제 configmap은 적용됨.

#### 방법은? 
  1. 새로운 configmap을 만들어서 적용
  2. [stakater/Reloader](https://github.com/stakater/Reloader)

---


## 업데이트 과정
``` bash
$ k set image deployment kubia nodejs=luksa/kubia:v3
deployment "kubia" image updated
```
- Pod template 정보 변경 -> deployment trigger

과정은 전에 설명한 레플리케이션컨트롤러 롤링 업데이트 과정에서 발생하는 이벤트와 매우 유사. 하지만 우리가 어떤 다른 명령어를 하지 않아도 된다.

그리고 추가적으로 이전 버전에 대한 레플리카셋을 그냥 나둔다. (k rolling-update는 삭제해버림)

---

## 롤백
배포 이후에 문제가 있다고 판단되는 경우 이전 버전으로 롤백할 수가 있다.

``` bash
$ k rollout undo deployment <deployment name> // 직전 버전으로 롤백

$ $ k rollout history deployment kubia // 이력 확인 => 이력을 보기 위해서 --record를 통해서 배포를 진행해야 됨

$ k rollout undo deployment kubia --to-revision=1 // 특정 버전으로 롤백
```

---

## 롤아웃 속도 제어
``` yaml
spec:
  replicas: 3
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
```

---

### maxSurge
해당 숫자만큼 추가적으로 인스턴스를 가질 수 있다. (replicas + maxSurge)

`replicas: 3, maxSurge: 1` 이라고 과정했을 때 롤링 업데이트 과정에서 최대 4개까지의 인스턴스가 동작할 수 있다.

### maxUnavailable
해당 숫자를 뺀 나머지 인스턴스는 동작하는 것을 보장해야 된다. (replicas - maxUnavailable)

`replicas: 3, maxUnavailable: 1` 이라고 과정했을 때 롤링 업데이트 과정에서 최대 2개의 인스턴스는 동작을 보장해야된다.

---

### 롤아웃 프로세스 일시 정지

``` bash
$ k set image deployment kubia nodejs=luksa/kubia:v4

$ k rollout pause deployment kubia

$ k rollout resume deploymnet kubia
```

---

- 카나리 배포

변경을 하자마자 deployment는 롤아웃을 진행할 것이고 그러면서 생긴 **새로운 버전에 v4 파드**가 만들어질 것이다. 

그 과정에서 **중간에 멈추게 되면 해당 v4 파드에 트래픽**이 들어가게 할 수 있다. 이런 방식으로 카나리 배포가 가능함. 

[kubectl rollout document](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#rollout)

---

### 잘못된 버전의 롤아웃 방지

- `minReadySecond`
파드를 사용 가능한 것으로 취급하기 전에 새로 만든 파드에 준비 시간을 줌.

`minReadySecond`가 지나기 전에 **새 파드가 제대로 작동하지 않고** 레디니스 프로브가 실패하기 시작하면 롤아웃이 효과적으로 차단된다.

실제 트래픽을 수신하기 시작한 후 파드가 준비 상태를 계속 보고할 수 있도록 `minReadySecond`를 널널하게 설정해야 된다.

---

``` yaml
spec:
  minReadySecond: 10
  ...
  template:
    ...
    spec: 
      containers:
      - image: kubia:v3
        name: kubia
        readinessProbe: 
          periodSeconds: 5
            httpGet:
              path: /
              port: 8080
```

> 주의사항 

만약 `minReadySecond`은 설정하지 않고 `readinessProbe`만 설정하는 경우 **readinessProbe**의 첫번째가 성공하면 배포가 가능한 것으로 판단해서 나머지도 모두 배포하게 된다. 

그러니 올바르게 적절한 값으로 설정하는 것을 추천한다.

---

### 롤아웃 데드라인

- `ProgressDeadlineSeconds`

``` yaml
spec:
  progressDeadlineSeconds: 600 // (단위는 초)
```

---

> 만약 데드라인을 넘어서면 디플로이먼트 컨트롤러는 디플로이먼트의 .status.conditions 속성에 다음의 디플로이먼트 컨디션(DeploymentCondition)을 추가한다.

``` text
Type=Progressing
Status=False
Reason=ProgressDeadlineExceeded
```

> 쿠버네티스는 Reason=ProgressDeadlineExceeded 과 같은 상태 조건을 보고하는 것 이외에 정지된 디플로이먼트에 대해 조치를 취하지 않는다. 더 높은 수준의 오케스트레이터는 이를 활용할 수 있으며, 예를 들어 디플로이먼트를 이전 버전으로 롤백할 수 있다.

[deployment 실패](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/#%EB%94%94%ED%94%8C%EB%A1%9C%EC%9D%B4%EB%A8%BC%ED%8A%B8-%EC%8B%A4%ED%8C%A8)

---

# chapter 10. statefulset

---

# 서론
- 클러스터된 스테이트풀 애플리케이션 배포
- 파드 레플리카 인스턴스에 별도의 스토리지 제공
- 파드 레플리카에 안정적인 이름과 호스트 이름 보장
- 예측 가능한 순서대로 파드 레플리카의 시작과 중지
- DNS 서비스 레코드를 통한 피어 디스커버리

---

# 스테이트풀 파드 복제
## 레플리카셋은?
레플리카셋은 이름과 ip 주소를 제외하고 모두 똑같은 파드를 만듬. (똑같은 파드 == 내용물이 똑같음)

그리고 모두 똑같은 볼륨을 사용함 => 다른 퍼시스턴트볼륨클레임은 사용 불가

![replicaSet](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig01_alt.jpg)

---

## 레플리카에 개별 스토리지 붙이기 (스토리지에 대한 안정적인 제공에 대한 고민)
- 수동으로 하나하나 pod를 생성
  - 나중에 너무 귀찮음. 죽으면 계속 살려야되는데 말이 안됨.

- 파드 인스턴스별로 하나의 레플리카셋 사용
  - 단일 레플리카를 사용하는 것에 비해 너무 불편함.
  - 하나 배포한다고 해도 각 레플리카에 대해서 배포를 직접 해야됨
  - 스케일도 귀찮음

![eachpod](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig02.jpg)

---

- 동일 볼륨을 여러 개 디렉토리로 분할 사용
  - 하나의 퍼시스턴스 볼륨 + 각 파트들이 다른 디렉토리 공간 지정해서 사용
  - 인스턴스가 실행되기 전에는 어차피 모두 같이 설정을 가지고 있기 때문에 하지 못함.
  - 인스턴스가 생성된 이후에 사용하지 않는 디렉토리를 선택할 수는 있도록 가능.
  - 하지만 잘 동작하는 것에 대한 보장이 안되고 볼륨 병목 현상 발생

![eachPodSameVolume](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig03_alt.jpg)

---

## 각 파드에 안정적인 아이덴티티 제공
새로운 파드를 교체해야되는 경우 (죽던지 혹은 배포를 했던지) 레플리카셋같은 **다른 호스트 네임과 IP**를 제공한다.

특정 애플리케이션에서 이전 인스턴스 데이터를 가지고 시작할 때 네트워크 아이덴티티로 인한 문제가 발생한다. 

간단한 예시로 클러스터 멤버마다 설정파일을 제공해주는 경우가 있는데 -> (?)

Pod이 종료되어 다시 올라오는 경우가 있을 때 지정된 ip가 바뀌어서 새로 셋업을 해야된다.

---

### 각 파드 인스턴스별 전용 서비스 사용

#### 각 Pod마다 서비스를 제공해보자!
- 서비스같은 경우 **변경되지 않는 ip**를 제공
- 파드별로 서비스를 가지고 있으면 새로운 파드가 만들어 진다고해도 문제가 되지 않지 않을까?
- 할수는 있겠지만 각 파드가 어떤 서비스에 매핑되는지 알기 어렵기 때문에 그 IP를 사용해 **다른 파드에 자신을 등록할 수 없다.** (?)

![eachService](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig04.jpg)

---

# 스테이트풀셋

## 스테이트풀셋과 레플리카셋 비교

### 레플리카셋 (가축)
- 파드끼리 내부 구성이 거의 동일함
- 이전 파드와 새로운 파드가 식별만 다르지 동작은 그래도.
- 대부분 stateless 언제든지 완전히 새로운 파드로 교체가능.

---

### 스테이트풀셋 (애완동물)
- 새로운 파드 인스턴스에 대해서 **이전 파드와 동일한 이름, 네트워크 아이덴티티, 상태**까지 모두 동일
- 레플리카를 줄인다고 해서 스토리지가 없어지지 않고 계속 남아있다가 레플리카를 다시 늘렸을 때 같이 매핑됨. -> 이전 상태에 대해서 계속 보관
- 완전히 무작위하게 생성되는게 아니고 규칙을 가지고 있음.

---

## 안정적인 네트워크 아이덴티티 제공
- 인덱스를 이용해서 파드의 이름과 호스트 이름, 스토리지 이름을 붙이는데 사용 (ex. hoonPod-0, hoonPod-1)

![comparingReplicaAndStateful](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig05_alt.jpg)

---

### 거버닝 서비스 
- 인스턴스에 대해서 동일한 이름을 제공하지만 `호스트 아름`를 통해서 접근을 해야되는 경우가 있음.
- 그 뿐만 아니라 스테이트리스 인스턴스들은 모두 상태가 없기 때문에 그 중에 아무나 선택하면 됨.
- 하지만 스테이트풀 파드들은 각자 **상태**가 다르기 때문에 **특별하게 인식할 필요**가 있다.

---

####  So, 그래서 거버닝 헤드리스 서비스를 생성해서 **호스트에 대한 네트워크 아이덴티티**를 제공
- 각 파드는 자체 DNS 엔트리를 가짐.
- 클러스터 내/외부에 있는 클라이언트는 특정 파드의 주소를 지정할 수 있음.

- 실제 예시
```
namespace: default
service name (거버닝 서비스): foo
pod name: a-0

FQDN: a-0.foo.default.svc.cluster.local
```

---

- `foo.default.svc.cluster.local` 라는 거비닝 서비스 도메인의 SRV레코드를 조회해서 **스테이트풀셋의 파드 이름**을 찾을 수도 있음.

- 특정 노드에 문제가 발생하는 경우
![comparing_2](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig06_alt.jpg)
ㅍ
---

### 스테이트풀셋 스케일링
만약 2개의 레플리카 파드가 있다가 과정했을 때 새로운 파드에 이름에는 다음 인덱스인 2가 붙여서 생성될 것이다. 

ex) a-0, a-1, **a-2 (New)**

또한 삭제될 때는 그 순서에 반대대로 삭제된다.

ex) a-0, a-1, a-2 (will be removed), **a-3 (is removed)**

스케일 다운시 좋은 점은 어떤 파드가 제거될지 알 수 있다는 점이다. <-> 레플리카셋: 어떤 것이 먼저, 그리고 지정해서 스케일 다운하지 못함.

![scailing](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig07_alt.jpg)

---

- 특정 스테이트풀 애플리케이션 같은 경우 빠르게 스케일 다운이 처리하기 어려워 **한 시점의 하나의 파드 인스턴스만 스케일 다운**함

예) 분산 데이터 스토어 -> 사실 내용을 이해하지 못함...

---

## 각 스테이트풀 인트선스에 안정적인 스토리지 제공
**안정적인 아이덴티티를 갖도록 보장**

이전 파드가 사라지고 새로 만들어진 파드에 대해서도 이전 스토리지을 똑같이 할당해주어야 함.

---

- 그러기 위해서 스테이트풀셋은?

스토리지는 영구적이어야하지만 파드와는 분리되어야한다.

=> Pod에 내부에 있다면 분명히 Pod가 스케줄링 되는 단계에서 데이터가 손실될 가능성이 있음.

---

### 볼륨 클레임 템플릿

팟을 만들 때에 대한 템플릿을 만드는 것과 비슷하게 **볼륨 클레임을 템플릿** 통해서 만듬.

6장에서 배운 것처럼 퍼시스턴트볼륨클레임을 이용해서 볼륨과는 1:1 매핑이 진행되고 

파드와 퍼시스턴스볼륨 클레임을 1:1 매핑해서 각자 스토리지를 가지도록 한다.

![볼륨클레임템플릿](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig08_alt.jpg)

> 이 가정에는 관리자가 프로비저닝한다거나 혹은 **동적 프로비저닝**이 지원한다고 가정

---

![새로운파드볼륨](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig09_alt.jpg)

- 레플리카를 줄여서 스케일 다운을 하게 되는 경우 Pod만 삭제된다. **PVC는 남는다.**
- 다시 스케일 업하게 되는 경우 Pod이 기존에 있던 PVC를 이용해서 볼륨을 사용

---

> 주의: 스케일 다운을 하고 남은 볼륨이 중요한 데이터라면 직접 관리하는 것이 중요하다. (퍼시스턴트볼륨클레임을 직접 삭제)

PVC를 그냥 삭제하게 된다면 정책에 따라서 PV이 삭제되거나 재활용될 것이다.

재활용이 된다고 했을 때 다시 스케일 업을 하게 되는 경우 데이터가 손상될 수 있다.

---

# 스테이트풀셋 사용

---

## format
``` yaml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: kubia
spec:
  serviceName: kubia
  replicas: 2
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia-pet
        ports:
        - name: http
          containerPort: 8080
        volumeMounts:
        - name: data
          mountPath: /var/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      resources:
        requests:
          storage: 1Mi
      accessModes:
      - ReadWriteOnce
```

---

- 배포 과정

`deployment`와 다르게 `sts`는 파드가 모두 완벽하게 뜬 이후에 다음 파드를 만든다.

여러 개를 동시에 만들다가 `race condition`을 발생할 가능성이 있기 때문이다.

---

### 실행 이후 모습
- Pod
``` yaml
$ kubectl get po kubia-0 -o yaml

---
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
  - image: luksa/kubia-pet
    ...
    volumeMounts:
    - mountPath: /var/data
      name: data
      ...
  ...
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data-kubia-0
  ...
```
---

- PVC
``` bash
$ kubectl get pvc
NAME           STATUS    VOLUME    CAPACITY   ACCESSMODES   AGE
data-kubia-0   Bound     pv-c      0                        37s
data-kubia-1   Bound     pv-a      0                        37s
```

---

# 스테이트풀셋의 피어 디스커버리
클러스터된 애플리케이션의 중요한 요구 사항으로 `피어 디스커버리` (클러스터된 다른 멤버를 찾는 기능)이라고 합니다.

쿠버네티스 API 서버에 요청을 해서 알아낼 수 있지만 그것은 애플리케이션에 독립적이지 못하기 때문에 좋지 않은 방법이다.

우리는 DNS에 SRV 레코드를 통해서 기능을 제공할 것이다.

---

## 헤드레스트 서비스
``` yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  clusterIP: None
  selector:
    app: kubia
  ports:
  - name: http
    port: 80
```

---

## SRV 레코드
- 특정 서비스를 제공하는 서버의 호스트 이름과 포트를 가리키는데 사용.
- 쿠버네티스는 헤드리스 서비스를 뒷밤침하는 파드의 호스트 이름을 가리키도록 SRV 레코드를 생성.

---

- dig (DNS lookup 도구)을 이용한 테스트
``` bash
dig SRV kubia.default.svc.cluster.local

...
;; ANSWER SECTION: // <- 헤드리스트 서비스를 뒷받침하는 파드의 SRV 레코드
k.d.s.c.l. 30 IN  SRV     10 33 0 kubia-0.kubia.default.svc.cluster.local.
k.d.s.c.l. 30 IN  SRV     10 33 0 kubia-1.kubia.default.svc.cluster.local.

;; ADDITIONAL SECTION: // <- A 레코드
kubia-0.kubia.default.svc.cluster.local. 30 IN A 172.17.0.4
kubia-1.kubia.default.svc.cluster.local. 30 IN A 172.17.0.6
...
```

위의 모습처럼 파드가 스테이트풀셋의 모든 파드 목록을 가져오려면 SRV DNS 룩업을 수행하기만 하면 된다.

---

## 피어 디스커버리를 통한 활용

---

### DNS를 통한 피어 디스커버리
쿠버네티스 서버는 모든 파드들에 대한 정보를 가지고 있다. 

하지만 클라이언트는 알기 위해서 API 서버에 요청을 해야되는데 너무 번거롭고 시간도 오래 걸린다는 문제

그런 상황에서 스테이트풀셋과 SRV 레코드를 사용하면 편하게 해결 가능하다.

![dnsLookUp](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig12_alt.jpg)

---

# 스테이트풀셋 노드 실패를 처리하는 과정

## 스테이트풀셋의 최대 하나 (p.448)

`최대 하나의 의미`: 스테이트풀셋은 동일한 아이덴티티를 가지는 파드를 가지는 것을 절대적으로 막는다

예) 쿠버네티스가 파드 상태를 확신할 수 없을때? 스테이트풀셋이 교체 파드를 생성하게 되면 동일한 아이텐티티를 가진 파드가 생긴고 동일한 볼륨에 데이터를 저장할 것이다.

그런 것을 막겠다! 두 개의 아이텐디디로 실행되지 않도록 동일한 PVC에 바인딩 되지 않도록 보장 -> 절대적인 확신이 있어야지만 교체한다.

---

## 상황

> 가정: 노드에 네트워크가 끊어져서 쿠버네티스가 해당 노드 내부의 정보를 얻지 못할 경우

- 해당 `Node` status -> NotReady
- 해당 `Node` 안에 `Pod` status -> Unknown

### Unknown 상태에 Pod은?
- 노드에 다시 연결이 되면 Running이 되겠지만 특정 시간(설정 가능)이 지나게 되면 노드에서 자동으로 삭제됨.
- `Kubelet`이 deletion을 확인하면 파드를 종료시킴
  - But, 현재 상황에서는 `Kubelet`에게 명령을 못내림.
  - Pod는 살아있는 상황이다.
  - 시간이 지나면 `Terminating`이라는 상태가 되지만 실제로 삭제는 안된다.

---

## 수동으로 삭제하기
``` bash
$ k delte po kubia-0
```
- 실제로 삭제되지 않음.
- 아까 상황에서 본 것처럼 네트워크 연결이 끊어졌기 때문에 해당 명령을 받을 수가 없는 것 (벌써 마스터는 이놈을 없애버리고 싶지만...)

---

### 강제 삭제하기
``` bash
$ k delete po kubia-0 --force --grace-period 0
```
확실하게 손절해버린다.

- `grace-period`: (default) 30s

---

약간 애매했던 것은 replicaSet은 손절하나 싶었는데 Unknown이면 결국 아무것도 못한다는 내용을 읽었습니다.

결국 Unknown이면 모두 저렇게 삭제하는 거 말고는 방법이...?

---

끝

---

- 클러스터된 애플리케이션 -> 주키퍼 이런 것들을 설치해보면 알 것이다.

컨시스턴스 알고리즘이 존재함.


- known은 쿠버네티스의 마스터가 모르는 것. 애매한 상태.

마스터에 있는 pod의 정보를 지우는 것. 삭제하는 것. 실제로 pod이 살아있을지는 모르는 것이다.

kubelet이 이상하거나, 네트워크가 이상할 수도 있고.

- Q. 만약 다시 살아나면 어떻게 되는 것이냐? 

완전히 남의 자식이 되는 것. 다시 떠도 마스터에서는 절대 관리하지 않음. kubelet에서 이상한놈이 떠있는지 확인할 수 있을 수는 있음.


- finalizers
  - 특정 팟에. 어플리케이션이 삭제해야되는 단계 중에 과정이 존재함.
  - 어노테이션으로 마킹을 해놓음.
  - 모든 자원 릴리즈가 끝나면 마킹을 없앰.
  - 근데 자원 릴리즈 과정에서 자원이 꼬여서 평생 안없어지는 경우가 있음.
  - 그래서 해당 값을 없애버려서 그냥 명시적으로 없애버리는것
  - 실제 자원이 없어졌는지 모른다.








