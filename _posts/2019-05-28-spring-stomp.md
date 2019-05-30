---
title: STOMP -  Web Socket(1)
description: STOMP -  Web Socket(1)
header: STOMP -  Web Socket(1)
tags: [Java, Spring, WebSocket, Socket, STOMP]
keywords: 
    - en: Broker
      ko: 규약/프로토콜
      cn: 协议[xiéyì]
    - en: Subscribe
      ko: 구독
      cn: 订阅[dìngyuè]
    - en: Coupling
      ko: 결합
      cn: 耦合[Ǒuhé]
    - en: Inject
      ko: 주입하다
      cn: 注入
    - en: Credentials
      ko: 자격증명
      cn: 账号
    - en: Interval
      ko: 간격
      cn: 间隔[jiàngé]
---

웹소켓 프로토콜은 텍스트와 바이너리(binary) 두가지 메시지유형으로 정의된다. 하지만 콘텐츠는 정의되지 않는다.
클라이언트와 서버가 하위 프로토콜 협상하는 메커니즘을 정의한다. 즉, 웹소켓 위에서 각 메시지를 전송할 수 있는 메시지 유형, 각 메시지의 형식 및 내용 등을 정의하는 데 사용할 수 있는 상위 수준의 메시징 프로토콜.
하위 프로토콜 사용은 선택적이다. 하지만 클라이언트와 서버 어느쪽에서 메시지 내용을 정의하는 일부 프로토콜에 합의할 필요가 있을 것이다.

## Overview


