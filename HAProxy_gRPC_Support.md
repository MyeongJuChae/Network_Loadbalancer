# HAProxy의 gRPC 지원


https://www.haproxy.com/blog/haproxy-1-9-2-adds-grpc-support 내용을 번역 & 정리

## gRPC의 귀환
****

지난 10년 동안 서비스 개발을 해왔다면 SOAP와 같은 무거운 원격 프로시저 호출 프로토콜에서 REST와 같은 더 가볍고 HTTP 친화적인 패러다임으로 이동하는 것을 보셨을 것입니다. 업계가 RPC에서 멀어짐에 따라 전체 성숙도 모델 (Richardson 성숙도 모델 참조)이 개발되어 누구의 예상보다 HTTP를 사용하는 영역으로 더 나아가게 되었습니다.

그러나 여기 저기 어딘가에서 우리 모두는 JSON이 서비스간에 데이터를 전송하는 가장 좋은 (유일한?) 방법이라는 개념에 정착했습니다. 일리가 있습니다. JSON은 유연하고 쉽게 파싱되며 주어진 언어의 객체로 쉽게 역직렬화됩니다.

이 만능의 접근 방식으로 인해 많은 사람들이 JSON 메시지를 전달하여 통신하는 백엔드 서비스를 구현했습니다. 많은 데이터를 보내고 받아야하는 서비스나 6개의 다른 서비스와 통신하는 서비스도 모두 JSON에 의존했습니다.

HTTP 경로 및 메서드 모음으로만 정의된 서비스를 지원하기 위해 각각 인수를 다르게 전송하는 방법(URL의 일부? JSON 요청에 포함?)을 정의할 수있는 가능성이 있으므로 개발자는 자체 클라이언트 라이브러리를 롤링해야 했습니다. — 조직 내에서 사용되는 모든 프로그래밍 언어에 대해 반복되어야 하는 프로세스입니다.

그런 다음 프로토콜 버퍼라는 고유한 바이너리 직렬화를 사용하는 RPC 스타일 프레임 워크인 gRPC가 등장했습니다. 이를 통해 메시지를 더 빠르고 효율적으로 전달할 수 있었습니다. 클라이언트와 서버 간의 데이터는 지속적으로 스트리밍 될 수도 있습니다. 프로토콜 버퍼를 사용하는 gRPC는 클라이언트 SDK 및 서비스 인터페이스를 자동 생성할 수 있습니다. 분명히 RPC 패러다임이 화려하게 돌아 왔습니다.


## gRPC 사례
****
gRPC란 무엇이며 어떤 문제를 해결하려고 할까요? 2015년에 Google은 Square 및 기타 조직과 공동으로 개발한 원격 절차 호출을 통해 분산 프로그램을 연결하기 위한 새로운 프레임 워크인 gRPC를 오픈 소스로 제공했습니다. 내부적으로 Google은 이미 대부분의 공개 서비스를 gRPC로 마이그레이션했습니다. 프레임 워크는 Google 서비스가 달성한 규모에 필요한 기능을 제공했습니다.

그러나 gRPC는 다른 업계에서도 발생하는 문제를 해결합니다. 서비스 지향 아키텍처가 어떻게 바뀌었는지 생각해보십시오. 처음에 일반적인 패턴은 클라이언트가 단일 백엔드 서비스에 요청하고 JSON 응답을 받은 다음 연결을 끊는 것입니다. 오늘날 애플리케이션은 종종 비즈니스 트랜잭션을 더 많은 단계로 분해합니다. 단일 트랜잭션에는 6개 서비스와의 통신이 포함될 수 있습니다.

gRPC 프로토콜은 유선을 통해 텍스트 기반 JSON 메시지를 보내는 대신 사용할 수 있습니다. 대신 바이너리 데이터로 전송되는 프로토콜 버퍼를 사용하여 메시지를 직렬화하여 메시지를 더 작고 빠르게 만듭니다. 서비스 수를 늘리면 서비스 간의 지연 시간을 줄이는 것이 더욱 눈에 띄고 중요해집니다.

