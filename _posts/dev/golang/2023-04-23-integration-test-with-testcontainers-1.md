---
title: "integration test과 testcontainers (1)"
description: ""

categories: 
  - dev
tags:
  - dev
  - test
  - testing
  - testcontainers
---

글에 대한 코드는 [링크](https://github.com/BaeJi77/exmple-testcontainers-golang)를 통해서 볼 수 있습니다.

# integration test란

> Integration testing (sometimes called integration and testing, abbreviated I&T) is the phase in software testing in which individual software modules are combined and tested as a group. 
> 
> Integration testing is conducted to evaluate the compliance of a system or component with specified functional requirements.
> 
> It occurs after unit testing and before system testing. 
> 
> Integration testing takes as its input modules that have been unit tested, groups them in larger aggregates, applies tests defined in an integration test plan to those aggregates, and delivers as its output the integrated system ready for system testing.

출처: [위키페디아](https://en.wikipedia.org/wiki/Integration_testing)

`unit test`는 해당 컴포넌트 내부에서 명확하게 해당 기능이 동작하는지를 테스트하기 위해서 진행했다면, 

`integration test`는 여러 컴포넌트 간에 정상적으로 기능이 동작하는지 테스트를 한다고 생각이 듭니다. 

## 내가 생각하는 integration test는

여러 component간 연결을 하기 때문에 해당 application과 연결된 다른 application이 필요하기도 하며 DB를 직접 접근하는 경우에는 DB가 필요하기도 합니다.

이런 경우 test를 진행하기 전에 configuration을 조정하여서 우리가 임시로 동작하게 만든 동작을 비슷하게 하는 application이나 db를 대신 호출할 수 있도록 하기도 합니다. (예시로는 test 과정에서 test server를 동작시키거나 application 내부 db를 동작시키는 것을 생각합니다.)

하지만 이런 경우 실제와 다른 동작을 해서 test가 정상적으로 되지 않은 경우가 있을 수 있으며 그런 과정에서 테스트에 대한 신뢰도가 떨어질수도 있습니다. (spring에서 사용하는 로컬db인 h2같은 경우 rdb처럼 동작하지만 실제로 mysql이나 여러 rdb와는 다른 부분이 존재합니다.)

그런 문제점을 해결하기 위해서 실제로 해당 application이나 db를 실행시키는게 좋지만 그것 또한 쉽지 않은 일입니다.

test를 위해서 해당 application나 db를 계속 동작시킨다면 리소스 낭비 및 실제로 테스트가 실패할 수 있습니다. 계속 실행시켜둬야되는 이유로는 테스트 코드 이전에 실행을 멈춰있다가 테스트 실행 전에 실행시키는 것이 매우 어렵다라고 판단합니다.

그리고 db같은 경우 데이터를 계속 적재한다거나 혹은 그렇지 않다고 하더라도 동시에 여러곳에서 테스트를 하게 되면 테스트가 정상적으로 동작하지 않을 수 있습니다.

# testcontainers란

- [Testcontainers 공식문서](https://www.testcontainers.org/)
- [Testcontainers github Organizations](https://github.com/testcontainers)

Testcontainer는 docker를 이용해서 test code 상에서 특정 docker image를 동작시킬 수 있도록 해주는 library입니다. docker container 실행에 대해서 여러 옵션을 줄 수 있습니다. (test시마다 container을 죽이고 뛰울 수 있으며 계속 하나의 container가 동작하게도 가능합니다.)

여러 언어를 지원하고 있으며 `java`, `go`, `rust`, `.net`, `node.js`, `python`, `haskell`를 지원하고 있습니다. `testcontainer-java`가 github star가 가장 많이 있습니다. 

## Testcontainer의 장점 in official document
- Data access layer integration tests: use a containerized instance of a MySQL, PostgreSQL or Oracle database to test your data access layer code for complete compatibility, but without requiring complex setup on developers' machines and safe in the knowledge that your tests will always start with a known DB state. Any other database type that can be containerized can also be used.
- Application integration tests: for running your application in a short-lived test mode with dependencies, such as databases, message queues or web servers.
- UI/Acceptance tests: use `containerized web browsers`, compatible with Selenium, for conducting automated UI tests. Each test can get a fresh instance of the browser, with no browser state, plugin variations or automated browser upgrades to worry about. And you get a video recording of each test session, or just each session where tests failed.
- Much more! Check out the various contributed modules or create your own custom container classes using `GenericContainer` as a base.

## Testcontainers와 integration test

제가 생각하는 integraion test를 준비하면서 생기는 문제들에 대해서 Testcontainers가 많은 부분을 해결해 준다고 생각합니다.

1. 간단하게 테스트하기 위해서 가짜로 동작하는 컴포넌트를 만들어서 테스트를 해야했음 -> 실제로 동작하는 컴포넌트를 코드 몇줄로 생성 가능.
2. 완벽한 테스트를 위해 실제 컴포넌트 관리에 대한 어려움과 리소스 낭비 -> 테스트때 마다 실제로 동작하는 컴포넌트이지만 임시적으로 만들기 때문에 관리 및 리소스 낭비를 많이 줄임.
3. 실제 컴포넌트 동작시 테스트가 정상적으로 되지 않을 수 있음(데이터가 축적되거나 동시에 테스트가 동작할 경우) -> 테스트마다 혹은 test group에 따라서 컴포넌트 라이프 사이클을 핸들링할 수 있기 때문에 해당 문제 해결 가능.

## 요구사항

특정 언어마다 요구사항이 다른것 같으며 어떤 환경에서 실행하냐에 따라서 요구사항이 다릅니다. 그럼에도 가장 기본적으로는 `docker`을 실행할 수 있는 환경이어야 됩니다.

- [Testcontainers-go system_requirements](https://golang.testcontainers.org/system_requirements/)
- [Testcontainers-java system_requirements](https://www.testcontainers.org/supported_docker_environment/)

## 사용법

Testcontainers-go를 이용해서 간단한 예시를 보여드리겠습니다.

### install

``` go
$ go get github.com/testcontainers/testcontainers-go
```

### Simple Redis containers

``` go
func RunRedisContainers(t *testing.T) testcontainers.Container {
	ctx := context.Background()
	req := testcontainers.ContainerRequest{
		Image:        "redis:latest",
		ExposedPorts: []string{"6379/tcp"},
		WaitingFor:   wait.ForLog("Ready to accept connections"),
	}
	redisC, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
		ContainerRequest: req,
		Started:          true,
	})
	if err != nil {
		t.Error(err)
	}

	return redisC
}

func TearDownContainers(ctx context.Context, t *testing.T, containers testcontainers.Container) {
	if err := containers.Terminate(ctx); err != nil {
		t.Fatalf("failed to terminate container: %s", err.Error())
	}
}
```

### Test code

``` go
func TestSimpleRedis(t *testing.T) {
	ctx := context.Background()
	redisC := RunRedisContainers(t)
	defer TearDownContainers(ctx, t, redisC)

	endpoint, err := redisC.Endpoint(ctx, "")
	if err != nil {
		t.Error(err)
	}

	client := redis.NewClient(&redis.Options{
		Addr: endpoint,
	})

	key := "hello"
	value := "world"

	client.Set(ctx, key, value, time.Minute)
	got, _ := client.Get(ctx, key).Result()

	assert.Equal(t, got, value)
}
```

## testcontainer의 옵션값.

### testcontainers.ContainerRequest

어떤 Container를 만들지에 대한 정보를 넣습니다. 

가장 중요한 정보로는 Container image에 대한 정보, Port number, CMD, EntryPoint와 같은 정보들을 모두 넣을 수 있습니다.

심지어 Dockerfile이 있는 경우 dockerfile을 읽어서 실행하는 것도 가능합니다.

### testcontainers.GenericContainerRequest

`testcontainers.ContainerRequest`를 기본적으로 가지고 있으며 거기에 `Started`, `ProviderType`, `Logger`, `Reuse` 이 4가지가 추가객 struct입니다.

다른 fields 값들은 특별한 경우가 아니면 사용할 일이 없다고 생각합니다. 

`Reuse`라는 필드값을 true로 하는 경우에 container name이 똑같은 것이 동작하고 있다면 해당 container를 사용하게 됩니다.

# 결론

오늘은 `Testcontainers`라는 test library에 대해서 간단하게 알아봤습니다. 이전에 integraion test를 어떻게 하지라는 생각을 많이 했었는데 해당 라이브러리를 사용하면 편하게 할 수 있겠구나 라는 생각을 하게 된 것 같습니다.

오늘은 간단하게 설명드렸지만 다음에는 어떻게 활용되고 있는지와 어떻게 더 활용할 수 있는지에 대해서 구체적으로 다뤄보겠습니다.
