---
title: "대규모 Agent를 어떻게 관리하면 좋을까요? (feat. OpAMP) (3)"
description: "서버에서 관리되는 agent를 관리하는 OTEL에서 만드는 프로토콜에 대한 이야기. remote configuration, package management, connection management, security에 대한 이야기."
categories:
  - dev
  - observability
tags:
  - otel
  - OpAMP
  - observability
---

# 이전 시리즈

- [대규모 Agent를 어떻게 관리하면 좋을까요? (feat. OpAMP) (1): OpAMP가 나온 배경과 Spec](https://baeji77.github.io/dev/observability/About-OpAMP-1/)
- [대규모 Agent를 어떻게 관리하면 좋을까요? (feat. OpAMP) (2): OpAMP가 Agent를 처리하는 과정 (1)](https://baeji77.github.io/dev/observability/About-OpAMP-2/)
- (현재 글) [대규모 Agent를 어떻게 관리하면 좋을까요? (feat. OpAMP) (3): OpAMP가 Agent를 처리하는 과정 (2)](https://baeji77.github.io/dev/observability/About-OpAMP-3/)

> **Warning**
> 해당 시리즈 글은 OpAMP 공식 문서 내용을 압축하고 번역하여서 소개합니다. 
> OpAMP는 현재 매우 발전하고 있는 프로토콜이기에 해당 내용을 금방 expired될 수 있습니다. 심지어 3개의 시리즈를 작성하면서도 새로 추가되고 없어지는 내용이 존재했습니다.
> 최신 정보와 추가적인 정보를 원하시는 공식 문서를 참고하기를 바랍니다.

# 개요

이번 글에서는 `OpAMP(Open Agent Management Protocol)`를 활용하여 서버와 클라이언트 간의 데이터 교환 및 처리 과정을 중점적으로 다룹니다. 특히 다음 네 가지 주요 영역에 초점을 맞췄습니다:

- **Configuration**: remote configuration을 통해 Agent의 설정을 관리하는 방법
- **Package Management**: 패키지 설치 및 관리 절차
- **Connection Management**: 클라이언트와 서버 간의 연결 설정 및 유지
- **Security**: 보안 고려 사항 및 권장 사항

이 글은 OpAMP의 실질적인 적용 사례를 통해 대규모 Agent 관리의 복잡성을 해소하는 방법을 제시할 것입니다.

## TL;DR

- **Remote Configuration**: 서버에서 Agent로 설정을 원격으로 전송할 수 있으며, 이 과정에서 무한 루프를 방지하고 보안을 고려해야 합니다.
- **Package Management**: 해시 기반으로 패키지의 변경 여부를 확인하고 필요한 경우 다운로드합니다. 보안이 중요한 요소입니다.
- **Connection Management**: 클라이언트와 서버 간의 연결 및 장애 상황 처리 절차가 포함됩니다. 특히, 재시도 시 서버 부하를 줄이기 위한 백오프 전략이 필요합니다.
- **Security**: 악의적인 서버로부터의 보안 위협을 최소화하기 위해 권한을 최소화하고, Client 측에서 configuration에 대해서 최대한 보수적으로 접근해야 합니다.
- OpAMP는 지속적으로 발전하는 프로토콜로, 새로운 추가되는 기능에 대해서 고려하여서 설정해야됩니다.

# 이전 글 요약

`OpAMP`에서 Client와 Server와 서로 status를 통신할때 가장 최신화된 정보만을 교환하며 데이터를 압축해서 통신합니다. 가장 최신화된 정보만을 교환하면 그 중에서 통신이 제대로 되지 않았다면 모든 정보를 교환하는 방어장치도 존재합니다.

OpAMP에 대해서 지원하는 서버만을 위한 것이 아닌 OTLP 혹은 다른 서버들에 대한 connection setting을 동적으로 관리하는 방법이 존재합니다.

OpAMP Client가 자체적으로 문제에 대해서 OTLP 기반으로 서버로 상태를 전송할 수 있습니다.

# 용어 정리

- **Server**: OpAMP Server이며, OpAMP를 통해서 OpAMP client와 통신하여서 Agent에 대한 관리를 도와준다.
- **Client**: OpAMP Client이며, OpAMP 서버와 통신하며 Agent에 대한 같은 host에서 직접적인 관리 및 실행을 담당한다.
- **Agent**: host에서 실행되는 process이며 otel에서 사용되는 가장 대표적인 예는 otel collector가 있다. 문서에서 Agent는 OpAMP Client + Agent 개념으로 사용하기도 합니다.

# Operation

OpAMP를 기반으로 서로 교환하는 메시지들을 통해서 Server와 Client는 상호작용을 한다. 이전 내용에 연장으로 밑에서는 여러가지 상호작용에 대해서 더 알아갈 것입니다.

## Configuration

해당 기능을 통해서 Server에서 Agent에 대한 configuration을 전송할 수 있습니다.

- Server는 `ServerToAgent` 메시지의 `remote_config` 필드를 설정하여 Agent에 remote configuration을 제공할 수 있습니다.
- Client는 Agent가 remote configuration을 수락할 수 있는 경우 `AgentToServer.capabilities`의 `AcceptsRemoteConfig` 비트를 설정해야 합니다. 
- Agent 입장에서는 remote에서 오는 configuration에 전부를 받을 필요는 없습니다. 그리고 local에서 선언된 configuration도 있기 때문에 remote configuration이 꼭 client 혹은 agent에 전체 configuration은 아닙니다.
- client는 `effective_config` 필드를 통해서 OpAMP Server에서 실질적으로 적용된 configuration에 대한 정보를 보냅니다. 그 과정에서 `AgentToServer.capabilities`의 `ReportsEffectiveConfig` 비트가 설정되어있어야 합니다.

밑에는 remote configuration의 적용 시퀀스 다이어그램입니다. 

```
     Agent       Client                             Server

       │           │ AgentToServer{}                   │   ┌─────────┐
       │           ├──────────────────────────────────►├──►│ Process │
       │           │                                   │   │ Status  │
Local  │    Remote │                                   │   │ and     │
Config │    Config │ ServerToAgent{AgentRemoteConfig}  │   │ Fetch   │
  │    │  ┌────────┤◄──────────────────────────────────┤◄──┤ Config  │
  ▼    │  ▼        │                                   │   └─────────┘
 ┌─────────┐       │                                   │
 │ Config  │       │                                   │
 │ Merger  │       │                                   │
 └─────┬───┘       │                                   │
       │           │                                   │
       │Effective  │                                   │
       │Config     │ AgentToServer{}                   │
       └──────────►├──────────────────────────────────►│
                   │                                   │
                   │                                   │
```

Client에서 특정한 변경이 없는 경우에도 Server에서 remote configuration에 대한 요청을 전달할 수 있습니다.

```
    Agent      Client                             Server

       │           │                                   │
       │           │                                   │
       │           │                                   │   ┌────────┐
Local  │    Remote │                                   │   │Initiate│
Config │    Config │  ServerToAgent{AgentRemoteConfig} │   │and     │
    │  │  ┌────────┤◄──────────────────────────────────┤◄──┤Send    │
    ▼  │  ▼        │                                   │   │Config  │
  ┌─────────┐      │                                   │   └────────┘
  │ Config  │      │                                   │
  │ Merger  │      │                                   │
  └────┬────┘      │                                   │
       │           │                                   │
       │Effective  │                                   │
       │Config     │ AgentToServer{}                   │
       └──────────►├──────────────────────────────────►│
                   │                                   │
                   │                                   │
```

해당 procedure 과정에서 중요한 것은 실질적으로 적용된 configuration에 대한 변경이 있는 경우에만 적용된 configuration을 Server로 전송합니다. 실질적으로 configuration이 적용된 것이 없다면 Server로 요청을 보내지 않습니다.

만약 그렇지 않으면 무한적으로 계속 메시지를 주고 받는 구조가 될 수 있습니다.

### Messages

```protobuf
message AgentRemoteConfig {
  AgentConfigMap config = 1;
  bytes config_hash = 2;
}

message AgentConfigMap {
  map<string, AgentConfigFile> config_map = 1;
}

message AgentConfigFile {
  bytes body = 1;
  string content_type = 2; // ex) text/yaml, text/json, ...
}
```

`AgentConfigMap` 안에 있는 `config_map`에 key 값은 파일이름입니다. 하나의 configuration file만 있는 경우에는 key는 비어있습니다.

`AgentConfigFile`안에 있는 `content_type`은 body 데이터에 대한 `MIME Content-Type`입니다. 해당 값은 Server가 UI를 깔끔하게 시각화하는데 사용할 수 있습니다.

## Package (Beta)

`OpAMP`는 host에 특정 패키지를 설치하고 관리하도록 하는 기능을 제공합니다. 

- 최상위 패키지와 하위 패키지라는 두 가지 유형의 패키지가 존재합니다.
  - 최상위 패키지는 Agent의 주요 기능을 구현한 것이며 일반적으로 하나의 패키지가 존재합니다.
  - 하위 패키지는 addon 혹은 plugin이라고 알려져 있으며 Agent의 추가 기능을 도와줍니다.

패키지당 하나의 파일만 다운로드 할수 있기 때문에 패키지 안에 zip이나 tar 파일 형식으로 하는것을 추천합니다.

- Server는 `ServerToAgent` 메시지의 `packages_available` 필드를 설정해야 됩니다.
- Agent가 `AcceptsPackages` 비트를 설정해야지만 Server가 패키지 정보를 제공합니다.

### Package download

Agent가 PackagesAvailable 메시지를 수신한 후 절차는 밑에와 같습니다.

1. Agent가 보유한 모든 패키지의 해시값과 Server가 제공해주는 `all_packages_hash` 필드값과 비교합니다.
  - 만약 해당 값이 똑같다면 다운로드 절차가 완료된 것으로 간주합니다. 그렇지 않다면 2단계를 진행합니다.
2. Server가 제공하는 각 패키지에 대해서 Agent는 어떤 패키지를 다운로드 해야되는지 확인합니다.
  - 해당 패키지 이름이 없는 경우 3단계를 진행합니다.
  - 해당 패키지 이름은 있지만 hash 필드 값이 Server에서 제공해준 것과 다르다면 3단계를 진행합니다.
  - Agent가 보유하고 있지만 Server에서 제공하지 않는 패키지는 Agent에서 삭제합니다.
3. Server에서 제공한 패키지 정보를 기반으로 다운로드 할지 결정합니다.
  - `download_url` 필드에 지정된 위치에서 파일을 다운로드 합니다.
  - `download_url`는 HTTP Get 메시지를 통해서 다운로드 할수 있어야 되며 `headers` 필드에서 제공하는 값들을 함께 보내서 권한 문제를 해결할수도 있어야 됩니다.

### Agent status report

Client는 `AgentToServer` 메시지에 `package_statuses` 필드에 값을 넣어서 Server에 전송합니다.

모든 패키지에 대한 다운로드 및 설치가 완료되면 (성공 혹은 실패)에 대해서 Server에 보고합니다.

밑에는 일반적인 패키지 다운로드 및 status report 시퀀스 다이어그램입니다.

```
    Download        Agent/Client                          OpAMP
     Server                                              Server
       │                 │                                  │
       │                 │  ServerToAgent{PackagesAvailable}│
       │                 │◄─────────────────────────────────┤
       │   HTTP GET      │                                  │
       │◄────────────────┤                                  │
       │ Download file #1│                                  │
       ├────────────────►│                                  │
       │                 │ AgentToServer{PackageStatuses}   │
       │                 ├─────────────────────────────────►│
       │   HTTP GET      │                                  │
       │◄────────────────┤                                  │
       │ Download file #2│                                  │
       ├────────────────►│                                  │
       │                 │ AgentToServer{PackageStatuses}   │
       │                 ├─────────────────────────────────►│
       │                 │                                  │
       .                 .                                  .

       │   HTTP GET      │                                  │
       │◄────────────────┤                                  │
       │ Download file #N│                                  │
       ├────────────────►│                                  │
       │                 │ AgentToServer{PackageStatuses}   │
       │                 ├─────────────────────────────────►│
       │                 │                                  │
```

### hash 계산

가장 중요한 것은 **Agent에서는 어떤 hash 계산을 하지 않으며 Server에서 제공받은 것을 저장**하고 비교만을 합니다.

file hash, package hash, all package hash 3가지 hash가 존재합니다.

- file hash

`DownloadableFile` 메시지의 `content_hash` 필드에 저장. 

특정 파일이 Server에서 제공해주는 것과 다른지 비교.

- package hash

`PackageAvailable` 메시지의 `hash` 필드에 저장.

전체 패키지(패키지 이름 및 파일 콘텐츠)를 식별하는 패키지 해시.

- all package hash

`PackagesAvailable` 메시지의 `all_packages_hash` 필드에 저장

모든 패키지의 집계 해시는 특정 Agent에 대한 모든 패키지의 집계 해시

### Messages

```protobuf
message PackagesAvailable {
    map<string, PackageAvailable> packages = 1;
    bytes all_packages_hash = 2;
}

message PackageAvailable {
    PackageType type = 1;
    string version = 2;
    DownloadableFile file = 3;
    bytes hash = 4;
}

enum PackageType {
    TopLevelPackage = 0;
    AddonPackage    = 1;
}

message DownloadableFile {
    string download_url = 1;
    bytes content_hash = 2;
    bytes signature = 3;
    headers headers = 4; // 상태: [development]
}
```

`DownloadableFile`에 `signature`는 보안을 위해서 존재하며 `headers`는 `download_url`에 파일 다운로드를 보낼때 필요한 정보를 넣을수 있도록 하는 필드입니다.

## Custom message (Development)

### motivation

OpAMP 프로토콜은 Agent의 원격 관리 및 구성의 모든 측면을 다루도록 설계되었습니다. OpAMP 프로토콜의 범위에 포함되지 않거나 다른 Agent 유형에 일반화할 수 없는 특정 Agent에 매우 특화된 경우에는 지원되지 않을 것입니다. Server와 Agent 간에 다른 연결을 열거나 완전히 새로운 프로토콜을 정의하지 않고도 사용자 정의 동작을 구현할 수 있도록 합니다.

### CustomCapabilities message

Agent와 Server 모두 해당 메시지를 통해서 Custom message를 교환합니다. `ServerToAgent` 및 `AgentToServer`에서 지원됩니다.

기능은 역방향 FQDN과 선택적 버전 정보로 식별되어야 합니다. 예를 들어 `"com.company.capability/v2"`는 `"company.com"`에서 만든 `"capability"의 버전 2`를 식별합니다.

지원되지 않는 기능이 있는 `CustomMessage`를 수신한 경우 메시지를 무시할 수 있습니다.

```protobuf
message CustomCapabilities {
  repeated string capabilities = 1; // 상태: [개발 중]
}
```

### CustomMessage message

```protobuf
message CustomMessage {
    string capability = 1;
    string type = 2;
    bytes data = 3;
}
```

- `CustomMessage.capability`: 기능을 고유하게 식별하고 CustomCapabilities 메시지의 기능 중 하나와 일치하는 역방향 FQDN
- `CustomMessage.data`: 메시지의 이진 데이터입니다. 기능은 정의하는 각 사용자 정의 메시지 유형에 대한 데이터의 내용 형식을 지정해야 합니다.

### Examples

해당 예제는 제가 공부를 진행하면서 존재하던 예제있으면 추가적인 예제들을 원한다면 [해당 링크](https://opentelemetry.io/docs/specs/opamp/#examples)를 통해서 얻을 수도 있습니다.

#### 일시중지 및 재개

- Agent 연결

Agent에서 Server와 연결할때 `AgentToServer` 메시지에 `CustomCapabilities`에 밑에 같이 전송하게 되면 `"io.opentelemetry.pause"` 기능을 제공해준다는 것을 알려주는 것입니다.

```
CustomCapabilities {
  capabilities: ["io.opentelemetry.pause"]
}
```

- `ServerToAgent` message

```
CustomMessage {
  capability: "io.opentelemetry.pause"
  type: "pause"
}
```

- `AgentToServer` message

```
// 성공시
CustomMessage {
  capability: "io.opentelemetry.pause"
  type: "status"
  data: {
    "paused": true
  }
}

// 실패시
CustomMessage {
  capability: "io.opentelemetry.pause"
  type: "error"
  data: {
    "error": "Unable to pause"
  }
}
```

- resume 를 통한 Agent 실행 재개

밑에와 같이 `resume` 유형의 `CustomMessage`를 포함여서 전송합니다.

```
CustomMessage {
  capability: "io.opentelemetry.pause"
  type: "resume"
}
```

# Connection management

## 연결 설정

Client는 HTTP(s) 연결을 설정하여서 Server와 연결합니다.

연결이 설정된 후 Client는 첫번째 status report를 보내고 응답을 기대합니다.

Client가 Server와 연결을 할 수 없는 경우 연결 재시도를 하고 Server 과부하를 주지 않도록 jitter를 이용한 지수 backoff 전략을 사용해야 합니다.

연결 재시도를 할때 Client는 Server로 부터 제공받는 throttling 룰을 지켜야 됩니다.

## 연결 종료

### WebSocket, OpAMP Client

Client는 `agent_disconnect` 필드가 설정된 `AgentToServer` 메시지를 먼저 보내야 합니다. 그런 다음 WebSocket Close 제어 프레임을 보내고 WebSocket 표준에 정의된 절차를 따라야 합니다.

### WebSocket, OpAMP Server

연결을 종료하려면 Server는 WebSocket Close 제어 프레임을 보내고 WebSocket 표준에 정의된 절차를 따라야 합니다.

### Plain HTTP

HTTP 응답이 완료된 후 Client 연결이 사라진 것으로 항상 암시되므로 Client가 `agent_disconnect` 필드가 설정된 `AgentToServer` 메시지를 보낼 필요는 없습니다.

Server에서는 지속적으로 HTTP 요청을 오는 Clent와 특정 기간동안 HTTP 요청이 오지 않는 Client에 대한 비지니스를 다르게 사용할수도 있습니다.

## WebSocket 연결 복원

설정된 WebSocket 연결이 예기치 않게 끊어진 경우 Client는 즉시 다시 연결을 시도해야 합니다. 연결이 실패하는 경우 backoff를 사용하여 연결을 시도합니다.

## 중복 WebSocket 연결

Server는 동일한 Client 인스턴스가 두 개 이상의 동시 연결을 가지고 있거나 여러 Agent 인스턴스가 동일한 `instance_uid`를 사용하는 경우 요청을 연결 해제하거나 제공하지 않을 수 있습니다.

Server는 중복 `instance_uid`를 감지해야 합니다(예: Agent가 잘못된 UID 생성기를 사용하거나 Agent가 실행되는 VM을 복제한 경우 발생할 수 있음). 중복 `instance_uid`가 감지되면 Server는 새 `instance_uid`를 생성하고 이를 `AgentIdentification`의 `new_instance_uid` 값으로 보냅니다.

## Authentication (Beta)

Client와 Server는 HTTP에서 지원하는 인증 방법을 사용할 수 있으며, `basic 인증` 또는 `Bearer 인증`과 같은 방법이 있습니다.

Client 인증이 실패한 경우 Server는 **401 Unauthorized**로 응답해야 합니다.

## bad request

Server가 잘못된 형식의 `AgentToServer` 메시지를 수신한 경우 Server는 `error_response`가 적절히 설정된 `ServerToAgent` 메시지로 응답해야 합니다.

Client는 `BAD_REQUEST` 응답을 수신한 `AgentToServer` 메시지를 **다시 보내지 않아야 합니다.**

## 메시지 재시도

Client는 다음과 같은 경우 `AgentToServer` 메시지를 다시 보낼 수 있습니다:

- 응답이 필요한 `AgentToServer` 메시지를 전송했지만 합리적인 시간 내에 응답이 오지 않은 경우. (timeout 설정 가능)
- 응답이 필요한 `AgentToServer` 메시지가 전송되었지만 응답을 받기 전에 연결이 끊어진 경우.
- Server로부터 **UNAVAILABLE** 응답을 받은 후.

Server에서는 언제나 Client가 여러번 요청이 올 수 있다는 전제하에 멱등성이 있게 메시지를 수신해야합니다.

OpAMP는 Client는 특별한 경우가 아니면 항상 Client와 Agent의 모든 데이터가 아닌 업데이트가 된 혹은 최신화가 된 정보들만 전송합니다. 그러기 때문에 보내야될 데이터를 지속적으로 큐와 같은 곳으로 저장할 필요없이 최신 데이터를 모아서 한번만 전송하면 됩니다.

Server 또한 Client 혹은 Agent에 전달하고자 하는 최신 메시지만 유지하고 연결해서 하나의 메시지만 전송하면 됩니다.

## Throttling

### WebSocket

Server가 `AgentToServer` 메시지를 처리하지 못하는 경우 Server는  `error_response`가 type 필드가 **UNAVAILABLE**로 설정된 `ServerToAgent` 메시지로 응답해야 합니다. Client는 연결을 끊고 대기한 후 다시 연결을 재개해야 합니다.

`retry_after_nanoseconds`는 Client가 다시 연결하기 전에 대기해야하는 시간을 지정합니다.

```protobuf
message RetryInfo {
    uint64 retry_after_nanoseconds = 1;
}
```

`retry_info`가 설정되지 않은 경우 Client는 재시도 간격을 점진적으로 증가시키기 위해 지수 백오프 전략을 구현해야 합니다.

### Plain HTTP

Server는 **HTTP 503 Service Unavailable** 또는 **HTTP 429 Too Many Requests** 응답을 반환할 수 있으며, `Retry-After` 헤더를 설정하여 Client가 다시 연결을 시도해야 하는 시점을 나타낼 수 있습니다.

Agent가 다시 연결하는 동안 클라이언트는 정기적인 하트비트 메시지를 보내지 않아야 합니다.

**최소 권장 재시도 간격은 30초**입니다.

# Security

`remote configuration`, `remote package download`는 상당한 보안 위험을 초래합니다. 악의적인 Server 측 구성을 보내거나 악의적인 패키지를 보내면 Server는 Agent에게 바람직하지 않은 작업을 수행하도록 강제할 수 있습니다.

이 섹션의 지침은 구현에 선택 사항이지만 **민감한 애플리케이션에 대해 강력히 권장됩니다**.

## General Recommendations

Agent는 **zero-trust security model**을 사용하고 Server로부터 수신한 remote configuration이나 다른 제안을 **자동으로 신뢰하지 않는 것이 좋습니다**.

- Agent는 민감한 파일에 액세스하거나 높은 권한 작업을 수행하지 않도록 최소한의 권한으로 실행해야 합니다. Agent는 루트 사용자로 실행되어서는 안됩니다.
- Agent가 로컬 데이터를 수집할 수 있는 경우 수집을 특정 디렉토리 세트로 제한해야 합니다. 이 제한은 로컬로 지정되어야 하며 remote configuration을 통해 재정의할 수 없어야 합니다. 만약 그렇지 않다면 Agent는 머신에 민감한 정보에 액세스할 수도 있습니다.
- Agent는 외부 코드를 실행할 수 있는 가지고 있기 때문에 Agent는 특정 디렉토리에 위치한 특정 스크립트로만 이 기능을 제한해야 합니다. 이 제한은 로컬에서만 지정하며 remote configuration에 통해서는 재정의를 막아야합니다. 그렇지 않다면 remote configuration 기능이 Agent의 머신에서 임의의 코드실행을 수행하는데 도움을 줄 수 있습니다.

## Configuration Restrictions

Agent는 remote configuration을 통해 강제할 수 있는 작업을 제한하는 것이 좋습니다.

Agent가 수집할 수 있는 디렉토리나 파일에 대한 Agent 측 제한을 두는 것이 좋습니다. remote configuration 수신했을때 Agent는 설정에 대해서 검증하고 제한을 위반하는 경우 configuration을 완전히 혹은 부분적으로 적용하지 않도록 하거나 금지된 디렉토리 및 파일에 대한 데이터를 수집하지 못하도록 해야합니다.

Agent가 실행할 수 있는 실행 파일에 대한 Agent 측 제한을 두는 것이 좋습니다. `deny list`로 지정하는 것보다 `allow list`로 정리하는 것이 좋흣ㅂ니다. 제한은 하드코딩된 local configuration 파일로 하는 것이 좋습니다. 그리고 Server에서 정의된 configuration으로 해당 정보를 수정하지 못하도록 해야됩니다.

## Opt-in remote configuration

remote configuration 기능은 기본적으로 Agent에서 활성화되지 않는 것이 좋습니다. 기능은 사용자가 Opt-in해야 합니다.

## Code Signing

패키지의 일부인 실행 가능한 코드는 서명되어야 하며, 손상된 Server가 Agent에 악의적인 코드를 전달하는 것을 방지해야 합니다.

- 다운로드 가능한 실행 가능한 코드(예: 실행 가능한 패키지)는 코드 서명이 필요합니다.
- Agent는 다운로드한 파일의 실행 가능한 코드가 유효한 코드 서명을 가지고 있는지 확인해야 합니다.
- 코드 서명에 인증 기관이 사용되는 경우 인증 기관과 그 개인 키가 OpAMP Server와 함께 위치하지 않는 것이 좋습니다. 손상된 Server가 악의적인 코드를 서명할 수 없도록 하기 위해서입니다.

# Interoperability

## 부분 구현의 상호 운용성

OpAMP는 Agent와 Server에 대한 여러 기능을 정의합니다. 이러한 기능의 대부분은 선택 사항입니다. Agent 또는 Server는 피어가 특정 기능을 지원하지 않을 수 있음을 준비해야 합니다.

Agent와 Server는 초기 메시지 교환 중에 지원하는 기능을 나타냅니다. Client는 `AgentToServer` 메시지에서 기능 비트 필드를 설정하고, Server는 `ServerToAgent` 메시지에서 기능 비트 필드를 설정합니다.

Server는 특정 Agent의 기능을 알게 된 후 Agent가 지원하지 않는 기능을 사용하는 것을 중지해야 합니다.

마찬가지로, Agent는 Server의 기능을 알게 된 후 Server가 지원하지 않는 기능을 사용하는 것을 중지해야 합니다.

## 미래 기능의 상호 운용성

### 무시할 수 있는 기능 확장

이 새로운 기능을 구현하는 발신자는 새로운 필드를 설정합니다. 새로운 기능을 알지 못하는 사양의 이전 버전을 구현하는 수신자는 새로운 필드를 단순히 무시합니다. Protobuf 인코딩은 나머지 필드가 여전히 수신자에 의해 성공적으로 역직렬화되도록 보장합니다.

### 무시할 수 없는 기능 확장

`AgentToServer` 및 `ServerToAgent` 메시지의 기능 필드는 예약된 비트 수를 포함합니다.

Client와 Server는 메시지를 보낼 때 이러한 예약된 비트를 0으로 설정해야 합니다. 이를 통해 OpAMP의 최신 버전을 구현하는 수신자는 발신자가 새로운 기능을 지원하지 않음을 알 수 있으며, 이에 따라 동작을 조정할 수 있습니다.

# 마무리

이번에는 remote configuration, package download, connection management, security와 그 밖에 추후 기능에 대한 대처에 대해서 알아봤습니다.

3개의 시리즈를 통해서 물리 서버에서 동작하는 대규모 Agent에 대한 관리를 위한 프로토콜인 OpAMP에 대해서 알아봤습니다. 이렇게 정리를 통해서 OpAMP는 어떤 식으로 Agent를 관리하려고 하는지 간단하게는 알수 있게 된것 같습니다.

현재 비슷한 프로덕트를 개발한 입장에서 확장 가능성에 대한 고려와 Server 장애시에 대한 Client 처리와 같은 좋은 인싸이트를 얻었다고 생각합니다.

추후 글에서는 현재 내가 개발했던 프로덕트와 OpAMP에 대한 차이점과 장단점을 비교하는 글을 작성하고 싶습니다.

# ref

- [opamp-spec](https://github.com/open-telemetry/opamp-spec/blob/main/specification.md)
- [OpAMP Specification github](https://github.com/open-telemetry/opamp-spec?tab=readme-ov-file)
- [opamp supervisor](https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/cmd/opampsupervisor/specification/README.md)
- [opampsupervisor in otel-collector-cotribe](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/cmd/opampsupervisor)
- [opamp extention](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/extension/opampextension)
- [otel management document](https://opentelemetry.io/docs/collector/management/)
- [opamp-go](https://github.com/open-telemetry/opamp-go)