업계의 또 다른 변화는 서비스가 송수신해야하는 데이터의 급속한 성장입니다. 이 데이터는 상시 가동되는 IoT 장치, 리치 모바일 애플리케이션 또는 자체 로깅 및 메트릭 수집에서 가져올 수 있습니다. gRPC 프로토콜은 클라이언트와 서비스 간의 양방향 스트리밍을 활성화하기 위해 내부적으로 HTTP/2를 사용하여 이를 처리합니다. 이를 통해 데이터가 오래 지속되는 연결을 통해 앞뒤로 파이프되어 요청/메시지별 응답 패러다임의 한계를 벗어날 수 있습니다.

프로토콜 버퍼는 코드 생성도 제공합니다. 프로토콜 버퍼 컴파일러 인 protoc을 사용하여 여러 프로그래밍 언어로 서비스에 대한 클라이언트 SDK 및 인터페이스를 생성 할 수 있습니다. 이렇게하면 클라이언트와 서비스를 동기화 상태로 유지하는 것이 더 쉬워지고 이 상용구(boilerplate) 코드를 직접 작성하는 시간이 줄어 듭니다.

SOAP와 같은 이전 프레임 워크가 XML을 사용하여 이 기종 프로그래밍 언어를 연결하는 방법과 유사하게 gRPC는 프로토콜 버퍼로 공유되지만 독립적인 서비스 설명 언어로 사용합니다. gRPC를 사용하면 언어에 구애받지 않는 함수 서명이 포함된 공유 .proto 파일에서 인터페이스 및 메서드 스텁이 생성됩니다. 그러나 이러한 기능의 구현은 직접 연결되지 않습니다. 실제로 클라이언트는 실제 구현 대신 모의 서비스를 교체하여 단위 테스트를 수행하거나 필요한 경우 완전히 다른 구현을 가리킬 수 있습니다.


## HAProxy HTTP/2 지원
****
gRPC를 지원하려면 HTTP/2 지원이 필요합니다. HAProxy 1.9가 출시되면 클라이언트와 HAProxy간에 그리고 HAProxy와 백엔드 서비스 간에 HTTP/2 트래픽 부하를 분산할 수 있습니다. 이를 통해 gRPC를 메시지 전달 프로토콜로 활용할 수 있습니다. 현재 대부분의 브라우저는 gRPC를 지원하지 않습니다. 그러나 gRPC 게이트웨이와 같은 도구를 HAProxy 뒤에 배치하여 JSON을 gRPC로 변환할 수 있으며, 물론 자체 네트워크 내에서 서비스 간 gRPC 통신의 부하를 분산 할 수 있습니다.

이 섹션의 나머지 부분에서는 HAProxy가 이러한 기능을 제공하게 된 방법에 대해 알아 봅니다. 그런 다음 gRPC를 통해 양방향 스트리밍을 사용하는 애플리케이션을 시연합니다.

### 클라이언트와 프록시 간의 HTTP/2
---
HAProxy는 2017년 말 1.8 릴리스를 통해 자체와 클라이언트(예: 브라우저)간에 HTTP/2에 대한 지원을 추가했습니다. 일반적으로 확인되는 지연 시간이 서버와 브라우저 사이의 인터넷을 통과하는 네트워크 세그먼트에서 발생하기 때문에 HAProxy를 사용하는 사람들에게는 큰 승리였습니다. HTTP/2는 바이너리 형식(사람이 읽을 수있는 HTTP/1.1의 텍스트 기반 형식과 반대), 헤더 압축 및 단일 TCP 연결을 통한 메시지 프레임의 다중화로 인해 데이터를보다 효율적으로 전송할 수 있습니다.

HAProxy에서 이것을 활성화하는 것은 매우 간단합니다. TLS를 통해 바인딩하고 있는지 확인하고 `frontend`의 `bind` 지시문에 `alpn` 매개 변수를 추가하기만 하면 됩니다. 
```
frontend fe_mysite
    bind :443  ssl  crt /path/to/cert.pem  alpn h2,http/1.1
    default_backend be_servers
```

