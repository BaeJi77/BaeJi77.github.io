---
title: "integration test과 testcontainers (2)"
description: ""

categories: 
  - dev
tags:
  - dev
  - test
  - testing
  - testcontainers
---

글에 대한 코드는 저의 github에서 볼 수 있습니다.

- [test code](https://github.com/BaeJi77/exmple-testcontainers-golang)
- [test custom container code](https://github.com/BaeJi77/exmple-testcontainers-golang-test-server)

# 이전 글 요약

`integration test`는 여러 컴포넌트 간에 정상적으로 동작하는지 테스트하는 것이라고 말씀드렸습니다.

여러 컴포넌트의 의미로는 (1) 해당 애플리케이션 내부에 있는 여러 class, 여러 모듈 간에 관계일 수도 있으며 (2) 외부 컴포넌트 간에 관계일 수도 있습니다.

이전 글에서는 (2)에 대해서 구체적으로 이야기했습니다.

(2)을 하기 위해서 첫번째로 비슷한 동작을 하는 컴포넌트를 테스트 실행시마다 동작시키는 방법과 두번째로는 다른 컴포넌트를 실제로 동작하는 것에 대해서 말을 했습니다.

해당 두가지 모두 그것마다 모두 문제가 있었으며 해당 문제를 도와주는 `testcontainers`라는 라이브러리에 대해서 소개했습니다.

이전 글에 대해서 더욱 알고 싶다면 해당 [링크](https://baeji77.github.io/dev/integration-test-with-testcontainers-1/)를 클릭하세요!

# 목표

이번 글에서는 저번 글에서 작성했던 것처럼 `testcontainers` 어떻게 활용하고 있는지와 그리고 어떻게 더 활용하면 좋을것 같은 지에 대해서 이야기 나워볼 생각입니다. 

그리고 제가 사용한다면 궁금한 내용들을 정리해서 알려드리겠습니다.

# 사용 사례

제가 말씀드리는 사용 사례는 `testcontainers` 공식 홈페이지에서 소개된 케이스이며 제가 코드를 보면서 분석한 것이기에 틀릴 수 있습니다.

## golang project 

### [telegraf](https://github.com/influxdata/telegraf)

`telegraf`에 대해서 간단하게 설명을 드리자면 plugin기반으로 메트릭을 수집 및 전송하는 agent입니다. golang으로 작성되어서 동작합니다.

`telegraf`같은 경우 여러 application과 연관되어서 동작하는 경우가 많습니다. 간단하게 예시를 들자면 redis에서 메트릭을 수집해서 코드를 테스트한다던지 어떤 특정 데이터에서 수집된 내용을 카프카로 전송하고 싶을때 해당 application이 직접 동작할 때 테스트하기가 가장 쉽고 정확하다고 생각합니다.

- [testutil/container](https://github.com/influxdata/telegraf/blob/master/testutil/container.go#L24-L40)

`telegraf`같은 경우는 test할때 container을 쉽게 관리하기 위해서 해당 struct를 정의해서 사용했습니다. 

- [redis input plugin](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/redis/redis_test.go#L43-L50)
- [mongodb input plugin](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/mongodb/mongodb_server_test.go#L18-L25)
- [mongodb output plugin](https://github.com/influxdata/telegraf/blob/master/plugins/outputs/mongodb/mongodb_test.go#L24-L34)

## java project

### [skywalking](https://github.com/apache/skywalking) 

> an APM(application performance monitor) system, especially designed for microservices, cloud native and container-based architectures.

apache 프로젝트이면서 apm 서비스로 알려져있는 `skywalking`같은 경우에도 `testcontainers`를 이용해서 테스트를 진행하고 있습니다.

- [skywalking내에서 testcontainers검색](https://github.com/search?q=repo%3Aapache%2Fskywalking+testcontainers&type=code&p=1)

추가적으로 `skywalking`에서는 docker compose를 이용해서 테스트를 진행하기도 합니다. [링크](https://github.com/apache/skywalking/blob/ea0c1b93542f72706e0198fbcc8ff14ad682a8b9/oap-server/server-configuration/configuration-apollo/src/test/java/org/apache/skywalking/oap/server/configuration/apollo/ApolloConfigurationIT.java#L69-L79)

### [spring-session](https://github.com/spring-projects/spring-session)

`spring framework`에서 session과 관련한 모듈을 담당하는 spring-session에서도 사용하고 있는 것을 확인할 수 있습니다.

- [integration test code](https://github.com/spring-projects/spring-session/blob/3ed01cbf95c5ad9da8ef2356a6727e853c4642e7/spring-session-samples/spring-session-sample-boot-redis/src/integration-test/java/sample/BootTests.java#L24)

integration test라는 이름으로 동작하고 있으면 e2e 느낌으로 테스트를 진행하고 있다고 생각합니다.

java project중에 더 다양한 예제를 원하신다면 공식으로 제공해주는 [예시](https://github.com/testcontainers/testcontainers-java/tree/main/examples)를 한번 참고해보세요.

## 이런 것도 가능하지 않을까? (1)

내가 테스트하고 싶은 테스트는 공식적으로나 개발자가 제공해주는 docker image로 존재하지 않는다 혹은 내가 동작시키고 싶은 컨테이너는 내가 직접 만들었거나 팀이 만든 컨테이너이다. 이런 경우는 어떻게 해야될까?

해당 경우를 위해서 임시 테스트 서버를 만들었습니다. [링크](https://github.com/BaeJi77/exmple-testcontainers-golang-test-server)

- 임시로 동작할 서버를 만들었습니다.

```golang
func main() {
	w := gin.Default()

	w.GET("/", func(context *gin.Context) {
		fmt.Printf("request=%v", context.Request)

		context.Status(200)
	})

	w.Run(":10000")
}
```

- 해당 테스트 서버를 docker image로 만들었습니다.

``` bash
$ docker build -t testcontainers-go-test:1.0.0 .

$ docker images
REPOSITORY                                          TAG       IMAGE ID       CREATED          SIZE
testcontainers-go-test                              1.0.0     9e6811e8e2bd   5 minutes ago    994MB
```

- 실제 코드에서 해당 container를 동작시키고 그것에 대해서 요청을 보내봤습니다.
  - testcontainer 선언부분
```golang
req := testcontainers.ContainerRequest{
	Image:        "testcontainers-go-test:1.0.0",
	ExposedPorts: []string{"10000/tcp"},
}
container, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
	ContainerRequest: req,
	Started:          true,
})
```

  - testcontainers로 만들어진 서버로 요청보내는 부분
	```golang
	customContainers := RunCustomContainers(t)
	defer TearDownContainers(ctx, t, customContainers)

	endpoint, err := customContainers.Endpoint(ctx, "")
	if err != nil {
		t.Error(err)
	}

	res, err := http.DefaultClient.Get(fmt.Sprintf("http://%s", endpoint))
	if err != nil {
		t.Error(err)
		return
	}

	assert.Equal(t, res.StatusCode, http.StatusOK)
	```

- 테스트 결과

```
=== RUN   TestSimpleCustomContainers
2023/05/07 14:34:23 github.com/testcontainers/testcontainers-go - Connected to docker: 
  Server Version: 20.10.17
  API Version: 1.41
  Operating System: Docker Desktop
  Total Memory: 5939 MB
2023/05/07 14:34:23 Starting container id: 2087765a040f image: docker.io/testcontainers/ryuk:0.3.4
2023/05/07 14:34:24 Waiting for container id 2087765a040f image: docker.io/testcontainers/ryuk:0.3.4
2023/05/07 14:34:24 Container is ready id: 2087765a040f image: docker.io/testcontainers/ryuk:0.3.4
2023/05/07 14:34:24 Starting container id: b699697ae80d image: testcontainers-go-test:1.0.0
2023/05/07 14:34:24 Container is ready id: b699697ae80d image: testcontainers-go-test:1.0.0
--- PASS: TestSimpleCustomContainers (1.00s)
PASS
```

위에서 보신 것처럼 실질적으로 우리가 테스트를 같이 해보고 싶은 컴포넌트와도 이렇게 테스트를 진행할 수 있습니다.

### 개인적으로 생각하는 한계

방금 제가 만든 컴포넌트는 순전히 해당 컴포넌트는 다른 컴포넌트와의 의존성이 없습니다. 

하지만 실제 우리가 통합으로 테스트를 원하는 컴포넌트는 외부와의 의존성이 있을 수 있습니다. 그런 경우 이전에 제가 말했던 이슈가 발생하면서 테스트가 어려워집니다.

그런 경우를 방지하기 위해서 결국 해당 컴포넌트를 순전히 테스트만을 위한 container로 만들어야되며 실질적으로 독립적으로 동작하는 것이 어려울수도 있습니다.

구체적으로 우리가 alpha나 prod과 같은 환경 분리를 통해서 test-containers라는 환경을 만들어서 configuration 값을 변경하더라도 해당 컴포넌트와 연결되어 있는 외부 컴포넌트(db일수도 있고 또 다른 컴포넌트일수도 있는)와의 의존성을 끊기 위해서는 코드적으로 많은 작업을 해야될수도 있습니다.

이렇게 만들었다고해도 해당 컴포넌트가 변경되는 경우 계속 그 test-containers에 대한 contribution을 지속해야되는 부담도 있을 수 있다고 생각합니다.

## 이런 것도 가능하지 않을까? (2)

현재 의존성이 있는 DB를 업데이트하거나 변경을 해야되는 경우 테스트에 용이하게 사용할 수 있지 않을까 생각해봤습니다. 

소프트웨어는 업데이트를 하면서 이전 기능을 지원해주지 않기도 합니다. 혹은 새로운 기능을 제공해주기도 합니다. 

`testcontainers`는 동작하는 containers에 대한 버전을 지정할 수 있습니다. 이런 것들을 통해서 기존에 사용하고 있는 쿼리들을 테스트하면서 문제를 확인할 수 있지 않을까 싶었습니다.

당연히 이런 중요한 업무를 했을 경우 직접 쿼리를 날리거나 해당 기능에 대해서 모두 검증을 할 것이며 이런 업무가 지속적으로 발생하는 것이 아니기 때문에 꼭 이렇게 할 필요는 없다는 생각을 합니다.

그럼에도 주기적으로 반드시 업데이트를 해야된다면 반복적인 작업을 한번 만들어진 코드로 만들어진 테스트로 대체할 수 있다면 편리하지 않을까 생각했습니다.

혹은 정말 큰 일이지만 데이터베이스를 변경해야되는 경우 integration이나 e2e로 만들어진 테스트에서 검증할때 도움이 되지 않을까 생각했습니다. 

# 알아두면 좋은 것들

# FAQ

- 어떤 언어를 지원하나요?
  
`java`, `go`, `rust`, `.net`, `node.js`, `python`, `haskell`를 지원하고 있습니다. 그 중에서 `java`가 가장 인기가 많은 것으로 보입니다.


- 어떤 환경에 동작하나요?
  
사용하는 언어마다 다른 것으로 보입니다. 하지만 기본적으로는 `Docker-API compatible container runtime`이 필요하다고 나옵니다.


- 유닛 테스트에서 사용하는 건 어떤가요?
  
유닛 테스트에서 해당 라이브러리를 쓰는 것은 목적과 맞지 않을수도 있습니다. 유닛 테스트는 모듈 자체적으로 기능을 제공할 수 있는지를 검증하는 테스트 중에 하나라고 생각합니다.


- 성능적으로 이슈는 없나요?
  
발생할 수 있습니다. 무엇보다 실질적으로 container를 동작시키는 것이기 때문에 테스트마다 발생하는 시간은 다르겠지만 단순히 코드로만 동작하는 테스트보다는 오래걸릴 것으로 예상합니다.

테스트마다 컨테이너를 만들도록 한다면 더 오래걸릴 것으로 예상합니다. 그런 것을 방지하기 위해서 테스트마다 옵션을 잘 조절해서 테스트를 진행해야된다고 생각합니다.

# 마무리

이번에는 실질적으로 여러 프로젝트에서 어떻게 사용하고 있는지에 대해서 알아봤습니다. 이번 글을 쓰면서 조금 더 다양한 생각이 들었고 그것을 직접 사용해본 경험도 좋았던 것 같습니다.

당연히 오픈소스 말고도 더 높은 수준으로 사용하고 있는 프로젝트도 있을 것이라고 생각합니다만 이 글을 통해서 글을 읽으신 분들이 여러 방법에 대해서 생각할 수 있는 기회였으면 좋겠다고 생각합니다.
