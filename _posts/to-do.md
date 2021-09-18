
- 인프런 스프링 강의 정리

- JPA 강의

- 메이븐이나 그레들 중 하나는 공부할 것  

- 읽고 싶은 책 : 이펙티브 자바, 오브젝트, 디자인 패턴, 리펙토링, 토비의 스프링, 클린코드
  
--------------------------------------------------------------

기본타입은 공유되지 않기때문에 사이드 이펙트가 발생하지 않음
Integer같은 레퍼 클래스의 객체들은 공유가 되지만(int b = a), 값을 변경할 수 없기때문에 (Integer setInteger()같은건 없음) 사이드 이펙트가 발생하지 않는다. 

임베디드 타입
새로운 값 타입을 직접 정의할 수 있다. 주로 기본 값 타입을 모아서 만듬 (ex. 좌표)

@Embeddable : 값 타입을 정의하는 곳에 표시
@Embedded : 값 타입을 사용하는 곳에 표시

둘 중에 하나만 명시해도 컴파일에 문제는 없으나 명확성을 위해 둘 다 명시할 것을 추천함

임베디드 타입 전후의 테이블은 똑같지만 객체를 임베디드 타입으로 묶는 것. 코드에서 좀 더 객체지향적으로 설계 가능 + 인스턴스 변수를 활용한 메소드 생성 가능 + 재사용 가능

만약 한 엔티티에서 같은 임베디드 타입을 사용하고 싶다면 `@AttributeOverrides`를 사용하면 된다.
```java
@Entity
public class Member{
    ....
    @Embeded
    private Address homeAddress;

    @Embeded
    @AttributeOverrides({
        @AttributeOverrid(name="city", column=@Column(name = "work_city")),
        @AttributeOverrid(name="street", column=@Column(name = "work_street")),
        @AttributeOverrid(name="zipcode", column=@Column(name = "work_zipcode"))
    })
    private Address workAddress;
}
```

임베디드 타입의 사이드 이펙트
```java
Address address = new Address("city", "street", "10000");
Member member = new Member();
member.setUsername("member1");
member.setHomeAddress(address);
em.persist(member);

Member member2 = new Member();
member2.setUsername("member2");
member2.setHomeAddress(address);
em.persist(member2);

member.getHomeAddress().setCity("newCity");
```

이렇게 되면, UPDATE 쿼리가 두 방 나가고, member1과 member2 모두 city가 newCity로 변경된다. 
이건 명백한 사이드 이펙트(부작용)이고, 이런 오류는 발견하기 굉장히 어렵다. 만약 이런 동작을 의도했다하더라도 이런 방식을 원하면 임베디드 타입이 아니라 엔티티로 공유했어야 한다. 

-> 값 타입은 불변 객체로 만들어야 한다. 생성자로만 값을 설정하고 Setter를 만들지 않으면 된다. 
따라서 값타입을 변경하고싶으면 setter를 쓰면 절대 안되고 새로운 인스턴스를 생성해서 갈아끼워야 함

인텔리제이 equals()&hashCode() 자동생성 지린다.
근데 이 때 getter사용 옵션을 켜야 프록시일때도 접근이 가능하니 주의

값타입 컬렉션
컬렉션을 DB로 표현할 수 없기때문에 따로 테이블로 빼야 함 (개념적으로 보면 일대다 매핑)
값타입 컬렉션도 결국 값타입이다, 값타입은 스스로의 생명주기를 가지지않고 엔티티에 종속적이기 때문에 컬렉션 내의 데이터들은 다른 테이블에 저장됨에도 불구하고 엔티티만 persist해도 컬렉션 내의 데이터들은 자동으로 테이블에 저장된다. 

컬렉션들은 기본이 지연로딩임

값타입 컬렉션 내의 값을 변경하고 싶을 때도 setter를 사용해서는 안되고, remove()를 통해서 해당 값을 지우고 add()를 통해서 새로운 값을 넣어야 한다. 

```java
findMember.getAddressHistory().remove(new Address("old1","street","10000"));
findMember.getAddressHistory().add(new Address("newCity1","street","10000"));
```
기본적으로 컬렉션들은 remove 시에 equals()를 통해 해당 데이터를 찾기 떄문에, 값타입의 equals를 항상 구현해두어야 함

이 때 쿼리를 까보면, ADDRESS 테이블에서 해당 member_id를 가진 데이터를 모두 delete해버리고 새로 갈아끼움(데이터의 개수만큼 insert 쿼리가 꽂힘) 그래서 실무에서 쓰기엔 힘듦

```java
@Entity
@Getter @Setter
@Table(name="ADDRESS")
public class AddressEntity{
    @Id @GeneratedValue
    private Long id;

    private Address address;
}
```
이렇게 엔티티로 한 번 감싸주고