ALPN에 익숙하지 않다면 다음과 같은 간단한 요약을 살펴 보겠습니다. HTTP/1.1에서 TLS를 사용할 때 관례는 포트 443에서 수신하는 것입니다. HTTP/2가 등장했을 때 의문이 생겼습니다. 사람들이 이미 익숙한 포트와 다른 포트? 그러나 서버와 클라이언트가 사용할 HTTP 버전을 알 수있는 방법이 있어야 했습니다. 물론 프로토콜을 협상하는 완전히 별개의 핸드 셰이크가 있을 수 있었지만 결국에는 이 정보를 TLS 핸드 셰이크로 인코딩하여 왕복 시간을 절약하기로 결정했습니다.

RFC 7301에 설명된 대로 ALPN(Application-Layer Protocol Negotiation) 확장은 애플리케이션 프로토콜에 동의하는 클라이언트 및 서버를 지원하도록 TLS를 업데이트했습니다. 특히 HTTP/2를 지원하기 위해 만들어졌지만 향후 협상이 필요한 다른 프로토콜에 유용할 것입니다.

ALPN을 사용하면 클라이언트가 TLS ClientHello 메시지의 일부로 지원하는 프로토콜 목록을 선호하는 순서대로 보낼 수 있습니다. 그러면 서버는 TLS ServerHello 메시지의 일부로 선택한 프로토콜을 반환할 수 있습니다. 따라서 보시다시피 각 측에서 지원하는 HTTP 버전을 전달할 수 있다는 것은 실제로 기본 TLS 연결에 의존합니다. 어떤 면에서는 적어도 같은 포트에서 HTTP/1.1과 HTTP/2를 모두 지원하려면 더 안전한 웹으로 우리 모두를 밀어 붙입니다.

### 백엔드에 HTTP/2 추가
---
버전 1.8 릴리스 이후 HAProxy 사용자는 프런트 엔드에서 HTTP/2를 켜는 것만으로도 이미 성능 향상을 볼 수 있었습니다. 하지만 gRPC와 같은 프로토콜을 사용하려면 백엔드 서비스에도 HTTP/2를 사용해야 합니다. HAProxy Technologies의 오픈 소스 커뮤니티와 엔지니어가 문제를 해결했습니다.

이 과정에서 HAProxy가 HTTP 메시지를 구문 분석하고 수정하는 방법의 핵심 부분을 리팩토링 할 적절한 때라는 것이 분명해졌습니다. HTTP 메시지를 처리하기 위한 완전히 새로운 엔진이 개발되었습니다. 이 엔진은 Native HTTP Representation 또는 HTX 모드로 명명되었으며 버전 1.9와 함께 출시되었습니다. HTX 모드에서 HAProxy는 HTTP 프로토콜의 모든 표현을 보다 쉽게 ​​조작할 수 있습니다. 백엔드에 HTTP/2를 사용하기 전에 `http-use-htx` 옵션을 추가해야 합니다.


HTTP/2가 필요하고 HTTP/1.1로 대체 할 수 없는 gRPC의 경우 http/1.1을 모두 생략할 수 있습니다. 단일 프로토콜을 지정할 때 `alpn` 대신 `proto` 매개 변수를 사용할 수도 있습니다. 다음은 `bind` 및 `server` 라인에서 `proto`를 사용하는 예입니다.
```
frontend fe_mysite
   bind :443  ssl  crt /path/to/cert.pem  proto h2
   default_backend be_servers

backend be_servers
   balance roundrobin
   server server1 192.168.3.10:3000  ssl  verify none  proto h2  check  maxconn 20
```

`proto`를 사용할 때 `ssl` 매개 변수를 통한 TLS 활성화는 선택 사항입니다. 사용하지 않으면 HTTP 트래픽이 일반 상태로 전송됩니다. 프런트 엔드에서는 `alpn`을, 백엔드에서는 `proto`를 사용할 수 있으며 그 반대도 마찬가지입니다.

### You Could Always Do Layer 4 Proxying 
---
항상 전송 계층(layer 4) 프록시(예: 설정 모드 tcp)를 사용하여 HTTP/2 트래픽을 프록시 할 수 있습니다. 이 모드에서는 연결을 통해 전송되는 데이터가 HAProxy에 대해 불투명하기 때문입니다. 흥미로운 소식은 http 모드를 사용할 때 HTX를 통해 애플리케이션 계층 (layer 7)에서 트래픽을 종단 간 프록시 할 수 있다는 것입니다.

