---
title: "Spring boot docker image만들기 최적화 방법 (+ 새로운 gradle task)"

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
Spring boot를 활용하여서 도커 이미지를 만들 때 단점을 이야기하며 그것에 대한 개선 방법에 대해 이야기합니다.

# 용어 설명
gradle task: gradle 프로젝트의 작업 단위. gradle에서 제공해주는 기본 task도 있으며 새로운 task를 만들거나 기존 task를 커스텀에서 사용가능하다. 

[OCI (OPEN CONTAINER INITIATIVE)](https://opencontainers.org/): 컨테이너 기술에 대한 표준화입니다. docker 뿐만 아니라 여러 컨테이너 기술에 대한 표준화로 컨테이너를 모두 같은 동작을 할 수 있도록 했습니다. [참고글 (kor)](http://www.opennaru.com/kubernetes/open-container-initiative/)


# Spring dockerized

## 기본 방식
```docker
FROM openjdk:8-jdk-alpine
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
현재 위에 있는 `Dockerfile`은 인터넷이나 기본적으로 많이들 볼 수 있는 형식이다.
이것을 통해서 간단하게 jar 파일을 통해서 spring 관련 도커 이미지를 만들 수 있다.
스프링같은 경우 `./gradlew bootJar`를 통해서 Spring 프로젝트를 jar로 파일로 패키징할 수 있고 위에 코드를 활용해서 도커 이미지로 만들 수 있다.

하지만 여기서 중요한 것은 jar 파일이 무거워 지는 경우 docker image를 만드는 것이 비효율이 발생한다.
도커는 레이어마다 캐쉬를 활용할 수 있고 그것을 통해서 빠른 docker 이미지를 만들 수 있는 장점이 존재한다.
이런 구조에서는 자바의 모든 구조가 jar 파일로 되는 것이기 때문에 캐쉬를 적용하기가 어렵다. (소스 코드 한 줄이 바뀌더라도 캐쉬가 깨지기 때문에 다시 연산을 해야한다.)

## [효율적인 방식](https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1)
Spring을 이용해서 Jar 파일을 만들 때 4개의 영역으로 구분되어 지도록 만들 수 있습니다. [gradle 방식](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/gradle-plugin/reference/html/#packaging-layered-jars)
이 방법으로 Jar 파일을 만들고 밑에 방식을 통해서 jar 파일을 풀게 되면 4가지 폴더(layer)가 만들어 집니다.

```groovy
bootJar {
    layered()
}
```
```bash
java -Djarmode=layertools -jar JAR_FILE_NAME.jar extract 
```
위에 있는 코드는 `gradle.build` 에 추가하고 `./gradlew bootJar` 를 통해서 jar파일을 만들게 되는 경우 4가지 폴더로 나눠져서 jar가 구성되지는 모습을 볼 수 있다.

그리고 위에 있는 명력어를 통해서 jar 파일을 풀게 되면 4가지 폴도로 구성되어진 형식을 또 볼 수 있다.

|folder name|description(공식문서 설명)|
|---|---|
|application|application classes and resources|
|snapshot-dependencies|any dependency whose version contains SNAPSHOT|
|spring-boot-loader|the jar loader classes|
|dependencies|any dependency whose version does not contain SNAPSHOT|

정확하게 이렇게 4가지로 나눠지는 이유는 적혀있지 않았지만 application layer부터 밑으로 자주 바뀌지 않은 순서가 존재한다는 것이다. (application이 제일 자주 바뀌고 dependiencies가 제일 바뀌지 않는다)

그래서 실제로 docker image를 만들어 가는 과정에서 이 순서에 역순으로 작성을 해주어야 한다. (자주 바뀌지 않기 때문에 캐쉬가 깨질 가능성이 적어진다.)

![spring-boot-docker-image-1](/assets/images/spring-boot-dockerized:jar-layerable-folder.png)

위에 있는 사진은 실제로 압축을 풀었을 경우 나오는 폴더의 모습이다.

```docker
FROM openjdk:8-jdk-alpine as builder
WORKDIR application
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract 

FROM openjdk:8-jdk-alpine
WORKDIR application
ENV port 8080
ENV spring.profiles.active local
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./

ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```
이 방법으로 생긴 4개의 폴더에 대해서 모두 `COPY`하도록 한다.
이 것을 통해서 어떤 부분이 수정되더라도 최소한의 범위에 대해서 캐쉬가 깨지기 때문에 효율적으로 docker image를 만들 수 있다.


```bash
$ docker build -t YOUR_PROJECT_TAG DOCKER_FILE_LOCATION
$ docker run -e spring.profiles.active=local -p 8080:8080 YOUR_DOCKER_IMAGE
```
이렇게 docker image를 실행할 수 있다. 당연히 이전에 만들었던 방식과 똑같이 만들어 질 것이다. 

# [Spring boot 2.3](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/gradle-plugin/reference/html/#build-image)

이전에도 플러그인 설치하는 경우에 컨테니어 이미지를 만드는데 도와주는 플러그인 존재했음. 이 task는 별도 플러그인 없이 바로 실행 가능함.
Spring boot 2.3 에서 gradle (maven도 지원) [bootBuildImage](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/gradle-plugin/api/org/springframework/boot/gradle/tasks/bundling/BootBuildImage.html) 라는 task가 생겼다.
Docker image를 만들어주는데 Dockerfile 없이 만드는 것이 가능하다.
docker host 정보도 셋업 가능함 + build 관련해서 builder를 바꾼다던지 image 이름을 바꾸는 옵션이 존재함.

![spring-boot-docker-image-2-1](/assets/images/spring-boot-dockerized:gradle-task.png)
![spring-boot-docker-image-2-2](/assets/images/spring-boot-dockerized:docker-images.png)

gradle task 실행과 함께 docker image를 만들게 된다. 
그리고 맨 마지막 줄에 있는 spring-boot-dockerized 이름을 가진 image가 만들어 진것을 확인할 수 있다. (gcr.io, paketobuilderpacks 이 친구들은 Buildpacks을 위해서 필요함)
기본적으로 docker daemon을 사용하고 있기 때문에 docker가 설치되어 있어야지 동작할 수 있다.

## [Buildpacks](https://buildpacks.io/)
실제 bootBuildImage task는 buildpack 이라는 기술을 활용해서 구현했다.
OCI 이미지와 최신 컨테이너 표준을 사용하며 어플리케이션을 컨테이너 이미지로 만드는데 잘 활용될 수 있는 기술이다.[Features](https://buildpacks.io/features/)

![spring-boot-docker-image-3](/assets/images/spring-boot-dockerized:comaprison.png)


## Java Buildpack memory configuration
`buildpack` 으로 이미지를 만드는 경우에 있어서 기본적으로 자동으로 jvm 세팅을 해준다.
자세한 jvm memory 계산 방법에 대해서 궁금하신 경우는 여기를 참고하세요. 

### Default
```bash
$ docker run -e spring.profiles.active=local -p 8080:8080 spring-boot-dockerized:0.0.1-SNAPSHOP

Setting Active Processor Count to 6
WARNING: Container memory limit unset. Configuring JVM for 1G container.
Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -Xmx400163K -XX:MaxMetaspaceSize=136412K -XX:ReservedCodeCacheSize=240M -Xss1M (Total Memory: 1G, Thread Count: 250, Loaded Class Count: 21670, Headroom: 0%)
Adding 127 container CA certificates to JVM truststore
Spring Cloud Bindings Enabled
Picked up JAVA_TOOL_OPTIONS: -Djava.security.properties=/layers/paketo-buildpacks_bellsoft-liberica/java-security-properties/java-security.properties -agentpath:/layers/paketo-buildpacks_bellsoft-liberica/jvmkill/jvmkill-1.16.0-RELEASE.so=printHeapHistogram=1 -XX:ActiveProcessorCount=6 -XX:MaxDirectMemorySize=10M -Xmx400163K -XX:MaxMetaspaceSize=136412K -XX:ReservedCodeCacheSize=240M -Xss1M -Dorg.springframework.cloud.bindings.boot.enable=true
```
기본적으로 어떤 옵션을 주지 않은 상태에서 실행을 했을 경우 Container size 1G라고 설정하고 나머지 jvm 옵션 + buildpack에서 만든 휴리스틱으로 알아서 셋업해줍니다.

### Container memory set-up
```bash
$ docker run -e spring.profiles.active=local -p 8080:8080 -m=2G  spring-boot-dockerized:0.0.1-SNAPSHOP

Setting Active Processor Count to 6
Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -Xmx1448739K -XX:MaxMetaspaceSize=136412K -XX:ReservedCodeCacheSize=240M -Xss1M (Total Memory: 2G, Thread Count: 250, Loaded Class Count: 21670, Headroom: 0%)
Adding 127 container CA certificates to JVM truststore
Spring Cloud Bindings Enabled
Picked up JAVA_TOOL_OPTIONS: -Djava.security.properties=/layers/paketo-buildpacks_bellsoft-liberica/java-security-properties/java-security.properties -agentpath:/layers/paketo-buildpacks_bellsoft-liberica/jvmkill/jvmkill-1.16.0-RELEASE.so=printHeapHistogram=1 -XX:ActiveProcessorCount=6 -XX:MaxDirectMemorySize=10M -Xmx1448739K -XX:MaxMetaspaceSize=136412K -XX:ReservedCodeCacheSize=240M -Xss1M -Dorg.springframework.cloud.bindings.boot.enable=true
```
기본으로 셋업되어 있는 1G가 아닌 container의 메모리를 늘리게되면 그것만큼 알아서 나머지 jvm 셋팅을 해준다. 해당 계산법에 대해서는 위에 링크를 확인 바란다. 
자동으로 셋업되는 것이 아닌 자기가 직접 jvm 셋업을 하고 싶은 경우 docker run을 하면서 JAVA_OPTS으로 셋업할 수 있다. 

![spring-boot-docker-image-4](/assets/images/spring-boot-dockerized:exetution-capture.png)
![spring-boot-docker-image-5](/assets/images/spring-boot-dockerized:exetution-capture-2g.png)

# Ref
[Packagin Layered Jars](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/gradle-plugin/reference/html/#packaging-layered-jars)

[Spring boot docker image docs](https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1)

[Spring boot Packaging OCI Images](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/gradle-plugin/reference/html/#build-image)

[Buildpacks homepage](https://buildpacks.io/)

[java buildpack memory calculator](https://github.com/cloudfoundry/)

[java buildpack 4.0](https://www.cloudfoundry.org/blog/just-released-java-buildpack-4-0/)

참고 블로그: https://zgundam.tistory.com/181?category=440149
