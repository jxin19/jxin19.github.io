---
title: Web Socket
description: Future
header: Asynchronous - Future
tags: [Java, Spring, WebSocket, Socket]
keywords: 
    - en: Full-duplex
      ko: 전이중
      cn: 全双工
    - en: Two-way
      ko: 양방향
      cn: 双向
    - en: Interaction
      ko: 상호작용
      cn: 相互作用
    - en: Send
      ko: 송신
      cn: 发送
    - en: Receive
      ko: 수신
      cn: 接收
    - en: Real time
      ko: 실시간
      cn: 实时
    - en: Proxy
      ko: 프록시,대리
      cn: 代理
    - en: Mechanism
      ko: 매커니즘
      cn: 机理
---

## 개요

웹소켓 프로토콜(RFC 6455)는 표준화된 방식으로 서버와 클라이언트 간의 단일 TCP 연결을 통해 전이중(full-duplex), 양방향 통신 채널을 지원한다.
HTTP의 다른 TCP 프로토콜과 다르다. 하지만, HTTP를 통해 동작하도록 설계되었고, 포트 80/443을 사용하여 기존 방화벽 규칙을 재사용할 수 있도록 설계되었다.

HTTP 호출로 웹소켓 상호작용이 시작된다. HTTP Upgrades 헤더를 사용하여 업그레이드하거나 WebSocket 프로토콜로 교체한다. 

```bash
GET /spring-websocket-portfolio/portfolio HTTP/1.1
Host: localhost:8080
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: Uc9l9TMkWGbHFD2qnFHltg==
Sec-WebSocket-Protocol: v10.stomp, v11.stomp
Sec-WebSocket-Version: 13
Origin: http://localhost:8080
```

상태코드 200 대신 웹소켓 프로토콜을 반환한다.

```bash
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 1qVdfYHU9hPOl4JYYNXF623Gzn0=
Sec-WebSocket-Protocol: v10.stomp
```

성공적인 핸드쉐이크가 끝나면 클라이언트와 서버의 지속적인 송수신을 위해 HTTP 업그레이드 요청을 처리하는 TCP 소켓은 열린상태로 유지된다.

이 문서의 범위를 뛰어넘어 웹소켓이 어떻게 동작하는지 설명이 끝났다. RFC 6455(HTML5의 웹소켓 챕터)나 튜토리얼, 많은 자료들을 읽어보길 바란다.

참고로 웹소켓 서버가는 웹서버(e.g. nginx) 뒤에서 동작한다. 웹소켓 서버에 웹소켓 업그레이드 요청을 전달하도록 구성해야 할 수도 있다.
마찬가지로 어플리케이션이 클라우드 환경에서 실행되면 웹소켓 지원과 관련된 내용을 확인해야 한다.

#### HTTP vs WebSocket

웹소켓은 HTTP과 호환되도록 설계되었고 HTTP요청으로 시작할수 있지만, 두 프로토콜(HTTP, 웹소켓)이 다른 아키텍쳐와 어플리케이션 프로그래밍 모델이라는 것을 이해하는 것이 중요하다.

HTTP와 REST에서 많은 어플리케이션은 많은 URL로 모델링된다. 어플리케이션 클라이언트와 상호작용을 하려면 요청과 응답 형태의 URL 접근을 해야한다. 서버는 HTTP URL, 메서드 및 헤더를 기반으로 적절한 처리기로 요청을 라우팅합니다.

웹소켓과 대조되는 것은 일반 URL로 최초 연결하고 이후 모든 어플리케이션 메시지 흐름이 동일한 TCP 연결로 처리된다. 이 점은 전체적으로 다른 비동기, 이벤트 기반, 메시징 아키텍쳐이다.

웹소켓은 마치 낮은 단계 전송 프로토콜로 HTTP와는 달리 메시지 내용에 대한 어떠한 의미도 규정하지 않는다.
이 말은 클라이언트와 서버가 메시지 의미론에 동의하지 않으면 메시지를 라우팅하거나 처리할 수 없다. 

HTTP 핸드쉐이크(handshake) 요청의 'Sec-WebSocket-Protocol' 헤더를 통해 웹소켓 클라이언트와 서버는 높은 단계의 사용, 메시징 프로토콜(e.g. STOMP)를 사용을 협상할 수 있다.

#### 언제 사용하는가?

웹소켓은 동적이고 상호작용적인 웹 페이지를 만들 수 있다. 하지만 Ajax와 HTTP 스트리밍(streaming) 또는 긴 폴링(long polling) 등의 복합적인 사례는 간단하고 효과적인 방법을 제공한다.

