---
title: "minikube에서 로컬 docker image를 올리고 싶을때는? (How to use local docker image in minikube(kubernetes))"

categories:
  - dev
  - infra
  - kubernetes
tags:
  - dev
  - kubernetes
  - docker
  - minikube
---

# Overview
minikube를 하면서 로컬에 존재하는 docker image를 사용해보고 싶었는데 잘 되지 않더군요. 그래서 한번 방법을 찾아봤습니다.

# 상황
저는 로컬에 있는 이미지를 `minikube` 위에서 동작하도록 설정하기 위해서 밑에와 같은 방식으로 `.yaml` 파일을 만들던가 `kubectl`을 만들어서 사용했습니다.
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-dockerize-default-pod
spec:
  containers:
  - name: spring-dockerize-default-pod
    image: spring-boot-dockerized:0.0.1-SNAPSHOT
```

``` bash
$ kubtctl run [쿠버네티스 위에서 이름] --image=[이미지이름]:[태그이름]
```

이렇게 동작을 했을 때 정상적으로 동작할 줄 알았는데... 잘 돌아가지 않더군요.

![kubectl_get_pods](/assets/images/before_get_pods.png)
위에 사진 처럼 `ErrImagePull`라는 상태메시지가 나왔습니다. 자세하기 보기 위해서 `kubectl describe [resource name]` 을 통해서 확인해봤습니다.

![describe_pods](/assets/images/before_error.png)

``` bash
Failed to pull image "[이미지이름]:[태그이름]": rpc error: code = Unknown desc = Error response from daemon: 
pull access denied for cms, repository does not exist or may require 'docker login': denied: requested access to the resource is denied 
```

위에서 보이는 것 처럼 뭔가 `pull`을 하지 못하는 부분이 보였습니다. 

여기저기 에러 메시지를 검색도 해보고 혹시 로컬에 존재하는 이미지를 가져오지 못하는 것이 아닐까 싶어서 검색을 해보니 방법이 있더군요. 

현재 제가 로컬에서 동작하고 있는 docker image 모습입니다. 
``` bash
$ docker ps
```

![before_docker_ps](/assets/images/before_docker_ps.png)


[minikube document](https://kubernetes.io/ko/docs/setup/learning-environment/minikube/#docker-%EB%8D%B0%EB%AA%AC-%EC%9E%AC%EC%82%AC%EC%9A%A9%EC%9D%84-%ED%86%B5%ED%95%9C-%EB%A1%9C%EC%BB%AC-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0) 를 보게 되면 local에서 사용하기 위한 방법이 존재했습니다.

## 방법
아주 간단하다. `minikube`와 현재 나의 `docker` host와 연결하는 방법인데 

``` bash
$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://127.0.0.1:32778"
export DOCKER_CERT_PATH="/Users/Hoon/.minikube/certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"

# To point your shell to minikube's docker-daemon, run:
# eval $(minikube -p minikube docker-env)

$ eval $(minikube -p minikube docker-env)
``` 
``$ minikube docker-env``를 입력하게 되면 저런 결과 값들이 나온다. 그리고 마지막 줄에 존재하는 ``$ eval $(minikube -p minikube docker-env)``를 실행하게 되면 연결되었다!!

![after_docker_ps](/assets/images/after_docker_ps.png)

위에 사진은 이전과 다르게 `docker ps`를 했을 경우 나오는 모습이다. `docker` 이미지가 `minikube`에서 동작하는 driver(docker)와 연결되어진 것이기 때문에 **추가적으로** 만들어지 `docker image`는 `minikube` 위에 올라가게 된다!

이젠 `minikube`에 올리고 싶은 친구가 있다면 위에 있는 명령어를 입력한 이후에 쿠버네티스에서 동작하게 되면 아주 깔끔하게 동작할 것이다.

## 실제 실행 결과
![after_docker_images](/assets/images/after_docker_images.png)

![after_kubectl_run](/assets/images/kubectl_run.png)

![after_kubectl_get_pods](/assets/images/after_kubectl_get_pods.png)

실제 도커 이미지를 빌드하니 `minikube` 내부 `docker image`에 등록되는 모습을 볼 수 있다. 그 이후에 `kubectl run`을 통해서 동작시켜보니 정상적으로 동작하는 것을 볼 수 있다.

## 주의
정말 이상하게도 ``$ eval $(minikube -p minikube docker-env)`` 명령어를 친 터미널에서는 가능하고 터미널에서 다른 창을 켜서 실행해보니 다른 곳에서는 또 적용이 안되더라고요... 만약 minikube에서 올리고 싶은 경우는 해당 명령어를 친 곳에서 `docker build`를 진행하는게 맞을 것 같습니다. 


# 결론
쿠버네티스를 자주 사용하지는 않아 셋업하는 과정에서 생각보다 시간을 많이 투자하게 된다. 혹시 모를 누군가에게 적은 시간으로 해결하기 바라면서 마친다.

# ref
[minikube](https://kubernetes.io/ko/docs/setup/learning-environment/minikube/)
