---
title: "[JPA] Spring Data JPA"
author: "yeoji21"
date: 2021-10-03
categories: [Backend, JPA]
tags: [java, jpa, spring]
---

## 들어가면서
인프런에 있는 김영한님의 [실전! 스프링 데이터 JPA](https://inf.run/2wyU) 강의를 정리한 글입니다. 세부사항이나 설정 등은 포스팅하지 않으니, 자세한 내용은 강의를 통해 확인해주시길 바랍니다.

## 목차
- 공통 인터페이스 기능
- 쿼리 메소드 기능
- 페이징과 정렬
- 벌크성 수정 쿼리
- @EntityGraph




### **공통 인터페이스 기능**
스프링 데이터 JPA가 제공하는 공통 인터페이스를 통해 Repository 생성
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

}
```

`JpaRepository`를 확장한 인터페이스는 스프링 데이터 JPA가 구현 클래스를 대신 생성해서 꽂아준다. 

원래는 `AppConfig`에서 `@EnableJpaRepositories`를 통해 위치를 지정해주어야 하지만 스프링 부트 사용 시 `@SpringBootApplication`이 자동으로 위치를 지정해준다.

이 때 스프링 데아터 JPA가 컴포넌트 스캔을 자동으로 처리하기 때문에 `@Repository`를 생략해도 된다.

이러한 `JpaRepository` 인터페이스는 공통 CRUD 기능을 제공한다.

<img src="assets/../../../assets/img/jpa-basic/25.png" width=500>
> 그림 오류 T findOne(ID) -> Optional<T> findById(ID)

#### JpaRepository 주요 메소드 
- `save(S)` : 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합한다.
- `delete(T)` : 엔티티 하나를 삭제한다. 
- `findById(ID)` : 엔티티 하나를 조회한다.
- `getOne(ID)` : 엔티티를 프록시로 조회한다.
- `findAll()` : 모든 엔티티를 조회한다. 정렬(Sort)이나 페이징(Pageable) 조건을 파라미터로 제공할 수 있다.

### **쿼리 메소드 기능**

쿼리 메소드의 기능 3가지
- 메소드 이름으로 쿼리 생성
- 메소드 이름으로 `JPA NamedQuery` 호출
- `@Query` 어노테이션을 사용해서 레포지토리 인터페이스에 쿼리 직접 정의

#### 메소드 이름으로 쿼리 생성

메소드명을 분석해서 자동으로 JPQL 쿼리를 실행한다. 

예를 들어, 순수 JPA 사용 시 이름과 나이를 기준으로 회원을 조회하려고 한다면 아래와 같이 JPQL을 직접 작성한다.

```java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age){
    return em.createQuery("select m from Member m where m.username=:username " +
              "and m.age >:age", Member.class).setParameter("username", username)
              .setParameter("age",age).getResultList();
}
```

스프링 데이터 JPA에서는 메소드명만 규칙에 따라 지어주면 된다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

이 때 메소드명으로 사용되는 키워드는 다음과 같다. 

|키워드|예시|JPQL|
|--:|--:|--:|
|Distinct|findDistinctBy…|select distinct … where … |
|And|findByNameAndAge|… where x.name =? and x.age =?|
|Or|findByNameOrNickName|… where x.name = ? or x.nickname = ?|
|Is, Equals|findByName, findByNameIs,<br>findByNameEquals|… where x.name = ?|
|Between|findByStartDateBetween|… where x.startDate between ? and ?|
|LessThan|findByAgeLessThan|… where x.age < ?|
|LessThanEqual|findByAgeLessThanEqual|… where x.age <= ?|
|GreaterThan|findByAgeGreaterThan|… where x.age > ?|
|GreaterThanEqual|findByAgeGreaterThanEqual|… where x.age >= ?|
|After|findByStartDateAfter|… where x.startDate > ?|
|Before|findByStartDateBefore|… where x.startDate < ?|
|IsNull, Null|findByAge(Is)Null|… where x.age is null|
|IsNotNull, NotNull|findByAge(Is)NotNull|… where x.age not null|
|Like|findByNameLike|… where x.firstname like ?|
|NotLike|findByNameNotLike|… where x.firstname not like ?|
|StartingWith|findByNameStartingWith|… where x.firstname like ? <br> (parameter bound with appended %)|
|EndingWith|findByNameEndingWith|… where x.firstname like ? <br> (parameter bound with prepended %)|
|Containing|findByNameContaining|… where x.firstname like ? <br> (parameter bound wrapped in %)|
|OrderBy|findByAgeOrderByNameDesc|… where x.age = ?1 order by x.name desc|
|Not|findByNameNot|… where x.lastname <> ?|
|In|findByAgeIn(Collection\<Age\> ages)|… where x.age in ?|
|NotIn|findByAgeNotIn(Collection\<Age\> ages)|… where x.age not in ?|
|True|findByActiveTrue()|… where x.active = true|
|False|findByActiveFalse()|… where x.active = false|
|IgnoreCase|findByNameIgnoreCase|… where UPPER(x.name) = UPPER(?)|

- 조회 : `find...By`, `read...By`, `query...By`, `get...By`  
    - 이 때 `...`에는 식별을 위한 설명이 아무거나 들어가도 된다.

- COUNT : `count...By` -> 반환타입 `Long`

- EXISTS : `exists...By` -> 반환타입 `boolean`

- 삭제 : `delete...By`, `remove...By` -> 반환타입 `long`

- LIMIT : `findFirst`, `findFirst3`, `findTop`, `findTop5`

관련된 더 많은 정보는 [스프링 데이터 JPA 공식 문서](https://docs.spring.io/spring-data/jpa/docs/2.5.5/reference/html/#jpa.query-methods)를 참조하자.


#### @Query

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m where m.username= :username nad m.age =:age")
    List<Member> findUser(@Param("username")String username,@Param("age")int age);
}
```
실행할 메소드에 정적 쿼리를 직접 작성하기때문에 이름없는 Named 쿼리라고도 볼 수 있다. 그렇기 떄문에 JPA named 쿼리처럼 애플리케이션 실행 시점에서
문법 오류를 발견할 수 있다.

