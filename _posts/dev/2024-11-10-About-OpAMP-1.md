---
title: "About OTEL OpAMP (1)"
description: "서버에서 관리되는 에이전트를 관리하는 OTEL에서 만드는 프로토콜에 대한 이야기"
categories:
  - dev
  - observability
tags:
  - otel
  - OpAMP
  - observability
---

# 개요

OpenTelemetry(소위 otel)의 `Open Agent Management Protocol(OpAMP)`은 대규모 에이전트 관리의 필요성을 충족시키기 위해 개발된 프로토콜입니다. 이 글에서는 OpAMP의 주요 기능과 구현 방법에 대해 설명합니다. 

현재 OpAMP는 베타 버전이며(충분히 변경 가능성이 큽니다!), OpAMP는 실질적으로 구현체를 제공하기 보다는 Spec를 제공해주기 때문에 구현체가 다양할 수 있습니다. 

현재 client 구현체는 다양하게 있으면 otel에서도 `opamp-go`라는 구현체가 있기도 합니다. 여기서 살짝 문제라면 문제인것은 서버에 대한 구현체가 pulbic으로 공개된 것이 없는 것으로 알고 있습니다. 현재 문서에서는 **observIQ**라는 곳에서 지원한다고 하는데 유료로 제공하고 있습니다.

# OpenTelemetry(otel)란 무엇인가?

OpenTelemetry(otel)는 클라우드 네이티브 소프트웨어의 성능과 동작을 모니터링하기 위한 오픈 소스 프레임워크입니다. 이는 분산 추적, 메트릭, 로그를 수집하고 처리하여 시스템의 가시성을 높이는 것을 목표로 합니다. OpenTelemetry는 다양한 언어와 플랫폼을 지원하며, 표준화된 API와 SDK를 통해 개발자가 쉽게 통합할 수 있도록 설계되었습니다. 

OpenTelemetry의 주요 목적은 다음과 같습니다:

- **통합된 관측 데이터 수집**: 분산 시스템에서 발생하는 다양한 형태의 데이터를 통합적으로 수집합니다.
- **표준화된 데이터 형식**: 데이터를 표준화된 형식으로 변환하여 다양한 백엔드 시스템과 호환성을 유지합니다.
- **확장 가능한 아키텍처**: 다양한 데이터 소스와 수집기를 지원하여 확장성을 제공합니다.

# OpAMP의 필요성

OpenTelemetry(otel)는 **다양한 에이전트를 통해 데이터를 수집**합니다. 이러한 에이전트는 각기 다른 환경에서 실행되며, 수많은 에이전트를 효율적으로 관리하고 모니터링하는 것이 중요합니다. **OpAMP는 이러한 필요성을 충족시키기 위해 개발**되었습니다. OpAMP는 에이전트의 상태를 중앙에서 모니터링하고, 원격으로 구성 및 업데이트를 관리할 수 있는 기능을 제공합니다.

# 배경

OpAMP는 Kubecon 2022에서 가장 많이 요청된 기능 중 하나로, 대규모 데이터 수집 에이전트를 원격으로 관리하기 위한 네트워크 프로토콜입니다. OpAMP는 에이전트가 서버에 상태를 보고하고, 서버로부터 구성 및 설치 패키지 업데이트를 받을 수 있도록 합니다. 이 프로토콜은 벤더에 구애받지 않으며, 다양한 벤더의 에이전트를 원격으로 모니터링하고 관리할 수 있습니다.

특히, OpAMP Supervisor는 OpenTelemetry Collector를 관리하기 위한 외부 프로세스로, Collector의 설정 파일에 원하는 값을 주입하는 등의 작업을 수행할 수 있습니다. Supervisor는 OpAMP 프로토콜의 클라이언트 측을 구현하여 OpAMP 백엔드와 통신하며, Collector 프로세스를 시작 및 중지하고, 필요한 경우 Collector를 재시작합니다.

![OpAMP Supervisor 이미지]()

# 핵심 기능

OpAMP의 핵심 기능은 다음과 같습니다:

- **에이전트의 원격 구성**: 서버는 에이전트의 설정을 원격으로 변경할 수 있습니다.
- **상태 보고**: 에이전트는 자신의 상태를 서버에 보고합니다.
- **에이전트의 자체 텔레메트리 보고**: 에이전트는 자신의 프로세스 메트릭과 에이전트 전용 메트릭을 OTLP 호환 백엔드로 보고할 수 있습니다.
- **에이전트 하트비트**: 에이전트는 주기적으로 하트비트를 서버에 전송하여 자신의 가용성을 알립니다.
- **다운로드 가능한 에이전트 전용 패키지 관리**: 서버는 에이전트에 설치할 패키지를 제공할 수 있으며, 에이전트는 이를 자동으로 다운로드하고 설치할 수 있습니다.
- **보안 자동 업데이트 기능**: 에이전트는 서버로부터 제공받은 패키지를 통해 자동으로 업그레이드 또는 다운그레이드할 수 있습니다.
- **연결 자격 증명 관리**: 클라이언트 측 TLS 인증서의 취소 및 회전을 포함한 연결 자격 증명을 관리합니다.

