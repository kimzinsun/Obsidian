---
tags:
  - 스프링
  - AOP
date: 24-08-04
---
# AOP란?

Aspect Oriented Programming의 약자로 **관점 지향 프로그래밍**이다

로직을 **핵심 기능**과 **부가 기능**의 관점으로 나누어서 보고 각각을 모듈화 하는것이 AOP의 핵심이다


쇼핑몰로 예를 들면 상품을 검색하고, 장바구니에 담고, 결제하는 부분은 핵심적인 비즈니스 로직이라고 볼 수 있고, 유저별로 홈페이지에 체류하는 시간을 측정하는 부분은 부가 기능에 해당한다.

AOP는 부가기능을 Aspect로 정의하여 핵심 기능에서 부가기능을 분리한다.

![[스크린샷 2024-08-02 17.43.22.png]]

# AOP 주요 개념

- Join point : Aspect가 적용될 수 있는 시점
	- @Around : 핵심기능 수행 전과 후에 어드바이스 기능 수행
	- @Before : 핵심기능 호출 전에 수행
	- @After : 핵심기능 수행 성공/실패 여부와 상관없이 언제나 동작
	- @AfterReturning : 핵심기능 호출 성공 시 수행
	- @AfterThrowing : 핵심기능 호출 실패 시 수행

- Advice : Aspect의 기능을 정의한 것
- Point cut : Advice를 적용할 메소드 범위 지정

```java
@Slf4j(topic = "UseTimeAop")  
@Aspect  
@Component  
@RequiredArgsConstructor  
public class UserTimeAop {  
    private final ApiUseTimeRepository apiUseTimeRepository;  
  
    @Pointcut("execution(* com.sparta.myselectshop.controller.ProductController.*(..))")  
    private void product() {  
    }  
  
    @Pointcut("execution(* com.sparta.myselectshop.controller.FolderController.*(..))")  
    private void folder() {  
    }  
  
    @Pointcut("execution(* com.sparta.myselectshop.naver.controller.NaverApiController.*(..))")  
    private void naver() {  
    }  
  
    @Around("product() || folder() || naver()")  // 메소드 실행 전후에 적용할 내용  
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {  
        // 측정 시작 시간  
        long startTime = System.currentTimeMillis();  
        try {  
            // 핵심기능 실행  
            Object output = joinPoint.proceed();  
            return output;  
        } finally {  
            // 측정 종료 시간  
            long endTime = System.currentTimeMillis();  
            long runTime = endTime - startTime;  
  
            // 사용자 정보 가져오기  
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();  
            if(auth!=null && auth.getPrincipal().getClass() == UserDetailsImpl.class) {  
                UserDetailsImpl userDetails = (UserDetailsImpl) auth.getPrincipal();  
                User loginUser = userDetails.getUser();  
  
                ApiUseTime apiUseTime = apiUseTimeRepository.findByUser(loginUser).orElse(null);  
  
                if(apiUseTime == null) {  
                    apiUseTime = new ApiUseTime(loginUser, runTime);  
                } else {  
                    apiUseTime.addUseTime(runTime);  
                }  
  
                apiUseTimeRepository.save(apiUseTime);  
            }  
  
        }  
    }  
  
}
```


# Spring에서의 AOP

스프링 AOP는 프록시 기반의 AOP 구현체이다.

스프링이 프록시 객체를 중간에 삽입하여 동작하는 형태이다.

![[스크린샷 2024-08-05 00.57.39.png]]

