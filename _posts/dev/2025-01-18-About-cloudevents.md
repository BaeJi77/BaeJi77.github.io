---
title: "Event를 발급할때 어떤 Spec을 사용하는게 좋을까? (feat. cloudevents)"
description: "우리는 event를 발급하고 그것을 사용할 때 어떤 spec을 이용할지 고민하는 경우가 한번쯤은 있을 것이다. CNCF에서 지정하여 발전했던 event spec인 cloudevents에 대해서 알아보자."
categories:
  - dev
  - cloud
  - event
tags:
  - dev
  - cloud
  - event
  - cloudevents
---

예제에 있는 코드들은 [해당 링크](https://github.com/BaeJi77/blog-code/blob/main/2025-01/cloudevents/src/main/kotlin/Main.kt)를 통해서 모두 볼 수 있습니다.

### 요약

CloudEvents는 다양한 서비스와 플랫폼 간의 이벤트 데이터를 표준화하는 데 초점을 맞춘 사양입니다. 이 글에서는 CloudEvents의 개념, 사양, 주요 특징, 실제 사용 사례, 장점과 한계를 종합적으로 다룹니다. 이를 통해 CloudEvents가 어떻게 멀티 클라우드 환경에서 데이터 상호 운용성과 개발 효율성을 제공하는지 이해할 수 있습니다.

- 왜 필요할까? **클라우드 환경에서 이벤트 데이터의 다양성과 일관성 부족으로 인한 문제를 해결.**
- 무엇을 제공할까? **표준화된 이벤트 형식과 프로토콜 독립성을 제공하며, 다양한 SDK와 데이터 형식을 지원.**
- 주요 이점: **시스템 통합성을 향상시키고, 멀티 클라우드 아키텍처에서 이벤트 데이터를 쉽게 교환.**

# CloudEvents란?

cloudEvents는 이벤트 데이터를 일관된 방식으로 설명하기 위한 사양(Spec)입니다. 이 기술은 다양한 서비스, 플랫폼 및 시스템 간의 상호 운용성을 제공하는 것을 목표로 합니다.

cloudEvents는 다음과 같은 주요 특징을 가지고 있습니다:

- 이벤트 데이터를 위한 표준화된 형식 제공
- 다양한 프로그래밍 언어를 위한 SDK 지원
- HTTP, AMQP, MQTT 등 다양한 프로토콜 바인딩 제공
- JSON, Avro, Protobuf 등 다양한 이벤트 형식 지원
- CNCF(Cloud Native Computing Foundation)에서 주관하는 프로젝트

# 등장배경

1. 이벤트 형식의 다양성: 클라우드 제공업체와 서비스마다 서로 다른 이벤트 형식을 사용하고 있었습니다.
2. 개발자 생산성 저하: 개발자들은 각 시스템마다 다른 이벤트 형식을 학습해야 했고, 이는 생산성 저하로 이어졌습니다.
3. 도구 및 인프라 제한: 이벤트 데이터의 일관성 부족으로 SDK, 이벤트 라우터, 추적 시스템 등의 개발이 제한되었습니다.
4. 이벤트 데이터의 이식성 부족: 서로 다른 환경 간에 이벤트 데이터를 이동하거나 활용하는 데 어려움이 있었습니다.
5. 상호 운용성 필요: 멀티 클라우드 및 하이브리드 클라우드 환경에서 이벤트 기반 아키텍처의 상호 운용성이 중요해졌습니다.

# Spec

CloudEvents 사양은 필수 필드와 선택적 필드로 구성되어 있습니다.

CloudEvents의 필드들을 자세히 살펴보겠습니다. 각 필드에 대해 타입, 설명 및 특징, 그리고 다양한 예시를 제공하겠습니다.

## 필수 필드

### id
- 타입: `String`
- 설명 및 특징: 
  - 이벤트의 고유 식별자로, 생산자가 정의합니다.
  - source + id의 조합은 각 이벤트를 고유하게 식별해야 합니다.
  - 동일한 이벤트가 재전송되는 경우(예: 네트워크 오류로 인한 재시도) 동일한 id를 사용할 수 있습니다.
  - 소비자는 동일한 source와 id를 가진 이벤트를 중복으로 간주할 수 있습니다.
- 예시:
  - UUID: "1234-5678-9101-1121"
  - 타임스탬프 기반: "2023-01-18T09:00:00.123Z"
  - 순차적 번호: "event-001"
  - 해시값: "f58d4b9a3e8c47a6b7f3c1d2e5f6g7h8"

### source
- 타입: `URI-reference`
- 설명 및 특징:
  - 이벤트가 발생한 컨텍스트를 식별합니다.
  - URI 형식을 사용하여 이벤트의 출처를 명확히 합니다.
  - 분산 시스템에서 여러 생산자를 구분하는 데 중요합니다.
- 예시:
  - "/myapp/user/profile"
  - "https://github.com/cloudevents"
  - "urn:uuid:6e8bc430-9c3a-11d9-9669-0800200c9a66"
  - "kafka://my-cluster/my-topic"

### specversion
- 타입: `String`
- 설명 및 특징:
  - 사용된 CloudEvents 사양의 버전을 나타냅니다.
  - 이벤트 소비자가 올바른 파싱 및 해석 방법을 결정할 수 있게 합니다.
  - 현재 가장 널리 사용되는 버전은 1.0입니다.
- 예시:
  - "1.0"
  - "0.3"

### type
- 타입: `String`
- 설명 및 특징:
  - 이벤트의 종류를 설명합니다.
  - 이벤트 생산자가 정의하며, 소비자가 이벤트를 분류하고 처리하는 데 사용됩니다.
  - 일반적으로 도메인 특화된 이벤트 유형을 나타냅니다.
- 예시:
  - "com.github.pull.create"
  - "com.example.object.delete.v2"
  - "aws.s3.object.created"
  - "google.cloud.pubsub.topic.publish"

## 선택적 필드

### datacontenttype
- 타입: `String`
- 설명 및 특징:
  - data 필드의 컨텐츠 타입을 지정합니다.
  - MIME 타입 형식을 사용합니다.
  - data 필드가 있는 경우 이 필드를 포함하는 것이 좋습니다.
- 예시:
  - "application/json"
  - "text/xml"
  - "application/octet-stream"
  - "text/plain; charset=utf-8"

### dataschema
- 타입: `URI`
- 설명 및 특징:
  - data 필드의 스키마를 참조하는 URI입니다.
  - 이벤트 데이터의 구조를 정의하고 검증하는 데 사용됩니다.
  - JSON Schema, Avro 스키마 등을 참조할 수 있습니다.
- 예시:
  - "https://schema.org/UserProfile"
  - "http://example.com/schemas/user-created.json"
  - "urn:schema:com.example:user:1.0"

### subject
- 타입: `String`
- 설명 및 특징:
  - 이벤트와 관련된 특정 주체나 리소스를 식별합니다.
  - source 내에서 이벤트의 컨텍스트를 더 구체적으로 설명합니다.
- 예시:
  - "user123@example.com"
  - "/users/1234"
  - "sensor-1234"
  - "order-9876"

### time
- 타입: `Timestamp`
- 설명 및 특징:
  - 이벤트가 발생한 시간을 나타냅니다.
  - RFC 3339 형식의 타임스탬프를 사용합니다.
  - 시간대 정보를 포함해야 합니다.
- 예시:
  - "2023-01-18T09:00:00Z"
  - "2023-01-18T09:00:00.123456789+09:00"
  - "2023-01-18T09:00:00-07:00"

### data
- 타입: `Any`
- 설명 및 특징:
  - 이벤트와 관련된 도메인별 정보를 포함합니다.
  - 구조화된 데이터(예: JSON 객체) 또는 비구조화된 데이터(예: 문자열, 바이너리)일 수 있습니다.
  - datacontenttype 필드와 함께 사용하여 데이터의 형식을 명시합니다.
- 예시:
  - JSON 객체: {"userId": "1234", "action": "login"}
  - 문자열: "File successfully uploaded"
  - Base64 인코딩된 바이너리: "SGVsbG8gQ2xvdWRFdmVudHMh"

## 실제 사용 예시

1. AWS EventBridge: CloudEvents 기반으로 이벤트를 전송하며, 다양한 AWS 서비스와의 통합성을 보장합니다.
   ```json
   {
     "specversion": "1.0",
     "id": "event-1234-5678",
     "source": "aws.s3",
     "type": "com.amazonaws.s3.object.created",
     "time": "2023-01-18T09:00:00Z",
     "datacontenttype": "application/json",
     "data": {
       "bucket": "my-bucket",
       "object": "path/to/file.txt",
       "size": 1024
     }
   }
   ```

2. Google Cloud Pub/Sub: CloudEvents를 활용하여 메시지를 전달하고, 데이터의 일관성을 보장합니다.
   ```json
   {
     "specversion": "1.0",
     "id": "9876543210",
     "source": "//pubsub.googleapis.com/projects/myproject/topics/mytopic",
     "type": "google.cloud.pubsub.topic.publish",
     "datacontenttype": "application/json",
     "time": "2023-01-18T09:05:00-07:00",
     "data": {
       "message": "Hello CloudEvents!",
       "attributes": {
         "key": "value"
       }
     }
   }
   ```

3. Knative Eventing: Kubernetes 환경에서 CloudEvents를 활용하여 이벤트 처리를 단순화합니다.
   ```json
   {
     "specversion": "1.0",
     "id": "abc-123-456",
     "source": "/apis/v1/namespaces/default/ping",
     "type": "dev.knative.sources.ping",
     "datacontenttype": "application/json",
     "time": "2023-01-18T09:10:00Z",
     "data": {
       "message": "Ping event triggered"
     }
   }
   ```

# 특징

CloudEvents는 이벤트 데이터를 표준화된 방식으로 설명하기 위한 개방형 사양으로, 여러 가지 중요한 특징과 장점을 가지고 있습니다. 이러한 특징과 장점들을 단계적으로 살펴보겠습니다.

## 1. 표준화된 이벤트 형식

### 특징

CloudEvents는 이벤트 데이터를 위한 공통 형식을 제공합니다. 이 형식은 필수 속성과 선택적 속성으로 구성되어 있어, 다양한 시스템에서 일관된 방식으로 이벤트를 표현할 수 있습니다.

표준화된 이벤트 형식은 이벤트 데이터의 표준화된 형식은 서로 다른 시스템, 플랫폼, 서비스 간의 상호 운용성을 크게 향상시킵니다.

### 예시

예를 들어, 한 기업이 여러 cloud provider의 서비스를 사용하고 있다고 가정해 봅시다. AWS Lambda에서 생성된 이벤트, Azure Event Grid에서 처리되는 이벤트, 그리고 Google Cloud Eventarc로 라우팅되는 이벤트가 모두 동일한 CloudEvents 형식을 사용한다면, 이 기업은 이벤트 처리 로직을 단순화하고 통합할 수 있습니다.

cloud provider뿐만 아니라 여러 오픈소스에서 cloudevents 호환가능한 데이터 포맷을 사용하고 있습니다. 더 많은 호환을 보기 위해서는 [공식 홈페이지](https://cloudevents.io/)를 참고하기를 바랍니다.

## 2. 확장성과 유연성

### 특징

CloudEvents는 기본 속성 외에도 확장 속성을 정의할 수 있는 기능을 제공합니다. 이를 통해 특정 도메인이나 사용 사례에 필요한 추가 정보를 포함할 수 있습니다. `확장 속성(Extension Attributes)`을 통해 확장성을 제공합니다.

이러한 확장성은 다양한 애플리케이션 및 도메인 시나리오에 CloudEvents를 적용할 수 있게 해주며, 특정 요구사항에 맞춰 이벤트 데이터를 커스터마이즈할 수 있습니다.

### 예시 및 상황

금융 서비스 회사에서 거래 이벤트를 처리하는 경우를 생각해봅시다. 기본 CloudEvents 속성 외에도 '거래 ID', '계좌 번호', '거래 금액' 등의 확장 속성을 정의하여 사용할 수 있습니다. 이렇게 하면 표준 형식을 유지하면서도 금융 거래에 특화된 정보를 포함할 수 있습니다[3][7].

## 3. 프로토콜 독립성

### 특징

CloudEvents는 다양한 전송 프로토콜을 지원합니다. HTTP, AMQP, MQTT 등 여러 프로토콜에 대한 바인딩을 제공하여, 다양한 환경에서 이벤트를 전송할 수 있습니다.

이러한 프로토콜 독립성은 CloudEvents를 다양한 시스템 아키텍처에 적용할 수 있게 해주며, 기존 인프라와의 통합을 용이하게 합니다.

## 그 외에 특징

### event size limit

CloudEvents 사양은 이벤트 크기를 64KB로 제한하도록 권장하지만, 실제 구현에서는 이보다 큰 이벤트를 허용하는 경우가 많습니다. 이는 권장사항일 뿐이며, 특정 사용 사례에 따라 더 큰 크기의 이벤트를 사용할 수 있습니다.

- size limit sdk implementaion
  - 대부분의 CloudEvents SDK는 64KB 제한을 **명시적으로 강제하지 않습니다**.
  - 대신, 이벤트 생성 시 크기를 확인하거나 경고하는 기능을 제공하는 경우가 있습니다.
- 유연성
  - 많은 SDK들은 64KB 이상의 이벤트도 허용합니다.
  - 이는 다양한 사용 사례와 시스템 요구사항을 수용하기 위함입니다.
- 권장사항
  - SDK들은 일반적으로 64KB 제한을 권장사항으로 취급합니다.
  - 개발자들에게 가능한 한 작은 크기의 이벤트를 사용하도록 권장합니다.

정리하자면 실질적으로 코드적(지원하는 sdk)으로 제한하지는 않습니다. 권장사항이기에 대안방안으로 event 크기가 크다면 링크를 제공해서 해당 데이터를 직접 접근해서 사용하는 방식을 권장합니다.

### version 관리

CloudEvents의 버전 관리는 이벤트 생산자와 소비자 간의 호환성을 유지하면서 이벤트 스키마를 진화시키는 중요한 측면입니다. CloudEvents 사양은 특정 버전 관리 패턴을 강제하지 않지만, 몇 가지 권장 사항과 접근 방식을 제시합니다.

## CloudEvents의 버전 관리 접근 방식

CloudEvents는 주로 두 가지 속성을 통해 버전 관리를 지원합니다:

1. `type` 속성
   - 소비자가 이벤트 유형을 쉽게 식별하고 필터링할 수 있습니다.
   - 주요 변경사항을 명확히 표시할 수 있습니다.
   - 예시:
        ```
        "type": "com.example.object.created.v1"
        "type": "com.example.object.created.v2"
        ```
2. `dataschema` 속성
   - 스키마 변경을 유연하게 관리할 수 있습니다.
   - 소비자가 스키마를 직접 확인하고 처리해야 할 수 있습니다.
   - 예시:
        ```
        "dataschema": "https://example.com/schemas/user-created.v1.json"
        "dataschema": "https://example.com/schemas/user-created.v2.json"
        ```

# 실제 사용기

## protobuf

CloudEvents의 Protobuf message를 이용해서 cloudevents를 사용하는 방법이 존재합니다.

### CloudEvents Protobuf 구조

CloudEvents Protobuf 형식은 다음과 같은 구조를 가집니다:

```protobuf
message CloudEvent {
  // 필수 속성들
  string id = 1;
  string source = 2;
  string spec_version = 3;
  string type = 4;

  // 선택적 속성들과 확장 속성들
  map<string, CloudEventAttributeValue> attributes = 5;

  // 데이터
  oneof data {
    bytes binary_data = 6;
    string text_data = 7;
    google.protobuf.Any proto_data = 8;
  }
}

message CloudEventAttributeValue {
  oneof attr {
    bool ce_boolean = 1;
    int32 ce_integer = 2;
    string ce_string = 3;
    bytes ce_binary = 4;
    string ce_uri = 5;
    string ce_uri_reference = 6;
    google.protobuf.Timestamp ce_timestamp = 7;
  }
}

oneof data {
    // Binary data
    bytes binary_data = 2;

    // String data
    string text_data = 3;

    // Protobuf Message data
    google.protobuf.Any proto_data = 4;
}
```

### 실제 사용 코드 예시

밑에 있는 예제 코드는 java 코드를 기반으로 만들었습니다. 

protobuf sdk 같은 경우 data에 `byte`, `text`, `protobuf` 이 3가지 유형에 값을 넣을 수 있습니다. 

```java
public static CloudEvent plainTextExample() {
  CloudEvent.Builder ceBuilder = CloudEvent.newBuilder();

  ceBuilder
    //-- REQUIRED Attributes.
    .setId(UUID.randomUUID().toString())
    .setSpecVersion("1.0")
    .setType("io.cloudevent.example")
    .setSource("producer-1")

    //-- Data.
    .setTextData("This is a plain text message");

  //-- OPTIONAL Attributes
  withCurrentTime(ceBuilder, "time");
  withAttribute(ceBuilder, "datacontenttype", "text/plain");

  // Build it.
  return ceBuilder.build();
}

private static Spec.CloudEvent protoExample() {

  //-- Build an event data protobuf object.
  Test.SomeData.Builder dataBuilder = Test.SomeData.newBuilder();

  dataBuilder
    .setSomeText("this is an important message")
    .setIsImportant(true);

  //-- Build the CloudEvent.
  CloudEvent.Builder ceBuilder = Spec.CloudEvent.newBuilder();

  ceBuilder
    .setId(UUID.randomUUID().toString())
    .setSpecVersion("1.0")
    .setType("io.cloudevent.example")
    .setSource("producer-2")

    // Add the proto data into the CloudEvent envelope.
    .setProtoData(Any.pack(dataBuilder.build()));

  // Add the protto type URL
  withAttribute(ceBuilder, "dataschema", ceBuilder.getProtoData().getTypeUrl());

  // Set Content-Type (OPTIONAL)
  withAttribute(ceBuilder, "datacontenttype", "application/protobuf");

  //-- Done.
  return ceBuilder.build();

}
```

## kafka

제가 생각하기에 event 같은 경우 kafka를 이용해서 많이들 이용할 가능성이 높다고 생각합니다. 그래서인지 sdk 차원에서 cloudevents가 잘 지원되고 있다고 생각합니다. 

kakfa는 데이터를 보내는 형식이 두가지 입니다.

1. kafka header에 context attritubes 데이터들을 넣어서 보내고 실제 데이터는 kafka value에 넣는 방법.

해당 방식으로 하게 되면 data를 제외한 metadata가 kafka headers에 들어간 것을 볼 수 있습니다. 이렇게 되면 kafka의 schema를 확인하는 기능까지 활용할수도 있습니다.

```
------------------ Message -------------------

Topic Name: mytopic

------------------- key ----------------------

Key: mykey

------------------ headers -------------------

ce_specversion: "1.0"
ce_type: "com.example.someevent"
ce_source: "/mycontext/subcontext"
ce_id: "1234-1234-1234"
ce_time: "2018-04-05T03:56:24Z"
content-type: application/avro
       .... further attributes ...

------------------- value --------------------

            ... application data encoded in Avro ...

-----------------------------------------------
```

2. kafka header에 content-type를 cloudevent라고 명시적으로 기록한 이후에 kafka value에 cloudevent를 json으로 바꾼 것을 모두 넣어서 보내는 방식

해당 방법은 kafka headers에 content-type만 넣고 모든 cloudevents에 대한 값을 json으로 value에 넣는 것입니다.

```
------------------ Message -------------------

Topic Name: mytopic

------------------- key ----------------------

Key: mykey

------------------ headers -------------------

content-type: application/cloudevents+json; charset=UTF-8

------------------- value --------------------

{
    "specversion" : "1.0",
    "type" : "com.example.someevent",
    "source" : "/mycontext/subcontext",
    "id" : "1234-1234-1234",
    "time" : "2018-04-05T03:56:24Z",
    "datacontenttype" : "application/xml",

    ... further attributes omitted ...

    "data" : {
        ... application data encoded in XML ...
    }
}

-----------------------------------------------
```

### 실제 사용 코드 예시

producer와 consumer에서 configuration에 Seriallization Deserialization에 대해서 cloudevent에 대한 타입을 지정하며 CloudEventSerializer, CloudEventDeserializer를 통해서 자동으로 가능하다.

위에서 설명했던 kafka에 value를 어떻게 넣냐는 producer에서 Seriallization 하는 configuration에서 설정할 수 있습니다. 자세한 사용은 [해당 링크](https://github.com/cloudevents/sdk-java/blob/main/kafka/src/main/java/io/cloudevents/kafka/CloudEventSerializer.java)를 참고하세요.

저는 특별히 kafka 환경까지 모두 구성하기에는 힘들기에 간단하게 kotlin 코드에서 consumer쪽에 코드를 상상하며 구현해봤습니다. 실질적으로 코드와는 다르지만 해당 로직을 참고하여서 여러 가지 것들을 할 수 있을것으로 예상됩니다.

현재 코드상에서는 많은 것을 압축했지만 `consumerCloudEvents`라는 메소드가 지속적으로 kafka로 오는 데이터를 읽는다는 가정입니다. **이 상황에서 코드에서는 cloudevents의 `type`와 `dataSchema`를 확인하여서 해당 version을 처리할지 말지 결정할 수 있습니다.**

```kotlin

data class MyEvent @JsonCreator constructor(
    @JsonProperty("id") val id: String,
    @JsonProperty("type") val type: String
)

val objectMapper = ObjectMapper()

fun consumerCloudEvents(events: CloudEvent) {
    val cloudEventsData =
        events.data?.let { PojoCloudEventDataMapper.from(objectMapper, MyEvent::class.java).map(it) }?.value

    when (events.type) {
        "com.example.someevent" -> {
            // do something
        }
        // add more cases here
        else -> {
            error("Unknown event type: ${events.type}")
        }
    }

    when (events.dataSchema) {
        URI.create("http://example.com/schema.json") -> {
            // do something
        }
        // add more cases here
        else -> {
            error("Unknown schema: ${events.dataSchema}")
        }
    }

    println(cloudEventsData)

    // do something ...
    println("do something...")
}

=== print
MyEvent(id=test-id, type=test-type)
do something...

```

# 나의 생각

cloudevents는 event를 어떻게 다루고 처리할지 그리고 어떤 데이터 형식으로 교환할지에 대한 생각에서 나왔다고 생각합니다. 개발자들은 비슷한 상황과 고민을 경험하고 그것을 해결하고 싶어서 만들어졌다고 생각합니다. 그것에 대한 공감으로 CNCF 지원을 통해서 성장할수도 있었다고 생각합니다.

개인적으로 모든 event 타입에 대해서 해당 cloudevents를 사용하는 것은 귀찮음과 실제로 효율적이지 않을수도 있습니다. 제가 생각하는 단점은 기본적으로 순수 데이터 뿐만 아니라 메타데이터를 통해서 (해당 데이터들이 디버깅에 도움이 되지만) 데이터 크기 상승과 처리 속도를 낮출수도 있다고 생각합니다. 실제로 그렇게 크지 않지만 많은 데이터를 처리할때는 영향을 줄 수 있는 부분도 있다고 생각합니다.

만약 제가 사용한다면 `외부 시스템 혹은 배포 사이클이 다른 컴포넌트끼리 통합을 위해서 메시지를 만들어야될 때` 장점이 있다고 생각합니다. 특히 버전 관리를 할 수 있는 부분이 가장 좋으며 실제 consumer 입장에서 데이터를 단순히 읽는 것이 아니라 일단 데이터를 받고 처리할 수 있는지 없는지를 type와 dataschema를 통해서 판단해서 미리 처리할 수 있다는 것이 장점이라고 생각합니다.

이런 장점은 같은 배포 사이클에 있는 컴포넌트끼리에는 큰 장점이 아니라고 생각하면 외부 시스템과 통신할때 가장 큰 효과를 발휘할 수 있지 않을까 생각합니다.

# 마무리

이번에 cloudevents라는 event data spec에 대해서 알아봤습니다. 한번쯤 Event를 어떤 형식으로 교환하면 좋을지에 대해서 생각해볼만한데 해당 Spec을 보면서 이런 관점이 있다는 것을 알 수 있었습니다.

나중에 현재 cloudevents를 사용하면서 어떻게 사용 더 좋을지에 대한 후기를 남길 수 있으면 좋겠습니다.

# ref

- https://github.com/cloudevents/spec
- https://www.cncf.io/projects/cloudevents/
- https://cloudevents.io/
