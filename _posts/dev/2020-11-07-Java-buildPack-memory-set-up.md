---
title: "Java/Spring 쿠버네티스에서 buildpack으로 만들어진 이미지 메모리 설정하기 (Set up java memory configration by using kubernetes resource)"

categories:
  - spring
  - dev
tags:
  - dev
  - spring
  - java
  - gradle
---

# Overview 
[기존에 블로깅했던 내용](https://baeji77.github.io/spring/dev/Spring-boot-gradle-build/) 설명한 방식을 통해서 특별한 추가 명령어 없이 바로 Jar를 만들고 docker image를 만들 수 있었다. 그 이후 Java 메모리 설정에 대해 말해보겠다.

# bootBuildImage gradle task
이전 자료를 읽으면 자세하게 나오겠지만 아주 간단한 방법으로 docker image를 만들 수 있습니다. (`gradle task bootBuildImage`)

그리고 메모리 설정은 docker를 실행할 때 환경 변수를 지정하면서 메모리 설정을 할 수 있따.

# Java buildpack 메모리 설정
`buildpack`같은 경우 기본적으로 자신들에 휴리스틱 알고리즘으로 메모리 설정이 되어 있다. `Max memory`에 대해서 알아서 계산을 해준다. 

다시 한번 보면 컨테이너의 메모리를 설정하고 그것에 맞춰져서 실행된다.

## default
``` bash
$ docker run -e spring.profiles.active=local -p 8080:8080 spring-boot-dockerized:0.0.1-SNAPSHOP

Setting Active Processor Count to 6
WARNING: Container memory limit unset. Configuring JVM for 1G container.
Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -Xmx400163K -XX:MaxMetaspaceSize=136412K -XX:ReservedCodeCacheSize=240M -Xss1M (Total Memory: 1G, Thread Count: 250, Loaded Class Count: 21670, Headroom: 0%)
Adding 127 container CA certificates to JVM truststore
Spring Cloud Bindings Enabled
Picked up JAVA_TOOL_OPTIONS: -Djava.security.properties=/layers/paketo-buildpacks_bellsoft-liberica/java-security-properties/java-security.properties -agentpath:/layers/paketo-buildpacks_bellsoft-liberica/jvmkill/jvmkill-1.16.0-RELEASE.so=printHeapHistogram=1 -XX:ActiveProcessorCount=6 -XX:MaxDirectMemorySize=10M -Xmx400163K -XX:MaxMetaspaceSize=136412K -XX:ReservedCodeCacheSize=240M -Xss1M -Dorg.springframework.cloud.bindings.boot.enable=true
```

## Container memory set-up
``` bash
$ docker run -e spring.profiles.active=local -p 8080:8080 -m=2G  spring-boot-dockerized:0.0.1-SNAPSHOP

Setting Active Processor Count to 6
Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -Xmx1448739K -XX:MaxMetaspaceSize=136412K -XX:ReservedCodeCacheSize=240M -Xss1M (Total Memory: 2G, Thread Count: 250, Loaded Class Count: 21670, Headroom: 0%)
Adding 127 container CA certificates to JVM truststore
Spring Cloud Bindings Enabled
Picked up JAVA_TOOL_OPTIONS: -Djava.security.properties=/layers/paketo-buildpacks_bellsoft-liberica/java-security-properties/java-security.properties -agentpath:/layers/paketo-buildpacks_bellsoft-liberica/jvmkill/jvmkill-1.16.0-RELEASE.so=printHeapHistogram=1 -XX:ActiveProcessorCount=6 -XX:MaxDirectMemorySize=10M -Xmx1448739K -XX:MaxMetaspaceSize=136412K -XX:ReservedCodeCacheSize=240M -Xss1M -Dorg.springframework.cloud.bindings.boot.enable=true
```
아무 옵션도 주지 않은 경우에는 `Total Memory: 1G` 나온다. 기본적으로 메모리 1G를 가지고 메모리 구성을 진행한다.

컨테이너 메모리로 2G로 설정하니 `Total Memory: 2G`라고 나오는 것을 볼 수 있고 나머지 부분에 대해서 추가적으로 설정하지 않아도 `Xmx`, `CacheSize`와 같은 친구들을 자동으로 셋업해준다. 

참고로 휴리스틱 알고리즘으로 설정되어 있는데 제한으로는 매우 작은 메모리를 할당하는 경우 제대로 동작하지 않을 수 있다. [Java-buildpack-4.0](https://www.cloudfoundry.org/blog/just-released-java-buildpack-4-0/)

휴리스틱 알고리즘은 힙에 대한 영역에 대한 나머지 부분에 대해서 최적화를 진행하고 추가적인 메모리 공간에 대해서 힙에 용량이 커지도록 설정된다. 그래서 전체 메모리가 작은 경우 힙 메모리가 적게 할당되게 되어서 정상적으로 동작이 안될 수 있다.

# 그러면 쿠버네티스에서는?

쿠버네티스에서도 이미지를 동작할 때 환경 변수를 입력하는 것처럼 사용하면된다. 하지만 그 방법말고 간단하게 할 수 있는 방법이 있다. 

쿠버네티스에서는 기본적으로 자원에 대해서 `Request/limit` 이라는 영역이 존재한다. (이 부분에 대해서는 다른 블로그님들이 잘 작성되어 있습니다!)

해당 부분에 대해서 어떻게 셋업하느냐에 따라서 자동으로 메모리를 할당해주는 경우가 존재한다.

## k8s pod yaml file

### default
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

### request
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-dockerize-memory-request-2g-pod
spec:
  containers:
  - name: spring-dockerize-memory-request-2g-pod
    image: spring-boot-dockerized:0.0.1-SNAPSHOT
    resources:
      requests:
        memory: "2G"
```

### limit
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-dockerize-memory-limit-2g-pod
spec:
  containers:
  - name: spring-dockerize-memory-limit-2g-pod
    image: spring-boot-dockerized:0.0.1-SNAPSHOT
    resources:
      limits:
        memory: "2G"  
```

## 실제 실험 결과

### default
![default](/assets/images/k8s-spring-memory-default.png)

### request
![request](/assets/images/k8s-spring-memory-request.png)

### limit
![limit](/assets/images/k8s-spring-memory-limit.png)

사진을 잘 보게 되면 default는 아무것도 설정하지 않았기 때문에 당연히 기존에 알던대로 1G로 설정된다.

그리고 `request` 또한 별다른 특별한 경우가 없다.

하지만 `limit`을 설정하는 경우 해당 컨테이너 이미지를 동작시킬 때 `-m=2G`을 주는 것과 같은 효과로 메모리 영역이 달라졌다.

# 결론
자바 같은 경우 메모리 설정이 매우 중요하다는 이야기를 들어서 궁금한 상태였는데 같이 업무를 하는 분에게 질문까지 받아서 그 내용에 대한 답변으로 공부해 봤다. `buildpack`을 이용해서 아주 간단하게 이미지를 만들 수 있는 만큼 생각보다 추가적인 부분이 자세하게 나오지 않아 공부해야되는 부분이 많은 것 같다.

# Ref
[Buildpacks homepage](https://buildpacks.io/)

[java buildpack memory calculator](https://github.com/cloudfoundry/)

[java buildpack 4.0](https://www.cloudfoundry.org/blog/just-released-java-buildpack-4-0/)
