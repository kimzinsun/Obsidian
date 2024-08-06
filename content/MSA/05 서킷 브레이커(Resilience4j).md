---
tags:
  - MSA
  - 서킷브레이커
  - Resilience4j
  - SpringCloud
---
# 서킷 브레이커란?

- [[01 MSA 개요|마이크로서비스]]간의 호출 실패를 감지하고 시스템의 전체적인 안정성을 유지하는 패턴
- 상태변화 : closed -> open -> half-open

# Resilience4j

- 서킷 브레이커 라이브러리
- 서킷 브레이커 상태
	- closed : 기본 상태, 모든 요청 통과
	- open : 실패율 초과시 오픈 상태로 전환, 모든 요청 실패 처리
	- half-open : 오픈상태에서 일정 대기 시간 후 전환, 제한된 수의 요청을 허용하며 시스템이 복구되었는지 확인하고 closed 상태로 전환
- Fallback : 호출 실패시 대체 로직
- 모니터링 : 서킷 브레이커 상태를 모니터링하고 관리할 수 있음

> Resilience4j dependency는 `io.github.resilience4j:resilience4j-spring-boot3:2.2.0` 사용하기 !

# Fallback 매커니즘
- 외부 서비스 호출 실패시 대체 로직을 제공하는 메서드
- 시스템의 안전성을 높이고, 장애가 발생해도 사용자에게 일정한 응답을 제공할 수 있음

