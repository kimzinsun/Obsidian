항상 페이지네이션을 만들어서 사용했었는데 Spring에 Pageable 기능이 있길래 사용해보았다

# Pagination

> [!NOTE] 페이지네이션이란?
> 
> 콘텐트를 여러 페이지로 나누어 다음 또는 이전 페이지로 이동하거나 특정 페이지로 이동할 수 있는 요소
> 

# Pageable

> [!NOTE] Pageable이란?
> 
> Spring 에서 제공하는 Pagination을 위한 인터페이스


```java
@GetMapping("")  
public ResponseEntity<?> getProducts(Pageable pageable) {  
    return ResponseEntity.status(200).body(productService.getProducts(pageable));  
}
```

- 호출 시에 `Pageable` 객체를 넘겨주면 스프링에서 매핑해준다
- `?page=1&size=10&sort=name,DESC` 의 패턴으로 API 호출하면 된다

> `page` : 한 페이지에서 확인할 수 있는 데이터의 양
> `page` : 확인할 페이지 번호
> `sort` : 정렬 기준


# 실제 사용

![[스크린샷 2024-08-09 10.17.32.png]]![[스크린샷 2024-08-09 10.18.00.png]]
- 실제로 호출해보면 아래와 같은 페이지 정보들이 나와서 활용할 수 있다


처음에 페이지 1번부터 호출했는데 순서가 이상하길래 뭐지? 했는데 알고보니 페이지도 0번부터 시작하는거였다.. 