```java
@ElementCollection
@CollectionTable(name="address", joinColumns = @JoinColumn(name="member_id"))
private List<Address> addressHistory = new ArrayList<>();
```
이랬었던 것을

```java
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
@JoinColumn(name="member_id")
private List<AddressEntity> addressHistory = new ArrayList<>();
```
이렇게 일대다 연관관계 매핑으로 바꿔야함 

값타입 컬렉션은 셀렉트 박스처럼 매우 간단한 테이블일때만 사용해야 함

```java
List<Member> result = em.createQuery("select m from Member m where m.username like '%kim%'", Member.class).getResultList();
```
위의 JPQL 쿼리에서 Member는 MEMBER 테이블이 아닌, Member 엔티티를 의미한다.

Criteria
```java
CriteriaBuilder cb = emgetCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);
Root<Member> m = query.from(Member.class);
CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> resultList = em.creatQuery(cq).getResultList();
```
가독성이 떨어져서 실무에서 잘 쓰이지는 않는 스펙

```java
List<Member> result = em.createNativeQuery("select MEMBER_ID,USERNAME from MEMBER", Member.class).getResultList();
```

파라미터 바인딩 - 이름기준
```java
Member member = em.createQuery("select m from Member m where m.username=:username", Member.class)
.setParameter("username","member1").getSingleResult();
```
순서기준은 순서가 바뀔 수 있으므로 사용하지 말것

엔티티 프로젝션하면 List로 반환된 것들이 모두 영속성 컨텍스트 내에서 엔티티로 관리된다.
```java
List<Member> result = em.createQuery("select m from Member m",Member.class).getResultList();
Member findMember = result.get(0);
findMember.setAge(20);
```
이 상황에서 findMember에 대한 UPDATE 쿼리가 나가서 영속성 컨텍스트에 의해 관리된다는 것을 알 수 있음


```java
em.createQuery("select m.team from Member m",Team.class)
em.createQuery("select t from Member m join m.team t", Team.class)
```
위와 아래의 쿼리는 내부적으로 데이터베이스에 같은 쿼리를 보낸다. (MEMBER와 TEAM을 join해서 가져옴) 하지만 실제 쿼리와 더 비슷한 아래의 쿼리로 작성하는 것이 가독성 면에서 더 좋다.

임베디드 타입은 해당 테이블에 속해있기때문에 따로 join하지 않고 불러옴

스칼라 프로젝션
```java
List<Object[]> resultList = em.createQuery("select m.username, m.age from Member m").getResultList();
Object[] result = resultList.get(0);
System.out.println("username = " + result[0]);
System.out.println("age = " + result[1]);
```

new키워드로 가져오기
```java
package jpql;

@AllArgsConstructor
@Getter @Setter
public class MemberDTO{
    private String username;
    private int age;
}
````
```java
List<MemberDTO> result = em.createQuery("select new jpql.MemberDTO(m.username, m.name) from Member m", MemberDTO.class).getResultList();
MemberDTO memberDTO = result.get(0);
System.out.println("username = " + memberDTO.getUsername());
System.out.println("age = " + memberDTO.getAge());
```
패키지를 다 적어줘야 하는 단점이 있음


페이징
```java
List<Member> result = em.createQuery("select m from Member m order by m.age desc", Member.class)
.setFirstResult(0).setMaxResult(10).getResultList();
```

내부조인 외부조인 공부할것
내부조인 : 둘 다 공통적으로 있는 거 조인
와부조인 : 공통되는 컬럼중에 없는 애는 null로
세타조인 : 카테시안 곱으로 다 합침

`on`을 통해 join 시 조건을 줄 수 있음

메인쿼리와 서브쿼리는 상관없음. m과 m2. m을 긁어오면 성능 별로임

```java
String query = "select m.username, 'HELLO', true from Member m " + "where m.type=:userType";
List<Object[]> result = em.createQuery(query).setParameter("userType",MemberType.ADMIN).getResultList();
```

```java
List<Item> result = em.creaeteQuery("select i from Item i where type(i) = Book", Item.class).getResultList();
```

JPQL 기본함수
```java
String query = "selct size(t.members) from Team t";
```

사용자 정의 함수 등록
```java
public class MyH2Dialect extends H2Dialect{
    public MyH2Dialect(){
        registerFunction("group_concat",new StandardSQLFunction("group_concat",StandardBasicTypes.STRING));
    }
}
```
외우지말고 클래스까보고 따라하기 추천
이렇게 바꾼 Dialect를 persistence.xml에 등록해줘야 동작함

```java
String query1 = "select function('group_concat', m.username) from Member m";
String query1 = "select group_concat(m.username) from Member m";
```
둘이 같은 쿼린데 아래가 더 직관적


JPQL에서 단일 값 연관 경로를 사용하면 내부적으로 inner join을 사용하는 쿼리가 나간다. => 조인같은게 묵시적으로 나가면 좋지않음. 직관적이지도 않고 운영/튜닝이 너무 힘들어짐

컬렉션 값 연관 경로도 묵시적인 내부 조인이 발생하는데, 탐색은 안됨. 탐색은 안된다는 말은 즉, 해당 컬렉션의 멤버변수에 접근할 수 없다는 것을 의미함. 대신 `.size`로 컬렉션의 크기를 구할 수는 있음
=> 명시적 join을 통해 탐색가능

하지만 가장 좋은 방법은 묵시적 조인을 사용하지않고 명시적 조인을 사용하는 것임

패치 조인
: 즉시로딩으로 가져올 때와 비슷한데, 쿼리로 원하는 객체 그래프를 조회할지 동적으로 정할 수 있다는 장점

```java
Team teamA = new Team();
teamA.setName("A팀");
em.persist(teamA);