예를 들어 뉴스, 메일, 소셜미디어 등에서 동적 업데이트가 필요하다. 하지만 매 분마다 그렇게 하는 것이 좋다. 게임과 금융 앱 등은 실시간과 밀접하다.

대기시간만으로는 결정적인 요인이 아니다. 만약 메시지양이 적은 경우(e.g. 네트워크 오류 모니터링) HTTP 스트리밍 또는 폴링이 효과적인 방법이 될 수 있다.
짧은 대기시간, 매우 빈번하고 많은 양의 전송일 경우, 웹소켓을 사용하는 것이 가장 좋다.


## WebSocket API

스프링 프레임워크는 웹소켓 메시지를 처리하는 클라이언트와 서버 쪽(side) 어플리케이션을 작성하는데 사용할 수 있는 웹소켓API를 제공한다.

#### WebSocketHandler

웹소켓 서버 생성은 `WebSocketHandler`를 구현하거나 `TextWebSocketHandler` 또는 `BinaryWebSocketHandler`를 확장하는 것처럼 매우 간단하다.

```java
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.TextMessage;

public class MyHandler extends TextWebSocketHandler {

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) {
        // ...
    }

}
```

```java
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

위 내용은 Spring MVC 어플리케이션과 DispatcherServlet의 설정이 포함되어야 한다. 하지만 스프링의 웹소켓은 Spring MVC를 의존하지 않는다.
이는 비교적 간단히 WebSocketHandler를 사용하여 WebSocketHandler를 다른 HTTP 제공 환경에 통합하는 것이다.

#### WebSocket Handshake

최초 HTTP 웹소켓 핸드쉐이크(handshake) 요청을 가장 쉽게 사용자 정의하는 것은 핸드쉐이크 메소드의 "before"와 "after"를 노출하는 HandshakeInterceptor를 사용하는 것이다. 
이런 인터셉터를 사용하여 핸드쉐이크를 막거나 WebSocketSession 모든 속성을 사용가능하도록 하는 것이다.
예를 들어 이것은 HTTP 속성이 웹소켓 세션을 통과하는 빌트인(built-in) 인터셉터이다.

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new MyHandler(), "/myHandler")
            .addInterceptors(new HttpSessionHandshakeInterceptor());
    }

}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
        <websocket:handshake-interceptors>
            <bean class="org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor"/>
        </websocket:handshake-interceptors> 
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

추가적인 옵션은 `DefaultHandshakeHandler`를 확장하는 것이다. 웹소켓 핸드쉐이크의 단계적 성능, 클라이언트 원점(origin) 유효성 포함, 서브 프로토콜 협상 등.
아직 지원되지 않는 웹소켓 서버엔진 및 버전에 맞추기 위해 임의의 `RequestUpgradeStrategy`를 구성해야 하는 경우 옵션을 사용해야 할 수도 있다.(Deployment 참조)
Java-config, XML 네임스페이스 모두 사용자정의 `HandshakeHandler`를 구성할 수 있다.

> 스프링은 `WebSocketHandlerDecorator` 기반 클래스 제공한다. 추가적인 행동을 포함한 `WebSocketHandler`을 꾸밀 수 있다.
> 로그와 예외처리는 웹소켓 Java-config나 XML 네임스페이스를 기본적으로 사용할 때 제공하고 추가한다.
> `ExceptionWebSocketHandlerDecorator`는 모든 `WebSocketHandler` 메소드에서 발생하는 확인되지 않는 예외를 포착하고서버 오류를 나타내는 상태 1011로 웹소켓 세션을 닫는다.


#### Deployment

스프링 웹소켓 API은 Spring MVC 어플리케이션에 쉽게 통합 할 수 있다. DispatcherServlet은 HTTP 웹소켓 핸드쉐이크와 다른 HTTP 요청을 제공한다.
`WebSocketHttpRequestHandler`를 호출하여 다른 HTTP 처리 프로세스 시나리오를 쉽게 통할 할 수 있다. 이해하기도 쉽다.
단, JSR-356 런타임에 대해서는 특별한 고려사항이 적용된다.

자바 웹소켓 API(JSR-356)는 두가지 매커니즘이 있다.
첫번째로 시작할 때 서블릿 컨테이너 클래스 경로 스캔(Classpath scan) (서블릿3 기능)을 포함하고, 다른 하나는 서블릿 컨테이너 초기화 시 사용할 등록 API를 포함한다.
첫 번째는 시작할 때 Servlet 컨테이너 클래스 경로 스캔(Servlet 3 기능)을 포함하고, 다른 하나는 Servlet 컨테이너 초기화 시 사용할 등록 API를 포함한다.
이러한 메커니즘 중 어느 것도 웹소켓 핸드쉐이크와 스프링 MVC의 `DispatcherServlet`과 같은 모든 HTTP 처리 싱글 "Front Controller"를 사용할 수 있도록 하지 않는다.

스프링의 웹소켓 지원은  JSR-356 런타임에서 실행될 때 서버별 `RequestUpgradeStrategy`를 처리한다.
Tomcat, Jetty, GlassFish, WebLogic, WebSphere, and Undertow (and WildFly)에 이러한 전략이 있다.

> 자바 웹소켓 API의 초과요청은 WEBSOCKET_SPEC-211을 따라 새로 작성될 수 있다.
> Tomcat, Undertow, WebSphere는 가능한 대체 API를 제공한다. 이는 Jetty와 함께 사용할 수 있다.

두번째 고려사항은 JSR-356을 지원하는 서블릿 컨테이너가 어플리케이션이 시작을 급격히 느려지게 하는 `ServletContainerInitializer` (SCI) 스캔의 실행을 기대한다. 
포괄적인 영향으로 JSR-356 지원하는 서블릿 컨테이너 버전을 업그레이드 한 후 관찰되는 것이다.
web.xml의 요소중 <b><absolute-ordering /></b> 의 사용을 통한 Web Fragments (그리고 SCI 스캐닝) 사용여부 선택할 수 있다. 

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering/>

</web-app>
```

