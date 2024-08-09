---
tags:
  - API게이트웨이
  - ServerWebExchange
date: 2024-08-09
---
> [!Question]
> 
> 모든 API 의 Response Header 에 Server-Port Key 로 현재 실행중인 서버의 포트를 추가해주세요.

# 헤더에 서버 포트 추가하기

처음에는 [[06 API 게이트웨이|API게이트웨이]] 할 때 했던 방법대로 각 서비스 컨트롤러에 `@Value`로 `server.port`를 가져와 Response Header에 담아 보냈다. 

하지만 모든 메소드에 추가를 하려고 하니 `이게 맞나...` 라는 생각이 들었다. 

그러다 문득 어차피 모든 서비스들은 게이트웨이를 통과하는데 게이트웨이 필터에서 해주면 되겠다고 생각했고 `ServerWebExchange`의 Request URL에서 port번호를 추출해 해결할 수 있었다.

```java
private final static String HEADER_SERVER_PORT = "Server-Port";  
  
@Override  
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {  
    return chain.filter(exchange).then(Mono.fromRunnable(() -> addServerPortHeader(exchange)));  
}  
  
private void addServerPortHeader(ServerWebExchange exchange) {  
    URI uri = exchange.getAttribute(ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR);  
    exchange.getResponse().getHeaders().add(HEADER_SERVER_PORT, uri.getPort() + "");  
}
```

![[스크린샷 2024-08-09 19.30.50.png]]

문제는 해결했지만 이 exchange에 대해 궁금해졌다.

# ServerWebExchange

> [!NOTE] ServerWebExchange
> 
> Spring WebFlux에서 제공하는 인터페이스로, **HTTP 요청과 응답의 전체 생애 주기를 관리하는 객체**

# 주요 역할

1. 요청 및 응답 관리
	- 클라이언트로부터 들어온 HTTP 요청과 서버가 클라이언트로 반환할 HTTP 응답을 모두 관리함
	- 이 객체를 통해 요청과 응답의 헤더와 본문을 읽거나 수정할 수 있음
2. 속성 관리
	- 요청 처리 중 필요한 데이터를 속성으로 저장하고 꺼내 사용할 수 있는 기능 제공
3. 비동기 처리
	- 비동기방식으로 요청과 응답 처리
	- Mono와 Flux같은 리액티브 타입을 사용해 비동기 작업 수행
4. 웹 필터 체인 관리
	- 웹 필터 체인 실행 관리


# ServerWebExchangeUtils

> [!NOTE] ServerWebExchangeUtils
> 
> ServerWebExchange와 관련된 다양한 작업을 도와주는 정적 메서드를 모아놓은 클래스
> 

# 주요 상수와 메서드

- `GATEWAY_ROUTE_ATTR`: 현재 요청에 대해 매칭된 라우트(Route) 객체를 나타내는 속성
- `GATEWAY_REQUEST_URL_ATTR`: 요청이 라우팅될 최종 URL을 나타내는 속성
- `GATEWAY_SCHEME_PREFIX_ATTR`: 요청이 라우팅될 때 사용된 스킴(scheme) 접두사를 나타내는 속성
- `PRESERVE_HOST_HEADER_ATTRIBUTE`: 요청 시 호스트 헤더를 유지하기 위한 속성

