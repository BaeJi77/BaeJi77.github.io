---
title:  "Spring application.yaml 파일 분리 및 사용 방법"

categories:
  - spring
  - dev
tags:
  - dev
  - spring
  - java
  - properties
  - application.yaml
---

# 배경
Java profile에 맞는 환경 마다 동작하게 만드는 properties 파일을 다르게 설정하고 싶었습니다. 이런 환경에서 다양한 방법이 있는데 제가 생각했을 때 편한 방법을 설명해드리겠습니다.

# `.properties` vs `.yaml`
둘 다 Spring에서 프로퍼티를 지정할 때 사용할 수 있어요. 밑에 간단한 예를 통해서 어떻게 작성되는지 비교해보겠습니다.
```yaml
properties file

spring.port=8080
server.number[0]=5
server.number[1]=3

---
yaml file

spring:
    port: 8080

server:
    number:
        - 5
        - 3
```
위에 보인느 것처럼 `.properties` 파일 같은 경우는 지속적으로 prefix을 만들어줘야 되기 때문에 귀찮은 부분이 있습니다. 그 이후에 배열을 표현할 때도 굉장히 불편함이 있다고 생각합니다.
사람마다 편한 방법이 있고 익숙한 방법이 있지만 처음부터 `.yaml`을 가지고 많이 활용하다보니 저는 `.yaml` 파일을 가지고 properties 파일을 만들었습니다.

그리고 `.yaml` 파일에서만 되는 프로퍼티 선언 방식이 있다. 하나의 파일에서 여러 profile을 정의해서 사용하는 방법이다.
```yaml
spring:
    profiles:
        active: local

---

spring:
    profiles: local

server:
    port:
        8080

---

spring:
    profiles: prod

server:
    port:
        80
```
위와 같은 방식으로 하나의 파일에서 여러 profiles을 선언하고 선택하는 방식으로 사용할 수가 있다. `.properties` 에서는 여러 파일을 만들어야되는데 형식은 `application-{profileName}.properties` 형식으로 만들고 profiles를 `profileName`으로 하게 되면 그 파일을 읽어서 사용한다.

# 방법
내가 사용한 방법은 사실 `.properties`를 여러 profiles 파일을 만들어서 관리하는 것과 동일한 방식이다. `.yaml`도 여러 파일을 만들어서 사용할 수가 있다.
![사진1](/assets/images/application.yaml-1.png)

위에 있는 사진처럼 profiles에 맞는 파일을 만들어서 사용할 수 있다.

그 뿐만 아니라 `active` 라는 키워드 사용하지 않고 실행하여도 profile에 맞도록 파일을 찾아서 읽는다. 공통적인 내용은 `application.yaml`에 넣어두고 사용하며 나머지 부분은 profiles에 넣어서 사용하면 됩니다.

# `.yaml` properties 정보 읽기
```java
@EnableConfigurationProperties({
        ServerProperties.class,
        ...,
})
@Configuration
public class ServerConfig {

}
```
위와 같은 방식으로 properties 파일에 대한 class를 사용하겠다고 선언을 진행한다. 그리고 밑에 같은 `.yaml` 파일이 있다고 했을 때 그 값을 밑와 같은 방식으로 선언해서 사용가능하다. (이것은 사람마다 취향이 다르지만 저는 항쪽에서 `Enable`과 관련된 것을 모두 선언하는 방식으로 하고 있습니다.)

```yaml
server:
    port: 8080
    name: Hoon
    info:
        hello: true
        world: false
```

```java
@ConfigurationProperties("server")
public class ServerProperties {

    private Integer port;
    private String name;
    private InfoProperties infoProperties;

    @Data
    public static class InfoProperties {
        private Boolean hello;
        private Boolean world;
    }
}
```
위와 같은 방법으로 `prefix`를 선언할 수 있고 그 밑에 존재하는 properties 값들을 가져올 수 있다. 그 뿐만 하위에 있는 것들에 대해서 `static class`를 만들어서 그 값을 또 가져올 수 있다.

```java
@Component
public class PropertiesComponent {

    private final ServerProperties serverProperties;

    public PropertiesComponent (ServerProperties serverProperties) {
        this.serverProperties = serverProperties;
    }

    public print() {
        System.out.println(serverProperties.getPort());
        System.out.println(serverProperties.getName());
        System.out.println(serverProperties.getInfoProperties.getHello());
        System.out.println(serverProperties.getInfoProperties.getWorld());
    }

}

---

8080
Hoon
true
false
```
이런 느낌으로 사용할 수가 있습니다. 이렇게 하게 되면 원하는 `.yaml` 파일을 자유자제로 사용할 수 있습니다. 

# reference

[갓대희님의 블로그](https://goddaehee.tistory.com/213)
