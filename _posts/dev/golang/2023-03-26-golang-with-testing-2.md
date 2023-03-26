---
title: "golang with testing 2 (feat. testify)"
description: ""

categories: 
  - dev
  - golang
tags:
  - dev
  - golang
  - test
  - testing
  - testify
---

글에 대한 코드는 [링크](https://github.com/BaeJi77/golang-testing)를 통해서 볼 수 있습니다.

# 이전 글
1. https://baeji77.github.io/dev/golang/golang-with-testing/

# 소개

이전 글에도 말 했듯이 테스트는 소프트웨어 개발의 필수적인 부분입니다. 좋은 테스트를 작성하면 코드가 예상대로 작동하고 프로덕션에서 버그가 발생할 가능성이 줄어듭니다. 이번 글에서는 테스트 작성을 위한 Golang 라이브러리인 `Testify`에 대해 설명합니다. `Testify`는 테스트 작성을 더 쉽고 효율적으로 만드는 몇 가지 기능을 제공합니다.

# `Testify`이란 무엇입니까?

- [testify github link](https://github.com/stretchr/testify)

`Testify`는 테스트 작성을 위한 일련의 유틸리티를 제공하는 유명한 Golang 라이브러리입니다. 소프트웨어 개발을 위한 오픈 소스 라이브러리 및 도구를 제공하는 회사인 Stretchr에서 개발했습니다. Testify는 assertion, mock 및 test suite를 포함하여 테스트 작성을 위한 풍부한 기능을 제공합니다. golang을 사용하는 많은 사람들이 사용하고 있으며 **19K star**를 보여하고 있습니다. (2023.03.26 기준)

# `Testify` 시작하기

`Testify`를 시작하려면 먼저 설치해야 합니다. 다음 명령을 사용하여 go get을 사용하여 Testify를 설치할 수 있습니다.

```bash
$ go get github.com/stretchr/testify
```

## `Testify`의 assertion

assertion은 `Testify`의 핵심 기능입니다. 이를 통해 코드가 예상대로 작동하는지 확인할 수 있습니다. `Testify`는 테스트 중인 값 유형에 따라 사용할 수 있는 여러 어설션 함수를 제공합니다. 일반적으로 사용되는 어설션 함수 중 일부는 다음과 같습니다.

- `assert.Equal`: 두 값이 같은지 확인합니다. 단순 유형에는 == 연산자를 사용하고 복합 유형에는 reflect.DeepEqual 함수를 사용합니다.
- `assert.NotEqual`: 두 값이 같지 않은지 검사합니다.
- `assert.Nil`: 값이 nil인지 확인합니다.
- `assert.NotNil`: 값이 nil이 아닌지 확인합니다.
- `assert.True`: 값이 true인지 확인합니다.
- `assert.False`: 값이 false인지 확인합니다.
- `assert.Error`: 함수 호출이 오류를 반환하는지 확인합니다.
- `assert.NoError`: 함수 호출이 오류를 반환하지 않는지 확인합니다.

```go
func errorFunc() error {
	return fmt.Errorf("hello world")
}

func noErrorFunc() error {
	return nil
}

func TestAssert(t *testing.T) {
	a := assert.New(t)

	a.Equal(1, 1, "1 should be equal to 1")
	a.NotEqual(1, 2, "1 should not be equal to 2")
	a.Nil(nil, "nil should be nil")
	a.NotNil("not nil", "not nil should not be nil")
	a.True(1 == 1, "1 should be equal to 1")
	a.False(1 == 2, "1 should not be equal to 2")

	err := errorFunc()
	a.Error(err, "someFuncThatReturnsAnError should return an error")

	err = noErrorFunc()
	a.NoError(err, "someFuncThatDoesNotReturnAnError should not return an error")
}
---

=== RUN   TestAssert
--- PASS: TestAssert (0.00s)
PASS
```

## `Testify`의 Mock

테스트에서 모킹은 종속성을 실제 종속성의 동작을 시뮬레이션하는 개체로 대체하는 데 사용되는 기술입니다. 이를 통해 개발자는 시스템의 다른 부분에 의존하지 않고 격리된 코드 조각을 테스트할 수 있습니다.

`Testify`는 mock이라는 패키지를 제공합니다. 다음은 mock 패키지를 사용하는 방법의 예입니다.

### 기본적인 Mock 사용법

```go
type Database interface {
	Get(key string) (string, error)
}

type MockDatabase struct {
	mock.Mock
}

func (m *MockDatabase) Get(key string) (string, error) {
	args := m.Called(key)
	return args.String(0), args.Error(1)
}

func TestGetData(t *testing.T) {
	mockDB := new(MockDatabase)
	mockDB.On("Get", "foo").Return("bar", nil)

	data, _ := mockDB.Get("foo")

	if data != "bar" {
		t.Errorf("Unexpected data: %s", data)
	}

	mockDB.AssertExpectations(t)
}
```

이 예제에서 MockDatabase 구조체는 목킹 프레임워크의 핵심 기능을 제공하는 `mock.Mock` 개체를 포함합니다. 

Get 메소드는 Database 인터페이스를 만족하도록 정의되었으며 `mock.Called` 메소드를 사용하여 함수에 전달된 인수를 기록하고 테스트 케이스에 지정된 반환 값을 반환합니다.

`On` 메소드는 Get 메소드의 예상 인수 및 반환 값을 지정하는데 사용됩니다. 테스트 종료 시 `AssertExpectations` 메서드가 호출되어 예상되는 모든 메서드 호출이 이루어졌는지 확인합니다.

### 의존성이 있는 경우에 대한 Mock

```go
type BComponent interface {
	Do() string
}

type RealComponent struct{}

func (r *RealComponent) Do() string {
	return "I'm real"
}

type MockBComponent struct {
	mock.Mock
}

func (md *MockBComponent) Do() string {
	args := md.Called()
	return args.String(0)
}

type AComponent struct {
	b BComponent
}

func (out *AComponent) CallBComponent() string {
	return out.b.Do()
}

func TestAComponent(t *testing.T) {
	// Create a mock dependency
	mockBComponent := new(MockBComponent)
	mockBComponent.On("Do").Return("mock result")

	// Inject the mock dependency into the object under test
	aComponent := AComponent{b: mockBComponent}

	// Test the object under test
	result := aComponent.CallBComponent()
	expectedResult := "mock result"
	if result != expectedResult {
		t.Errorf("Expected %s, but got %s", expectedResult, result)
	}

	// Verify that the mock dependency was called
	mockBComponent.AssertCalled(t, "Do")
}
```

많은 경우 Mock을 사용할 때는 의존성이 있는 경우라고 생각합니다. 

그런 경우에는 mock interface 구현체를 만들어서 의존성을 mock 구현체를 향하도록 변경해서 테스트할 수 있습니다.

## `Testify`의 `require`

```text
The require package provides same global functions as the assert package, but instead of returning a boolean result they terminate current test.

See t.FailNow for details.
```

위에 있는 문장은 testify의 README.md에 작성된 내용입니다. 

간단하게 말한다면 assert와 다른 것은 테스트 실패인 경우에 있어서 해당 테스트를 바로 중단한다는 것입니다.

## `testify`의 `sutie`

`suite`는 함께 그룹화된 테스트 모음입니다. 공통으로 설정되고 테스트 환경에 대한 release를 해야되는 테스트 세트가 있는 경우에 유용합니다. 

`Testify`에서 제품군은 `Suite` 인터페이스를 사용하여 정의되며 그것에 대해서 구현만 하게 되면 테스트 진행 중에 자동으로 실행됩니다. [interface code](https://github.com/stretchr/testify/blob/master/suite/interfaces.go)

```go

// Define the suite, and absorb the built-in basic suite
// functionality from testify - including assertion methods.
type ExampleTestSuite struct {
	suite.Suite
	value string
}

func (suite *ExampleTestSuite) SetupSuite() {
	fmt.Println("before test")
}

func (suite *ExampleTestSuite) SetupTest() {
	fmt.Println("before each test")
	suite.value = "hello"
}

func (suite *ExampleTestSuite) TearDownSuite() {
	fmt.Println("after test")
}

// before each test
func (suite *ExampleTestSuite) TearDownTest() {
	fmt.Println("after each test")
}

// All methods that begin with "Test" are run as tests within a
// suite.
func (suite *ExampleTestSuite) TestExample() {
	fmt.Println("test1")
	suite.Equal(suite.value, "hello")
}

func (suite *ExampleTestSuite) TestExample2() {
	fmt.Println("test2")
	suite.Equal(suite.value, "hello")
}

func (suite *ExampleTestSuite) TestExample3() {
	fmt.Println("test3")
	suite.Equal(suite.value, "hello")
}

// In order for 'go test' to run this suite, we need to create
// a normal test function and pass our suite to suite.Run
func TestExampleTestSuite(t *testing.T) {
	suite.Run(t, new(ExampleTestSuite))
}
---

=== RUN   TestExampleTestSuite
before test
=== RUN   TestExampleTestSuite/TestExample
before each test
test1
after each test
=== RUN   TestExampleTestSuite/TestExample2
before each test
test2
after each test
=== RUN   TestExampleTestSuite/TestExample3
before each test
test3
after each test
after test
--- PASS: TestExampleTestSuite (0.00s)
    --- PASS: TestExampleTestSuite/TestExample (0.00s)
    --- PASS: TestExampleTestSuite/TestExample2 (0.00s)
    --- PASS: TestExampleTestSuite/TestExample3 (0.00s)
PASS
```

# 결론

이번에 test를 할 때 유용하게 사용될 `testify`라는 golang에서 유명한 test library를 알아봤습니다. `assertion`, `require`을 통해서 더욱 정교하게 값에 대한 비교가 가능하며 `mock`를 통해서 의존성을 가지고 있는 코드를 해결할 수 있습니다. `suite`를 통해서 테스트 그룹핑과 그것을 쉽게 다룰수 있는 interface에 대해서 알아봤습니다. 

이전 `testing`에 비해 존재하지 않는 기능들도 존재하며 무엇보다 assertion이나 mock을 활용하면 더 좋은 테스트 코드를 만들때 좋은 도움이 될것이라고 생각이 들었습니다. 

조금 더 사용해보고 좋은 팁을 꼭 공유를 했으면 좋겠습니다.
