---
date: 2024-07-31
---

# Entity 연관 관계

> [!Note]
> 고객이 음식을 주문 시, 주문 정보는 어느 테이블에 들어가야 할까?

1. 고객 테이블 - 한 명의 고객은 음식을 여러개 주문할 수 있음 -> 음식 컬럼 중복
2. 음식 테이블 - 하나의 음식은 여러명의 고객에게 주문될 수 있음 -> 고객 컬럼 중복

-> 주문 테이블 생성으로 해결

> 고객 : 음식 = N : M 관계

- N:M 관계 테이블의 연관 관계 해결을 위해 주문 테이블과 같은 중간 테이블을 두고 관리한다
- DB 테이블에서는 테이블 사이의 연관 관계를 FK로 맺을 수 있고 방향 상관없이 조회 가능하다

그러나 **Entity는** **객체 형태**이므로 상대 Entity의 타입을 필드로 가지고 있어야 조회 할 수 있다 -> **방향 개념 존재**

# 1대 1 관계 `@OneToOne`

## 단방향 관계

외래키의 주인 설정

- 외래키의 주인만이 외래 키를 등록, 수정, 삭제 할 수 있으며, 주인이 아닌 쪽은 외래키 조회만 가능

![[스크린샷 2024-07-31 12.18.39.png]]

- 음식 Entity가 외래 키의 주인인 경우

```java
@Entity
@Table(name = "food")
public class Food {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;

    @OneToOne
    @JoinColumn(name = "user_id") // 외래키의 주인이 활용하는 애너테이션
    private User user;
}
```

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

## 양방향 관계

- 양방향일 경우 외래키의 주인이 누구인지 알려주어야 함
- 외래키의 주인이 **아닌** 쪽에서 mappedBy 옵션 사용해 외래키의 주인 지정

```java
@Entity
@Table(name = "food")
public class Food {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;

    @OneToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToOne(mappedBy = "user")
    private Food food;
}
```

> **mappedBy의 속성값은 외래키의 주인인 상대 Entity의 필드명을 의미한다**

```
@OneToOne
    @JoinColumn(name = "user_id")
    private User user; -> 이 부분..
```

> [!Warning]
>
> 1. 외래 키의 주인인 Entity에서 @JoinColumn을 사용하지 않아도 default 옵션이 적용되어 생략 가능
>
>    - 다만, 1:N 관계에서 생략한다면 외래 키를 저장할 컬럼을 파악할 수가 없어 중간 테이블이 생기기 때문에 그냥 애너테이션 쓰는게 좋음
>
> 2. 양방향 관계에서 mappedBy 옵션을 생략할 경우에도 파악 불가능, 반드시 설정하자 ~

# N 대 1 관계 `@ManyToOne`

## 단방향 관계

![[스크린샷 2024-07-31 12.35.59.png]]

- 음식 Entity가 N의 관계로 외래 키의 주인

```java
@Entity
@Table(name = "food")
public class Food {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

## 양방향 관계

![[스크린샷 2024-07-31 12.39.18.png]]

- 고객 Entit에서 Java 컬렉션을 사용하여 음식 Entity 참조
- 그렇다고 DB에 user_id 1,2 이렇게 저장되는건 아님

```java
@Entity
@Table(name = "food")
public class Food {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "user") // 여기도 마찬가지로 @JoinColumn으로 외래키를 가지는 필드 이름
    private List<Food> foodList = new ArrayList<>();
}
```

# 1 대 N 관계 `@OneToMany`

## 단방향 관계

![[스크린샷 2024-07-31 12.46.39.png]]

- 외래키의 주인은 음식이지만 실제 외래키는 고객 entity가 가지고 있음

```java
@Entity
@Table(name = "food")
public class Food {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;

    @OneToMany
    @JoinColumn(name = "food_id") // users 테이블에 food_id 컬럼
    private List<User> userList = new ArrayList<>();
}
```

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

- 외래키의 주인은 음식 entity지만 실제 DB에서 외래키는 고객 테이블이 갖고 있음
  -> INSERT시 UPDATE가 추가적으로 발생되는 문제가 생김

## 양방향 관계

1대 N 관계에서는 일반적으로 양방향 관계가 없음

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

		@ManyToOne
		@JoinColumn(name = "food_id", insertable = false, updatable = false)
		private Food food;
}
```

- insertable, updatable을 false로 설정하면 양방향처럼 설정할 수는 있음

# N 대 M 관계 `@ManyToMany`

## 단방향 관계

![[스크린샷 2024-07-31 14.17.05.png]]

- N : M 관계를 풀어내기 위해 중간 테이블 생성하여 사용

```java
@Entity
@Table(name = "food")
public class Food {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;

    @ManyToMany
    @JoinTable(name = "orders", // 중간 테이블 생성
    joinColumns = @JoinColumn(name = "food_id"), // 현재 위치인 Food Entity 에서 중간 테이블로 조인할 컬럼 설정
    inverseJoinColumns = @JoinColumn(name = "user_id")) // 반대 위치인 User Entity 에서 중간 테이블로 조인할 컬럼 설정
    private List<User> userList = new ArrayList<>();
}
```

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

- JPA에 의해 만들어진 테이블은 컨트롤이 어려워 추후에 테이블 변경이 있을때 문제가 발생할 수 있음

## 양방향 관계

![[스크린샷 2024-07-31 14.19.35.png]]

```java
@Entity
@Table(name = "food")
public class Food {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;

    @ManyToMany
    @JoinTable(name = "orders", // 중간 테이블 생성
    joinColumns = @JoinColumn(name = "food_id"), // 현재 위치인 Food Entity 에서 중간 테이블로 조인할 컬럼 설정
    inverseJoinColumns = @JoinColumn(name = "user_id")) // 반대 위치인 User Entity 에서 중간 테이블로 조인할 컬럼 설정
    private List<User> userList = new ArrayList<>();
}
```

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToMany(mappedBy = "userList")
    private List<Food> foodList = new ArrayList<>();
}
```

## 중간 테이블

![[스크린샷 2024-07-31 14.20.32.png]]

- 중간 테이블을 직접 생성하여 관리하면 변경 발생시 컨트롤이 쉬워짐

```java
@Entity
@Table(name = "food")
public class Food {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;

    @OneToMany(mappedBy = "food")
    private List<Order> orderList = new ArrayList<>();
}
```

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "user")
    private List<Order> orderList = new ArrayList<>();
}
```

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "food_id")
    private Food food;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```