메소드명을 통한 자동 쿼리 생성 기능은 파라미터가 많아지면 메소드 이름이 매우 길어지기 때문에 `@Query`를 사용하는 경우가 많다.

이 때 전달할 쿼리에 전달할 변수는 `@Param`을 통해 파라미터로 전달받도록 하자.

### **페이징과 정렬**

페이징과 정렬 파라미터
- `org.springframework.data.domain.Sort`
- `org.springframework.data.domain.Pageable`

특별한 반환 타입
- `org.springframework.data.domain.Page` : 추가 count 쿼리 결과를 포함하는 페이징 방식
- `org.springframework.data.domain.Slice` : 추가 count 쿼리 없이 다음 페이지만 확인 가능 (내부적으로 limit+1 조회)
- `List` : 추가 count 쿼리 없이 결과만 반환

페이징과 정렬 사용 예제
```java
Page<Member> findByUsername(String name, Pageable pageable);
Slice<Member> findByUsername(String name, Pageable pageable);
List<Member> findByUsername(String name, Pageable pageable);
List<Member> findByUsername(String name, Sort sort);
```

이것을 활용해서 다음 조건으로 페이징과 정렬을 사용하는 예제를 살펴보자.

- 검색 조건 : 나이가 10살 이상
- 정렬 조건 : 이름으로 내림차순
- 페이징 조건 : 첫 번째 페이지, 페이지당 보여줄 데이터는 3건 

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    Page<Member> findByAge(int age, Pageable pageable);
}
```

```java
@Test
void pagingTest(){
    //given
    memberRepository.save(new Member("member1", 10));
    memberRepository.save(new Member("member2", 10));
    memberRepository.save(new Member("member3", 10));
    memberRepository.save(new Member("member4", 10));
    memberRepository.save(new Member("member5", 10));
    PageRequest pageRequest = PageRequest.of(0, 3, 
                            Sort.by(Sort.Direction.DESC, "username"));

    //when
    Page<Member> page = memberRepository.findByAge(10, pageRequest);

    //then
    List<Member> members = page.getContent();   //조회된 데이터
    long totalElements = page.getTotalElements();   

    members.forEach(System.out::println);
    System.out.println(totalElements);

    assertThat(members.size()).isEqualTo(3);    //조회된 데이터 수
    assertThat(page.getTotalElements()).isEqualTo(5);   //전체 데이터 수 
    assertThat(page.getNumber()).isEqualTo(0);  //페이지 번호 (0번부터 시작)
    assertThat(page.getTotalPages()).isEqualTo(2);  //전체 페이지 수
    assertThat(page.isFirst()).isTrue();    //첫번쨰 항목인지
    assertThat(page.hasNext()).isTrue();    //다음 페이지가 있는지
    }
