---
title: "[JPA] @OneToOne 연관관계 매핑 시 주의사항"
author: "yeoji21"
date: 2022-07-17
categories: [Backend, ORM, JPA]
tags: [java, jpa, spring]
---

이번 포스트에서는 JPA의 `@OneToOne` 매핑을 사용하면서 마주한 문제에 대해 얘기해보겠습니다.

결론부터 말씀드리자면 JPA에서 OneToOne 관계로 엔티티를 연결할 경우, `Lazy Loading`이 보장되지 않을 수 있습니다.  

어떤 경우에 이런 문제가 발생하고, 어떻게 해결하면 좋을까요?  
문제 상황을 간단히 하면 다음과 같습니다. 

<img src="/assets/img/spring/shop_erd.png" width=600>

하나의 가게는 하나의 가게 위치를 갖습니다.  
이때 가게의 위치는 가게를 지도에 표시하기 위한 위도, 경도 정보를 비롯한 다양한 정보들이 추가될 수 있기 때문에 가게 위치 테이블로 별도로 분리하고 가게와 1:1 연관 관계를 맺고 있습니다.

여기서 두 엔티티간의 비즈니스적 관계를 보면 가게가 가게 위치를 가지는 주인의 역할을 하기 때문에 가게를 **주인 엔티티**, 가게 위치는 주인 엔티티에 속하는 **대상 엔티티**라고 부르겠습니다.
> 외래키를 가지는 JPA의 연관 관계의 주인과 무관하게 비즈니스적으로 두 엔티티를 바라본 관계입니다.

<br>

이런 관계에서 저는 대상 엔티티(가게 위치)를 연관 관계의 주인으로 설정했습니다.

```java
@Entity
public class Location{
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "location_id")
    private Long id;

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "shop_id")
    private Shop shop;

    @Column(name = "street_address")
    private String streetAddress;
}
```

대부분 저처럼 대상 엔티티가 주인 엔티티를 외래키로 가지는 형태로 구상하셨을거라 생각합니다.

이렇게 구성해야 주인 엔티티(가게)가 저장되고 주인 엔티티의 PK를 FK로 가지는 대상 엔티티(가게 주소)가 저장될 수 있기에 저장이든 조회든 우리의 생각대로 주인 엔티티와 대상 엔티티의 관계가 구현되기 때문입니다.

여기까지 좋습니다.  

하지만 이렇게 단방향으로 매핑하게되면 대상 엔티티만 주인 엔티티의 참조를 가지게 되는데,  
대부분의 비즈니스 로직에서 가게를 조회하고 가게에서 가게 주소로 접근하는 경우가 많지만 가게는 가게 주소의 참조를 가지고 있지 않기 때문에 접근할 수 있는 방법이 없죠

따라서 주인 엔티티에서 대상 엔티티의 참조를 가지는 **@OneToOne 양방향 매핑**을 설정합니다.

```java
@Entity
public class Shop {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;

    @Column(name = "shop_name")
    private String shopName;

    @Column(name = "phone_number")
    private String phoneNumber;

    @Column(name = "introduction")
    private String introduction;

    @OneToOne(
            fetch = FetchType.LAZY,
            cascade = CascadeType.ALL,
            mappedBy = "shop"
    )
    private Location location;
}
```

이제 가게에서도 가게 주소로 접근할 수 있게 되었습니다!  
@OneToOne 관계에서 잊지않고 Lazy Loading까지 설정해주었기 때문에 야무지게 두 엔티티의 관계를 매핑했다고 생각했는데..?

문제는 여기서부터 시작되었습니다 😨😨

@OneToOne 양방향 매핑을 설정하게 되면 연관 관계의 주인이 아닌 쪽(가게 엔티티)에서는 Lazy Loading을 적용할 수 없게됩니다.
> 위 예시처럼 명시적으로 fetch = FetchType.LAZY 설정을 추가하더라도 즉시 로딩으로 실행됩니다.

<br>

왜 이런 현상이 발생하는 것일까요?

JPA는 객체의 참조가 **프록시 기반**으로 동작합니다.   

즉, 지연 로딩을 설정하면 해당 객체의 참조는 비어있는 것이 아니고 프록시 객체를 가지고 있습니다.  

그런데 이때, 조회한 가게에 대한 가게 위치의 결과가 없다면 어떻게 해야할까요? 이 경우에는 당연히 프록시가 아닌 null이 반환되어야 합니다.

때문에 해당 객체가 null인지 아닌지 체크한 뒤에 null이 아니라면 프록시 객체를 넣어야겠죠  
> null 체크를 하지 않고 미리 프록시 객체를 넣어둔다면 해당 객체가 존재하지 않더라도 null이 아니라 프록시 객체를 반환해버리기 때문

하지만 지금처럼 대상 엔티티가 연관 관계의 주인인 양방향 매핑 방식에서는 주인 엔티티는 대상 엔티티의 데이터가 존재하는지 스스로 판단이 불가능합니다.
 
따라서 이 경우에는 대상 엔티티를 강제로 즉시 로딩해서 데이터가 있으면 해당 데이터를 넣고, 없다면 null을 넣는 방식으로 동작하기 때문에 지연 로딩으로 설정해도 즉시 로딩으로 값을 가져오는 것입니다.

반대로 연관 관계의 주인 입장에서는 외래키를 관리하고 있기때문에 해당 객체가 존재하는지 스스로 판단할 수 있어 Lazy Loading이 가능합니다.

