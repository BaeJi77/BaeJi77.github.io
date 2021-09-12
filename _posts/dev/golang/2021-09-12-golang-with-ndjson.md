---
title: "golang에서 ndjson 다루기"

categories:
  - dev
  - golang
tags:
  - dev
  - golang
  - ndjson
---

# Ndjson이란

[공식사이즈](http://ndjson.org/) 에 나와있는 내용을 토대로 작성을 하게 되면.

json 형식을 저장하지만 `\n` (new line)을 기반으로 나눠지도록 데이터가 저장된다. 한번 저장할 때 하나의 json이 저장된다고 생각하면 된다.

ndjson은 두가지를 만족해야 된다.

1. Line Separator is `\n`
2. Each Line is a Valid JSON Value

이번에는 json과 ndjson 모두 보여드리면서 설명드리겠습니다.

``` json
// json
[
    {
        "name": "baeji",
        "score": "1<<10"
    },
    {
        "name": "hoon",
        "score": "1<<10"
    }
]
---

// ndjson
{"name": "baeji","score": "1<<10"}
{"name": "hoon","score": "1<<10"}
```

첫번째 내용은 우리가 일반적으로 사용하는 json array의 형식이다. 배열 안에 json들이 저장되어 있는 형태이다.

두번째는 제가 말하려고 하는 ndjson 형식으로 어떤 배열에 표시가 없이 단순히 json의 형태가 `\n`로 구분되어지도록 만들어진다.

사실 자주 쓰이지 않으면 특정한 경우 데이터를 조금 쉽게 다루기 위해서 사용한다고 알고 있다. 

단순히 데이터를 편하게 쌓고 그것을 단순히 지속적으로 읽는 형식으로 사용되는 경우. 공식 홈페이지에서도 log와 같은 형식으로 사용될 수 있는 것이다.

# 나의 사용 목적

최근에 담당한 업무가 클라이언트 개발인데 해당 클라이언트를 테스트하기 위한 (데이터를 받아주기 위한) 테스트 서버가 필요했다. 

테스트 서버이었기에 클라이언트와 같이 뜨고 같이 없어지면 좋을 것 같다는 생각을 했고 주기적으로 데이터를 보내는 클라이언트 데이터를 저장하고 확인하기 위해서 테스트 서버를 만들었다.

단순 클라이언트가 아닌 agent느낌으로 개발자가 트리거를 주는것이 아닌 주기적으로 알아서 데이터를 보내는 형식이었다.


데이터는 파일로 저장했다. 메모리로 저장했어도 되었지만 나중을 위해서 파일로 처리했다.

# 실제 적용

[실제 소스 코드 참고](https://github.com/BaeJi77/blog-code/tree/main/2021-09/go-with-ndjson)

- `main.go` 실행 결과

``` bash 
$ ./ndjson                                                                                                                                                                                              
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /data                     --> main.(*Output).GetData-fm (3 handlers)
[GIN-debug] POST   /store                    --> main.(*Output).Handle-fm (3 handlers)
[GIN-debug] Listening and serving HTTP on :8080
[GIN] 2021/09/12 - 20:52:59 | 200 |     306.913µs |             ::1 | POST     "/store"
[GIN] 2021/09/12 - 20:57:33 | 200 |     109.986µs |             ::1 | POST     "/store"
```

- 요청 데이터

``` bash
$ curl --location --request POST 'localhost:8080/store' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'date=20210512123456' \
--data-urlencode 'data=hello hoon'

$ curl --location --request POST 'localhost:8080/store' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'date=20210512123456' \
--data-urlencode 'data=hello baeji'
```

- 저장된 데이터

``` ndjson
{"Data":"hello hoon","httpStatus":"200","Date":"20210512123456"}
{"Data":"hello baeji","httpStatus":"200","Date":"20210512123456"}
```

- 데이터 보기

``` bash
$ curl --location --request GET 'localhost:8080/data'
[{"Data":"hello hoon","httpStatus":"200","Date":"20210512123456"},{"Data":"hello baeji","httpStatus":"200","Date":"20210512123456"}]
```
위에 있는 데이터를 이쁘게 만들면 밑에 처럼 볼 수 있는 것이다.

``` json
[
    {
        "Data": "hello hoon",
        "httpStatus": "200",
        "Date": "20210512123456"
    },
    {
        "Data": "hello baeji",
        "httpStatus": "200",
        "Date": "20210512123456"
    }
]
```

# 마무리

이번이 아닌 자바를 이용해서 ndjson을 활용해본 적이 있는데 그때보다 너무 쉽게 구현을 해서 놀라웠다. 정말로 쉬웠다.

해당 방법이 일반적인 경우에는 많이 사용되지 않지만 스트리밍으로 데이터를 처리해야되는 경우 자주 사용될 수 있다고 알고 있기에 적용해본 경험이 나쁘지 않았다.
