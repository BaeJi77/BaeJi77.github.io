---
title: "대규모 Agent를 어떻게 관리하면 좋을까요? (feat. OpAMP) (2)"
description: "서버에서 관리되는 agent를 관리하는 OTEL에서 만드는 프로토콜에 대한 이야기"
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
- (현재 글) [대규모 Agent를 어떻게 관리하면 좋을까요? (feat. OpAMP) (2): OpAMP가 Agent를 처리하는 과정](https://baeji77.github.io/dev/observability/About-OpAMP-2/)

# 개요

무슨 이야기를 하고 싶은지. 어떤 이야기를 주제로 할것인지.

## TL;DR

해당 내용을 모두 이야기할 수 있는 요약

# 이전 글 요약

OpAMP 개요. otel은 무엇이고 해당 프로토콜이 나온 이유는 무엇이냐. 어떤 기능을 제공해주냐.

연결하는 방법. http 요청, websocket.

ServerToAgent, AgentToServer에 대한 데이터 교환 Spec

# 용어 정리

- Server: OpAMP Server이며, OpAMP를 통해서 OpAMP client와 통신하여서 Agent에 대한 관리를 도와준다.
- Client: OpAMP Client이며, OpAMP 서버와 통신하며 Agent에 대한 같은 host에서 직접적인 관리 및 실행을 담당한다.
- Agent: host에서 실행되는 process이며 otel에서 사용되는 가장 대표적인 예는 otel collector가 있다. 문서에서 Agent는 OpAMP Client + Agent 개념으로 사용하기도 합니다.

# Operation

OpAMP에 통해서 서로 교환하는 메시지들을 통해서 Server와 Client는 상호작용을 한다. 밑에서는 여러가지 상호작용에 대해서 알아갈것이다.

## Status Reporting

- Server에 연결한 직후 처음으로. 상태 보고서는 Client가 보내는 첫 번째 메시지여야 합니다.
- 이후에는 Agent의 상태가 변경될 때마다.

여기서 특별한 점은 처음에는 전체 메시지를 보내고 난 이후에 **변경에 대해서만 상태를 교환**한다는 것입니다.

`AgentToServer`와 `ServerToAgent` 메시지를 통해서 서로 통신하게 됩니다. 

Client가 Server에 대해서 데이터를 받아서 처리하고 그것에 대해서 Server로 다시 전송해줍니다.

```
          Agent   Client                                  Server

            │       │         ServerToAgent                 │
            ┌───────┤◄──────────────────────────────────────┤
            │       │                                       │
            ▼       │                                       │
        ┌────────┐  │                                       │
        │Process │  │                                       │
        │Received│  │                                       │
        │Data    │  │                                       │
        └───┬────┘  │                                       │
            │       │                                       │
            │Status │                                       │
            │Changed│         AgentToServer                 │   ┌─────────┐
            └──────►├──────────────────────────────────────►├──►│         │
                    │                                       │   │ Process │
                    │         ServerToAgent                 │   │ Status  │
                    │◄──────────────────────────────────────┤◄──┤         │
                    │                                       │   └─────────┘
```

```
              Agent  Client                                  Server

                │      │         ServerToAgent                 │
                ┌──────┤◄──────────────────────────────────────┤
                │      │                                       │
                ▼      │                                       │
            ┌────────┐ │                                       │
            │Process │ │                                       │
            │Received│ │                                       │
            │Data    │ │                                       │
            └───┬────┘ │                                       │
                │      │                                       │
                ▼      │                                       │
             No Status │                                       │
              Changes  │                                       │
                       │                                       │
                       │                                       │
```

여기서 중요한 것은 Server에서 받은 메시지를 통해서 특별한 변화가 있지 않다면 데이터를 보내지 않아야 됩니다. 만약 보내게 된다면 무한정으로 해당 메시지를 교환하게 됩니다.

### Agent Status Compression

Agent 상태가 특별히 변경되지 않았다면 메시지를 전송할때 해당 필드에 대해서는 데이터를 같이 보내지 않아도 됩니다. 해당 하는 필드들은 `AgentDescription, ComponentHealth, EffectiveConfig, RemoteConfigStatus, PackageStatuses, CustomCapabilities.` 입니다.

변경에 대해서 서로 통신을 하다보니 중간에 Server가 정상적이지 않았을 경우에 서로 데이터 손실이 있을수 있습니다. 그런 경우를 확인하기 위해서 `sequence_num`를 활용해서 확인할 수 있습니다. 해당 필드는 Client가 보낼 새로운 AgentToServer 메시지가 있을 때마다 증가하는 정수입니다.

Server은 이전에 Client와 통신할떄 `sequence_num`를 가지고 있는데 해당 값보다 정확하게 1차이가 나지 않다면 데이터 손실이 있다고 판단할 수 있습니다.

Server는 Agent에게 ServerToAgent 메시지를 보내고 `ServerToAgent` 메시지의 flags 필드에 `ReportFullState` 비트를 설정하게 되면 Client은 `AgentToServer`에 전체 상태를 전송합니다.

### AgentDescription message

```protobuf
message AgentDescription {
    repeated KeyValue identifying_attributes = 1;
    repeated KeyValue non_identifying_attributes = 2;
}
```

- identifying_attributes
  - Agent을 식별할 수 있는 데이터들을 가지고 있는 key/value 데이터 타입입니다.
  - otel에 대표적인 `otel collector` 같은 경우 `service.name, service.namespace, service.version, service.instance.id` 와 같은 값들을 가질 수 있습니다.
- non_identifying_attributes
  - Agent를 정확하게 식별하는 것은 아니지만 실행되는 위치에 대한 정보를 가집니다.
  - `os.type, os.version, cloud.*, host.*` 와 같은 값들을 가질 수 있습니다.

### ComponentHealth message

`ComponentHealth`은 Agent들에 대한 현재 health 상태를 나타냅니다.

```protobuf
message ComponentHealth {
    bool healthy = 1;
    fixed64 start_time_unix_nano = 2;
    string last_error = 3;
    string status = 4;
    fixed64 status_time_unix_nano = 5;
    map<string, ComponentHealth> component_health_map = 6;
}
```

각 필드에 대한 구체적인 설명은 [공식문서](https://opentelemetry.io/docs/specs/opamp/#componenthealth-message)를 참고하는 것을 추천드립니다.


### RemoteConfigStatus message

```
message RemoteConfigStatus {
    bytes last_remote_config_hash = 1;
    enum Status {
        // 상태 필드의 값이 설정되지 않았습니다.
        UNSET = 0;

        // 원격 구성이 Agent에 의해 성공적으로 적용되었습니다.
        APPLIED = 1;

        // Agent가 이전에 수신한 원격 구성을 현재 적용 중입니다.
        APPLYING = 2;

        // Agent가 이전에 수신한 구성을 적용하려고 했지만 실패했습니다.
        // 자세한 내용은 error_message를 참조하세요.
        FAILED = 3;
    }
    Status status = 2;
    string error_message = 3;
}
```

`last_remote_config_hash` 필드 값은 Agent가 받은 마지막으로 받은 configuration의 hash 값입니다. 

각 필드에 대한 구체적인 설명은 [공식문서](https://opentelemetry.io/docs/specs/opamp/#remoteconfigstatus-message)를 참고하는 것을 추천드립니다.

### PackageStatuses message

`PackageStatuses` 메시지는 Agent가 보유하거나 제공받은 모든 패키지의 상태를 설명합니다. 메시지는 다음과 같은 구조를 가집니다:

```
message PackageStatuses {
    map<string, PackageStatus> packages = 1;
    bytes server_provided_all_packages_hash = 2;
    string error_message = 3;
}
```

`PackageStatuses.server_provided_all_packages_hash`는 이 Agent가 이전에 Server로부터 PackagesAvailable 메시지를 통해 수신한 모든 패키지의 집계 해시입니다.

### PackageStatus Message

```
message PackageStatus {
    string name = 1;
    string agent_has_version = 2;
    bytes agent_has_hash = 3;
    string server_offered_version = 4;
    bytes server_offered_hash = 5;
    enum Status {
        INSTALLED = 0;
        INSTALLING = 1;
        INSTALL_FAILED = 2;
    }
    Status status = 6;
    string error_message = 7;
}
```

해당 메시지는 `PackageStatuses`와 같이 사용되는 메시지입니다. 

Agent가 패키지를 가지고 있냐에 따라서 설정되어야되는 값들이 달라질 수 있습니다. 다양한 케이스에 대해서 필드가 설정될 수 있습니다.

- `PackageStatus.agent_has_version`

Agent가 보유한 패키지의 버전입니다. Agent가 이 패키지를 보유한 경우 설정되어야 합니다.

Agent가 이 패키지를 보유하지 않은 경우 비어 있어야 합니다. 예를 들어 Server가 패키지를 제공했지만 설치에 실패했으며 Agent가 이전에 이 패키지를 보유하지 않은 경우가 있을 수 있습니다.

- `PackageStatus.agent_has_hash`

Agent가 보유한 패키지의 해시입니다. Agent가 이 패키지를 보유한 경우 설정되어야 합니다.

Agent가 이 패키지를 보유하지 않은 경우 비어 있어야 합니다. 예를 들어 Server가 패키지를 제공했지만 설치에 실패했으며 Agent가 이전에 이 패키지를 보유하지 않은 경우가 있을 수 있습니다.

- `PackageStatus.server_offered_version`

Server가 Agent에게 제공한 패키지의 버전입니다.

agent_has_version 및 server_offered_version 필드가 모두 설정되고 다른 값을 가질 수 있습니다. 예를 들어 Agent가 이미 패키지의 버전을 성공적으로 설치했으며, Server가 다른 버전을 제공했지만 Agent가 해당 버전을 설치하지 못한 경우 가능합니다.

- `PackageStatus.server_offered_hash`

Server가 Agent에게 제공한 패키지의 해시입니다.

agent_has_hash 및 server_offered_hash 필드가 모두 설정되고 다른 값을 가질 수 있습니다. 예를 들어 Agent가 이미 패키지의 버전을 성공적으로 설치했으며, Server가 다른 버전을 제공했지만 Agent가 해당 버전을 설치하지 못한 경우 가능합니다.

- `PackageStatus.status`

이 패키지의 상태입니다. 가능한 값은 다음과 같습니다:

`INSTALLED`: 패키지가 Agent에 의해 성공적으로 설치되었습니다. error_message 필드는 설정되지 않아야 합니다.

`INSTALLING`: Agent가 현재 패키지를 다운로드하고 설치 중입니다. server_offered_hash 필드는 Agent가 설치 중인 버전을 나타내기 위해 설정되어야 합니다. error_message 필드는 설정되지 않아야 합니다.

`INSTALL_FAILED`: Agent가 패키지를 설치하려고 했지만 설치에 실패했습니다. server_offered_hash 필드는 Agent가 설치하려고 했던 버전을 나타내기 위해 설정되어야 합니다. error_message는 실패에 대한 자세한 정보를 포함할 수도 있습니다.

- `PackageStatus.download_details`

download_details는 패키지 다운로드를 설명하는 추가 세부 정보를 포함합니다. 상태가 `DOWNLOADING`인 경우에만 설정되어야 합니다.

```
message PackageDownloadDetails {
  double download_percent = 1;
  uint64 download_bytes_per_second = 2;
}
```

## Connection Setting Management

OpAMP 서버와의 통신을 통해서 OpAMP를 포함하여 OTLP 서버 그리고 특정하지 않은 여러 서버들과의 connection을 할 수 있는 설정에 대한 정보를 교환할 수 있습니다.

OpAMP에서 connection setting에 대해서 다루는 이유는 보안적인 부분, 동적인 connection 관리, 운영 효율성적인 장점이 있습니다. 

앞으로 어떤 식으로 연결을 하면 어떤 메시지를 통해서 서로 통신하는지에 대해서 다뤄보겠습니다.

```
            ┌────────────┬────────┐           ┌─────────┐
            │            │ OpAMP  │  OpAMP    │ OpAMP   │
            │            │        ├──────────►│         │
            │            │ Client │           │ Server  │
            │            └────────┤           └─────────┘
            │                     │
            │            ┌────────┤           ┌─────────┐
            │            │OTLP    │ OTLP/HTTP │OTLP     │
            │  Agent     │        ├──────────►│Telemetry│
            │            │Client  │           │Backend  │
            │            └────────┤           └─────────┘
            │                     │
            │            ┌────────┤
            │            │Other   ├──────────► Other
            │            │        ├──────────►
            │            │Clients ├──────────► Destinations
            └────────────┴────────┘
```

OpAMP Server 및 다른 대상에 연결할 때 일반적으로 Agent(또는 Agent를 대신하여 연결하는 OpAMP Client)는 헤더 기반 인증 메커니즘(예: "Authorization" HTTP 헤더 또는 사용자 정의 헤더의 액세스 토큰)을 사용하고 선택적으로 TLS 연결을 위한 클라이언트 측 인증서(상호 TLS라고도 함)를 사용할 것입니다.

Server은 3가지 connection setting을 제공할 것입니다.

- OpAMP Server 그 자체.
- Telemetry(metrics, log and trace)를 OTLP/HTTP 프로토콜 전송받는 대상.
- Other Server. Agent 마다 또 연결해야되는 대상들이 추가적으로 있고 그 대상들을 다를 것이다.

Server는 Agent가 AgentToServer.capabilities 필드를 통해 연결을 사용할 수 있는 기능을 보고한 경우에만 특정 연결 클래스에 대한 제안을 할 수 있습니다:

`ReportsOwnLogs, ReportsOwnMetrics, ReportsOwnTraces, AcceptsOpAMPConnectionSettings, AcceptsOtherConnectionSettings`

### OpAMP Connection Setting Offer Flow

```
                   Client                                 Server

                     │                                       │    Initiate
                     │    Connect                            │    Settings
                     ├──────────────────────────────────────►│     Change
                     │                 ...                   │        │
                     │                                       │◄───────┘
                     │                                       │          ┌───────────┐
                     │                                       ├─────────►│           │
                     │                                       │ Generate │Credentials│
┌───────────┐        │ServerToAgent{ConnectionSettingsOffers}│ and Save │   Store   │
│           │◄───────┤◄──────────────────────────────────────┤◄─────────┤           │
│Credentials│ Save   │                                       │          └───────────┘
│   Store   │        │             Disconnect                │
│           ├───────►├──────────────────────────────────────►│
└───────────┘        │                                       │
                     │    Connect, New settings              │          ┌───────────┐
                     ├──────────────────────────────────────►├─────────►│           │
                     │                                       │ Delete   │Credentials│
┌───────────┐        │    Connection established             │ old      │   Store   │
│           │◄───────┤◄─────────────────────────────────────►│◄─────────┤           │
│Credentials│Delete  │                                       │          └───────────┘
│   Store   │old     │                                       │
│           ├───────►│                                       │
└───────────┘        │                                       │
```

1. Client에서 연결 요청 이후에 서버는 새로운 setting을 만들고 인증서 정보를 저장하여서 그 정보를 Client에 전송한다.
2. Client은 해당 정보를 받아서 설정을 끊고 새 요청을 보낸다.
3. 서버에서 받은 정보를 기반으로 새로운 Connection을 요청한다.
4. 그 요청으로 Server가 승인하게 되면 이전 인증서 정보를 삭제하고 연결 확인을 보낸다.

연결과정에 대한 디테일적인 부분을 생략하게되면 이렇게 연결을 진행한다고 파악할 수 있습니다.

### Trust On First Use

인증 정보가 없는 Clinet에게 연결 요청이 왔을 경우 서버에서 자동으로 혹은 수동으로 해당 연결을 승인하게 되면 추후 서로 교환한 인증서를 기반으로 통신하는 방법있습니다.

### Registration On First Use

대규모 Agent를 배포했을 경우 그리고 Client와 Server가 어느정도 인증을 위한 정보를 가지고 있을때 사용하면 좋습니다.

서로 약속한 정보가 있을 경우 자동으로 인증을 진행하는 방법입니다.

### Agent-initiated CSR Flow

Agent에서 key pair와 CSR를 Server로 전송하여서 Agent가 connection을 주도하는 방법입니다.

```
                   Client                                 Server

                     │ (1)           Connect                 │
                     ├──────────────────────────────────────►│
                     │                 ...                   │
┌───────────┐        │                                       │          ┌───────────┐
│ Generate  │  (2)   │ (3)     AgentToServer{CSR}            │(4)       │           │
│ Keypair   ├───────►├──────────────────────────────────────►├─────────►│  Approve  │
│ and CSR   │        │                 ...                   │          │           │
└───────────┘        │                                       │          └─────┬─────┘
                     │                                       │                │(5)
                     │                                       │                │
                     │                                       │                ▼
                     │                                       │          ┌───────────┐
┌───────────┐        │ServerToAgent{ConnectionSettingsOffers}│ (7)      │Create     │
│           │◄───────┤◄──────────────────────────────────────┤◄─────────┤Certificate│
│Credentials│ Save   │                                       │          │    (6)    │
│   Store   │        │             Disconnect                │          └───────────┘
│           ├───────►├──────────────────────────────────────►│
└───────────┘        │                                       │
                     │    Connect, New settings              │          ┌───────────┐
                     ├──────────────────────────────────────►├─────────►│           │
                     │                                       │ Delete   │Credentials│
┌───────────┐        │    Connection established             │ old      │   Store   │
│           │◄───────┤◄─────────────────────────────────────►│◄─────────┤           │
│Credentials│Delete  │                                       │          └───────────┘
│   Store   │old (8) │                                       │
│           ├───────►│                                       │
└───────────┘        │                                       │
```

간략하게 해당 flow를 설명한다면,

- Client는 일반 TLS을 사용해서 Server와의 신원을 검증합니다.
- Keypair와 CSR을 Server로 전송합니다.
- Server은 설정에 따라서 승인을 해줄수도 있고 자동으로 승인하도록 설정할 수 있습니다.
- 그 이후는 위에서 설명한 `OpAMP Connection Setting Offer Flow`와 똑같이 흘러갑니다.

#### Using instance_uid in the CSR

CSR payload에 `instance_uid`를 같이 넣어서 보내주면 Agent가 다른 Agent인 척하는 것을 방지할 수 있습니다.

### Revoking Access

Server은 header 정보와 인증서 정보를 기반으로 특정 Agent에 대해서 접근을 막을 수 있습니다.

OpAMP 서버뿐만 아니라 다른 설정에 대해서 credentials를 기반으로 연결을 차단할 수 있습니다.

### Agent to Server Message

```
message ConnectionSettingsRequest {
    OpAMPConnectionSettingsRequest opamp = 1;
}

message OpAMPConnectionSettingsRequest {
    CertificateRequest certificate_request = 1;
}

message CertificateRequest {
    bytes csr = 1;
}
```

#### ConnectionSettingsRequest Message

`opamp` 필드는 OpAMP 연결 설정 요청을 나타내기 위해 설정됩니다. 이 필드가 설정되지 않은 경우 Server에 대해 실행할 수 없습니다.

### OpAMPConnectionSettingsRequest Message

`certificate_request`는 Agent가 Server에 클라이언트 인증서를 생성하도록 요청할 때 설정됩니다. 이 필드는 필수입니다.

#### CertificateRequest Message

`csr` 필드는 클라이언트의 개인 키로 서명된 PEM 인코딩 클라이언트 인증서 서명 요청(CSR)입니다.

### Server to Agent Message

```
message ConnectionSettingsOffers {
    bytes hash = 1;
    OpAMPConnectionSettings opamp = 2;
    TelemetryConnectionSettings own_metrics = 3;
    TelemetryConnectionSettings own_traces = 4;
    TelemetryConnectionSettings own_logs = 5;
    map<string, OtherConnectionSettings> other_connections = 6;
}

message OpAMPConnectionSettings {
    string destination_endpoint = 1;
    Headers headers = 2;
    TLSCertificate certificate = 3;
    uint64 heartbeat_interval_seconds = 4;
}

message TelemetryConnectionSettings {
    string destination_endpoint = 1;
    Headers headers = 2;
    TLSCertificate certificate = 3;
}

message OtherConnectionSettings {
    string destination_endpoint = 1;
    Headers headers = 2;
    TLSCertificate certificate = 3;
    map<string, string> other_settings = 4;
}

message TLSCertificate {
    bytes cert = 1; // PEM-encoded certificate
    bytes private_key = 2; // PEM-encoded private key of the certificate
    bytes ca_cert = 3;
}
```

`ConnectionSettingsOffers.hash`는 전체 설정에 대한 hash 값이다. 

그 외에는 Server에서 Client에게 특정 서버와의 connection setting에 대한 정보를 제공해준다. 

각 message들은 거의 비슷한 구조를 가지고 있으면 모두 endpoint, headers, certificate 라는 정보를 가지고 있다.

## Own Telemetry Reporting

해당 기능은 OpAMP에서 Optional입니다.

Server는 Agent에게 OTLP를 지원하는 서버 정보를 제공하여서 Telemetey 데이터를 전달할 수 있도록 합니다.

```
            ┌────────────┬────────┐           ┌─────────┐
            │            │ OpAMP  │  OpAMP    │ OpAMP   │
            │            │        ├──────────►│         │
            │            │ Client │           │ Server  │
            │            └────────┤           └─────────┘
            │   Agent             │
            │            ┌────────┤           ┌─────────┐
            │            │OTLP    │ OTLP/HTTP │ OTLP    │
            │            │        ├──────────►│ Metric  │
            │            │Exporter│           │ Backend │
            └────────────┴────────┘           └─────────┘
```

Server은 처음 Agent가 connection을 할때 응답으로 ServerToAgent에 OTLP를 지원하는 서버에 대한 정보를 제공합니다.

만약 OTLP를 지원하는 서버 정보가 변경된다면 해당 정보에 대해서 추가적으로 데이터를 보내지만 만약 변경되지 않았다면 지속적으로 해당 정보는 보내지 않습니다. Agent에서 ServerToAgent에 connection_settings 필드에 어떤 변경도 없다면 지속적으로 Telemetey 데이터를 전송합니다.

권장 시간 간격은 10초입니다.

```
       Agent    Client                                Server
                                                            Metric
        │         │                                      │  Backend
        │         │ServerToAgent{ConnectionSettingsOffer}│
        ┌─────────│◄─────────────────────────────────────┤    │
        │                                                │    │
        ▼                                                     │
    ┌────────┐                                                │
    │Collect │                     OTLP Metrics               │ ──┐
    │Own     ├───────────────────────────────────────────────►│   │
    │Metrics │                                                │   │
    └────────┘                         ...                    .   │ Repeats
        │                                                         │
    ┌────────┐                                                │   │ Periodically
    │Collect │                     OTLP Metrics               │   │
    │Own     ├───────────────────────────────────────────────►│   │
    │Metrics │                                                │ ──┘
    └────────┘                                                │
```

해당 Telemetey 데이터를 전송할때 `AgentDescription` meseage에 identifying_attributes 필드, non_identifying_attributes 필드를 포함시켜서 전송합니다.

# 마무리

이번에는 OpAMP에서 실질적으로 Server와 Agent 사이세어 상태를 교환하고 그것을 처리하는 과정과 connection setting, 그리고 Agent 자체적으로 Telemetey를 처리하는 것에 대해서 알아봤습니다.

다음 시간에는 Package 관리, remote configuration 그리고 보안적인 부분들에 대해서 모두 이야기해 보겠습니다.

# ref

- [opamp-spec](https://github.com/open-telemetry/opamp-spec/blob/main/specification.md)
- [OpAMP Specification github](https://github.com/open-telemetry/opamp-spec?tab=readme-ov-file)
- [opamp supervisor](https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/cmd/opampsupervisor/specification/README.md)
- [opampsupervisor in otel-collector-cotribe](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/cmd/opampsupervisor)
- [opamp extention](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/extension/opampextension)
- [otel management document](https://opentelemetry.io/docs/collector/management/)
- [opamp-go](https://github.com/open-telemetry/opamp-go)
