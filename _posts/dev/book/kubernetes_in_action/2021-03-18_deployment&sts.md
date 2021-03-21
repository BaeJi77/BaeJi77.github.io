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
# 9. deployment
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

애플리케이션은 변경될 것이다. 쿠버네티스트에서 실행되는 애플리케이션을 업데이트하는 방법과 무중단 업데이트를 어떻게 하는지 설명할 것이다.

이 작업은 레플리케이션 컨트롤러 또는 레플리카셋을 사용해서 수행할 수 있지만 디플로이는 레플리카셋 기능을 활용해서 선언적으로 업데이트를 가능하게 한다.

---

# 파드에서 실행중인 애플리케이션 업데이트

기본 버전 v1을 태그로 가지는 애플리케이션을 외부에서 접근할 수 있다. 그런 상태에서 우리는 새로운 버전인 v2를 배포하고 싶다. 그러기 위해서는 우리는 기존 파드를 삭제하고 새로운 파드를 실행해야한다. 두가지 방법이 있다고 할 수 있다.

1. 기존 파드를 한번에 모두 삭제한 다음 새 파드를 실행
2. 새로운 파드를 실행하고 기존 파드를 삭제. 새로운 파드를 모두 띄우고 삭제하거나 그 중간 중간에 삭제와 배포를 같이 한다.

두가지 모두 장단점이 있다. 첫번째 같은 경우 일시적으로 서비스 접근이 불가능하다. 두번째 같은 경우 두가지 버전이 동시에 실행되어야 하기 때문에 서비스가 같은 기능을 제공하지 못할수도 있으며 데이터 저장소 같은 경우 새 버전이 이전 버전을 손상 시킬 수 있어서 데이터 스키마나 데이터 수정을 해서는 안된다.

---

# 롤링 업데이트
말 그래도 돌아가면서 업데이트가 진행됨.