즉, 헤더, URL 및 요청 방법을 포함한 HTTP/2 메시지의 내용을 검사할 수 있습니다. 트래픽을 필터링하거나 특정 백엔드로 라우팅하도록 ACL 규칙을 설정할 수도 있습니다. 예를 들어 콘텐츠 유형 헤더를 검사하여 gRPC 메시지를 감지하고 구체적으로 라우팅 할 수 있습니다.

다음 섹션에서는 HAProxy를 사용하여 gRPC 트래픽을 프록시하는 예를 볼 수 있습니다.

## HAProxy gRPC 지원
***
Github에서 샘플 HAProxy gRPC 프로젝트를 다운로드하여 따르십시오. Docker Compose를 사용하여 환경을 가동합니다. 서버에서 무작위로 새로운 코드 명을 얻는 것을 보여줍니다 (예 : Bold Badger 또는 Cheerful Coyote). 여기에는 간단한 gRPC 요청/응답 예제와 HAProxy가 중간에 있는 보다 복잡한 양방향 스트리밍 예제가 포함됩니다.

### 프로토 파일
---
먼저 `sample/codenamecreator/codenamecreator.proto` 파일을 살펴보십시오. 이것은 프로토콜 버퍼 파일이며 gRPC 서비스가 지원할 방법을 나열합니다.
```
syntax = "proto3";

option go_package = "codenamecreator";

message NameRequest {
    string category = 1;
}

message NameResult {
    string name = 1;
}

service CodenameCreator {
    rpc GetCodename(NameRequest) returns (NameResult) {}
    rpc KeepGettingCodenames(stream NameRequest) returns (stream NameResult) {}
}
```

맨 위에는 `NameRequest` 메시지 유형과 `NameResult` 메시지 유형이 정의되어 있습니다. 전자는 `category`라는 문자열을 매개 변수로 사용하고 후자는 `name`이라는 문자열을 사용합니다. `GetCodename`이라는 함수와 `KeepGettingCodenames`라는 다른 함수가 있는 `CodenameCreator`라는 서비스가 정의됩니다. 이 예제 프로젝트에서 `GetCodename`은 서버에서 단일 코드 이름을 요청한 다음 종료됩니다. `KeepGettingCodenames`는 서버로부터 끝없는 스트림으로 코드 명을 지속적으로 받습니다.

.proto 파일에서 함수를 정의 할 때 매개 변수 또는 반환 유형 앞에 `stream`을 추가하면 스트림을 스트리밍 할 수 있습니다.이 경우 gRPC는 연결을 열어두고 요청 및/또는 응답이 동일한 채널에서 계속 전송되도록 합니다. 스트리밍없이 gRPC 서비스, 클라이언트에서만 스트리밍, 서버에서만 스트리밍, 양방향 스트리밍을 정의할 수 있습니다.

이 `.proto` 파일에서 클라이언트 및 서버 코드를 생성하려면 `protoc` 컴파일러를 사용합니다. Golang, Java, C ++ 및 C #을 포함한 다양한 언어에 대한 코드는 적절한 플러그인을 다운로드하고 인수를 통해 protoc에 전달하여 생성할 수 있습니다. 이 예에서는 `protoc-gen-go` 플러그인을 설치하고 `–go_out` 매개 변수를 사용하여 지정하여 Golang `.go` 파일을 생성합니다. 또한 사용자 언어에 맞는 프로토콜 버퍼 및 gRPC 라이브러리를 설치해야 합니다. `golang:alpine` Docker 컨테이너를 사용하여 클라이언트 `Dockerfile`의 시작 부분은 다음과 같은 환경을 구성합니다:
```
FROM golang:alpine AS build
RUN apk add git protobuf
RUN go get -u google.golang.org/grpc
RUN go get -u github.com/golang/protobuf/protoc-gen-go

# Copy files to container
WORKDIR /go/src/app
COPY . .

# Build proto file
WORKDIR /go/src/app/codenamecreator
RUN protoc --go_out=plugins=grpc:. *.proto
```