# Spec

## Communication model

OpAMP는 Agent와 똑같은 라이프사이클을 가지지 않을 확률이 높습니다. 그러기 때문에 Supervisor라는 개념에 하위 개념에서 따로 동작하면서 OpAMP Server와 통신하며 Agent를 관리하게 됩니다.

OpAMP(OpAMP Client)는 OpAMP Server와 WebSocket과 HTTP 두 가지 방법을 통해 통신합니다. 각 방법은 특정 요구 사항에 맞게 선택할 수 있으며, 각각의 장단점이 있습니다.

```
      ┌───────────────────┐
      │ Supervisor        │
      │                   │
      │          ┌────────┤           ┌────────┐
      │          │ OpAMP  │  OpAMP    │ OpAMP  │
      │          │        ├──────────►│        │
      │          │ Client │           │ Server │
      └────┬─────┴────────┘           └────────┘
           │
           │
           ▼
      ┌──────────┐
      │          │
      │  Agent   │
      │          │
      └──────────┘
```

### WebSocket Transport

WebSocket은 OpAMP 프로토콜의 양방향, 비동기 통신을 지원하는 전송 방법입니다. 이 방식에서는 OpAMP 클라이언트가 WebSocket 클라이언트로, 서버가 WebSocket 서버로 동작합니다. WebSocket 메시지는 `header`와 `data`로 구성되며, `data`는 Protobuf로 인코딩된 메시지입니다. WebSocket을 사용하면 서버와 에이전트 간의 실시간 통신이 가능하여, 서버가 에이전트에 즉각적인 명령을 전달할 수 있습니다.

WebSocket을 통한 메시지 교환은 다음과 같이 이루어집니다:

```
        ┌────────────\ \────────┐                        ┌──────────────┐
        │            / /        │   Data:AgentToServer   │              │
        │            \ \ OpAmp  ├───────────────────────►│              │
        │     Agent  / /        │                        │    Server    │
        │            \ \ Client │   Data:ServerToAgent   │              │
        │            / /        │◄───────────────────────┤              │
        └────────────\ \────────┘                        └──────────────┘
```

WebSocket은 즉각적인 서버-에이전트 간의 통신이 필요할 때 주로 사용됩니다. 클라이언트는 서버에 연결된 후 `AgentToServer` 메시지를 보내며, 서버는 이에 대한 응답으로 `ServerToAgent` 메시지를 보냅니다. 이러한 메시지 교환은 특정 기능에 따라 정의된 순서에 따라 이루어집니다.

### Plain HTTP Transport

HTTP는 OpAMP 프로토콜의 동기, 반이중 통신을 지원하는 전송 방법입니다. 이 방식에서는 OpAMP 클라이언트가 HTTP 클라이언트로, 서버가 HTTP 서버로 동작합니다. 클라이언트는 서버에 POST 요청을 보내고, 서버는 응답으로 Protobuf 메시지를 반환합니다. HTTP는 주기적인 폴링을 통해 통신하며, 실시간 통신이 필요하지 않은 환경에서 유용합니다.

HTTP를 통한 메시지 교환은 다음과 같이 이루어집니다:

- 클라이언트는 `AgentToServer` 메시지를 포함한 HTTP POST 요청을 서버에 보냅니다.
- 서버는 `ServerToAgent` 메시지를 포함한 HTTP 응답을 클라이언트에 보냅니다.
- 에이전트가 서버에 보낼 메시지가 없을 경우, 클라이언트는 주기적으로 서버를 폴링하여 새로운 메시지를 수신합니다.

HTTP 전송은 서버가 에이전트에 메시지를 보낼 때 클라이언트가 서버를 폴링할 때까지 기다려야 한다는 점에서 WebSocket과 차이가 있습니다. 기본 폴링 간격은 30초이며, 클라이언트 설정에 따라 조정할 수 있습니다.

## AgentToServer Messages

`AgentToServer Messages`는 에이전트가 서버에 보내는 메시지로, 에이전트의 상태와 기능을 서버에 전달하는 메시지을 합니다. 이 메시지는 포함하고 있습니다. (각 필드에 대한 설명이 부족한 경우는 다음 시리즈에서 설명하겠습니다.)

