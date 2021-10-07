- 하루마다 조금씩 인프런 스프링 강의 정리

- JPA 강의

- 메이븐이나 그레들 중 하나는 공부할 것  

- 읽고 싶은 책 : 이펙티브 자바, 오브젝트, 디자인 패턴, 리펙토링, 토비의 스프링, 클린코드, 자바와 JUnit을 활용한 실용주의 단위 테스트

- 스프링 시큐리티 - 정수원 : https://inf.run/KtZy
- 스프링 마이크로 서비스 - https://inf.run/LN5e

https://docs.spring.io/spring-data/jpa/docs/2.5.5/reference/html/#jpa.query-methods

--------------------------------------------------------------

```java
@PersistenceContext
private EntityManager em;
```
스프링 부트가 어노테이션을 보고 EntityManager를 주입해준다.

@Transactional이 테스트에 붙으면 실행 후 DB 롤백함
이 때 `Rollback(false)`를 붙여주면 롤백되지 않음

address같은 값 타입(임베디드 타입)은 수정되면 안되기 때문에 Getter만 열어두고 생성자로 값을 세팅할 수 있게 하는 것이 좋다. 이 때 기본 생성자는 필수이므로 기본 생성자도 따로 만들어주어야 하고, 사용자가 기본 생성자를 사용하는 것을 막기 위해 접근 지정자를 `protected`로 할 수 있다. (private는 안됨)
 
```java
@PersistenceUnit
private EntityManagerFactory emf;
```
`@PersistenceUnit`은 EntityManagerFactory를 주입해준다. 쓸 일은 잘 없음


shift + f6 : 메소드명이나 변수명 일괄 변경

클래스에 `@Transactional`붙이면 public 메소드마다 트랜잭션에 걸려들어감
이 때 조회만하는 메소드에는 따로 `@Transcational(readOnly=true)`를 해주면 JPA가 성능최적화함

반대로 클래스에 `@Transcational(readOnly=true)`를 붙이고 변경가능성이 있는 메소드에만 `@Transcational`을 붙여줘도 됨

지금 `validateDuplicateMember` 메소드는 동시성에 문제를 가지기때문에 실무에서는 name에 대해 유니크 제약조건도 추가하는 것이 안전함

```java
@PersistenceContext
private EntityManager em;
```

스프링 부트를 사용할 경우, `@PersistenceContext` 대신에 `@Autowired`를 사용해도 `EntityManager`를 주입해준다. 따라서 `final`로 선언하고 `@RequiredArgsConstructor`와 함께 Repository에서도 편하게 의존성을 주입받을 수 있다.


테스트 메소드에서 @Transcational이 붙어있으면 아예 쿼리조차 보내지 않기 때문에
테스트에서 쿼리를 확인하고 싶으면 메소드에 @Rollback(false) 설정을 추가하거나,
@Autowired EntityManager em;로 EntityManager를 주입받은 뒤, em.flush()하면 쿼리를 확인할 수 있다.
대신 이렇게 하면 DB에 데이터가 남기때문에 주의가 필요하다.

테스트 시에 DB 띄우지 않고 in-memory로 테스트 하는 법
test 밑에 resources 폴더 만들고 yml파일 복사해서 넣고 url을 url: jdbc:h2:mem:test로 변경한다. 

심지어 여기에 넣는 application.yml 파일은 설정이 없어도 기본이 in-memory로 동작하기 때문에 안에 logging 부분 빼고 다 지워도 된다. 

해당 멤버변수를 가진 클래스에서 해당 로직을 실행하는 것이 객체지향적이기 때문에 Item 클래스 내부에 비즈니스 로직인 addStock과 removeStock을 추가한다.

cascade는 단일 주인을 가질 때만? 사용하자. 예를 들어, OrderItem과 Delivery는 각각 Order랑만 연관되기 때문에 cascade할 수 있는 것이다. 

OrderItem의 생성을 static 메소드로 정의해놓았는데, 다른 사람은 기본 생성자로 OrderItem 인스턴스를 만든 뒤 Setter를 통해 값을 넣으려고 할 수 있다. 이렇게 되면 유지보수가 굉장히 어려워지기 때문에 생성자를 막는 것이 좋다. (실무에서는 setter도 열어두지 않는 것이 좋음) 이 때, JPA에서는 기본 생성자가 필요한데, 접근지정자를 protected까지 허용해주므로 롬복을 활용해서 @NoArgsConstructor(access = AccessLevel.PROTECTED)를 추가해주도록 하자.


cmd+option+m : 메소드로 분리하기
cmd+option+p : 메소드 내에서 특정 변수 파라미터로 보내기