gRPC 서버를 위한 별도의 `Dockerfile`은 동일한 `.proto` 파일을 기반으로 코드를 생성해야 하기 때문에 여기까지 동일합니다. `codenamecreator.pb.go`라는 파일이 생성됩니다. 나머지 클라이언트 및 서버 `Dockerfile`은 gRPC 서비스를 구현하고 호출하는 각각의 Go 코드를 빌드하고 실행합니다.

다음 섹션에서는 서버 및 클라이언트 코드가 어떻게 구성되는지 살펴 보겠습니다.

### 서버 코드
***
gRPC 서비스의 server.go 파일은 다음과 같이 `.proto` 파일에 정의 된 `GetCodename` 함수를 구현합니다.

```
type codenameServer struct{}

func (s *codenameServer) GetCodename(ctx context.Context, request *creator.NameRequest) (*creator.NameResult, error) {
    generator := newCodenameGenerator()
    codename := generator.generate(request.Category)
    return &creator.NameResult{Name: codename}, nil
}
```

여기서 일부 사용자 지정 코드는 새로운 임의의 코드 이름(표시되지는 않지만 Github 저장소에서 사용 가능)을 생성하는 데 사용되며 이는 `NameResult`로 반환됩니다. 스트리밍 예제인 `KeepGettingCodenames`에는 훨씬 더 많은 작업이 있으므로 `codenamecreator.pb.go`에서 생성된 인터페이스를 구현한다고 하면 충분합니다.
```

func (s *codenameServer) KeepGettingCodenames(stream creator.CodenameCreator_KeepGettingCodenamesServer) error {
    // server implementation
}
```

아이디어를 제공하기 위해 서버는 `stream.Send()`를 호출하여 데이터를 채널로 보냅니다. 별도의 Go 루틴에서 동일한 `stream` 객체를 사용하여 클라이언트로부터 메시지를 수신하기 위해 `stream.Recv()`를 호출합니다. 서버는 포트 3000에서 연결 수신을 시작합니다. 다음과 같이 gRPC 서버를 만들 때 TLS 공개 인증서와 비공개 키를 제공하여 전송 계층 보안을 사용할 수 있습니다.
```
address := ":3000"
crt := "server.crt"
key := "server.key"

lis, err := net.Listen("tcp", address)
if err != nil {
    log.Fatalf("Failed to listen: %v", err)
}

creds, err := credentials.NewServerTLSFromFile(crt, key)
if err != nil {
    log.Fatalf("Failed to load TLS keys")
}

grpcServer := grpc.NewServer(grpc.Creds(creds))
```

HAProxy는 백엔드 `serer` 설정에 `ca-file /path/to/server.crt`를 추가하여 서버의 인증서를 확인할 수 있습니다. 인수없이 `grpc.NewServer`를 호출하여 TLS를 비활성화 할 수도 있습니다.

### 클라이언트 코드
***
protoc 컴파일러는 서비스에서 구현하는 Golang 인터페이스와 클라이언트에서 서비스 함수를 호출하는 데 사용할 클라이언트 SDK를 생성합니다. Golang의 경우이 모든 것이 생성된 단일 `.go` 파일에 포함됩니다. 그런 다음이 SDK를 사용하는 코드를 작성합니다.

클라이언트는 주소를 `grpc.Dial` 함수에 전달하여 서버에 대한 보안 연결을 구성합니다. 서버에 TLS를 사용하려면 `grpc.WithTransportCredentials` 함수를 사용하여 서버의 공개 키 인증서를 확인할 수 있어야 합니다.
```
address := os.Getenv("SERVER_ADDRESS") // haproxy URL
crt := os.Getenv("TLS_CERT") // haproxy.crt

creds, err := credentials.NewClientTLSFromFile(crt, "")
if err != nil {
    log.Fatalf("Failed to load TLS certificate")
}

conn, err := grpc.Dial(address, grpc.WithTransportCredentials(creds))
```