Team teamB = new Team();
teamB.setName("B팀");
em.persist(teamB);

Member member1 = new Member();
member1.setUsername("회원1");
member1.setTeam(teamA);
em.persist(member1);

Member member2 = new Member();
member2.setUsername("회원2");
member2.setTeam(teamA);
em.persist(member2);

Member member3 = new Member();
member3.setUsername("회원3");
member3.setTeam(teamB);
em.persist(member3);

em.flush();
em.clear();

String query = "select m from Member m";
List<Member> result = em.createQuery(query, Member.class).getResultList();
result.forEach(m -> System.out.println("member = "+ m.getUsername()+" : "+m.getTeam().getName()));

transaction.commit();
```

```console
Hibernate: 
    /* insert ch10.Team
        */ insert 
        into
            Team
            (name, team_id) 
        values
            (?, ?)
Hibernate: 
    /* insert ch10.Team
        */ insert 
        into
            Team
            (name, team_id) 
        values
            (?, ?)
Hibernate: 
    /* insert ch10.Member
        */ insert 
        into
            Member
            (age, team_id, username, id) 
        values
            (?, ?, ?, ?)
Hibernate: 
    /* insert ch10.Member
        */ insert 
        into
            Member
            (age, team_id, username, id) 
        values
            (?, ?, ?, ?)
Hibernate: 
    /* insert ch10.Member
        */ insert 
        into
            Member
            (age, team_id, username, id) 
        values
            (?, ?, ?, ?)
Hibernate: 
    /* select
        m 
    from
        Member m */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.team_id as team_id4_0_,
            member0_.username as username3_0_ 
        from
            Member member0_
Hibernate: 
    select
        team0_.team_id as team_id1_1_0_,
        team0_.name as name2_1_0_ 
    from
        Team team0_ 
    where
        team0_.team_id=?
member = 회원1 : A팀
member = 회원2 : A팀
Hibernate: 
    select
        team0_.team_id as team_id1_1_0_,
        team0_.name as name2_1_0_ 
    from
        Team team0_ 
    where
        team0_.team_id=?
member = 회원3 : B팀
```

`select m from Member m`로 JPQL을 작성하니, Team은 프록시 값이 들어가있어서 사용할 때마다 따로 쿼리가 나가는 것을 확인할 수 있다. 만약 회원이 100명있다면 쿼리가 100번이 나간다. 이런 현상을 N+1 문제라고 한다. 이것을 해결하는 방법이 패치 조인이다. 

`select m from Member m join fetch m.team` 사용
```java
Hibernate: 
    /* select
        m 
    from
        Member m 
    join
        fetch m.team */ select
            member0_.id as id1_0_0_,
            team1_.team_id as team_id1_1_1_,
            member0_.age as age2_0_0_,
            member0_.team_id as team_id4_0_0_,
            member0_.username as username3_0_0_,
            team1_.name as name2_1_1_ 
        from
            Member member0_ 
        inner join
            Team team1_ 
                on member0_.team_id=team1_.team_id
member = 회원1 : A팀
member = 회원2 : A팀
member = 회원3 : B팀
```

만약 Member에서 team을 @ManyToOne으로 매핑할 때 LAZY 옵션을 주지 않고 `select m from Member m`로 쿼리를 날리면 어떻게 될까?
```java
Hibernate: 
    /* select
        m 
    from
        Member m */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.team_id as team_id4_0_,
            member0_.username as username3_0_ 
        from
            Member member0_
Hibernate: 
    select
        team0_.team_id as team_id1_1_0_,
        team0_.name as name2_1_0_ 
    from
        Team team0_ 
    where
        team0_.team_id=?
Hibernate: 
    select
        team0_.team_id as team_id1_1_0_,
        team0_.name as name2_1_0_ 
    from
        Team team0_ 
    where
        team0_.team_id=?