```protobuf
message AgentToServer {
    bytes instance_uid = 1;
    uint64 sequence_num = 2;
    AgentDescription agent_description = 3;
    uint64 capabilities = 4;
    ComponentHealth health = 5;
    EffectiveConfig effective_config = 6;
    RemoteConfigStatus remote_config_status = 7;
    PackageStatuses package_statuses = 8;
    AgentDisconnect agent_disconnect = 9;
    uint64 flags = 10;
    ConnectionSettingsRequest connection_settings_request = 11; // Status: [Development]
    CustomCapabilities custom_capabilities = 12; // Status: [Development]
    CustomMessage custom_message = 13; // Status: [Development]
    AvailableComponents available_components = 14; // Status: [Development]
}
```

- **`instance_uid`**: 에이전트의 전역 고유 식별자입니다. 이 식별자는 에이전트가 서버와의 통신에서 자신을 식별하는 데 사용됩니다. `instance_uid`는 에이전트 프로세스의 수명 동안 변경되지 않아야 하며, `UUID v7` 사양을 사용하여 생성됩니다.

- **`sequence_num`**: 각 `AgentToServer` 메시지가 전송될 때마다 1씩 증가하는 시퀀스 번호입니다. 이를 통해 서버는 메시지가 누락되었는지 여부를 감지할 수 있습니다. 시퀀스 번호가 이전에 수신된 번호보다 정확히 1만큼 크지 않으면 서버는 메시지 손실을 감지할 수 있습니다.

- **`agent_description`**: 에이전트의 유형, 실행 환경 등을 설명하는 데이터입니다. 이 필드는 마지막 `AgentToServer` 메시지 이후 변경되지 않았다면 생략될 수 있습니다.