```

메소드의 파라미터로 전달받은 `Pageable`은 인터페이스다. 따라서 실제 사용 시에 해당 인터페이스를 구현한 `PageRequest` 객체를 사용한다.

`PageRequest` 생성자의 첫 번쨰 파라미터에는 현재 페이지를, 두 번째 파라미터에는 조회할 데이터 수를 입력한다. 추가적으로 정렬 정보도 전달할 수 있다.

Page 사용 시, 상황에 따라 적절히 카운트 쿼리를 분리할 수 있다.

```java
@Query(value = "select m from Member m left join m.team t", 
                        countQuery="select count(m.username) from Member m")
Page<Member> findByAge(int age, Pageable pageable)
```

이 때, 엔티티를 바로 반환하는 대신, 페이지를 유지하면서 DTO로 반환하기 위해서는 `map()`을 사용한다.

```java
Page<Member> page = memberRepository.findByAge(10, pageRequest);
Page<MemberDto> memberDtoPage = page.map(MemberDto::new);
```

### **벌크성 수정 쿼리**

스프링 데이터 JPA를 사용한 벌크성 수정 쿼리
```java
@Modifying
@Query("update Member m set m.age = m.age+1 where m.age >=:age")
int bulkAgePlus(@Param("age") int age);
```

벌크성 수정, 삭제 쿼리는 메소드에 `@Modifying` 어노테이션을 사용해야 쿼리가 내부적으로 `executeUpdate()`로 실행되기 때문에, 추가하지 않으면 예외가 발생한다.

앞서 JPA 기초에서 배웠듯이, 벌크성 쿼리를 실행하고 나면 꼭 영속성 컨텍스트를 비워줌으로써 DB와 영속성 컨텍스트 내 데이터를 일치시켜야 한다. 스프링 데이터 JPA에서는 `@Modifying(clearAutomatically = true)`로 사용하면 벌크 연산 뒤에 영속성 컨텍스트를 자동으로 비워준다.

### **@EntityGraph**
`@EntityGraph`는 연관된 엔티티들을 SQL 한 방에 조회하는 방법, 즉 `fecth join`을 사용한다.


JPQL 패치 조인 
```java
@Query("select m from Member m join fetch m.team t")
List<Member> findMemberFetchJoin();
```

스프링 데이터 JPA는 JPA가 제공하는 엔티티 그래프 기능을 편리하게 사용하도록 도와준다. 이 어노테이션을 사용하면 JPQL 없이 패치 조인을 사용할 수 있다. (JPQL에 적용할 수도 있다.)

```java
//공통 메소드 오버라이드
@Override
@EntityGraph(attributePaths={"team"})
List<Member> findAll();

//JPQL + 엔티티 그래프
@EntityGraph(attributePaths={"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

//메소드 이름으로 쿼리 + 엔티티 그래프
@EntityGraph(attributePaths={"team"})
List<Member> findByUsername(String username);
```

예시로 확인할 수 있듯이 사실상 패치 조인의 간편 버전이라고 볼 수 있으며, 이 때 내부적으로 `LEFT OUTER JOIN`을 사용한다.