member = 회원1 : A팀
member = 회원2 : A팀
member = 회원3 : B팀
```
select 쿼리가 N+1번 발생하는 것을 알 수 있다. 따라서 즉시 로딩이든 지연 로딩이든 상관없이 발생하는 N+1 문제를 fetch join이 해결해준다. 


컬렉션의 패치 조인 내용 중복
일대다 매핑에서 조인하면 데이터가 뻥튀기됨 현재 Team 테이블에는 2개의 데이터가 있지만 조인하면서 3개로 합쳐짐 -> 중복 


```java
String query = "select t from Team t join fetch t.members";
List<Team> result = em.createQuery(query, Team.class).getResultList();
result.forEach(t -> {
    System.out.print("team = "+ t.getName() +" : ");
    t.getMembers().forEach(m -> System.out.print(m.getUsername()+" "));
    System.out.println();
});
```
```console
team = A팀 : 회원1 회원2 
team = A팀 : 회원1 회원2 
team = B팀 : 회원3 
```

SQL의 DISTINCT만으로는 우리가 없애려는 중복을 모두 없앨 수 없어서 JPQL의 DISTINCT를 사용해야 함

`select distinct t from Team t join fetch t.members` 사용
```console
team = A팀 : 회원1 회원2 
team = B팀 : 회원3 
```

컬렉션에 패치 조인 사용 시 페이징 API를 사용하면 데이터가 짤려서 틀린 결과가 나올 수 있어서 사용하면 안됨 
-> 하이버네이트의 경우, 컴파일러가 경고 로그를 남기고 메모리에서 페이징하는데 이 때 테이블의 데이터를 모두 메모리로 가져와서 페이징하기 때문에 문제가 생길 여지가 큼
따라서 이런 경우, 다대일로 쿼리 방향을 뒤집어서 페이징 API를 사용할 수는 있는데 이러면 N+1 문제가 발생

```java
@Entity
public class Team{
    ...
    @BatchSize(size=100)
    @OneToMany(mappedBy="team")
    private List<Member> members = new ArrayList<>();
}
```
이렇게 하면 팀 테이블과 연관된 멤버를 최대 100개까지 한 번에 가져온다고 하는데 너무 깊은 내용일 수도
아무튼 BatchSize를 쓰면 쿼리를 N+1번이 아니라 테이블 수만큼만 가져갈 수 있다.

```java
<property name="hibernate.default_batch_fetch_size" value="100"/>
```
BatchSize는 persistence.xml에서 위 처럼 전역설정을 할 수도 있다. (값은 1,000 이하로 줄 것)

패치 조인은 객체 그래프를 유지할 때 사용하면 효과적이다 -> m.team처럼 멤버변수?를 찾아들어갈 때 효과적

Named 쿼리에서
애플리케이션 로딩 시점에 초기화 후 재사용 -> 애플리케이션이 시작되어 로딩될 때 Named 쿼리를 파싱해서 가지고 있다가 사용될 때마다 재사용하기 때문에 쿼릴르 변환할 때 필요한 코스트를 줄일 수 있다는 장점을 가지고, 애플리케이션 로딩 시점에 쿼리를 검사할 수 있다는 장점을 가진다. 

```java
@Entity
@NamedQuery(
    name = "Member.findByUsername",
    query = "select m from Member m where m.username =:username"
)
public class Member{
    ...
}
```
이 때 만약 query에 쿼리를 적는데 오타가 나거나 잘못적은게 있다면 컴파일 시점에 예외가 발생해서 잘못된 쿼리임을 검사할 수 있음 => 엄청난 장점


```java
List<Member> result = em.createNamedQuery("Member.findByUsername", Member.class).setParameter("username","회원1").getResultList();
```

이후에 Spring Data JPA를 사용하면, `@Query` 어노테이션을 사용해서 메소드 위에 Named Query를 만들 수 있어서 편리하게 사용 가능


벌크연산은 흔히 알고있는 UPDATE나 DELETE문이라고 보면 되는데, JPA의 변경 감지만으로만 실행하려면 너무 많은 SQL문이 실행되어야 하므로 벌크연산을 사용한다. 
```java
int resultCount = em.createQuery("update Member m set m.age = 20", Member.class).executeUpdate();
```

벌크연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리하기 때문에 
벌크 연산을 먼저 실행하거나 벌크연산 후 영속성 컨텍스트를 초기화해주어야만 데이터가 꼬이지 않음

JPQL 실행 시 자동으로 flush()는 되지만 영속성 컨텍스트가 clear()로 영속성 컨텍스트를 비워줘야 한다. 필수
Spring Data JPA에서는 `@Modifying`을 통해 벌크연산을 실행하면서 옵션을 통해 clear()까지 자동화할 수 있다. 