- **`capabilities`**: 에이전트의 기능을 나타내는 비트마스크입니다. 이 필드는 항상 설정되어야 하며, 에이전트가 지원하는 기능을 서버에 알립니다. 예를 들어, 원격 구성을 수락할 수 있는지, 패키지 상태를 보고할 수 있는지 등을 나타냅니다. 구체적인 bitmask 정보는 해당 링크를 통해서 확인하세요. [#AgentToServer.capabilities](https://github.com/open-telemetry/opamp-spec/blob/main/specification.md#agenttoservercapabilities)

- **`health`**: 에이전트와 하위 구성 요소의 현재 상태를 나타냅니다. 이 필드는 에이전트의 전반적인 건강 상태를 나타내며, 마지막 `AgentToServer` 메시지 이후 변경되지 않았다면 생략될 수 있습니다.

- **`effective_config`**: 에이전트가 현재 사용 중인 구성입니다. 이는 서버로부터 받은 원격 구성과 로컬 구성을 결합한 결과일 수 있습니다. 마지막 `AgentToServer` 메시지 이후 변경되지 않았다면 이 필드는 생략될 수 있습니다.

- **`remote_config_status`**: 서버로부터 이전에 받은 원격 구성의 상태입니다. 구성 적용 여부 및 오류 메시지를 포함할 수 있습니다. 마지막 `AgentToServer` 메시지 이후 변경되지 않았다면 이 필드는 생략될 수 있습니다.

- **`package_statuses`**: 에이전트가 설치한 패키지의 상태 목록입니다. 각 패키지의 설치 상태 및 버전 정보를 포함합니다. 마지막 `AgentToServer` 메시지 이후 변경되지 않았다면 이 필드는 생략될 수 있습니다.

- **`agent_disconnect`**: 클라이언트가 서버에 보낸 마지막 `AgentToServer` 메시지에서 설정되어야 하는 필드로, 에이전트가 서버와의 연결을 종료할 때 사용됩니다.

- **`flags`**: 특정 작업을 요청할 때 사용되는 플래그 비트마스크입니다. 예를 들어, 서버가 새로운 `instance_uid`를 생성하도록 요청할 수 있습니다.

- **`connection_settings_request`**: [Development] 연결 설정을 생성하도록 요청하는 메시지입니다. 이 필드는 에이전트가 연결 설정 생성을 시작하는 흐름에서 설정됩니다.

- **`custom_capabilities`**: [Development] 에이전트가 지원하는 사용자 정의 기능을 나타내는 메시지입니다.

- **`custom_message`**: [Development] 에이전트가 서버에 보내는 사용자 정의 메시지를 포함합니다.

- **`available_components`**: [Development] 에이전트에 사용 가능한 구성 요소 목록을 포함하는 메시지입니다. 이 필드는 `ReportsAvailableComponents` 기능이 설정된 경우에만 보고되어야 합니다.


## ServerToAgent Messages

`ServerToAgent Messages`는 서버가 에이전트에 보내는 메시지로, 서버가 에이전트에 전달해야 할 정보를 포함합니다. 이 메시지는 다음과 같은 필드를 포함하고 있습니다.

```protobuf
message ServerToAgent {
    bytes instance_uid = 1;
    ServerErrorResponse error_response = 2;
    AgentRemoteConfig remote_config = 3;
    ConnectionSettingsOffers connection_settings = 4; // Status: [Beta]
    PackagesAvailable packages_available = 5; // Status: [Beta]
    uint64 flags = 6;
    uint64 capabilities = 7;
    AgentIdentification agent_identification = 8;
    ServerToAgentCommand command = 9; // Status: [Beta]
    CustomCapabilities custom_capabilities = 10; // Status: [Development]
    CustomMessage custom_message = 11; // Status: [Development]
}
```

- **`instance_uid`**: 에이전트 식별자입니다. 이 필드는 서버가 에이전트에 응답할 때 사용되며, 여러 에이전트가 하나의 WebSocket 연결을 통해 통신할 때 어떤 에이전트에 메시지가 전달되는지를 구분하는 데 사용됩니다. 서버는 필요에 따라 `AgentIdentification` 필드를 통해 이 값을 재설정할 수 있으며, 에이전트는 제공된 값으로 `instance_uid`를 업데이트해야 합니다.

- **`error_response`**: 에이전트의 요청 처리 중 발생한 오류를 나타냅니다. 이 필드가 설정되면, 다른 모든 필드는 설정되지 않아야 하며, 반대로 다른 필드가 설정되면 `error_response`는 설정되지 않아야 합니다. 오류 유형과 메시지를 포함합니다.

- **`remote_config`**: 서버가 에이전트에 제공하는 원격 구성입니다. 서버가 에이전트에 새로운 구성을 제안할 때 이 필드를 설정합니다.

- **`connection_settings`**: 서버가 에이전트의 클라이언트 연결 설정(예: 목적지, 헤더, 인증서 등)을 변경하고자 할 때 설정되는 필드입니다. 이 필드는 연결 설정 관리에 대한 세부 사항을 포함합니다.

- **`packages_available`**: 서버가 에이전트에 제공할 수 있는 패키지 목록입니다. 서버가 에이전트에 새로운 패키지를 제공할 때 이 필드를 설정합니다.

- **`flags`**: 플래그 비트마스크로, 서버가 에이전트에 특정 작업을 요청할 때 사용됩니다. 예를 들어, `ReportFullState` 플래그는 서버가 에이전트에 전체 상태를 다시 보고하도록 요청할 때 사용됩니다.

- **`capabilities`**: 서버의 기능을 나타내는 비트마스크입니다. 서버가 지원하는 기능을 에이전트에 알리며, 첫 번째 `ServerToAgent` 메시지에서 설정되어야 합니다. 이후 메시지에서는 생략될 수 있습니다. 구체적인 필드는 [#servertoagentcapabilities](https://github.com/open-telemetry/opamp-spec/blob/main/specification.md#servertoagentcapabilities)를 통해서 확인하세요.

- **`agent_identification`**: 에이전트의 식별 정보를 포함합니다. 서버가 필요에 따라 새로운 `instance_uid`를 제공할 수 있으며, 에이전트는 이를 업데이트하여 모든 후속 통신에 사용해야 합니다. 구체적인 필드는 [#servertoagentagent_identification](https://github.com/open-telemetry/opamp-spec/blob/main/specification.md#servertoagentagent_identification)를 통해서 확인하세요.

- **`command`**: 서버가 에이전트에 특정 명령을 요청할 때 사용됩니다. 예를 들어, 에이전트를 재시작하도록 요청할 때 이 필드를 설정합니다. 이 필드는 `instance_uid` 또는 `capabilities` 필드와 함께 설정될 수 있으며, 다른 모든 필드는 무시됩니다.

- **`custom_capabilities`**: [Development] 서버가 지원하는 사용자 정의 기능을 나타내는 메시지입니다.

- **`custom_message`**: [Development] 서버가 에이전트에 보내는 사용자 정의 메시지를 포함합니다.

## ServerToAgentCommand Messages

ServerToAgentCommand 메시지는 서버가 에이전트에 특정 명령을 요청할 때 사용됩니다:

```protobuf
// ServerToAgentCommand is sent from the Server to the Agent to request that the Agent
// perform a command.
message ServerToAgentCommand {
    enum CommandType {
        // The Agent should restart. This request will be ignored if the Agent does not
        // support restart.
        Restart = 0;
    }
    CommandType type = 1;
}
```

해당 Command는 agent restart를 위해서 사용되며 `instance_uid`, `capabilities`, `command` 필드가 반드시 있어야되며 나머지 필드는 무시됩니다.

# 마무리

OpAMP는 대규모 에이전트 관리에 필수적인 다양한 기능을 제공하며, 이를 통해 효율적인 원격 관리가 가능합니다. 이번 글에서는 OpAMP가 나온 배경과 실제로 어떤 식으로 서로 통신하는지에 대해서 이야기했습니다.

다음 글에서는 OpAMP에서 실제로 해당 데이터들을 가지고 어떤 식으로 상호작용하는지 그리고 보안 측면에 대해 더 깊이 탐구하겠습니다.
