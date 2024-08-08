---
date : 24-08-07   
---

# QueryDsl 이란?

> 정적 타입을 사용해서 SQL과 같은 쿼리를 생성할 수 있도록 해주는 프레임워크

# 사용하는 이유?

상품 검색을 하려고 한다. 이름, 설명, 가격, 수량이 있다고 생각했을 때, 조건 검색을 위해 이름, 설명, 가격, 수량을 검색할 수도 있지만, 이름+설명, 설명+가격, 가격+수량 등등.. 의 조건이 늘어나게 되면 그만큼 if문을 사용해야 하고 그만큼 쿼리가 복잡해진다.

이때 QueryDsl을 사용하면 메서드를 활용해 쉽게 동적 검색에 사용할 수 있다.
~~(다른 곳에도 사용한다고 하지만 여기선 알아보지 말자)~~


# QueryDSL 설정

```
ext {  
    set('springCloudVersion', "2023.0.2")  
    set('querydslVersion', "5.0.0")  // QueryDSL 버전 명시적으로 설정  
}

dependencies {
	implementation "com.querydsl:querydsl-jpa:${querydslVersion}:jakarta"  
	annotationProcessor "com.querydsl:querydsl-apt:${querydslVersion}:jakarta"  
	annotationProcessor "jakarta.annotation:jakarta.annotation-api"  
	annotationProcessor "jakarta.persistence:jakarta.persistence-api"

	...
}

def querydslSrcDir = 'src/main/generated'  
clean {  
    delete file(querydslSrcDir)  
}  
tasks.named('test') { useJUnitPlatform() }

```

build.gradle에 denpendency를 추가하고 QueryDsl 버전과 저장될 위치를 지정해준다

```java
QueryResults<Product> results = queryFactory
    .selectFrom(product)
    .where(
        nameContains(searchDto.getName()),
        descriptionContains(searchDto.getDescription()),
        priceBetween(searchDto.getMinPrice(), searchDto.getMaxPrice()),
        quantityBetween(searchDto.getMinQuantity(), searchDto.getMaxQuantity())
    )
    .orderBy(orders.toArray(new OrderSpecifier[0]))
    .offset(pageable.getOffset())
    .limit(pageable.getPageSize())
    .fetchResults();
```


```java
private BooleanExpression nameContains(String name) {
    return name != null ? product.name.containsIgnoreCase(name) : null;
}
```

QueryDSL의 BooleanExpression을 사용하여 동적 where을 구성하고 Pageable 객체로부터 정렬 및 페이징 정보를 적용해 쿼리를 실행한다.

사실 아직도 크게 감이 잘 안온다.... 실습하면서 익숙해져야겠다