API를 만들 때는 절대 Entity를 반환하면 안된다.
그렇지 않으면, Entity에 멤버변수가 추가될 때마다 API 스펙이 바뀌기 떄문이고 숨기고 싶은 변수(password 등)가 들어나게 되기 때문


api와 template engine에서는 서로 패키지를 분리하는 것을 권장함. 보통 예외처리의 경우 패키지 단위로 설정하는 것이 많은데, html 페이지의 예외상황은 예외 페이지로 이동하는 경우가 많지만, api에서 예외가 발생하면 예와 JSON을 보내주어야 하기 때문에 서로 간의 예외 처리 방식이 다르다.

...(@RequestBody Member member)
하면 request body로 넘어온 JSON 데이터를 Member로 변환해줌

v1에서는 엔티티와 API 스펙이 1:1로 매핑되어 있어서 엔티티 내의 변수명을 바꾸던지 하면 모든 API가 깨져버림. API는 여러 곳에서 호출하기 떄문에 이것은 큰 문제를 발생시킬 수 있음
따라서 API 스펙을 위한 DTO를 만들어야 함
이렇게 DTO를 만들면, 이 API에서 넣어줘야 하는 스펙이 뭔지 한눈에 보이고 필요한 API에 validation을 넣을 수 있어서 안정적이고 편리함. 

절대 엔티티를 외부에 노출하거나, 파라미터로 직접 받지 말도록 하자. 


update는 변경감지를 사용하는 것이 좋다.

List로 반환하면 유연성 또한 너무 낮기때문에 객체로 한번 감싸주고 리턴하는 것이 좋음



ordersV1은 Order가 Member를 불러오고, Member가 Order를 불러오면서 무한루프에 빠짐
이렇게 되면 양방향으로 묶이는 한 쪽에 `@JsonIgnore`를 걸어줘야 함
그런데 이렇게 해도 실행해보면 오류가 발생한다. 
```
Type definition error: .... 
    org.phibernate.proxt.pojo.bytebuddy.ByteBuddyInterceptor...
```
이것은 Jackson 라이브러리가 데이터를 Json으로 변환하는데 프록시 객체를 만나서 이것을 처리할 수 없다고 알리는 오류이다. 

이 때 Jackson 라이브러리가 프록시 객체를 무시하도록 할 수 있는데 추가적인 라이브러리를 사용해야 한다.

```
implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'
```

v3버전과 v4버전은 장단이 있어서 뭐 하나가 더 좋다고 할 순 없음
v3는 엔티티 속성 모두 가져와서 다른 API에서도 사용할 수 있는 여지가 커서 재사용성이 높지만 v4는 필요한 속성만 가져오기때문에 재사용성이 낮은 대신 좀 더 성능이 최적화된다
> 그리고 v4는 코드가 지저분함. 그리고 대부분의 경우에서는 성능에 큰 차이가 있진 않음

그리고 리포지토리에 API의 스펙대로 쿼리를 짜는것은 리포지토리에서 화면에 의존하도록 한다. => API 스펙이 바뀌면 리포지토리를 수정해야함 => 글쎄?

그래서 리포지토리 밑에 패키지를 하나 더 빼서 쿼리 서비스나 쿼리 리포지토리로 따로 뽑는 것을 권장함

v2처럼 DTO의 멤버변수로 Entity가 있다하더라도 허용하면 안된다. 

v3에서 Order의 데이터는 2개이고, OrderItem의 데이터가 4개였지만, Order와 OrderItem을 fetch join해버려서 Order의 데이터가 4개로 뻥튀기 되어 결과가 중복으로 나온다.


v3와 v3.1 중에서 조회할 데이터의 양이 많이 않은 경우, 중복 데이터가 발생하더라도 한방 쿼리로 패치조인을 사용한 v3처럼 조회하는게 좋을 수 있고, 100개 1000개씩 많은 데이터를 퍼올려야할 경우에는 v3.1처럼 여러 쿼리가 나가더라도 중복없이 가져오는 것이 좋을 수도 있다. 상황이나 데이터양에 따라 달라진다. 다만 페이징을 해야하는 경우는 v3.1의 방식을 써야 한다. 

@Query에 사용하는 JPQL은 오타가 발생하면 마치 named query처럼 애플리케이션 실행 시점에 예외를 발생시킴. 동적쿼리는 그냥 querydsl 사용을 추천

--------------

1. 양방향 연관관계를 피해라
2. 다중성이 적은 방향을 선택하라  
    ```java
    class A{
        private List<B> bList;
    }
    class B{

    }
    ```
    보다는 
    ```java
    class A{

    }
    class B{
        private A a;
    }
    ```
3. 의존성이 필요없다면 재거하라
4. 패키지 사이의 의존성 사이클을 제거하라
