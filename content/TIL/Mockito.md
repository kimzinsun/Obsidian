---
tags:
  - 스프링
  - 테스트
  - spring
  - test
  - JUnit5
date: 24-08-01
---

지난 [[단위 테스트]] 개념에 이어서 Mockito를 연습해보자

productService에 있는 `update` 코드를 테스트해보자

```java
@Transactional
public ProductResponseDto updateProduct(Long id, ProductMypriceRequestDto requestDto) {
    int myprice = requestDto.getMyprice();
    if (myprice < MIN_MY_PRICE) {
        throw new IllegalArgumentException("유효하지 않은 관심 가격입니다. 최소 " + MIN_MY_PRICE + "원 이상으로 설정해 주세요.");
    }
    Product product = productRepository.findById(id)
            .orElseThrow(() -> new NullPointerException("해당 상품을 찾을 수 없습니다."));

    product.update(requestDto);

    return new ProductResponseDto(product);
}
```

여기서 들 수 있는 의문

테스트 코드에 생성자 주입을 넣을 수 있나? repository에 있는 `findById`는 어떻게 동작하지?
라는 의문이 들 수 있다

하지만 우리가 테스트 하고 싶은건 `findById`가 아니라 **서비스에 있는 `update` 코드 자체가 잘 처리되는지이다**
repository가 아니라 service만 테스트 하기 위해 `Mock Object`를 사용할 수 있다

그리고 그 Mock Object를 제공해주는 라이브러리가 Mockito 이다

# Mockito

```java
@ExtendWith(MockitoExtension.class) // @Mock 사용을 위해 설정합니다.
class ProductServiceTest {

    @Mock
    ProductRepository productRepository;

    @Mock
    FolderRepository folderRepository;

    @Mock
    ProductFolderRepository productFolderRepository;
    ...

```

Mockito를 사용하기 위해 따로 dependency를 추가할 필요는 없다

`@ExtendWith(MockitoExtension.class)` 애노테이션을 달아서 설정해주고 `Mock` 오브젝트로 사용할 가짜 객체(클래스나 인터페이스)를 @Mock 애노테이션을 달아주면 Mockito가 가짜객체를 만들어준다

```java
@Test
@DisplayName("관심 상품 희망가 - 최저가 이상으로 변경")
void test1() {
    // given
    Long productId = 100L;
    int myprice = ProductService.MIN_MY_PRICE + 3_000_000;

    ProductMypriceRequestDto requestMyPriceDto = new ProductMypriceRequestDto();
    requestMyPriceDto.setMyprice(myprice);

    ProductService productService = new ProductService(productRepository, productFolderRepository, folderRepository);

    // when
    ProductResponseDto result = productService.updateProduct(productId, requestMyPriceDto);

    // then
    assertEquals(myprice, result.getMyprice());
}
```

- given
  - productId와 myprice를 임의로 넣어준다
  - setMyprice를 이용해 requestMyPriceDto를 생성해준다
  - 그리고 productService를 하나 만들어준다
- when
  - 실제로 productService를 실행해 result값을 받는다
- then
  - assertEquals를 이용해 내가 넣은 myprice와 결과물의 myprice가 같은지 비교해 Update가 잘 수행되고 있는지 확인한다

하지만 이대로 실행하면 오류난다
가짜 객체를 만들어 전달해주기만 하는게 아니라 사용 케이스를 제대로 정의해서 넘겨줘야하기 때문이다

> 다시 말하지만 우리는 update가 제대로 되는가? 가 중요한 포인트이다

사용 케이스를 추가한 코드를 다시 보자

```java
@Test
@DisplayName("관심 상품 희망가 - 최저가 이상으로 변경")
void test1() {
    // given
    Long productId = 100L;
    int myprice = ProductService.MIN_MY_PRICE + 3_000_000;

    ProductMypriceRequestDto requestMyPriceDto = new ProductMypriceRequestDto();
    requestMyPriceDto.setMyprice(myprice);

    User user = new User();
    ProductRequestDto requestProductDto = new ProductRequestDto(
            "Apple <b>맥북</b> <b>프로</b> 16형 2021년 <b>M1</b> Max 10코어 실버 (MK1H3KH/A) ",
            "https://shopping-phinf.pstatic.net/main_2941337/29413376619.20220705152340.jpg",
            "https://search.shopping.naver.com/gate.nhn?id=29413376619",
            3515000
    );

    Product product = new Product(requestProductDto, user);

    ProductService productService = new ProductService(productRepository,folderRepository, productFolderRepository);

    given(productRepository.findById(productId)).willReturn(Optional.of(product));

    // when
    ProductResponseDto result = productService.updateProduct(productId, requestMyPriceDto);

    // then
    assertEquals(myprice, result.getMyprice());
}
```

product를 직접 만들어주는 부분이 추가되었다 (User, ProductRequestDto)
그리고 Mockito에 given이라는 method를 이용해 넣어줄 수 있다

![[스크린샷 2024-08-02 01.07.01.png]]

(편-안)
