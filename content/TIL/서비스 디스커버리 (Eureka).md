---
date: 2024-08-05
---

## 서비스 디스커버리란?

- MSA에서 각 서비스의 위치를 동적으로 관리하고 찾아주는 기능
- 등록 서버에 자신의 위치를 등록하고, 다른 서비스는 이를 조회하여 통신
- 주요기능 - 서비스 등록, 서비스 조회 ,헬스체크 등

## Eureka란?

- MSA에서 각 서비스의 위치를 동적으로 관리
- 클라이언트 애플리케이션은 Eureka 서버에서 필요한 서비스의 위치를 조회함

## Eureka 사용하기

![[스크린샷 2024-08-05 19.03.12.png]]

1. client와 server 프로젝트 생성

```java
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client' // client
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server' // server
```

2. client와 server쪽에 각각의 디펜던시 추가

```
#server 설정

server.port=19090

eureka.client.register-with-eureka=false # 다른 eureka 서버에 서버 등록 여부(false)
eureka.client.fetch-registry=false # 다른 eureka 서버의 레지스트리 사용 여부(false)
eureka.instance.hostname=localhost
eureka.client.service-url.defaultZone=http://localhost:19090/eureka
```

```
#client 설정

spring.application.name=second

server.port=19092
eureka.client.service-url.defaultZone=http://localhost:19090/eureka
```

3. server와 client의 application.properties 수정

```java
@SpringBootApplication
@EnableEurekaServer
public class ServerApplication {

    public static void main(String[] args) {
       SpringApplication.run(ServerApplication.class, args);
    }

}
```

4. 서버쪽에 `@EnableEurekaServer` 애너테이션

![[스크린샷 2024-08-06 09.28.33.png]] 5. 서버와 클라이언트 동시에 실행시키고 서버 포트 localhost:19090으로 들어가보면 first와 second client가 동시에 잡히는걸 확인할 수 있다!