## 과정
![rolling](https://drek4537l1klr.cloudfront.net/luksa/Figures/09fig04_alt.jpg)

## 레플리케이션컨트롤러와 kubectl을 이용한 rolling update

이전에 hoon:v1이라는 이미지가 동작하는 컨테이너가 있다가 가정할 때.

hoon:v2를 배포하고 싶다.

``` bash
$ k rolling-update hoon-v1 hoon-v2 --image=hoon:v2
```

---

## 지금은 사용하지 않는 이유
- 운영자가 아닌 쿠버네티스가 자원에 대해서 수정을 진행. 
- k8s client가 이 과정을 진행. 네트워크 이슈로 클라이언트와 연결이 끊어지면 큰일.
- 명령을 나타난다. 쿠너베티스는 상태를 선언하고 그것을 달성하는 것이 베스트라고 생각하기에.

---

# deployment

애플리케이션을 배포하고 선언적으로 업데이트하기 위한 높은 수준의 리소스 <-> 레플리케이션컨트롤러 또는 레플리카셋과 같은 낮은 수준의 개념

![deployment](https://drek4537l1klr.cloudfront.net/luksa/Figures/09fig08.jpg)

```
deployment -> 레플리카셋 -> pods
```
---

## 왜 사용?
레플리카셋, 컨트롤러로도 충분히 배포는 가능.

하지만 아까 보여준 것 처럼 단점이 분명이 존재하며 배포와 관련한 리소스에 대한 상위 개념이 존재해야 됨.

---

## 형식

``` yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: hoonDeployment
spec:
  replicas: 3
  template:
    metadata:
      name: hoonPod
      labels:
        app: hoonApp
    spec:
      containers:
      - image: github/hoonImage:v1
        name: hoonContainer
```
레플리케이션컨트롤러를 만드는 것과 다르지 않다. 
---

## deployment를 통한 업데이트

---

### 레플리케이션 컨트롤러 업데이트 
- 레플리케이션컨트롤러은 명시적으로 하라면 명령도 내리고 새로운 레컨 이름도 지정해줘야했음. 
- 업데이트가 끝나면 이전 레플리케이션컨트롤러을 삭제함. 
- 터미널을 열어두고 끝날 때까지 기달려야 했음.

---

### deployment 업데이트 전략
- (default) RollingUpdate: 이전에 보여준 것처럼 하나씩 하나씩. 이전 버전과 현재 버전이 동시에 동작하는 것이 가능한 경우.
- Recreate: 한번에 모두 삭제한 다음 새로운 파드를 만듬. 이전 버전을 완전히 싸그리 없애버리고 새로 만듬. 짧은 다운타임이 생김

---

**파드 템플릿**을 변경하지 않은 경우에 디플로이먼트는 배포를 하지 않는다. 디플로이먼트 스펙, 레플리카 수, 정책과 같은 배포 속성은 바꿔도 롤아웃이 안됨.

**파드 템플릿**이 컨피그맵(또는 시크릿)을 참고하고 있고 해당 컨피그맵이 업데이트된다고해서 배포가 되지는 않는다. 

---

> 방법은? 
> 1. 새로운 configmap을 만들어서 적용
> 2. [stakater/Reloader](https://github.com/stakater/Reloader)

---

### 과정
---

``` bash
$ k set image deployment hoonDeployment hoonContainer=github/hoonImage:v2
```
해당 명령어를 트리거 -> 팟 템플릿의 변경에 대해서 트리거

과정은 전에 설명한 레컨 롤링 업데이트 과정에서 발생하는 이벤트와 매우 유사. 하지만 우리가 어떤 다른 명령어를 하지 않아도 된다.

그리고 추가적으로 이전 버전에 대한 레플리카셋을 그냥 나둔다. (k rolling-update는 삭제해버림)

---

### 롤백
배포 이후에 문제가 있다고 판단되는 경우 이전 버전으로 롤백할 수가 있다.

---

``` bash
$ k rollout undo deployment hoonDeployment // 직전 버전으로 롤백

$ $ k rollout history deployment hoonDeployment // 이력 확인 => 이력을 보기 위해서 --record를 통해서 배포를 진행해야 됨

$ k rollout undo deployment hoonDeployment --to-revision=1 // 특정 버전으로 롤백
```

---

### 롤아웃 속도 제어
``` yaml
spec:
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
```

---

#### maxSurge
해당 숫자만큼 추가적으로 인스턴스를 가질 수 있다. (replicas + maxSurge)

`replicas: 3, maxSurge: 1` 이라고 과정했을 때 롤링 업데이트 과정에서 최대 4개까지의 인스턴스가 동작할 수 있다.

---

#### maxUnavailable
해당 숫자를 뺀 나머지 인스턴스는 동작하는 것을 보장해야 된다. (replicas - maxUnavailable)

`replicas: 3, maxUnavailable: 1` 이라고 과정했을 때 롤링 업데이트 과정에서 최대 2의 인스턴스는 동작을 보장해야된다.

---

### 롤아웃 프로세스 일시 정지

``` bash
$ k set image deployment hoonDeployment hoonContainer=github/hoonImage:v4

$ k rollout pause deployment hoonDeployment

$ k rollout resume deploymnet hooneDeployment
```

---

변경을 하자마자 deployment는 롤아웃을 진행할 것이고 그러면서 생긴 새로운 버전에 v4 파드가 만들어질 것이다. 

그 과정에서 중간에 멈추게 되면 해당 v4 파드에 트래픽이 들어가게 할 수 있다.

카나리 배포가 가능함.

---

위에 설명한 것에 대한 명령어 다큐먼트 존재.

[kubectl rollout document](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#rollout)

---

### 잘못된 버전의 롤아웃 방지

---

- `minReadySecond`

파드를 사용 가능한 것으로 취급하기 전에 새로 만든 파드에 준비 시간을 줌.

`minReadySecond`가 지나기 전에 새 파드가 제대로 작동하지 않고 레디니스 프로브가 실패하기 시작하면 롤아웃이 효과적으로 차단된다.

실제 트래픽을 수신하기 시작한 후 파드가 준비 상태를 계속 ㅂ보고할 수 있도록 `minReadySecond`를 널널하게 설정해야 된다.
---

``` yaml
spec:
  minReadySecond: 10
  ...
  template:
    ...
    spec: 
      containers:
      - image: hoon:v3
        name: hoon
        readinessProbe: 
          periodSeconds: 1
            httpGrt:
              path: /
              port: 8080
```
---

만약 `minReadySecond` 올바르게 설정하지 않은 경우,

`readinessProbe`의 첫번째가 성공하면 배포가 가능한 것으로 판단해서 나머지도 모두 배포하게 된다. 

그러니 적절한 값으로 설정해야 된다.

---

### 롤아웃 데드라인

`ProgressDeadlineSeconds`라는 것을 설정할 수 있음. 

기본 10분으로 설정되어 있음.

``` yaml
kind: Deployment
...
spec:
  progressDeadlineSeconds: 600 // (단위는 초)
```

---

# StatefulSet

---

# 서론
- 클러스터된 스테이트풀 애플리케이션 배포
- 파드 레플리카 인스턴스에 별도의 스토리지 제공
- 파드 레플리카에 안정적인 이름과 호스트 이름 보장
- 예측 가능한 순서대로 파드 레플리카의 시작과 중지
- DNS 서비스 레코드를 통한 피어 디스커버리

---

# 스테이트풀 파드 복제
레플리카셋은 이름과 ip 주소를 제외하고 모두 똑같은 파드를 만듬. 

그리고 모두 똑같은 볼륨을 사용함 = 각자 다른 퍼시스턴트볼륨은 사용 불가

---
## 레플리카에 개별 스토리지 붙이기

**그러면 과연 각 파드에 볼륨을 붙일 수 있을까?**

---

- 수동으로 파드 생성
  - 나중에 너무 귀찮음. 죽으면 계속 살려야되는데 말이 안됨.
---

- 파드 인스턴스별로 하나의 레플리카셋 사용
  - 단일 레플리카를 사용하는 것에 비해 너무 불편함.
  - 하나 배포한다고 해도 각 레플리카에 대해서 배포를 직접 해야됨
  - 스케일도 귀찮음
![eachpod](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig02.jpg)

---
- 동일 볼륨을 여러 개 디렉토리로 분할 사용
  - 하나의 퍼시스턴스 볼륨 + 각 파트들이 다른 디렉토리 공간 지정해서 사용
  - 인스턴스가 생성되는 시점에 사용하지 않는 디렉토리를 선택할 수는 있음.
  - 하지만 잘 동작하는 것에 대한 보장이 안되고 볼륨 병목 현상 발생

![eachPodSameVolume](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig03_alt.jpg)
---

## 각 파드에 안정적인 아이덴티티 제공
새로운 파드를 교체해야되는 경우 레플리카셋같은 경우 이전과 다른 새로운 호스트 네임과 Ip를 제공한다.

이전 인스턴스 데이터를 가지고 시작할 때 네트워크 아이덴티티로 인한 문제가 발생한다. 

스테이트풀셋은 넘버링 (index)를 통해서 이름에 붙인다.

![comparingReplicaAndStateful](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig05_alt.jpg)

---
그리고 파드별 네트워크 관련해서 동일한 정보를 제공하고 싶다.

### 각 파드 인스턴스별 전용 서비스 사용
서비스같은 경우 변경되지 않는 ip를 제공한다고 생각하기 때문에 파드별로 서비스를 가지고 있으면 새로운 파드가 만들어 진다고해도 문제가 되지 않지 않을까?

할수는 있겠지만 각 파드가 어떤 서비스에 매핑되는지 알기 어렵기 때문에 해당 파드에 접근하기가 어렵다.

---

# 스테이트풀셋 이해

---

## 스테이트풀셋과 레플리카셋 비교

- 레플리카셋
다른 파드들과 거의 대부분 동일. 이전 파드와 새로운 파드가 식별만 다르지 동작은 그래도.

대부분 스테이트리스로 언제든지 완전히 새로운 파드로 교체가능.

---

- 스테이트풀셋
스테이트풀 파드는 새로운 파드 인스턴스는 이전 파드와 동일한 이름, 네트워크 아이덴티티, 상태를 똑같이 있어야된다. 

replica의 수를 줄이고 늘리는대도 규칙이 있다. 줄였다가 다시 늘리는데 이전 파드와 동일한 정보

![comparing_2](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig06_alt.jpg)

---

### 스테이트풀셋 스케일링
만약 2개의 레플리카 파드가 있다가 과정했을 때 새로운 파드에 이름에는 다음 인덱스인 2가 붙여서 생성될 것이다. 

ex) a-0, a-1, a-2 (New)

또한 삭제될 때는 그 순서에 반대대로 삭제된다.

ex) a-0, a-1, a-2 (will be removed), a-3 (be removed)

![scailing](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig07_alt.jpg)

---

### 각 스테이트풀 인트선스에 안정적인 스토리지 제공
새로 만들어진 파드에 대해서 이전 스토리지와 같은 스트리지가 할당되도록 해야된다.

스토리지는 영구적이어야되지만 파드와는 분리되어야한다. 쿠버네티스에 볼륨이 있어야되는 이유?

6장에서 배운 것처럼 퍼시스턴트볼륨클레임을 이용해서 볼륨과는 1:1 매핑이 진행되고 파드와 퍼시스턴스볼륨 클레임을 1:1 매핑해서 각자 스토리지를 가지도록 한다.

``` text
pod 1 : 1 pvc 
pvc 1 : 1 pv

pod -> pvc -> pv
```
---

이렇게 하기 위해서 **볼륨 클레임 템플릿**을 이용한다. (팟을 선언적으로 만들도록 했던 템플릿과 비슷)

![볼륨클레임템플릿](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig08_alt.jpg)
---

이 가정에는 관리자가 프로비저닝한다거나 혹은 동적 프로비저닝이 지원한다고 가정

> 주의: 스케일 다운을 하고 남은 볼륨이 중요한 데이터라면 직접 관리하는 것이 중요하다. (퍼시스턴트볼륨클레임을 직접 삭제)
> 다시 스케일 업을 하게 되는 경우 데이터가 손상되거나, 전략에 의해서 데이터가 삭제될 수도 있다.

![새로운파드볼륨](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig09_alt.jpg)

---

# 스테이트풀셋 사용

---

## format
``` yaml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: hoonSts
spec:
  replicas: 3
  template:
    metadata:
      name: hoonPod
      labels:
        app: hoonApp
    spec:
      containers:
      - image: github/hoonImage:v1
        name: hoonContainer
volumeCliamTemplates:
- metadata:
    name: data
  spec:
    resources:
      request:
      storage: 1Mi
    accessModes:
    - ReadWriteOnce
```

---
deployment와 다르게 sts는 파드가 모두 완벽하게 떴을 경우에는 다음 파드를 만든다.

두개를 동시에 만들다가 레이스 컨디션을 발생할 가능성이 있기 때문이다.

---

- 실행 이후 모습
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
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-r2m41
      readOnly: true
  ...
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data-kubia-0
  - name: default-token-r2m41
    secret:
      secretName: default-token-r2m41
```

---

### 실제 pvc 모습
``` bash
$ kubectl get pvc
NAME           STATUS    VOLUME    CAPACITY   ACCESSMODES   AGE
data-kubia-0   Bound     pv-c      0                        37s
data-kubia-1   Bound     pv-a      0                        37s
```

---

# 스테이트풀셋의 피어 디스커버리

같은 스테이트풀셋에 존재하는 파드들에 대해서 찾는 것.

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

## 거버닝 서비스
스테이트풀셋 같은 경우 이름이 이전과 똑같다.

하지만 동일한 이름말고도 **호스트이름을 통해서 어떤 접근을 해야될때가 있음**

그래서 거버닝 헤드리스 서비스를 생성해서 호스트에 대한 네트워크 아이덴티티를 찾을 수 있도록 도와줌.

또한 SRV 레코드를 조회해 모든 스테이트풀셋의 파드이름을 가져올 수 있음.

---

```
namespace: default
service name: foo
pod name: a-0

FQDN: a-0.foo.default.svc.cluster.local
```

---

### SRV 레코드
특정 서비스를 제공하는 서버의 호스트 이름과 포트를 가리키는데 사용.

쿠버네티스는 헤드리스 서비스를 뒷밤침하는 파드의 호스트 이름을 가리키도록 SRV 레코드를 생성.

---

``` bash
dig SRV kubia.default.svc.cluster.local

...
;; ANSWER SECTION:
k.d.s.c.l. 30 IN  SRV     10 33 0 kubia-0.kubia.default.svc.cluster.local.
k.d.s.c.l. 30 IN  SRV     10 33 0 kubia-1.kubia.default.svc.cluster.local.

;; ADDITIONAL SECTION:
kubia-0.kubia.default.svc.cluster.local. 30 IN A 172.17.0.4
kubia-1.kubia.default.svc.cluster.local. 30 IN A 172.17.0.6
...
```

위의 모습처럼 파드가 스테이트풀셋의 모든 파드 목록을 가져오려면 SRV DNS 룩업을 수행하기만 하면 된다.

---

## DNS를 통한 피어 디스커버리
쿠버네티스 서버는 모든 파드들에 대한 정보를 가지고 있다. 

하지만 클라이언트는 모른다. 알기 위해서 서버에 요청을 해야되는데 너무 번거롭고 시간도 오래 걸림.

그런 상황에서 스테이트풀셋과 SRV 레코드를 사용하면 편하게 해결 가능하다.

![dnsLookUp](https://drek4537l1klr.cloudfront.net/luksa/Figures/10fig12_alt.jpg)

---

## 클러스터된 데이터 저장소 사용
전략에 따라 다를 수 있겠지만 우리는 스케일 다운해서 새로운 올린 파드는 이전 파드에 연결되어진 스토리지가 있다.

그러기 때문에 이전에 있던 파일정도 스토리지에 저장된 결과는 달라지지 않는다. 그리고 바로 반환 가능하다.


# 스테이트풀셋 노드 실패를 처리하는 과정

책을 참조...


# 궁금증
왜 클러스턴된 애플리케이션에서는 피어 디스커버리(동일 클러스터에 있는 다른 멤버를 찾는 기능)
