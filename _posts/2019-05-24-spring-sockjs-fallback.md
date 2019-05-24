---
title: SockJS Fallback -  Web Socket
description: Future
header: Asynchronous - Future
tags: [Java, Spring, WebSocket, Socket]
keywords: 
    - en: Full-duplex
      ko: 전이중
      cn: 全双工
---

Over the public Internet, restrictive proxies outside your control may preclude WebSocket interactions either because they are not configured to pass on the Upgrade header or because they close long lived connections that appear idle.

The solution to this problem is WebSocket emulation, i.e. attempting to use WebSocket first and then falling back on HTTP-based techniques that emulate a WebSocket interaction and expose the same application-level API.

On the Servlet stack the Spring Framework provides both server (and also client) support for the SockJS protocol.


## Overview

## Enable SockJS

## IE 8,9

## Heartbeats

## Client disconnects

## SockJS and CORS

## SockJS Client

> 참고<br/>
> https://docs.spring.io/spring-framework/docs/5.0.5.RELEASE/spring-framework-reference/web.html
