---
title: SockJS Fallback -  Web Socket
description: SockJS Fallback -  Web Socket
header: SockJS Fallback -  Web Socket
tags: [Java, Spring, WebSocket, Socket, SockJS]
keywords: 
    - en: Protocol
      ko: 규약/프로토콜
      cn: 协议[xiéyì]
    - en: Interaction
      ko: 상호작용
      cn: 相互作用
---

공용 인터넷을 통해 통제할 수 없는 제한된 프록시는 업그레이드 헤더를 설정으로 전달하지 않거나 유휴 상태로 보일만큼 오래 지속된 연결을 단기 때문에 웹소켓의 상호작용을 방해할 수 있다.  

해결 방법으로는 웹소켓 에뮬레이션. 즉, 웹소켓을 먼저 사용하고 웹소켓 상호작용을 에뮬레이션 하는 것과 동일한 어플리케이션 레벨의 API를 노출 하는 HTTP 기반 기술을 되돌리려 시도한다.

스프링 프레임워크의 서블릿 스택은 SockJS 프로토콜을 위한 서버(클라이언트) 모두를 지원한다. 

<br/>

## Overview

SockJS의 목표는 어플리케이션이 웹소켓 API를 사용하도록 하는 것이다. 하지만, 런타임에 필요할 때 비웹소켓(non-WebSocket)으로 대체한다. (i.e. 어플리케이션 코드 변경이 필요하지 않다.)

<br/>

SockJS 구성:

 - SockJS 프로토콜은 실행가능한 narrated 테스트 형식을 정의한다.
 - SockJS 자바스크립트 클라이언트 - 브라우저에서 사용하는 클라이언트 라이브러리
 - SockJS 서버는 스프링 프레임워크 spring-websocket 모듈 중 하나를 포함하여 구현한다.
 - 4.1 spring-websocket은 SockJS 자바 클라이언트를 제공하는 것 같다. 

<br/>