[STOMP](https://stomp.github.io/stomp-specification-1.2.html#Abstract)는 Ruby, Python, Perl 등의 스크립트 언어를 위해 만들어진 간단한 문자 중심의 메시징 프로토콜이다. 
일반적으로 사용되는 메시징 패턴의 최소한 서브셋을 다루록 설계되었다.
STOMP는 TCP와 웹소켓과 같은 신뢰할 수 있는 양방향 스트리밍 네트워크 프로토콜에서 사용할 수 있다.
STOMP는 텍스트 지향 프로토콜이지만, 메시지 페이로드(payload) 텍스트나 바이너리일 수 있다.

<br/>

STOMP은 HTTP에서 모델링된 프레임 기반 프로토콜이다. STOMP 프레임의 구조:

<br/>

```
COMMAND
header1:value1
header2:value2

Body^@
```

<br/>

클라이언트는 SEND나 SUBSCRIBE 명령을 통해 메시지의 내용과 수신 대상을 설명하는 "destination" 헤더와 함께 메시지에 대해 전송이나 구독을 할 수 있다.
이것은 브로커를 통해 다른 연결된 클라이언트로 메시지를 보내거나 서버로 메시지를 보내 일부 작업을 수행하도록 요청할 수 있는 간단한 게시-구독(publish-subscribe) 메커니즘을 가능하게 한다. 

<br/>

스프링의 STOMP 지원을 할 때, 스프링 웹소켓 어플리케이션은 클라이언트에게 STOMP 브로커 역할을 한다.
메시지는 @Controller 메시지 처리 방법이나 구독자를 추적하여 구독중인 사용자에게 메시지를 전파(broadcast)하는 단순한 인메모리 브로커에게 라우팅된다.
실제 메시지 전파를 위해 STOMP 브로커(e.g. RabbitMQ, ActiveMQ 등)와 함께 작동하도록 스프링에서 구성할 수 있다.
이 사례로 스프링은 브로커와 TCP연결을 유지할 수 있고, 메시지를 중계하여, 연결된 웹소켓 클라이언트에 메시지를 전달한다.
스프링 웹 어플리케이션은 일관된 HTTP 기반 보안, 일반적 유효성검사, 친숙한 프로그래밍 모델 메시지 처리 작업에 의존할 수 있다.

<br/>

서버가 주기적으로 배출할 수 있는 주식시세(stock quotes)를 받기 위해 클라이언트 구독을 예로 들었다.
`SimpMessagingTemplate`를 통해 브로커에게 메시지를 보내는 스케줄링된 작업은 다음과 같다.  

<br/>

```
SUBSCRIBE
id:sub-1
destination:/topic/price.stock.*

^@
```

<br/>

아래 예제는 클라이언트에서 매매 요청을 보내는 에제이다. 서버에서 `@MessageMapping` 방법을 통해 처리할 수 있으며 나중에 실행 후 거래확인 메시지 및 세부정보를 클라이언트애게 전할 수 있다.

```
SEND
destination:/queue/trade
content-type:application/json
content-length:44

{"action":"BUY","ticker":"MMM","shares",44}^@
```

<br/>

목적지(destination)의 의미는 [STOMP](https://stomp.github.io/stomp-specification-1.2.html#Abstract) 스펙 상 의도적으로 불투명하게 남겨져 있다.
어떠한 string을 사용할 수 있고, 지원하는 목적지 구문과 의미를 정의하는 것은 전적으로 STOMP 서버에 달려있다.
하지만, 목적지가 경로와 같은 문자열이 되는 것은 매우 일반적이다. `"/topic/.."` 게시-구독(일 대 다)를 의미하고, `"/queue/"`는 Point to Point(일 대 일) 메시지 교환을 의미한다. 

<br/>

STOMP 서버는 MESSAGE 명령어를 사용하여 모든 구독자(subscribers)에게 전파할 수 있다.
다음은 서버가 주식 시세를 구독하는 클라이언트에게 전송하는 예제이다. 

```
MESSAGE
message-id:nxahklf6-1
subscription:sub-1
destination:/topic/price.stock.MMM

{"ticker":"MMM","price":129.45}^@
```

<br/>

중요한 것은 서버가 요청하지 않은 메시지를 보낼 수 없다는 것이다.
서버의 모든 메시지는 특정 클라이언트 구독의 결과에 응당해야 하고, 서버 메시지의 "subscription-id" 헤더는 클라이언트 구독의 "id" 헤더와 반드시 일치시켜야 한다.

<br/>

위 개요는 STOMP 프로토콜의 가장 기본적 이해를 제공할 목적을 가지고 있다. 모든 프로토콜 [스펙](https://stomp.github.io/stomp-specification-1.2.html)을 둘러보기를 추천한다.

## Benefits

STOMP를 서브 프로토콜로 사용하면 스프링 프레임워크와 스프링 시큐리티가 원시 웹소켓 사용에 비해 더 풍부한 프로그래밍 모델을 제공할 수 있다.

<br/>

동일한 점은 HTTP와 원시 TCP 비교 방법과 스프링 MVC 와 다른 웹 프레임워크는 풍부한 기능을 제공하는 방법이다.
혜택은 아래와 같다.

 - 사용자 정의 메시징 프로토콜과 메시지 형식을 개발할 필요가 없음.
 - STOMP 클라이언트는 스프링 프레임워크에서 자바 클라이언트를 포함하여 사용할 수 있다.
 - 메시지 브로커는 RabbitMQ, ActiveMQ 등으로 구독관리 및 전파 메시지를 (선택적으로) 사용할 수 있다.
 - 어플리케이션 로직은 @Controller 의 많은 수의 지정된 연결에 대해 단일 `WebSocketHandler`로 원시 웹소켓 메시지를 처리하는 것과 STOMP 목적지 헤더 기반의 라우팅 된 메시지로 구성할 수 있다. 
 - 스프링 시큐리티는 STOMP 목적지 와 메시지 유형을 기반으로 한 보안 메시지를 사용 한다.

## Enable STOMP

STOMP over WebSocket 지원은 `spring-messaging` 및 `spring-websocket` 모듈에서 사용할 수 있다.
이러한 종속성이 생기면 아래와 같이 [SockJS Fallback](http://minjoon.com/spring-sockjs-fallback)이있는 웹소켓을 통해 STOMP 엔드포인트를 노출 할 수 있습니다.                                 

```java
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS();  
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.setApplicationDestinationPrefixes("/app"); 
        config.enableSimpleBroker("/topic", "/queue"); 
    }
}
```

1. `"/portfolio"`는 필요한 웹소켓(or SockJS) 클라이언트가 웹소켓 핸드쉐이크에 대한 연결을 위해 연결할 필요가 있는 엔드포인의 HTTP URL이다.

2. STOMP 메시지는 `"/app"`로 목적지 헤더가 시작하는 곳의 `@Controller`의 `@MessageMapping` 메소드로 라우팅되된다.

3. 구독 및 전파 중계에는 내장된 메시지 브로커를 사용하라. `"/topic"`나 `"/queue"`로 시작하는 메시지를 브로커에게 전달한다.

<br/>

> "/topic"와 "/queue"로 시작하는 심플(simple) 브로커는 어떤한 특별한 의미도 없다.
그것들은 단지 pub-sub 대 point-to-point 메시징(즉, 많은 가입자 대 하나의 소비자)을 구별하기 위한 컨벤션일 뿐이다.
외부 브로커를 사용할 때, STOMP 페이지를 확인해서 어떤 종류의 STOMP 목적지와 접두사를 지원하는지 확인하자.

<br/>

브라우저에서 연결하려면 SockJS의 경우 [sockjs-client](https://github.com/sockjs/sockjs-client)를 사용할 수 있습니다.
STOMP의 많은 어플리케이션은 [jmesnil/stomp-websocket](https://github.com/jmesnil/stomp-websocket) 라이브러리 (stomp.js라고도 함)를 사용하여 기능이 완성되었으며, 수년 동안 업데이트가 되지 않는다. 
[JSteunou/webstomp-client](https://github.com/JSteunou/webstomp-client)는 가장 업데이트가 활성화되어 있고 진화된 버전이다. 아래 예제는 이 라이브러리를 기반으로 한다.

```java
var socket = new SockJS("/spring-websocket-portfolio/portfolio");
var stompClient = webstomp.over(socket);

stompClient.connect({}, function(frame) {
}
```

<br/>

SockJS를 사용하지 않고 웹소켓 연결하는 경우,

```java
var socket = new WebSocket("/spring-websocket-portfolio/portfolio");
var stompClient = Stomp.over(socket);

stompClient.connect({}, function(frame) {
}
```

예제를 참고해 보자.

 - [Using WebSocket to build an interactive web application getting started guide.](https://spring.io/guides/gs/messaging-stomp-websocket/)
 - [Stock Portfolio sample application.](https://github.com/rstoyanchev/spring-websocket-portfolio)

<br/>

참고로 위의 stompClient는 특정 로그인과 패스코드 헤더가 필요하지 않다.
인증정보에 대한 자세한 내용은 브로커와 인증 섹션에서 확인하자.

## Flow of Messages

STOMP 엔드포인트가 노출되면 스프링 어플리케이션은 클라이언트 연결을 위해 STOMP 브로커가 된다.
이번 섹션에서는 서버쪽의 메시지 흐름을 설명한다.

<br/>

`spring-messaging` 모듈은 스프링 통합에서 비롯된 메시징 어플리케이션에 대한 기본적인 지원을 포함하고 있고, 많은 스프링 프로젝트와 어플리케이션 시나리오에서 광범위하게 사용하기 위해 스프링 프레임워크 추출 및 통합되었다.

다음은 사용 가능한 메시징 추상화의 목록입니다.

 - Message — 헤더와 페이로드가 포함된 메시지에 대한 간단한 표현
 - MessageHandler — 메시지 처리 계약
 - MessageChannel — 생산자와 소비자 간 느슨한 결합(coupling)을 가능케하는 메시지 전송 계약
 - SubscribableChannel — `MessageHandler subscribers`를 가진 `MessageChannel`.
 - ExecutorSubscribableChannel — 메시지 전송을 위한 `Executor`를 사용하는 `SubscribableChannel`

<br/>

자바 설정 (i.e. `@EnableWebSocketMessageBroker`)은 위 컴포넌트를 사용하여 메시지 워크플로우(workflow) 를 구성한다.
아래 다이어그램은 내장 메시지 브로커가 활성화될 때 사용되는 구성요소를 보여준다. 

![Simple Broker](/img/spring-websocket/message-flow-simple-broker.png)

<br/>

위 다이어그램의 3가지 메시지 채널:

 - `"clientInboundChannel"` — 웹소켓 클라이언트에서 전달된 메시지를 전달한다.
 - `"clientOutboundChannel"` — 서버 메시지를 웹소켓 클라이언트에 보낸다.
 - `"brokerChannel"` — 서버쪽 어플리케이션 코드에서 메시지 브로커에세 메시지를 보낸다.

<br/>

다음 다이어그램에서 외부 브로커(e.g. RabbitMQ)는 메시지 구독과 전파을 위한 설정을 사용하는 컴포넌트를 보여준다.

![Broker Relay](/img/spring-websocket/message-flow-broker-relay.png)

위 다이어그램 중 주요 차이점은 TCP를 통해 외부 STOMP 브로커에게 메시지를 전달하고 브로커에서 구독한 클라이언트로 메시지를 전달하기 위해 "브로커 릴레이"를 사용하는 것이다. 
웹소켓 연결로부터 메시지를 전달받을 때, STOMP 프레임으로 디코딩된 다음 스프링 메시지 표현으로 변환하고, 추가 처리를 위해 `"clientInboundChannel"`로 전달한다.
예를 들어 목적지 헤더가 `"/app"`로 시작하는 STOMP 메시지는 어노테이션이 달린 컨트롤러의 `@MessageMapping` 메소드로 라우팅할 수 있고, `"/topic"`와 `"/queue"` 메시지는 메시지 브로커에게 직접 라우팅될 수 있다.

클라이언트에서의 STOMP 메시지 처리하는 어노테이션이 달린 `@Controller`가 "brokerChannel"을 통해 메시지 브로커에게 메시지를 보낸다.
브로커는 "clientOutboundChannel"를 통해 일치하는 구독자에게 메시지를 전파한다.
동일한 컨트롤러로 HTTP 요청 응답하여 동일한 작업을 할 수 있다. 클라이언트는 HTTP Post 수행할 수 있고, @PostMapping 방법은 메시지 브로커가 구독 클라이언트에게 메시지를 전파할 수 있다.

서버를 아래 예제와 같이 설정할 수 있다.

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio");
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setApplicationDestinationPrefixes("/app");
        registry.enableSimpleBroker("/topic");
    }

}

@Controller
public class GreetingController {

    @MessageMapping("/greeting") {
    public String handle(String greeting) {
        return "[" + getTimestamp() + ": " + greeting;
    }

}
```

<br/>

1. 클라이언트는 `"http://localhost:8080/portfolio"`로 연결하여 웹소켓 연결이 설정되면 STOMP 프레임이 그 위에서 흘러간다.

2. 클라이언트가 `"/topic/greeting"` 목적지 헤더인 `SUBSCRIBE` 프레임을 전송한다. 수신된 내용을 디코딩한 메시지를 `"clientInboundChannel"`로 전송한다. 클라이언트 구독을 저장하는 메시지 브로커로 라우팅한다.

3. 클라이언트는 `"/app/greeting"`에 `SEND` 프레임을 전송한다. `"/app"` 접두사는 어노테이션이 달린 컨트롤러로 라우팅하는데 도움을 준다. `"/app"` 접두사 다음 부분인 `"/greeting"` 부분은 GreetingController에서 `@MessageMapping` 메소드에 매핑된다.

4. GreetingController 의 반환 값은 반환 값과 `"/topic/greeting"` 의 기본 목적지 헤더를  기반으로 한 페이로드가 있는 스프링 메시지로 변환 한다. "brokerChannel"와 메시지 브로커의 처리를 결과메시지로 전송한다.

5. 메시지 브로커는 일치하는 모든 구독자 찾고, 메시지가 STOMP 프레임으로 인코딩된 메시지와 웹소켓 연결로 전송되는 `"clientOutboundChannel"`를 통해 `MESSAGE` 프레임을 전송한다.

<br/>

다음 섹션에서 인수 및 반환 값의 종류를 포함하여 주석 처리 방법에 대한 자세한 내용을 제공한다.

## Annotated Controllers

어플리케이션은 `@Controller` 어노테이션이 붙은 클래스를 사용하여 클라이언트에서 메시지를 처리할 수 있다.
이러한 클래스는 다음 설명된 것 처럼 `@MessageMapping`, `@SubscribeMapping`, `@ExceptionHandler` 메소드를 선언할 수 있다.

<br/>

#### @MessageMapping
`@MessageMapping` 어노케이션은 목적에 따라 메시지를 라우팅하는 메소드에 사용될 수 있다.
이는 메소드 수준 뿐만 아니라 유형 수준에서도 지원된다.
`@MessageMapping` 유형 수준은 컨테이너의 모든 메소드 맵핑에 대한 맵핑을 공유하는데 사용된다.

<br/>

기본 목적지 맵핑은 이런 패턴으로 예상할 수 있다. e.g. `"/foo*"`, `"/foo/**"`
패턴은 템플릿 변수(e.g. `"/foo/{id}"`)에 대한 지원을 포함한다. `@DestinationVariable` 메소드 인자를 참조할 수 있다. 

<br/>

`@MessageMapping` 메소드는 다음 인자들을 가진 유연한 특징을 가질 수 있다.

| 메소드 인자 | 설명 |
| :-------------- | :---------- |
| Message | 완전한 메시지에 액세스 하려면 |
| MessageHeaders | 메시지 내의 헤더에 액세스하려면 | 
| MessageHeaderAccessor, <br/>SimpMessageHeaderAccessor, <br/>StompHeaderAccessor | 입력된 접근자 방법을 통해 헤더에 액세스하려면 |
| @Payload | 메시지의 페이로드에 엑세스 하려면 구성된 MessageConverter를 통해 변환(e.g. JSON)하시오. <br> 일치하는 인자가 없으면 기본적으로 이 주석은 필요하지 않다. <br> 페이로드 인자는 자동으로 유효성검사 어노테이션(@javax.validation.Valid 나 스프링의 @Validated)이 달린다. |
| @Header | 필요하다면 org.springframework.core.convert.converter.Converter를 이용하여 특정 헤더 값에 엑세스 한다. |
| @Headers | 메시지의 모든 헤더에 접근한다. 이 인자는 java.util.Map로 지정해야만 한다. |
| @DestinationVariable | 메시지 목적지에서 추출된 템플릿 변수에 엑세스 한다. 값을 필요한 선언된 메서드 인자 유형으로 변환한다. |
| java.security.Principal | 웹소켓 HTTP 핸드쉐이크시, 로그인한 사용자를 반영한다. |

`@MessageMapping` 메소드에서 값을 반환 할때, 기본적인 값은 직렬화하여 설정된 `MessageConverter`를 통해 페이로드 된다. 그리고 메시지를 `"brokerChannel"`로 보내 구독자에게 전파한다. 
아웃바운드 메시지의 목적지는 인바운드 메시지와 동일하다. 하지만 `"/topic"`가 붙는다.

<br/>

`@SendTo` 메소드 어노테이션을 사용하여 목적지에서 페이로드할 대상을 커스터마이징할 수 있다.
`@SendTo`는 기본 대상 목적지에 메시지를 보내는 것을 클래스 수준에서 공유하는데 사용할 수 있다.
`@SendToUser`는 변형하여 메시지를 메시지와 관련된 사용자에게만 보낸다.

<br/>

`@MessageMapping` 메소드에서 반환되는 값은 `ListenableFuture`로 감싸진다. `CompletableFuture`나 `CompletionStage`를 사용하여 페이로드를 비동기 적으로 생성 할 수 있습니다.

<br/>

`@MessageMapping` 메소드에서 페이로드를 반환하는 대신 `SimpMessagingTemplate`을 사용하여 메시지를 보낼 수도 있다. 이 메소드는 반환 값을 처리하는 방법이기도 하다.

<br/>

#### @SubscribeMapping

`@SubscribeMapping` 어노테이션은 `@MessageMapping`과 함께 사용하기 위해 구독 메시지로 맵핑을 좁힌다.
이러한 시나리오에서 `@MessageMapping` 어노테이션은 대상을 지정하는 반면 `@SubscribeMapping`은 구독 메시지에 대한 관심만 나타낸다.

<br/>

`@SubscribeMapping` 메소드는 맵핑에 대한 것과 인수를 입력하는 `@MessageMapping` 메소드와 일반적으로 다르지 않다.
예를 들어 Type 수준 `@MessageMapping` 메소드를 공유 대상 접두사로 표현하는 것과 결합할 수 있고, `@MessageMapping` 메소드와 동일한 메소드 인자를 사용할 수 있다.

<br/>

`@SubscribeMapping`가 다른 것은 페이로드와 전송 시 직렬화 한 메소드 결과값을 "brokerChannel"이 아니라 "clientOutboundChannel"로 보내 브로커를 통해 전파하는 것이 아닌 직접 클라이언트에게 전송하는 것이다.
이는 일회성 메시지 교환을 실행하고, 회신 요청 메시지 교환을 수행하며, 가입 상태를 유지하지 않는 데 유용하다.
일회성 처리를 하고 요청-회신 메시지 교환, 구독 상태를 유지하지 않는 것이 유용하다.
이 패턴에 대한 일반적인 시나리오는 데이터가 로드(loaded)되고 출력되어야만 어플리케이션 초기화가 된다. 

<br/>

`@SubscribeMapping` 메소드는 `@SendTo`로 어노테이션을 달 수 있으며, 이 경우 반환 값이 명시적으로 특정 대상의 `"brokerChannel"`로 전송된다.

<br/>

#### @MessageExceptionHandler

어플리케이션은 `@MessageExceptionHandler` 메소드를 `@MessageMapping` 메소드 예외처리에 사용할 수 있다.
예외 인스턴스에 엑서스하려면 어노테이션 자체나 메소드 인자를 통해 예외처리를 선언할 수 있다. 

```java
@Controller
public class MyController {

    // ...

    @MessageExceptionHandler
    public ApplicationError handleException(MyException exception) {
        // ...
        return appError;
    }
}
```

<br/>

`@MessageExceptionHandler` 메소드는 유연한 메소드 시그니처를 지원하고, 동일한 메소드 인자 유형과 `@MessageMapping` 메소드의 반환 값을 지원한다.   

<br/>

일반적으로 `@MessageExceptionHandler` 메소드는 선언된 `@Controller` 클래스(또는 계층구조) 안에서 지원한다.  
이런 메소드를 컨트롤러 전체에 적용하려면 `@ControllerAdvice`가 표시된 클래스에서 선언 할 수 있다.
이것은 Spring MVC의 비슷한 지원과 비슷하다.

## Send Messages

어플리케이션 컴포넌트는 `"brokerChannel"`로 메시지를 전송한다.
가장 쉬운 방법으로 `SimpMessagingTemplate`를 주입하여 메시지 전송 기능을 사용한다.
일반적으로 유형별 주입되는 것이 쉽다.

```java
@Controller
public class GreetingController {

    private SimpMessagingTemplate template;

    @Autowired
    public GreetingController(SimpMessagingTemplate template) {
        this.template = template;
    }

    @RequestMapping(path="/greetings", method=POST)
    public void greet(String greeting) {
        String text = "[" + getTimestamp() + "]:" + greeting;
        this.template.convertAndSend("/topic/greetings", text);
    }

}
```

<br/>

동일한 유형의 다른 빈(Bean)이 존재 한다면 `"brokerMessagingTemplate"`이름으로도 자격을 얻을 수 있다. 

## Simple Broker

내장되어있는 심플 메시지 브로커는 클라이언트의 구독 요청, 메모리 저장공간, 일치하는 목적지와 연결된 클라이언트에 메시지 전파하는 것을 제어한다.
브로커는 앤트 스타일(Ant Style) 목적지 패턴에 대한 구독을 포함하여 경로와 같은 목적지를 지원한다.

> 어플리케이션은 닷(.)으로 구분된 대상도 사용할 수 있다.

## External Broker

심플 브로커는 처음 시작하기에 좋다. 하지만, STOMP 명령어(e.g. no acks, receipts, etc.)의 서브넷만 지원하고, 간단한 메시지 전송 루프나 클러스터링에는 적합하지 않다. 
대체할 수 있는 것은 어플리케이션이 Full-Featured 메시지 브로커로 업그레이드 하는 것이다.

<br/>

선택한 메시지 브로커(e.g. RabbitMQ, ActiveMQ, etc.)의 STOMP 문서를 확인해서, 브로커를 설치하고 STOMP 지원을 활성화 한 후 실행한다. 
그리고, 심플 브로커 대신 스프링 구성에서 STOMP 브로커 릴레이를 활성화한다.

<br/>

full-featured 브로커 설정 예제:

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableStompBrokerRelay("/topic", "/queue");
        registry.setApplicationDestinationPrefixes("/app");
    }

}
```

<br/>

위 설정의 `"STOMP broker relay"`은 외부 메시지 브로커로 포워딩하여 메시지를 처리하는 스프링 `MessageHandler`이다.
그렇게하기 위해 브로커에 대한 TCP 연결을 설정하고 모든 메시지를 브로커에 전달한 다음, 브로커에서 수신 한 모든 메시지를 웹소켓 세션을 통해 클라이언트에 전달한다.
기본적으로 메시지를 양방향으로 전달하는 `"릴레이"`역할을 합니다.

> TCP 연결관리를 위해 org.projectreactor:reactor-net, io.netty:netty-all 디펜던시를 프로젝트에 추가하자.

<br/>
  
Furthermore, application components (e.g. HTTP request handling methods, business services, etc.) can also send messages to the broker relay, as described in Send Messages, in order to broadcast messages to subscribed WebSocket clients.
또한 어플리케이션 컴포넌트 (예 : HTTP 요청 처리 메소드, 비즈니스 서비스 등)는 메시지 전송에 설명 된대로 브로커 릴레이에 메시지를 전송하여 구독중인 웹소켓 클라이언트에 메시지를 전파 할 수 있다.

<br/>

결과적으로 브로커 릴레이는 강력하고 확장 가능한 메시지 전파를 가능하게합니다.

## Connect to Broker

STOMP 브로커 릴레이는 브로커에 대한 단일 "시스템" TCP 연결하여 유지한다.
이 연결은 메시지를 수신하지 않고 서버측 어플리케이션에서만 발생하는 메시지에 사용된다. 
연결을 위한 STOMP 자격증명(즉, STOMP 프레임 로그인 및 패스코드 헤더)을 설정할 수 있다.  

<br/>

STOMP 브로커 릴레이는 연결된 모든 웹소켓 클라이언트에 대해 별도의 TCP 연결을 만든다.
클라이언트의 대신 모든 TCP 연결 생성하기 위해 STOMP 자격증명 사용을 설정할 수 있다. 

<br/>

> STOMP 브로커 릴레이는 항상 클라이언트를 대신하는 브로커에게 포워딩하는 모든 CONNECT의 로그인과 패스코드 헤더를 설정한다.
따라서 웹소켓 클라이언트는 헤더를 설정하는게 필요하지 않다. 그것들은 무시될 것이다.
웹소켓 클라이언트 대신 웹소켓 엔드포인트를 보호하고 클라이언트 ID 설정을 위해 HTTP 인증에 사용해야 한다.
인증 섹션에서 설명한 것처럼 WebSocket 클라이언트는 WebSocket 끝점을 보호하고 클라이언트 ID를 설정하기 위해 HTTP 인증을 사용해야합니다.

<br/>

STOMP 브로커 릴레이는 "시스템" TCP 연결을 통해 메시지 브로커와 heartbeat를 송수신한다.
heartbeat를 송수신하는 것에 대해 간격(interval) 설정할 수 있다. (기본 설정은 10초)
만약 브로커가 연결이 끊겼다고 보면, 브로커 릴레이는 매 5초마다 성공 할 때까지 재연결을 시도한다.

<br/>

스프링 빈은 브로커가 "시스템" 연결을 잃어버리거나 다시 설정할 때 수신 알림을 위해 `ApplicationListener<BrokerAvailabilityEvent>`를 구현한다.
예를 들어 주식 시세 서비스에서 주식 시세를 전파하는 것은 "시스템" 연결없을 때 메시지 전송을 중지할 수 있다.

<br/>

기본적으로 STOMP 브로커 릴레이는 항상 연결되어 있고, 연결이 끊기면 필요할 때 호스트와 포트로 재연결을 한다.  
다중 주소 지원을 원할 경우, 매 연결시도마다 주소 제공자를 설정할 수 있다. 대신 호스트와 포트는 정해져 있어야 한다.

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    // ...

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableStompBrokerRelay("/queue/", "/topic/").setTcpClient(createTcpClient());
        registry.setApplicationDestinationPrefixes("/app");
    }

    private ReactorNettyTcpClient<byte[]> createTcpClient() {

        Consumer<ClientOptions.Builder<?>> builderConsumer = builder -> {
            builder.connectAddress(()-> {
                // Select address to connect to ...
            });
        };

        return new ReactorNettyTcpClient<>(builderConsumer, new StompReactorNettyCodec());
    }
}
```

<br/>

STOMP 브로커 릴레이는 가상호스트 설정을 할 수 있다.
프로퍼티 값은 모든 `CONNECT` 프레임 호스트 헤더에 설정하여 클라우드 환경 같은데서 유용하다.
TCP 연결이 설정되는 실제 호스트 클라우드 기반 STOMP 서비스를 제공하는 호스트에서 다르다. 

## Dot as Separator

메시지가 `@MessageMapping` 메소드에 라우팅 되면 `AntPathMatcher`와 일치하며 기본 패턴은 슬러시 "/" 구분자를 사용한다.
이것은 웹 어플리케이션에서 좋은 규칙이고 HTTP URL과 비슷하다.
하지만 더 많은 메시지 규칙을 사용하려면 "." 구분자를 사용할 수 있다.

<br/>

자바 설정:

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    // ...

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setPathMatcher(new AntPathMatcher("."));
        registry.enableStompBrokerRelay("/queue", "/topic");
        registry.setApplicationDestinationPrefixes("/app");
    }
}
```


그리고 컨트롤러는 `@MessageMapping` 메서드에서 구분자로 점 "."을 사용할 수 있습니다.

<br/>

```java
@Controller
@MessageMapping("foo")
public class FooController {

    @MessageMapping("bar.{baz}")
    public void handleBaz(@DestinationVariable String baz) {
        // ...
    }
}
```

클라이언트는 `"/app/foo.bar.baz123"`로 메시지를 바로 보낼 수 있다.

<br/>

위 예제에서 외부 메시지 브로커에 전적으로 의존하기 때문에 `"broker relay"`의 접미사를 바꿀 수 없다.

<br/>

반면에 `"simple broker"`는 구성된 `PathMatcher`에 의존하므로 브로커에도 적용될 구분 기호를 전환하고 메시지의 대상과 구독의 패턴을 일치시키는 방법을 사용합니다.

<br/>

> 참고<br/>
> https://docs.spring.io/spring-framework/docs/5.0.5.RELEASE/spring-framework-reference/web.html
