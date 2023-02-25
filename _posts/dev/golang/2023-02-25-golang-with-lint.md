---
title: "golang with golangci-golint (feat.github action)"
description: ""

categories: 
  - dev
  - golang
tags:
  - dev
  - golang
  - golint
---

테스트를 위해 진행했던 코드는 [github repo](https://github.com/BaeJi77/init-golang)를 참고하세요.

# About lint(linter)

``` txet
린트(lint) 또는 린터(linter)는 소스 코드를 분석하여 프로그램 오류, 버그, 스타일 오류, 의심스러운 구조체에 표시(flag)를 달아놓기 위한 도구들을 가리킨다.[1] 이 용어는 C 언어 소스 코드를 검사하는 유닉스 유틸리티에서 기원한다.
```

위에 있는 내용은 [위키페디아](https://ko.wikipedia.org/wiki/%EB%A6%B0%ED%8A%B8_(%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4))에 있는 내용을 가져온 것입니다. 

lint를 적용하는 이유는 간단하게 말해서 버그가 발생할 수 있는 부분에 대한 경험적인 부분을 보여준다고 생각합니다. 프로그래밍 언어중에 거의 필수적으로 lint를 적용하는 언어로는 js가 있습니다. (jslint, eslint, ...)

golang도 그런 의미에서 활용할 수 있는 [golangci-golint](https://github.com/golangci/golangci-lint)가 있습니다.

# Install

공식 [install guide](https://golangci-lint.run/usage/install/)

현재 공식 사이트에서는 CI로 연동한다면 github action과 연동하는 것을 추천하고 있습니다.(configuration와 관련해서도 github 전용 output도 존재합니다.)

그럼에도 다양하게 활용이 가능합니다. 자신이 원하는 방법이 있는지에 대해서는 공식 guide를 참고해보세요.

## [Commandline](https://golangci-lint.run/usage/install/#local-installation)

첫번째로 설명해볼 내용은 로컬에 설치해서 활용하는 방법입니다. 실제 패키지를 다운받아서 사용할 수 있습니다. 

저는 현재 MacOS를 사용하고 있기에 `brew`를 이용한 방법에 대해서 소개하겠습니다.

``` bash
$ brew install golangci-lint
$ brew upgrade golangci-lint
---

$ brew upgrade golangci-lint

==> Downloading https://formulae.brew.sh/api/formula.json
######################################################################## 100.0%
==> Downloading https://formulae.brew.sh/api/cask.json
######################################################################## 100.0%
Warning: golangci-lint 1.51.2_1 already installed
```
실제로는 저는 최근에 업그레이드까지 진행해서 사용하고 있기 때문에 이런 메시지가 나오고 있습니다.

``` bash
$ golangci-lint
Smart, fast linters runner.

Usage:
  golangci-lint [flags]
  golangci-lint [command]

Available Commands:
  cache       Cache control and information
  completion  Generate the autocompletion script for the specified shell
  config      Config
  help        Help
  linters     List current linters configuration
  run         Run the linters
  version     Version

Flags:
      --color string              Use color when printing; can be 'always', 'auto', or 'never' (default "auto")
  -j, --concurrency int           Concurrency (default NumCPU) (default 10)
      --cpu-profile-path string   Path to CPU profile output file
  -h, --help                      help for golangci-lint
      --mem-profile-path string   Path to memory profile output file
      --trace-path string         Path to trace output file
  -v, --verbose                   verbose output
      --version                   Print version

Use "golangci-lint [command] --help" for more information about a command.
```

## Goland

Goland에서 `Command` + `,`를 누르게 되면 Preference 창이 나오게 됩니다.

```
Path: Preferences | Tools | Go Linter
Link: jetbrains://GoLand/settings?name=Tools--Go+Linter
```
링크를 누르게 되면 Goland가 깔려있는 경우 Goland에 대한 옵션이 나오도록 이동시켜줍니다.

![그림1_링크 클릭했을 경우](/assets/images/2023-02-25-golang-with-lint/그림1_goland링크.png)

![그림2](/assets/images/2023-02-25-golang-with-lint/%EA%B7%B8%EB%A6%BC2_goland-golinter.png)

위에 사진은 실제 이동했을 경우 Go Linter 설정에 대한 화면입니다. (저는 미리 설치하고 테스트를 진행했기 때문에 아무 설정도 안했을 경우와 화면이 다를 수 있습니다.)

위에 사진처럼 `GET LATEST`을 누르게 되면 밑에 그림처럼 원하는 곳에 설치할 수 있게 선택할 수 있습니다.

# [Integration](https://golangci-lint.run/usage/integrations/)

사용하는 방법은 다양한지만 Goland(local)과 CI에 적용하는 방법에 대해서 말씀드리겠습니다.

## [Goland](https://golangci-lint.run/usage/integrations/#goland)

[plugin github](https://github.com/xxpxxxxp/intellij-plugin-golangci-lint)

### File Watcher

공식 문서에서는 File Watcher기능(Preferences에 검색하면 나옵니다)을 이용하라고 나와있습니다. 

File Watcher라는 기능은 File 수정이나 저장을 하게 되면 triggering 되어서 동작하게 되는 기능입니다.

![그림3](/assets/images/2023-02-25-golang-with-lint/%EA%B7%B8%EB%A6%BC3_file-watcher.png)

![그림4](/assets/images/2023-02-25-golang-with-lint/%EA%B7%B8%EB%A6%BC4_file-watcher-golanglint.png)

### Go Linter

`Goland`에 `Preferences`를 보게되면 `Go Linter`라는 설정탭이 있습니다. (golint install에서 소개했습니다.)

![그림5](/assets/images/2023-02-25-golang-with-lint/%EA%B7%B8%EB%A6%BC5_golinter%EC%84%A4%EC%A0%95.png)

해당 UI를 보게 되면 Linter을 선택할 수 있습니다. 정말 다양한 Linter들이 있는데 Enable과 Disenable을 선택으로 할 수 있습니다.

`File Watcher`와 비슷하게 파일에 대한 변경에 대해서 감시를 하고 그것에 대해서 알려줍니다.

# [Configuration](https://golangci-lint.run/usage/configuration/)

lint실행에 대해서 다양한 옵션들을 사용할 수 있습니다. 그것에 대해서 설명하겠습니다.

## [Configuration file](https://golangci-lint.run/usage/configuration/#config-file)

공식 사이트에서는 file을 만들어서 권장한다고 생각합니다. 저도 비슷한 생각인 것으로 command-line으로 만든 경우 추가적으로 공유하거나 사용자가 설정을 해야됩니다. 하지만 file을 만들어서 같은 repo에 저장해둔다면 모두 공통적인 lint rule을 공유하면서 사용자는 특별히 설정할 일이 없기 때문입니다.

공식 가이드에서는 정말 다양한 옵션이 있기 때문에 모두 설명 드리는 것은 의미가 없다고 생각하며 전체적으로 어떤 설정을 할 수 있는지에 대해서만 가볍게 이야기드리겠습니다.

``` yml
# Options for analysis running.
run:
  # See the dedicated "run" documentation section.
  option: value
# output configuration options
output:
  # See the dedicated "output" documentation section.
  option: value
# All available settings of specific linters.
linters-settings:
  # See the dedicated "linters-settings" documentation section.
  option: value
linters:
  # See the dedicated "linters" documentation section.
  option: value
issues:
  # See the dedicated "issues" documentation section.
  option: value
severity:
  # See the dedicated "severity" documentation section.
  option: value
```

총 6가지에 대한 큰 옵션이 존재합니다. 

- `run`
  - `concurrency`, `timeout`, `skip-dirs` 등과 같이 lint가 동작하는 것에 대한 전체적인 rule을 설정할 수 있습니다.
- `output`
  - `format`과 같이 실행 이후 결과에 대한 설정을 할 수 있습니다. 
  - `format`에는 `colored-line-number|line-number|json|tab|checkstyle|code-climate|junit-xml|github-actions`이 가능합니다.
- `linters-settings`
  - [guide](https://golangci-lint.run/usage/linters/)에는 사용가능한 linter부터 각각 추가적인 설정이 가능합니다.
- `linters`
  - linter enable과 disenable에 대한 설정이 가능합니다.
- `issues`
  - lint 사용에 있어서 모든 사용자가 마음에 드는 룰이 있는 것은 아니고 그것을 위해서 무시한다거나 반드시 지켜야되는 룰을 설정할 수 있습니다.
  - 추가적으로 해당 글을 읽는 것을 추천합니다. [false-positives](https://golangci-lint.run/usage/false-positives/)
- `severity`
  - lint로 인해서 발견된 issue들에 대해서 어느정도 심각성을 부여할 것인지에 대한 설정입니다.
  - `default-severity`도 존재하며 각 linter에 대해서도 설정이 가능합니다.

## [Command-line](https://golangci-lint.run/usage/configuration/#command-line-options)

``` bash
$ golangci-lint run -h
```

위에 명령어를 통해서 볼 수 있습니다. 엄청나게 다양한 설정을 할 수 있으며 어떤 linter을 사용할지부터 format, timeout, exclude, skip과 관련한 설정을 할 수 있습니다. 

공식 사이트에서는 linter별로 설정하는 방법에 대해서 설정되어 있지 않은 것 같습니다. (혹은 존재하지 않을수도 있습니다.)

## Warning

``` text
The config file has lower priority than command-line options. If the same bool/string/int option is provided on the command-line and in the config file, the option from command-line will be used. Slice options (e.g. list of enabled/disabled linters) are combined from the command-line and config file.
```

공식 사이트에서 나온 문장을 가져온 것입니다. 한마디로 file에서 설정한 값이 있다고 하더라고 command-line을 실행했던 설정 값으로 사용된다는 것입니다. (Slice로 된 경우에는 combine가 됩니다.)

# With Github Action

[golangci-lint-action github](https://github.com/golangci/golangci-lint-action)를 참고하여서 적용했습니다.

``` yml
# This workflow will build a golang project
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-go

name: Go

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-go@v3
        with:
          go-version: '1.19'
      - uses: actions/checkout@v3
      - name: golangci-lint
        uses: golangci/golangci-lint-action@v3
        with:
          # Optional: version of golangci-lint to use in form of v1.2 or v1.2.3 or `latest` to use the latest version
          version: latest
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Set up Go
      uses: actions/setup-go@v3
      with:
        go-version: 1.19

    - name: Build
      run: go build -v ./...

    - name: Test
      run: go test -v ./...
```

기본적으로 제공해주는 github acion에서 go template과 lint을 이용해서 만들었습니다.

![그림6](/assets/images/2023-02-25-golang-with-lint/%EA%B7%B8%EB%A6%BC6_github-action-%EA%B2%B0%EA%B3%BC.png)

![그림7 PR action 결과](/assets/images/2023-02-25-golang-with-lint/%EA%B7%B8%EB%A6%BC_github-action-pr.png)


# 마무리

지금까지 `golangci-golint`에 대해서 알아봤습니다. 

lint가 정답이라고 생각하지는 않지만 convention을 맞추거나 버그가 발생하기 전에 미리 방지하 수 있다는 것에서 좋게 사용할 수 있다고 생각합니다.

특히 github action과 연동되어서 활용할 수 있다는 것이 특히 큰 장점이라고 생각합니다.