Web Fragments 이름 설정을 선택적으로 할 수 있다. 스프링의 `SpringServletContainerInitializer`는 서블릿3 자바 초기화 API을 지원한다.

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering>
        <name>spring_web</name>
    </absolute-ordering>

</web-app>
```

#### Server Config

근본적으로 웹소켓 엔진은 설정 프로퍼티를 노출시킨다. 컨트롤 런타임 특징은 대략 메시지 버퍼 크기, idle timeout 크기 등이 있다.
Tomcat, WildFly, GlassFish은 `ServletServerContainerFactoryBean`을 웹소켓 구성에 추가한다.

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
        return container;
    }

}
```

> 클라이언트의 웹소켓 설정을 위해 반드시 `WebSocketContainerFactoryBean` 나 `ContainerProvider.getWebSocketContainer()`을 사용해야 한다.

Jetty를 위해 Jetty WebSocketServerFactory 사전구성을 지원하고 웹소켓 자바 설정을 통해 스프링의 `DefaultHandshakeHandler`을 연결한다.

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(echoWebSocketHandler(), "/echo").setHandshakeHandler(handshakeHandler());
    }

    @Bean
    public DefaultHandshakeHandler handshakeHandler() {

        WebSocketPolicy policy = new WebSocketPolicy(WebSocketBehavior.SERVER);
        policy.setInputBufferSize(8192);
        policy.setIdleTimeout(600000);

        return new DefaultHandshakeHandler(new JettyRequestUpgradeStrategy(new WebSocketServerFactory(policy)));
    }

}
```

#### Allowed Origins

웹소켓을 위한 기본 동작은 스프링 프레임워크 4.1.5 이며, 오직 동일한 origin 요청만 허가한다. 이 것은 정확한 origins 목록이나 전체를 설정할 수 있다.
이 확인은 브라우저 클라이언트에서 대부분 고안된 설계이다. 브라우저에서 Origin 헤더 값을 다른 값으로 수정하는 것을 제한하진 않는다(see RFC 6454: The Web Origin Concept for more details).

세가지 가능한 동작

 - Allow only same origin requests (default): in this mode, when SockJS is enabled, the Iframe HTTP response header X-Frame-Options is set to SAMEORIGIN, and JSONP transport is disabled since it does not allow to check the origin of a request. As a consequence, IE6 and IE7 are not supported when this mode is enabled.
   
 - Allow a specified list of origins: each provided allowed origin must start with http:// or https://. In this mode, when SockJS is enabled, both IFrame and JSONP based transports are disabled. As a consequence, IE6 through IE9 are not supported when this mode is enabled.
   
 - Allow all origins: to enable this mode, you should provide * as the allowed origin value. In this mode, all transports are available.
   
웹소켓과 SockJS는 아래와 같이 origins를 설정할 수 있도록 한다.

```java
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").setAllowedOrigins("http://mydomain.com");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```