HAProxy는 클라이언트와 서버 사이에 위치하므로 주소는 로드 밸런서의 주소여야 하고 공개 키는 HAProxy `frontend`의 `bind` 라인에 지정된 .pem 파일의 인증서 부분이어야 합니다. TLS를 전혀 사용하지 않도록 선택하고 `grpc.Dial`의 두 번째 인수로 `grpc.WithInsecure()`를 전달할 수도 있습니다. 이 경우 TLS없이 수신하도록 HAProxy 구성을 변경하고 `proto` 인수를 사용하여 HTTP/2를 지정합니다.
```
bind :3001 proto h2
```
client.go 파일은 동일한 코드에서 구현된 것처럼 ``GetCodename`` 및 ``KeepGettingCodenames``를 호출할 수 있습니다. 이것이 바로 RPC 서비스의 힘입니다.
```
client := creator.NewCodenameCreatorClient(conn)
ctx := context.Background()

// simple, unary function call
result, err := client.GetCodename(ctx, &creator.NameRequest{Category: category})

// stream example, keeps connection open
fmt.Println("Generating codenames...")
stream, err := client.KeepGettingCodenames(ctx)
```

`GetCodename`과 같이 스트림을 사용하지 않는 gRPC 함수를 호출하면 이 함수는 서버에서 결과를 반환하고 종료됩니다. 이것은 아마도 대부분의 서비스가 작동하는 방식 일 것입니다.

스트리밍 예제의 경우 클라이언트는 `KeepGettingCodenames`를 호출하여 스트림 객체를 가져옵니다. 거기에서 `stream.Recv()`는 서버로부터 데이터를 받기 위해 무한 루프에서 호출됩니다. 동시에 `stream.Send`를 호출하여 10초 마다 데이터(이 경우 Science와 같은 새로운 범주)를 서버로 다시 보냅니다. 이러한 방식으로 클라이언트와 서버 모두 동일한 연결을 통해 병렬로 데이터를 보내고 받습니다.

다음 섹션에서는 레이어 7에서 gRPC 트래픽을 프록시하도록 HAProxy를 구성하는 방법을 알아 봅니다.

### HAProxy 구성
---
gRPC의 HAProxy 구성은 실제로 HTTP/2 호환 구성입니다.
```
global
    log stdout local0
    maxconn 50000
    ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
    ssl-default-bind-options ssl-min-ver TLSv1.1

defaults
    log global
    maxconn 3000
    mode http
    timeout connect 10s
    timeout client 30s
    timeout server 30s
    option httplog
    option logasap
    option http-use-htx

frontend fe_proxy
    bind :3001 ssl  crt /path/to/cert.pem  alpn h2
    default_backend be_servers

backend be_servers
    balance roundrobin
    server server1 server:3000 check  maxconn 20  ssl  alpn h2  ca-file /usr/local/etc/haproxy/pem/server.crt
```
`frontend` 내에서 `bind` 라인은 `alpn` 매개 변수 (또는 `proto`)를 사용하여 HTTP/2 (`h2`)가 지원되도록 지정합니다. 마찬가지로 `alpn` 매개 변수가 `backend`의 `server` 라인에 추가되어 엔드 투 엔드 HTTP/2를 제공합니다. 이 작업을 수행하려면 `http-use-htx` 옵션이 필요합니다.

몇 가지 주의 사항이 있습니다. 첫 번째는 클라이언트와 서버 간에 양방향으로 데이터를 스트리밍 할 때 HAProxy는 기본적으로 전체 요청/응답 트랜잭션이 완료되었을 때만 트래픽을 로깅하기 때문에 `option logasap`을 사용하여 HAProxy에 즉시 연결을 기록하도록 지시해야합니다. 이렇게 해야만 요청이 시작될 때 메시지를 기록합니다.
```
<134>Jan 15 14:38:46 haproxy[8]: 172.28.0.4:34366 [15/Jan/2019:14:38:46.988] fe_proxy~ be_servers/server1 0/0/2/0/+2 200 +79 - - ---- 1/1/1/1/0 0/0 "POST /CodenameCreator/KeepGettingCodenames HTTP/2.0"
```
디버그 로깅을 활성화하기 위해 `global` 섹션에 `debug` 추가 할 수도 있습니다. 그러면 요청 및 응답의 모든 HTTP/2 헤더가 표시됩니다.
```
POST /CodenameCreator/KeepGettingCodenames HTTP/2.0
content-type: application/grpc
user-agent: grpc-go/1.18.0-dev
te: trailers
host: haproxy:3001

HTTP/2.0 200
content-type: application/grpc
```
클라이언트에서 서버로 데이터를 스트리밍 할 때 `http-buffer-request` 옵션을 설정하지 마십시오. 이렇게 하면 전체 요청 본문을 수신할 때까지 HAProxy가 일시 중지되며 스트리밍시 오랜 시간이 걸립니다.

