---
title: "goss: infra 및 application 검증을 위한 서버 테스트 도구"
description: ""

categories: 
  - dev
tags:
  - dev
  - test
  - testing
  - goss
---

글에 대한 코드는 [링크](https://github.com/BaeJi77/goss-test)를 통해서 볼 수 있습니다.

# What is `goss`?

- [goss github repository](https://github.com/goss-org/goss)

Goss는 사용자가 파일 콘텐츠, 서비스 상태 및 네트워크 가용성을 포함하여 시스템 상태의 다양한 측면에 대한 기대치를 가지고 간단한 YAML 파일을 작성할 수 있게 해주는 서버 테스트 도구입니다. Golang으로 작성되어 빠르고 효율적이며 구성 파일, 인프라 및 애플리케이션을 검증하는 데 사용할 수 있습니다.

## goss는 어떻게 작동하나요?
Goss는 테스트 사례가 포함된 YAML 파일을 읽고 대상 시스템에 대해 실행합니다. 테스트는 로컬 또는 원격으로 실행할 수 있으므로 분산 시스템 테스트에 이상적입니다. 이 도구는 명령줄 인터페이스를 사용하고 색상화된 출력을 생성하여 각 테스트의 성공 또는 실패를 나타냅니다.

## goss를 사용하면 어떤 이점이 있나요?
Goss는 인프라와 애플리케이션의 테스트 및 검증 프로세스를 간소화합니다. 가볍고 빠르기 때문에 CI/CD(지속적인 통합 및 배포) 파이프라인에 사용하기에 이상적입니다. Goss는 사용자가 각 테스트에 대한 사용자 지정 명령 및 시간 제한을 지정할 수 있도록 구성이 매우 쉽습니다. 즉, Goss를 사용하여 인프라와 애플리케이션이 예상대로 작동하는지 확인하고, 개발 프로세스 초기에 버그를 포착하고, 문제를 디버깅하고 수정하는 데 드는 시간과 비용을 줄일 수 있습니다.

# How to use `goss`?

- [goss 메뉴얼](https://github.com/goss-org/goss/blob/master/docs/manual.md)

메뉴얼에서 처음부터 나오지만 현재 linux에서 완전히 지원하는 툴입니다. 그러기 때문에 macOS이거나 windows에서는 부분적으로 지원하며 공식 github `README.md`에서 제공하는 방식이 아니라 다른 방식으로 패키지를 다운받아서 사용해야됩니다.

## How to install `goss`? 

- linux용 다운로드

``` sh
curl -L https://github.com/goss-org/goss/releases/latest/download/goss-linux-amd64 -o /usr/local/bin/goss
chmod +rx /usr/local/bin/goss
```

## Usage

### cli

- `validate`: YAML 파일에 정의된 테스트를 실행하고 결과를 볼 수 있게 만들어주는 command입니다.

goss를 사용하면서 cli를 통해서 결과를 보기 위해서 실행해야되는 명령어라고 생가하면 됩니다.

```bash
$ goss validate --format documentation
File: /go/src/app: exists: matches expectation: [true]

Total Duration: 0.001s
Count: 1, Failed: 0, Skipped: 0
```

- `add` 및 `autoadd`: 다양한 시스템 구성 요소에 대한 YAML 규칙 파일을 생성하는 데 사용됩니다.

```bash
root@2c16a256cb0e:/go/src/app# goss add package nginx
Adding Package to './goss.yaml':

nginx:
  installed: false


root@2c16a256cb0e:/go/src/app# goss add service nginx
Adding Service to './goss.yaml':

nginx:
  enabled: false
  running: false


root@2c16a256cb0e:/go/src/app# goss add file nginx
Adding File to './goss.yaml':

/go/src/app/nginx:
  exists: false
  contains: []

root@2c16a256cb0e:/go/src/app# cat goss.yaml 
file:
  /go/src/app:
    exists: true
    contains: []
  /go/src/app/nginx:
    exists: false
    contains: []
package:
  nginx:
    installed: false
service:
  nginx:
    enabled: false
    running: false
```

- `serve`: 사용자가 웹 브라우저에서 테스트 결과를 볼 수 있는 웹 서버를 실행합니다.

- `render`: goss configuration과 관련해서 다양한 파일을 만들어서 사용할 수 있습니다. 현재 validate를 할 때 어떤 test suite들이 있는지에 대해서 전체를 보여줍니다.

```bash
$ cat goss_httpd_package.yaml
package:
  httpd:
    installed: true
    versions:
    - 2.2.15

$ cat goss_httpd_service.yaml
service:
  httpd:
    enabled: true
    running: true

$ cat goss_nginx_service-NO.yaml
service:
  nginx:
    enabled: false
    running: false

$ cat goss.yaml
gossfile:
  goss_httpd_package.yaml: {}
  goss_httpd_service.yaml: {}
  goss_nginx_service-NO.yaml: {}

$ goss -g goss.yaml render
package:
  httpd:
    installed: true
    versions:
    - 2.2.15
service:
  httpd:
    enabled: true
    running: true
  nginx:
    enabled: false
    running: false
```

### Handle goss.yaml file

- [주의사항](https://github.com/goss-org/goss/blob/master/docs/manual.md#important-note-about-goss-file-format)

특정 필드에 대해서 여러 개를 만들어서 테스트를 하고 싶을 수 있다. 하지만 그런 경우 이전에 작성된 configuration이 overriding 되어서 정상으로 동작하지 않는다.

- **NOT GOOD**
```yaml
file:
  A:
    exists: true

service:
  httpd:
    enabled: true
    running: true

file:
  B:
    filetype: directory
    exists: true
```

여기서 A는 무시당합니다. B만을 검사합니다.

- **GOOD**

```yaml
file:
  A:
    exists: true
  B:
    filetype: directory
    exists: true

service:
  httpd:
    enabled: true
    running: true
```

위에 있는 것과 차이는 `file`에 대한 필드에 대해서 한번에 다 작성했다는 것입니다.

### [goss available tests](https://github.com/goss-org/goss/blob/master/docs/manual.md#available-tests)

goss는 정말 다양한 경우에 대해서 테스트를 할 수 있습니다.

- addr
- command
- dns
- file
- gossfile
- group
- http
- interface
- kernel-param
- matching
- mount
- package
- port
- process
- service
- user

위에 있는 모든 케이스에 대해서 설명하기 보다 직접 구체적으로 설정가능한 옵션들을 찾아보는 것을 추천드립니다. 

저는 위에 있는 목록 중에 `gossfile`에 대해서만 간단하게 설명하겠습니다.

- `gossfile`: 어떤 goss file을 설정에 넣을 것인지 고를 수 있도록 하는 필드입니다.

밑에 있는 코드들은 goss file에 대한 설정들입니다.

```yaml
file:
  /go/src/app:
    exists: true

gossfile:
  a:
    file: new-goss.yaml
    skip: false
  b:
    file: skip-goss.yaml
    skip: true
```

```yaml
file:
  /hello-world:
    exists: true
```

```yaml
file:
  /hello-world-2:
    exists: true
```

위에서 부터 `goss.yaml`, `new-goss.yaml`, `skip-goss.yaml`입니다. `goss.yaml`이 실제로 실행되면서 두번째 있는 `new-goss.yaml` 정보를 가져오는 것을 볼 수 있습니다.

```bash
root@e23b8ea63550:/go/src/app# goss render
file:
  /go/src/app:
    exists: true
    contains: []
  /hello-world:
    exists: true
    contains: []
```

### 간단하게 작성된 예시 ([코드](https://github.com/BaeJi77/goss-test))

golang을 이용해서 파일을 만드는 코드가 있습니다. 그 코드와 함께 goss를 이용해서 정상적으로 동작하는지 확인해보겠습니다. (macOS를 사용하고 있어서 docker을 이용해서 동작시켰습니다.)

```golang
package main

import (
	"fmt"
	"os"
	"time"
)

func main() {
	fmt.Println("hello world")

	os.WriteFile("hello-world.txt", []byte{}, 0644)

	time.Sleep(1000 * time.Minute)
}
```

```yaml
file:
  /go/src/app:
    exists: true
    filetype: directory
  hello-world.txt:
    exists: true
    filetype: file
    mode: "0644"

gossfile:
  a:
    file: new-goss.yaml
    skip: false
  b:
    file: skip-goss.yaml
    skip: true
```

```bash
root@a574985d1674:/go/src/app# goss v -f documentation
File: /go/src/app: exists: matches expectation: [true]
File: /go/src/app: filetype: matches expectation: ["directory"]
File: /hello-world: exists:
Expected
    <bool>: false
to equal
    <bool>: true
File: hello-world.txt: exists: matches expectation: [true]
File: hello-world.txt: mode: matches expectation: ["0644"]
File: hello-world.txt: filetype: matches expectation: ["file"]


Failures/Skipped:

File: /hello-world: exists:
Expected
    <bool>: false
to equal
    <bool>: true

Total Duration: 0.002s
Count: 6, Failed: 1, Skipped: 0
```

위에서 볼 수 있듯이 파일이 존재하는지. 그리고 파일에 대한 타입, 그리고 mode까지 확인할 수 있습니다.

### 그 밖에 유용한 것 같은 친구들

- [template](https://github.com/goss-org/goss/blob/master/docs/manual.md#templates)

단순히 고정된 값만들을 사용하는 것이 아니라 다양한 값들을 외부에서 받아서 사용할수도 있고 혹은 배열을 이용해서 다양한 것들에 대해서도 테스트 할 수 있습니다.

- [pattern](https://github.com/goss-org/goss/blob/master/docs/manual.md#patterns)

goss test suite를 작성할 때 고정된 string 값을 비교하는 경우도 있지만 pattern을 이용해서 비교할 수 있는 경우도 있습니다. file 내부에 어떤 값이 있는 다던지, http response body에 어떤 값이 있는지에 대해서 사용할 수 있습니다.

# 마무리

제가 회사에서 주로 하고 있는 agent 개발을 진행하면서 테스트하기 어려운 케이스들이 존재했습니다. 예를 들면 특정 path에 파일이 만들어지는 것을 확인한다던지, 정상적으로 process가 동작하고 있는지 확인한다던지에 대한 케이스들이 잇엇습니다. 

오늘 설명한 내용은 단순히 사용법에 대해서 설명했지만 github action과 연동하거나 goss에서 추가 개발한 [dgoss](https://github.com/goss-org/goss/tree/master/extras/dgoss)를 사용한다면 더 다양한 것을 할 수 있을 것 같다는 생각을 했습니다.
