---
title: "Golang에서 build를 os별로 하기 위해서는? (+ release phase별로 빌드하기)"
description: ""

categories: 
  - dev
  - golang
tags:
  - dev
  - golang
  - build
---

- [예제 소스코드](https://github.com/BaeJi77/blog-code/tree/main/2021-12/golang-build)

# cross compiler

> 크로스 컴파일러는 컴파일러가 실행되는 플랫폼이 아닌 다른 플랫폼에서 실행 가능한 코드를 생성할 수 있는 컴파일러이다. (출처: wiki)

wiki에서 설명하듯 현재 개발하고 있는 환경과 별개로 내가 프로그램을 동작하는 OS에 맞게 컴파일을 할 수 있게 해주는 컴파일러입니다. 

만약 우리가 개발하고 있는 리눅스 환경이 아닌 윈도우에서나 다른 리눅스 환경에서 동작하게 하고 싶은 경우 다른 바이너리 파일을 만들 수 있다는 것입니다.

## golang의 cross compile

golang에서는 기본적으로 환경변수가 설정되어 있고 환경에서 어떤 OS에서 동작할 것인지에 대한 설정을 하여서 빌드를 할 수 있습니다.

``` bash
$ go env
...
GOOS="darwin"
GOARCH="amd64"
...
```

`$ go env` 를 하게 되면 현재 내가 가지고 있는 go에 대한 환경변수가 나옵니다. 그 중에서 `GOOS`, `GOARCH`에 값을 변경하게 되면 특정 OS나 CPU아키텍처에서 동작하는 프로그램을 빌드할 수 있습니다. (기본값은 현재 local에 대한 값으로 설정되어 있습니다.)

만약 이것을 cli로 빌드시에 변경하고 싶다면 어떻게 해야될까요?

``` bash
$ GOOS=darwin GOARCH=amd64 go build ...
```

위와 같이 환경변수를 변경하여서 build를 할 수 있습니다.


# 목적

나의 상황은 사실 조금 다랐습니다. 특정 OS에 맞게 바이너리 파일을 만들 수 있었지만 특정 OS가 동작할 때 빌드되는 파일을 변경하고 싶었습니다.

linux에서 동작하는 파일과 window에서 동작하는 파일이 다르게 하고 싶었습니다. (그렇게 해야만 했습니다.)

그런 경우 `go build constraint` 을 통해서 가능합니다.

## build constraint

[공식 문서](https://pkg.go.dev/cmd/go#hdr-Build_constraints)

특정 파일에 대한 빌드 제약을 걸고 싶다면 파일 내용 가장 앞에 주석을 달면 됩니다.

- In file
``` go
//go:build
// +build
```

- CLI
``` $
$ go build -tags=... xxxxx
```

첫번째 같은 경우 1.16 버전보다 위에 있는 버전에 대해서 지원하고 있으면 (1.17부터 지원) 두번째 같은 경우 1.16 버전 이하에 대해서 지원합니다. 

그렇다고 해서 1.17에서 두번째 방법을 지원하지 않는 것은 1.17버전에서도 저렇게 작성하는 것은 가능합니다. 

> //go:build lines
> 
> The go command now understands //go:build lines and prefers them over // +build lines. The new syntax uses boolean expressions, just like Go, and should be less error-prone. As of this release, the new syntax is fully supported, and all Go files should be updated to have both forms with the same meaning. To aid in migration, gofmt now automatically synchronizes the two forms. For more details on the syntax and migration plan, see https://golang.org/design/draft-gobuild.

위에 내용은 1.17 릴리즈 노트에 나와있는 내용을 가져와봤습니다.

### 실전

1. Only linux
``` 
//go:build linux
```
2. linux or drawin
``` 
//go:build linux drawin
```
3. Not linux 
``` 
//go:build !linux
```
4. linux and 386
```
//go:build (linux && 386)
```


- 특정 파일에서 내가 컴파일하지 않은 파일이 있는 경우에는 어떻게 될까?

![](/assets/images/2021-12-05-golang-build/golang-build.png)

``` bash
$ go build -tags=drawin                                                                                                                              (imon/lens)
# golang-build
./main.go:9:2: undefined: PrintLinux
```

drawin을 태그해서 빌드해보겠습니다. 과연 어떻게 될까요?

위에 보이는 것과 같이 해당 파일을 찾지 못하고 에러가 발생합니다. 

### tip

팁이라고 할수도 있고 아닐수도 있지만 공식문서에 파일 naming과 관련된 내용이 나옵니다.

``` 
*_GOOS
*_GOARCH
*_GOOS_GOARCH
```

그러니까 예를 들어서 `source_windows_amd64.go` 이렇게 되는 경우 window에 amd64를 명확하게 보여주자는 것입니다. 단순히 글을 읽을 때는 안이상했는데 과연 naming만 지키고 주석을 지운다면 어떻게 될까나?

실제로 주석을 지우고 실행해도 잘 동작합니다. 그러니까 파일 naming을 지킨 이후에 `// +build`를 하지 않아도 제대로 빌드가 됩니다.

역으로 파일 naming을 지키지 않아도 `// +build`만 추가하여도 제대로 동작하게 됩니다.

주의사항으로는 `_test`와 같은 접미사가 없는 경우에 된다고 적혀있습니다.

## release phase별로 build하기

이거는 제대로 권장하는지는 모르겠습니다. 이와 같이 test와 release 환경에 대해서 동작하는 파일을 다르게 하고 싶을 수 있다고 생각합니다.

``` go
테스트 환경
//go:build test

릴리즈 환경
//go:build release
```

``` bash
$ go build -tags=release
```

이렇게 실행시키면 자신이 원하는대로 특정 phase에 대한 파일만 빌드할 수 있다고 생각합니다!

- 만약 파일을 그냥 배재하고 싶다면?

``` go
//go:build ignore
```

이거는 태그와 별개로 완벽하게 빌드에서 제외하게 됩니다.

# ref

- [공식 문서](https://pkg.go.dev/cmd/go#hdr-Build_constraints)
- https://go.dev/doc/go1.17
- https://pkg.go.dev/go/build

