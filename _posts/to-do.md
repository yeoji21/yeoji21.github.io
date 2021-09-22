- 하루마다 조금씩 인프런 스프링 강의 정리

- JPA 강의

- 메이븐이나 그레들 중 하나는 공부할 것  

- 읽고 싶은 책 : 이펙티브 자바, 오브젝트, 디자인 패턴, 리펙토링, 토비의 스프링, 클린코드, 자바와 JUnit을 활용한 실용주의 단위 테스트

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


