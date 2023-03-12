---
title: "golang with testing 1 (feat. golang testing package)"
description: ""

categories: 
  - dev
  - golang
tags:
  - dev
  - golang
  - test
  - testing
---

글에 대한 코드는 [링크](https://github.com/BaeJi77/golang-testing)를 통해서 볼 수 있습니다.

# Test

## Test는 무엇인가?

> Software testing is the act of examining the artifacts and the behavior of the software under test by validation and verification. Software testing can also provide an objective, independent view of the software to allow the business to appreciate and understand the risks of software implementation. 
>
> 출처: https://en.wikipedia.org/wiki/Software_testing

위에 있는 내용은 wiki에서 가져온 `software testing`에 대한 글입니다. 

간단하게 말하면 테스트는 `검증 및 확인을 통해 소프트웨어의 아티팩트 및 동작을 검사하는 행위`입니다.

## 왜 Test을 하는가?

해당 부분에 대해서는 사람마다 장점이라고 생각하는 부분이 다를 수 있다고 생각합니다. 그럼에도 여러 글을 읽고 제가 생각하는 이유에 대해서 적어보겠습니다.

테스트 코드가 잘 만들어졌든 그렇지 않든 어느정도 기능을 했을 때를 가정하겠습니다.

1. 유지보수성 향상

특정 기능에 대한 변경을 해야되는 경우에 Test가 없다면 이전에 제공해주고 있는 기능에 대해서 변경 이후에도 제대로 동작하는지에 대한 보장을 하기 어렵습니다.

2. 코드 및 기능 이해

단순 코드만을 보고 코드가 제공해주는 기능을 이해하기 어려운 경우가 있습니다. 그런 경우 테스트 코드가 도움을 줄 수 있다고 생각합니다.

3. 기능 동작 보장

기능 개발에 있어서 해당 동작이 제대로 돌아가는 것을 실제로 동작시키지 않고도 확인하여 버그를 줄일 수 있다고 생각합니다.

## 어떤 종류가 있는가?

Test의 종류를 말하기 위해서는 테스트를 보는 접근법에 대해서 이야기할 수 있으며 Test를 진행하는 `Level`에 대해서도 이야기할 수 있습니다.

저는 여기서는 Level에 대해서 이야기해보겠습니다.

- Unit test
- Integation test
- E2E test

이번 글에서는 `Unit test`에 대해서 자세하게 말해보겠습니다.

# golang에서는 어떻게 Test를 하는가?

golang에서는 기본적으로 제공해주는 `testing package`가 있습니다. [godoc](https://pkg.go.dev/testing)

기본적으로 제공해주는 패키지를 제외하고 여러 라이브러리가 있습니다. Unit testing을 도와주는 라이브러리입니다. [testify](https://github.com/stretchr/testify), [ginkgo](https://github.com/onsi/ginkgo), [gomega](https://github.com/onsi/gomega)

추가적으로 팀원의 공유로 알게된 integration test를 도와주는 [testcontainer-go](https://github.com/testcontainers/testcontainers-go) 도 있습니다.

오늘은 golang에서 기본적으로 제공해주는 `testing package`에 대해서 이야기해보겠습니다.

# [testing package](https://github.com/golang/go/tree/master/src/testing)

golang에서 test를 하고 싶은 경우 test file에 대해서 `_test`에 대한 suffix을 붙이면 됩니다. ex) `main_test.go`

test file에서 test case를 추가하고 싶은 경우 함수명을 `TestXxx`로 만들면 됩니다. Xxx에서 X는 반드시 대문자여야 합니다. (소문자로 하는 경우 test case라고 생각하지 않아서 인식이 되지 않습니다.)

## test - [testing.T](https://pkg.go.dev/testing#T)

> T is a type passed to Test functions to manage test state and support formatted test logs.

지금까지 말한 test case로서 test를 실행시키기 위해서는 해당 `testing.T`를 사용해야 됩니다.

```golang
func TestXxx(t *testing.T) {...}
```

`testing.T`에는 다양한 메소드를 가지고 있습니다.

```golang
func TestT(t *testing.T) {
	t.Run("sub-test-1", func(t *testing.T) {
		fmt.Println("sub-test-1")
	})

	t.Error()
	t.Errorf("error string format")

	t.Deadline()

	t.Fail()
	t.FailNow()
	t.Failed()

	t.Skip()
	t.Skipf("skip string format")
	t.SkipNow()
	t.Skipped()

	t.Helper()
}
```

이것 말고도 `Log()`와 같은 메소드들을 추가로 지원하고 있습니다.

`Run()`는 test method 안에서 추가적인 sub test를 실행시킬 때 사용됩니다.

`Helper()`는 실제로 테스트 실행이 아닌 test에 대한 설명을 도와주는 역할을 합니다.

## benchmark - [testing.B](https://pkg.go.dev/testing#B)

> B is a type passed to Benchmark functions to manage benchmark timing and to specify the number of iterations to run.

```golang
func BenchmarkXxx(b *testing.B) {...}
```

특정 메소드와 관련했거나 특정 상황에 대해서 벤츠마크 측정을 하고 싶은 경우에 `testing.B`를 이용하면 편합니다.

```golang
func BenchmarkRandInt(b *testing.B) {
    for i := 0; i < b.N; i++ {
        rand.Int()
    }
}

---
BenchmarkRandInt-8   	68453040	        17.8 ns/op

```

commandline을 통해서 실행을 원하시는 경우 이렇게 실행해보십니오.

```bash
$ go test -bench .
```

golang test에 대한 beachmark에 대해서 모두 실행할 수 있을 것 입니다.

자세한 옵션에 대해서는 [Testing flags](https://pkg.go.dev/cmd/go#hdr-Testing_flags)을 통해서 확인할 수 있습니다.

## coverage

테스트 코드에 대한 전체적인 coverage에 대해서 궁금할 수 있는데 그 부분에 대해서는 test를 실행할 때 하나의 옵션만 추가하면 됩니다.

``` bash
$ go test -cover
```

``` bash
$ go test -cover -coverprofile cover.out .
```

위와 같이 `-coverprofile`이라는 옵션을 추가하게 되면 coverage에 대한 정보를 파일로도 만들어줍니다.

```text
mode: set
github.com/BaeJi77/golang-testing.git/main.go:5.13,7.2 1 0
```

옵션에 대해서는 밑에 영어 설명과 같이 작성해두겠습니다.

```text
-cover
    Enable coverage analysis.
    Note that because coverage works by annotating the source
    code before compilation, compilation and test failures with
    coverage enabled may report line numbers that don't correspond
    to the original sources.

-coverprofile cover.out
    Write a coverage profile to the file after all tests have passed.
    Sets -cover.
```

## TestMain

testing 패키지에 대해서 test 실행에 대해서 실행 이전과 이후에 대해서 핸들링을 할 수 있는 함수입니다. package 내부에서 오직 하나만 존재해야만 합니다.

한마디로 패키지 내에 존재하는 test 실행에 있어서 before과 after에 대해서 처리할 수 있는 함수입니다. 보는 것과 같이 `beforeEach`와 `afterEach`와 같이 동작하지는 않습니다.

```golang
func TestMain(m *testing.M) {
	fmt.Println("before Run()")
	code := m.Run()
	fmt.Println("After Run()")

	os.Exit(code)
}

func TestFoo(t *testing.T) {
	fmt.Println("TestFoo")
}

func TestBar(t *testing.T) {
	fmt.Println("TestBar")
}
---

before Run()
=== RUN   TestFoo
TestFoo
--- PASS: TestFoo (0.00s)
=== RUN   TestBar
TestBar
--- PASS: TestBar (0.00s)
PASS
After Run()
```

# 마무리

이번에는 golang에서 존재하는 standard package로 `testing`에 대한 설명을 했습니다.

기본적으로 제공하다보니 범용적으로 사용할 수 있는 기능들이 많이 있습니다. 추가적으로 test를 하는데 추가적인 기능이 필요하다면 `testify`와 같은 라이브러리에 대해서 알아보는 것을 추천합니다.