그런데 한 가지 의문이 생깁니다. 

@ManyToOne - @OneToMany 양방향 매핑을 한 경우에는 연관 관계의 주인이 아닌 쪽에서도 Lazy Loading을 잘만써왔는데 이 경우는 왜 허용되는 것일까요?

이 경우에는 단일 객체가 아닌 **컬렉션**을 가지기 때문입니다.  
컬렉션 자체적으로 데이터가 없는 Empty를 표현할 수 있기 때문에 컬렉션의 null 체크를 하지 않아도 됩니다. 따라서 컬렉션은 항상 지연 로딩으로 동작할 수 있는 것이죠

아주 치밀하게 설계가 되어있구나..라는 생각이 듭니다 🤔


<br>

다시 예시로 돌아가서 마저 얘기해보겠습니다.

Lazy Loading을 적용할 수 없다는 것은 꽤나 치명적입니다. 만약 가게 위치를 포함하지 않는 기본적인 가게 정보만 제공하는 API를 개발한다고 했을 때도 필요없는 가게 위치 엔티티까지 조회하게 되죠

이때 심지어 즉시 로딩으로 인한 N+1 문제가 발생하게 됩니다. 10개의 가게를 조회한다면 11개의 쿼리가 나가고, 100개의 가게를 조회한다면 101개의 쿼리가 나가게 됩니다. 

가게를 조회할 때 항상 fetch join을 통해 가게 위치까지 가져온다면 N+1 문제는 피할 수 있겠지만 가게 위치 데이터를 사용하지 않을 때도 함께 조회해야하는 자원 낭비는 막을 수 없습니다.

그렇다면 아쉽지만 주인 엔티티에서 대상 엔티티로의 참조를 제거하고 단방향 매핑으로만 사용해볼까요?

위 예시에서 많은 비즈니스 로직에서 가게와 가게 위치를 함께 사용하게 되는데, 이때마다 가게가 아닌 가게 위치 엔티티를 조회한 뒤, 객체 참조를 통해 가게 엔티티로 접근해야 하기에 객체지향의 관점에서 생각해보면 달갑지 않은 방식입니다. 
> 주인 인티티와 대상 엔티티가 함께 사용되는 경우가 많지 않고, 대상 엔티티는 홀로 사용되는 경우가 대부분이라면 고려해볼 수 있을 것 같습니다.

<br>

그렇다면 어떻게 매핑하는 것이 좋을까요?

먼저, 정말 1:1 관계가 필요한지 생각해봐야 합니다. 만약 엔티티 간의 관계가 1:N으로 변경될 가능성이 있다면 애초에 1:N으로 설계하는 것도 좋은 방법입니다. 

만약 명확한 1:1 관계라고 생각된다면 주인 엔티티에서 대상 엔티티로 접근하는 비즈니스 로직이 많은지 생각해보고, 대상 엔티티가 주로 단독으로 사용된다면 양방향이 아닌 단방향으로 연결해야 합니다.

하지만 위 예시처럼 명확한 1:1 관계이면서 주인 엔티티에서 대상 엔티티로 접근하는 비즈니스 로직이 많다면 주인 엔티티가 연관 관계의 주인이 되는 일대일 단방향 매핑으로 설계하는 것이 좋다고 생각합니다. 

```java
@Entity
public class Shop {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;

    @Column(name = "shop_name")
    private String shopName;

    @Column(name = "phone_number")
    private String phoneNumber;

    @Column(name = "introduction")
    private String introduction;

    @OneToOne(
            fetch = FetchType.LAZY,
            cascade = CascadeType.ALL
    )
    @JoinColumn(name = "location_id")
    private Location location;
}
```
이렇게 연결하면 주인 테이블에서 대상 테이블의 PK를 FK로 가지기 때문에 문제없이 지연 로딩을 적용할 수 있으면서 주인 엔티티가 대상 엔티티로의 참조를 가지기 때문에 비즈니스 로직 개발 시에도 용이합니다.

하지만 만약 엔티티 간의 관계가 1:N으로 변경된다면 적용하기가 훨씬 어려워지겠죠. 때문에 이렇게 연결을 하기 전에는 명확한 1:1 매핑이 맞는지 다시 한번 생각해봐야겠습니다.


### 참조

- [JPA 성능 N+1 문제와 해결 방법 Moon`s Development Blog](https://gmoon92.github.io/spring/jpa/hibernate/n+1/2021/01/12/jpa-n-plus-one.html)  
- [JPA 도입 — OneToOne 관계에서의 LazyLoading 이슈 #1 by Yorath Jang Medium](https://yongkyu-jang.medium.com/jpa-%EB%8F%84%EC%9E%85-onetoone-%EA%B4%80%EA%B3%84%EC%97%90%EC%84%9C%EC%9D%98-lazyloading-%EC%9D%B4%EC%8A%88-1-6d19edf5f4d3)  
- [java:jpa:one-to-one 권남](https://kwonnam.pe.kr/wiki/java/jpa/one-to-one)  
- [연관관계 질문이 있습니다 - 인프런 질문 & 답변](https://www.inflearn.com/questions/224187)  
- [JPA Fetch 전략 공부하기 - @OneToOne 양방향 매핑에서 Lazy가 동작하지 않는 이유](https://loosie.tistory.com/788)