### 헤더 및 URL 경로 검사
---
gRPC 트래픽 프록시의 Layer 7 기능 중 일부를 보여주기 위해 애플리케이션 프로토콜을 기반으로 트래픽을 라우팅해야하는 필요성을 고려합니다. 예를 들어 동일한 `frontend`를 사용하여 gRPC 및 비 gRPC 트래픽을 모두 제공하고 각각을 적절한 `backend`로 보낼 수 있습니다. acl 문을 사용하여 트래픽 유형을 확인한 다음 다음과 같이 `use_backend`로 백엔드를 선택합니다.
```
frontend fe_proxy
    bind :3001 ssl  crt /path/to/cert.pem  alpn h2
    acl isgrpc req.hdr(content-type) -m str "application/grpc"
    use_backend grp_servers if isgrpc
    default_backend be_servers
```
헤더를 검사하는 또 다른 용도는 메타 데이터에서 작동하는 기능입니다. 메타 데이터는 요청에 포함할 수 있는 추가 정보입니다. 이를 활용하여 JWT 액세스 토큰 또는 비밀 암호를 전송하여 이를 포함하지 않는 모든 요청을 거부하거나 더 복잡한 검사를 수행할 수 있습니다. 클라이언트에서 메타 데이터를 보낼 때 gRPC 코드는 다음과 같습니다 (메타 데이터 패키지는 google.golang.org/grpc/metadata 임).
```

client := creator.NewCodenameCreatorClient(conn)
ctx := context.Background()

// Add some metadata to the context
ctx = metadata.AppendToOutgoingContext(ctx, "mysecretpassphrase", "abc123")
```
다음은 `http-request deny`를 사용하여 비밀 암호를 보내지 않는 요청을 거부하는 예입니다.
```
frontend fe_proxy
    bind :3001 ssl  crt /path/to/cert.pem  alpn h2
    http-request deny unless { req.hdr(mysecretpassphrase) -m str "abc123" }
    default_backend be_servers
```
다음과 같이 `capture request header`줄을 `frontend`에 추가하여 HAProxy 로그에 메타 데이터를 기록할 수도 있습니다.
```
capture request header mysecretpassphrase len 100
```
`mysecretpassphrase` 헤더가 중괄호로 묶인 로그에 추가됩니다.
```
<134>Jan 15 15:48:44 haproxy[8]: 172.30.0.4:35052 [15/Jan/2019:15:48:44.775] fe_proxy~ be_servers/server1 0/0/1/0/+1 200 +79 - - ---- 1/1/1/1/0 0/0 {abc123} "POST /CodenameCreator/KeepGettingCodenames HTTP/2.0"
```
HAProxy는 URL 경로에 따라 다른 백엔드로 라우팅 할 수도 있습니다. gRPC에서 경로는 서비스 이름과 기능의 조합입니다. 이를 알고 있으면 다음 예와 같이 예상 경로 `/CodenameCreator/KeepGettingCodenames`와 일치하는 ACL 규칙을 선언하고 그에 따라 트래픽을 라우팅 할 수 있습니다.
```
frontend fe_proxy
    bind :3001 ssl  crt /path/to/cert.pem  alpn h2
    acl is_codename_path path /CodenameCreator/KeepGettingCodenames
    acl is_otherservice_path path /AnotherService/SomeFunction
    use_backend be_codenameservers if is_codename_path
    use_backend be_otherservers if is_otherservice_path
    default_backend be_servers
```


## 결론
***
지금까지 HAProxy가 서비스 간 통신에 gRPC를 사용할 수 있도록 HTTP/2를 완벽하게 지원하는 방법을 배웠습니다. HAProxy를 사용하여 gRPC 요청을 적절한 백엔드로 라우팅하고, 서버간에 균등하게 부하를 분산하고, HTTP 헤더 및 gRPC 메타 데이터를 기반으로 보안 검사를 시행하고, 트래픽을 관찰 할 수 있습니다.

