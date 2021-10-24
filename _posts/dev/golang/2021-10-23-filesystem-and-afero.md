---
title: "golang에서 filesystem api를 더 잘 활용해보자 (feat. spf13/afero)"

categories:
  - dev
  - golang
tags:
  - dev
  - golang
  - filesystem
  - spf13
  - afero
---

# 개요

최근 golang을 이용해서 filesystem을 활용해야되는 경우가 많이 있었습니다. filesystem을 조금 더 잘 사용해 보기 위해서 사용해본 라이브러리에 대해서 설명하겠습니다.

# golang에서 filesystem

기본적으로 `os` 패키지를 이용해서 파일을 열고 폴더를 만드는 과정을 진행할 수 있습니다. 

그리고 `io/ioutil` 패키지를 사용하게 되면 파일을 읽거나 쓰기를 하는데 필요한 함수를 제공했었습니다. 1.16 버전에서는 조금 변경이 있었습니다.

> https://golang.org/doc/go1.16#ioutil

``` go
Discard => io.Discard
NopCloser => io.NopCloser
ReadAll => io.ReadAll
ReadDir => os.ReadDir (note: returns a slice of os.DirEntry rather than a slice of fs.FileInfo)
ReadFile => os.ReadFile
TempDir => os.MkdirTemp
TempFile => os.CreateTemp
WriteFile => os.WriteFile
```

패키지 이동과 함께 이름이 바뀌는 변경이 있었습니다. 실제 기능에 대해서 바뀌지 않았을 것이라고 생각합니다.

그리고 1.16 버전에서는 `fs.FS` 라는 새로운 타입이 나왔습니다. Open을 지원해야되는 정말 가벼운 type입니다. 그것을 통해서 `File` 이라는 데이터를 받을 수 있습니다. 해당 데이터는 `Read()`, `Stat()`, `Close()`를 지원합니다.

`fs.FS`와 관련해서 몇개 타입을 지원하지만 `write`에 대한 메소드를 지원하지 않습니다.

# afero

[link](https://github.com/spf13/afero)

`spf13` 라는 사람이 만든 filesystem를 추상화한 라이브러리입니다. (여러 다양한 golang 라이브러리를 만들었습니다.)

golang에서 기본적으로 제공하는 `os`, `io` 패키지에서 제공하는 메소드를 모두 제공한다고 생각하면 됩니다. 

## 기능

``` md
- A single consistent API for accessing a variety of filesystems
- Interoperation between a variety of file system types
- A set of interfaces to encourage and enforce interoperability between backends
- An atomic cross platform memory backed file system
- Support for compositional (union) file systems by combining multiple file systems acting as one
- Specialized backends which modify existing filesystems (Read Only, Regexp filtered)
- A set of utility functions ported from io, ioutil & hugo to be afero aware
- Wrapper for go 1.16 filesystem abstraction `io/fs.FS`
```

위에 있는 내용은 github에 있는 내용을 가져온 것입니다. 

제가 생각하기에 큰 장점은 interface를 제공하기 filesystem에 대한 추상화하고 backend에 대해서 자신이 원하는 것을 적용할 수 있는 부분이라고 생각합니다.

그것과 함께 여러 filesystem을 실제로 지원을 하고 있습니다. 그러기에 기본적인 fs말고도 여러 fs을 적용할 수 있습니다.

## 선택 이유

추상화라는 큰 장점이 있지만 무엇보다 backend를 내가 자유롭게 선택할 수 있다는 점이 있었습니다. 

그것과 함께 1.16에 존재하는 filesystem에 대해서 메모리에 독립적인 filesystme처리 동작하도록 하는 방식이 있는데요. 그것에 대해서 기본적으로 제공하고 있었다라고 생각합니다.

- [afero memory backend](https://github.com/spf13/afero#memory-backed-storage)
- [Using Afero for Testing](https://github.com/spf13/afero#using-afero-for-testing)

- Afero For test
  - Much faster than performing I/O operations on disk
  - Avoid security issues and permissions
  - Far more control. 'rm -rf /' with confidence
  - Test setup is far more easier to do
  - No test cleanup needed

메모리에서 완벽하게 동작하기 때문에 그것을 통해 내가 구현한 파일시스템에 대해서 테스트에 매우 용이하게 사용할 수 있습니다. 그리고 어떤 테스트 끼리 간섭이 없고 그것을 위해서 따로 처리하지 않아도 됩니다. 그리고 완벽하게 cleanup되기 때문에 편리하게 사용가능합니다.

## 적용

### step 1
``` bash
$ go get github.com/spf13/afero
```

`go get` 을 통해서 해당 라이브러리를 가져온다.

### step 2

``` go
var fs = afero.NewMemMapFs()

---

var fs = afero.NewOsFs()
```

자신이 원하는 backend에 대해서 구현체를 만들 수 있습니다.

## Test

``` go
func TestExist(t *testing.T) {
	appFS := afero.NewMemMapFs()
	// create test files and directories
	appFS.MkdirAll("src/a", 0755)
	afero.WriteFile(appFS, "src/a/b", []byte("file b"), 0644)
	afero.WriteFile(appFS, "src/c", []byte("file c"), 0644)
	name := "src/c"
	_, err := appFS.Stat(name)
	if os.IsNotExist(err) {
		t.Errorf("file \"%s\" does not exist.\n", name)
	}
}
```

위에 있는 코드는 github code에 존재하는 `afero.NewMemMapFs()` 와 관련된 코드입니다.

테스트 진행 이후에 어떤 파일도 남지 않고 그것과 함께 os와 독립적으로 진행할 수 있기에 테스트 진행에 장점이 있습니다.

## 생각

해당 라이브러리에 대해서 너무 큰 interface를 가지고 있는 것에 대해서 아쉽다는 이야기에 대한 글을 읽은 적도 있었습니다.

하지만 1.16에서 개편한 fs에 대해서 아쉬운 점이 있었기에 충분히 장점이 있다는 생각이 들었습니다. 

# 소감

최근 filesystem과 관련해서 작업을 해야되는 경우가 많았고 그것에 대해서 테스트 진행이나 filesystem을 어떻게 관리할지에 대한 고민이 있었습니다.

공식 패키지가 아니지만 그럼에도 충분히 신뢰성이 높은 오픈소스라는 생각이 들었고 fs를 할 때 자주 사용할 것 같다는 생각을 했습니다.