브라우저에서 사용하기 위해 설계된 SockJS이다. 이는 다양한 기법을 사용하여 광범위한 버전을 지원하는데 매우 긴 시간이 걸린다.
SockJS 전송유형과 브라우저의 전체 목록은 [SockJS 클라이언트 페이지](https://github.com/sockjs/sockjs-client/)를 참조하시오.
3가지 일반적인 전송실패를 구분하면 WebSocket, HTTP Streaming, and HTTP Long Polling.
전체 내용에 대한 내용은 [블로그](https://spring.io/blog/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/)에서 볼 수 있다.

SockJS 클라이언트는 "GET /info" 정보를 전송하여 서버에서 기본 정보를 받는다. 
그리고, 어떤 전송 수단을 사용할지 선택해야 한다. 
가능한 웹소켓을 사용하고, 그렇지 않으면 대부분의 브라우저에는 최소 하나의 HTTP 스트리밍 옵션이 있거나 HTTP (긴) 폴링을 사용한다.

모든 전송요청은 URL 구조를 따른다:

```
http://host:port/myApp/myEndpoint/{server-id}/{session-id}/{transport}
```

 - {server-id} - 클러스터에서 라우팅 요청에 유용하지만 그렇지 않으면 사용되지 않음.
 - {session-id} - SockJS 세션에 속하는 HTTP 요청의 상관 관계.
 - {transport} - 전송 유형을 나타냄. (e.g. "websocket", "xhr-streaming")

<br/>

웹소켓 전송은 오직 단일 HTTP 요청으로 웹소켓 핸드쉐이크를 실행하는데 필요하다. 모든 메시지 전송은 소켓으로 전환된다.

HTTP 전송은 더 많은 요청이 필요하다. 예를 들어 Ajax/XHR 스트리밍은 서버/클라이언트간 메시지의 장시간 요청과 서버/클라이언트간 메시지에 대한 추가적인 HTTP Post 요청에 의존한다.
긴 폴링은 서버/클라이언트 간 각각의 전송 후 현재 요청이 종료되는 점을 제외하면 비슷한다.

SockJS는 최소한의 메시지 프레임을 추가한다.
예를 들어, 서버에서 첫 문자 o ("open" frame) 만 전송하고, 메시지는 "message1","message2"(JSON 인코딩된 배열), 기본적으로 25초 동안 메시지가 없으면 h ("heartbeat" frame) 문자를 보여주고, 세션이 닫히면 c ("close" frame) 문자를 보여준다.

자세한 내용을 보려면 브라우저에서 HTTP 요청을 하고 확인해야 한다.
SockJS 클라이언트는 전송 목록 수정을 허용하므로, 매 전송을 한 번에 하나씩 볼 수 있다.
SockJS 클라이언트는 브라우저 콘솔의 유용한 메시지 사용할 수 있게 하는 디버깅 구분자를 제공한다.
서버쪽에서 org.springframework.web.socket 의 TRACE 로깅 사용가능하다.
자세한 내용은 SockJS narrated 테스트에서 확인하자.

<br/>

## Enable SockJS

SockJS는 JAVA 설정을 매우 쉽게할 수 있다.

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").withSockJS();
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

<br/>

The above is for use in Spring MVC applications and should be included in the configuration of a DispatcherServlet.
However, Spring’s WebSocket and SockJS support does not depend on Spring MVC. 
It is relatively simple to integrate into other HTTP serving environments with the help of SockJsHttpRequestHandler.

On the browser side, applications can use the sockjs-client (version 1.0.x) that emulates the W3C WebSocket API and communicates with the server to select the best transport option depending on the browser it’s running in. 
Review the sockjs-client page and the list of transport types supported by browser. 
The client also provides several configuration options, for example, to specify which transports to include.

<br/>

## Heartbeats

SockJS 프로토콜은 프록시가 중단되는 것을 막기위해 서버가 Heartbeat 메시지를 전송하는데 요구한다.
스프링 SockJS 설정은 임의의 빈도수로 HeartbeatTime을 호출한 프로퍼티를 가지고 있다.
기본적으로 heartbeat는 연결상태에서 다른 메시지가 없다고 가정했을 때 25초 후에 보낸다.
25초라는 값은 범용인터넷에 대해 [IETF 권고](https://tools.ietf.org/html/rfc6202)를 따른 것이다.

> WebSocket/SockJS에서 STOMP를 사용할 때 STOMP 클라이언트와 서버가 교환할 heartbeat를 협상하는 경우, SockJS heartbeat는 비활성화 된다.

스프링 SockJS 지원을 통해 `TaskScheduler`가 heartbeats 작업을 스케줄링하는데 사용할 수 있도록 한다.
작업 스케줄러는 사용가능한 프로세서의 수를 기반으로 기본 설정을 한 스레드 풀(Thread Pool)에 의해 지원된다.
어플리케이션은 특정 요구에 따라 설졍하는 커스터마이징을 고려해야 한다.

<br/>

## Client disconnects

HTTP 스트리밍과 HTTP 긴 폴링 SockJS 전송은 사용가능한게 하나라도 연결되어 있어야 한다. 자세한 내용은 [블로그 참조](https://spring.io/blog/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/)

서블릿 컨테이너안에서 완료되도록 서블릿3 비동기 지원한다. 요청을 처리하는 서블릿 컨테이너 스레드를 종료할 수 있고, 다른 스레드로부터의 응답에 계속 쓰는 것을 허용한다. 

특정 문제는 서블릿 API가 사라진 클라이언트에 대한 알림을 제공하지 않는다는 것이다.(`SERVLET_SPEC-44` 참조)
하지만 서블릿 컨테이너는 이후에 응답을 쓰려고 시도할 때 예외가 발생한다.
스프링의 SockJS 서비스는 서버 발송 heartbeat를 지원한다.(기본적으로 매 25초)
that means a client disconnect is usually detected within that time period or earlier if messages are sent more frequently.

> 결과로는 네트워크 입출력 실패는 클라이언트의 연결종료로 단순히 발생할 수있으며, 불필요한 스택로그가 쌓일 수 있다. 
> Spring은 (각 서버마다 특정) 클라이언트 연결을 끊는 네트워크 장애를 식별하고 `AbstractSockJsSession`에 정의된 전용 로그 범주 `DISCONNECTED_CLIENT_LOG_CATEGORY`를 사용하여 최소 메시지를 기록하기 위해 최선의 노력을 기울인다.
> 스택 추적을 봐야 한다면 TRACE 로그 카테고리를 참조하자.

<br/>

## SockJS and CORS

만약 크로스 도메인(cross-origin)을 허용한다면 SockJS 프로토콜은 XHR 스트리밍과 폴링 전송에서 크로스도메인 지원에 대해 CORS를 사용한다. 
따라서 반응에 CORS 헤더가 없는 경우 CORS 헤더가 자동으로 추가된다.
어플레케이션이 이미 CORS 지원을 제공하도록 구성된 경우, 서블릿 필터를 통해 스프링 `SockJsService` 부분을 건너뛸 것이다.

스프링의 SockJsService에서 `suppressCors` 속성을 통해 이러한 CORS 헤더의 추가를 비활성화할 수도 있다.

SockJS가 예상하는 헤더와 값 목록:

 - `"Access-Control-Allow-Origin"` - "Origin" 요청 헤더의 최초 설정 값.
 - `"Access-Control-Allow-Credentials"` - 항상 `true`로 설정.
 - `"Access-Control-Request-Headers"` - 동일한 요청 헤더로부터의 초기 값.
 - `"Access-Control-Allow-Methods"` - 전송 시 지원하는 HTTP 방식.
 - `"Access-Control-Max-Age"` - 31536000로 설정 (1년).

<br/>

정확한 구현을 위해 `AbstractSockJsService`의 `addCorsHeaders`와 소스 코드의 `TransportType` enum을 참조하자.

또는 CORS 구성에서 SockJS 끝점 접두어가있는 URL을 제외하도록 고려하면 Spring의 SockJsService가 처리하도록 허용 할 수 있다.

<br/>

## SockJS Client

브라우저를 사용하지 않고 SockJS 엔드포인트를 원격으로 연결하기 위해 SockJS 자바 클라이언트를 제공한다.
이는 양방향 통신간 2개 서버 이상일 때, 네트워크 프록시에서 웹소켓 프로토골의 사용을 방해할 수 있는 경우 매우 유용하다.
SockJS 자바 클라이언트는 많은 수의 동시 사용자를 시뮬레이션 하는 경우 매우 유용하다.  

SockJS 자바 클라이언트는 "websocket", "xhr-streaming", "xhr-polling" 전송을 지원한다. 나머지 것들은 브라우저에서만 사용할 수 있다. 

`WebSocketTransport` 설정할 수 있는 것:

 - JSR-356 런타임에서의 `StandardWebSocketClient`
 - Jetty 9 이상 네이티브 웹소켓 API 사용하는 `JettyWebSocketClient`
 - 스프링의 `WebSocketClient`의 어떠한 구현

<br/>

XhrTransport는 정의상 "xhr-streaming"과 "xhr-polling"을 모두 지원한다. 클라이언트 관점에서는 서버에 연결하는 데 사용되는 URL 외에 다른 차이가 없기 때문이다. 현재 두 가지 구현이 있다.

 - `RestTemplateXhrTransport` uses Spring’s `RestTemplate` for HTTP requests.
 - `JettyXhrTransport` uses Jetty’s `HttpClient` for HTTP requests.

<br/>

어떻게 SockJS 클라이언트와 SockJS 엔드포인트 연결을 생성하는지 아래 예제를 보자.

```java
List<Transport> transports = new ArrayList<>(2);
transports.add(new WebSocketTransport(new StandardWebSocketClient()));
transports.add(new RestTemplateXhrTransport());

SockJsClient sockJsClient = new SockJsClient(transports);
sockJsClient.doHandshake(new MyWebSocketHandler(), "ws://example.com:8080/sockjs");
```

<br/>
 
많은 동시 사용자를 시뮬레이션하기 위한 `SockJsClient`를 사용하려면 충분한 수의 연결과 스레드를 허용하도록 기본 HTTP 클라이언트(XHR 전송용)를 구성해야 한다. 예: Jetty:

```java
HttpClient jettyHttpClient = new HttpClient();
jettyHttpClient.setMaxConnectionsPerDestination(1000);
jettyHttpClient.setExecutor(new QueuedThreadPool(1000));
```

<br/>

서버쪽 SockJS 관련 프로퍼티에 대해 커스터마이징를 고려할 수 있다. (자세한 내용은 Javadoc 참조)

```java
@Configuration
public class WebSocketConfig extends WebSocketMessageBrokerConfigurationSupport {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/sockjs").withSockJS()
            .setStreamBytesLimit(512 * 1024)
            .setHttpMessageCacheSize(1000)
            .setDisconnectDelay(30 * 1000);
    }

    // ...
}
```

<br/>

> 참고<br/>
> https://docs.spring.io/spring-framework/docs/5.0.5.RELEASE/spring-framework-reference/web.html
