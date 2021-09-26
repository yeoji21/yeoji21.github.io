---
title: "[Spring] Validation "
author: "yeoji21"
date: 2021-09-26
categories: [Backend, Spring]
tags: [java, spring]
---

## 들어가면서
인프런에 있는 김영한님의 [스프링 MVC 2편 - 백엔드 웹 개발 활용 기술](https://inf.run/rtcu) 강의를 정리한 글입니다. 세부사항이나 설정 등은 포스팅하지 않으니, 자세한 내용은 강의를 통해 확인해주시길 바랍니다.


## 목차
- BindingResult
- 오류 코드와 메시지 처리
- MessageCodesResolver
- Validator 분리
- Bean Validation 소개

### **BindingResult**

`BindingResult`는 스프링이 제공하는 검증 오류를 보관하는 객체이다. 검증 오류가 발생하면 이 객체에 보관하면 된다. 따라서 `BindingResult`가 있으면 `@ModelAttribute`에 데이터 바인딩 시 오류가 발생해도 컨트롤러가 정상 호출된다. 만약 `BindingResult`가 없으면 `400 오류`가 발생하면서 컨트롤러가 호출되지 않고 오류 페이지로 이동한다.

`BindingResult`에 검증 오류를 적용하는 3가지 방법은 다음과 같다.
- `@ModelAttribute`의 객체에 타입 오류 등으로 바인딩이 실패하는 경우, 스프링이 `FiledError`를 생성해서 자동으로 넣어준다.
- 개발자가 직접 넣어준다.
- `Validator`를 사용한다.

주의해야할 점으로, `BindingResult`는 검증할 대상 바로 다음에 와야한다는 점이다. 따라서 순서가 중요하다. 이 때 `BindingResult`는 자동으로 `Model`에 포함된다.
```java
@PostMapping("/...")
public String exController(@ModelAttribute Item item, BindingResult bundingResult,
                                    RedirectAttributes redirectAttributes){
    ...
}
```
<br>

이제 단계별로 차근차근 `BindingResult`를 사용하는 방법을 알아보자. 
```java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult
                                    , RedirectAttributes redirectAttributes){
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item","itemName"
                                                ,"상품이름은 필수입니다."));
    }
    if (item.getPrice() == null || item.getPrice() < 1000 
                    || item.getPrice() > 1000000) {
        bindingResult.addError(new FieldError("item","price",
                "가격은 1,000 ~ 1,000,000까지 허용합니다."));
    }
    if (item.getPrice() != null & item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.addError(new ObjectError("item",
                    "가격*수량의 합은 10,000원 이상이어야 합니다."));
        }
    }
    if (bindingResult.hasErrors()) {
        log.warn("error = {}", bindingResult);
        return "validation/v2/addForm";
    }

    // ...
    // 밑은 Validation에 문제 없을 시, 성공 로직
}
```
<br>

**필드 오류 - `FieldError`**  
```java
public FieldError(String objName, String field, String defaultMessage){}
```

- 객체의 필드에 오류가 있으면 `FieldError`객체를 생성해서 `bindingResult`에 담아두면 된다.
- objName : `@ModelAttribute` 이름
- field : 오류가 발생한 필드명
- defaultMessage : 오류 기본 메시지


**글로벌 오류 - `ObjectError`**  
```java
public ObjectError(String objName, String defaultMessage){}
```

- 하나의 특정 필드를 넘어서는 오류가 있으면 `ObjectError`객체를 생성해서 `bindingResult`에 담아주면 된다.


### **오류 코드와 메시지 처리**
FieldError는 두 가지 생성자를 제공한다.
```java
public FieldError(String objName, String field, String defaultMessage){}

public FieldError(String objName, String field, @Nullable Object rejectedValue,
                boolean bindingFailure, @Nullable String[] codes, 
                @Nullable Object[] arguments, @Nullable String defaultMessage)
```

- rejectedValue : 거절된 값
- bindingFailure : 타입 오류같은 바인딩 실패인지, 검증 실패인지 구분 값
- codes : 메시지 코드
- arguments : 메시지에서 사용되는 인자
  
`FieldError`, `ObjectError`의 생성자는 `errorCode`, `arguments`를 제공한다. 이것은 오류 발생 시 오류 코드로 메시지를 찾기 위해 사용된다. 

#### errors 메시지 파일 생성
먼저 스프링 부트가 해당 메시지 파일을 인식할 수 있게 다음 설정을 추가해야 한다. 

**application.properties**  
```properties
spring.messages.basename=messages,errors
```

**errors.properties 추가**  
```properties
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0}~{1}까지 허용합니다.
max.item.quantity=수량은 최대 {0}까지 허용합니다.
totalPriceMin=가격*수량의 합은 {0}이상이어야 합니다. 현재 값 = {1}
```

이제 위의 예제를 errors에 등록한 메시지를 사용하도록 적용해보자.

```java
if (!StringUtils.hasText(item.getItemName())) {
    bindingResult.addError(new FieldError("item","itemName",item.getItemName()
    ,false ,new String[]{"required.item.itemName"},null, null));
}
if (item.getPrice() == null || item.getPrice() < 1000 
                || item.getPrice() > 1000000) {
    bindingResult.addError(new FieldError("item","price", item.getPrice(),
            false, new String[]{"max.item.quantity"}, new Object[]{9999}, null));
}
if (item.getPrice() != null & item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
        bindingResult.addError(new ObjectError("item"
        ,new String[]{"totalPriceMin"}, new Object[]{10000, resultPrice},null));
    }
}
```
실행해보면 `errors`에 있는 `MessageSource`를 찾아서 메시지를 조회하는 것을 확인할 수 있다. 하지만 이렇게 되면 `FieldError`와 `ObjectError`는 다루기 너무 번거롭다. 오류 코드도 좀 더 자동화할 수 있지 않을까? 

`BindingResult`가 제공하는 `rejectValue()`와 `reject()`를 사용하면 `FieldError`와 `ObjectError`를 직접 생성하지 않고, 깔끔하게 검증 오류를 다룰 수 있다.

거두절미하고, `rejectValue()`와 `reject()`를 사용해서 단순화한 코드를 살펴보자.

```java
if (!StringUtils.hasText(item.getItemName())) {
    bindingResult.rejectValue("itemName", "required");
}
if (item.getPrice() == null || item.getPrice() < 1000 
            || item.getPrice() > 1000000) {
    bindingResult.rejectValue("price", "range", new Object[]{1000,1000000}, null);
}
if (item.getPrice() != null & item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
        bindingResult.reject("totalPriceMin"
                             ,new Object[]{10000, resultPrice}, null);
    }
}
```

**`rejectValue()`**  
```java
void rejectValue(@Nullable String field, String errorCode, 
                 @Nullable Object[] errorArgs, @Nullable String defaultMessage);
```

- field : 오류 필드명
- errorCode : 오류 코드(messageResolver가 처리할 코드)
- errorArgs : 오류 메시지에서 {0}를 치환하기 위한 값
- defaultMessage : 오류 메시지를 찾을 수 없을 때 사용하는 기본 메시지


**`reject()`**  
```java
void reject(String errorCode, @Nullable Object[] errorArgs, 
            @Nullable String defualtMessage)
```

<br>

`BindingResult`는 검증할 객체 바로 뒤에 오기 때문에 어떤 객체를 대상으로 검증하는지 알고 있다. 따라서 `target(item)`에 대한 정보는 없어도 된다. 따라서 `field`로 `price`만 전달하면 된다. 

```java
bindingResult.rejectValue("price", "range", new Object[]{1000,1000000}, null);
```

그런데, `FieldError`를 직접 다룰 때는 오류 코드를 `range.item.price`와 같이 모두 입력했는데, `rejectValue()`에서는 오류 코드를 `range`로 간단하게 입력했다. 그런데도 결과를 보면 오류 메시지를 잘 찾아서 출력한다.

무언가 규칙이 있는 것으로 보이는데, 이것을 이해하기 위해서는 `MessageCodesResolver`를 이해해야 한다. 

#### 오류 메시지
오류 코드를 만들 때 다음과 같이 자세히 만들 수도 있고, 
```properties
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0}~{1}까지 허용합니다.
```

또는 다음과 같이 단순하게 만들 수도 있다.
```properties
required=필수값입니다.
range=범위 모류입니다.
```

가장 좋은 방법은 단순한 메시지를 범용적으로 사용하다가, 세밀하게 작성해야 하는 경우에는 세밀한 내용이 적용되도록 메시지에 단계를 두는 방법이다. 

예를 들어, `required`라는 메시지만 있으면 이 메시지를 선택해서 사용하고, 오류 메시지에 `required.item.itemName`와 같이 객체명과 필드명을 조합한 세밀한 메시지 코드가 있으면 이 메시지를 높은 우선순위로 사용하는 것이다. 스프링은 `MessageCodesResolver`라는 것으로 이러한 기능을 지원한다.

### **MessageCodesResolver**
우선 테스트 코드로 `MessageCodesResolver`를 알아보자.

```java
public class MessageCodesResolverTest {

    MessageCodesResolver codesResolver = new DefaultMessageCodesResolver();

    @Test
    void messageCodesResolverObject(){
        String[] messages = codesResolver.resolveMessageCodes("required", "item");
        assertThat(messages).containsExactly("required.item", "required");
    }

    @Test
    void messageCodesResolverField(){
        String[] messages = codesResolver.resolveMessageCodes("required", 
                                            "item", "itemName", String.class);
        assertThat(messages).containsExactly(
                "required.item.itemName",
                "required.itemName",
                "required.java.lang.String",
                "required"
        );
    }
}
```

코드로 알 수 있듯이 `MessageCodesResolver`는 인터페이스이고, `DefaultMessageCodesResolver`가 기본 구현체이다. 


#### DefaultMessageCodesResolver의 기본 메시지 생성 규칙

**객체 오류**  
```
1. code + "." + object name
2. code
```


예) 오류 코드 : `required`, object name : `item`
1. `required.item`
2. `required`


**필드 오류**  

```
1. code + "." + object name + "." + field
2. code + "." + field
3. code + "." + field type
4. code
```

예) 오류 코드 : `typeMismatch`, object name : `user`, field : `age`, field type: `int`
1. `typeMismatch.user.age`
2. `typeMismatch.age`
3. `typeMismatch.int`
4. `typeMismatch`


#### 동작 방식
`rejectValue()`, `reject()`는 내부에서 `MessageCodesResolver`를 사용한다. `FieldError`와 `ObjectError`의 생성자를 보면, 여러 오류 코드를 가질 수 있는데, 
`MessageCodesResolver`를 통해서 생선된 순서대로 오류 코드를 보관한다.



### **Validator 분리**
컨트롤러에서 검증 로직이 차지하는 부분은 매우 크다. 이런 경우 별도의 클래스로 역할을 분리하는 것이 좋다. 

```java
@Component
public class ItemValidator implements Validator {
    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Item item = (Item) target;
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "itemName", "required");
        
        if (!StringUtils.hasText(item.getItemName())) {
            errors.rejectValue("itemName", "required");
        }
        if (item.getPrice() == null || item.getPrice() < 1000 
                    || item.getPrice() > 1000000) {
            errors.rejectValue("price", "range", 
                                new Object[]{1000,1000000}, null);
        }
        if (item.getPrice() != null & item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                errors.reject("totalPriceMin", 
                                new Object[]{10000, resultPrice}, null);
            }
        }
    }
}
```

`itemValidator` 직접 호출하기
```java
private final ItemValidator itemValidator;

@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult){
    itemValidator.validate(item, bindingResult);
    ...
```


`@Validated`적용
```java
@PostMapping("/add")
public String addItemV1(@Validated @ModelAttribute Item item, 
                        BindingResult bindingResult){
    if(bindingResult.hasErrors()){
        ...
    }
    ...
```
`validator`를 직접 호출하지 않고 검증 대상 앞에 `@Validated`만 붙여줘도 기존과 동일하게 잘 동작한다. `@Validated`는 검증기를 실행하라는 어노테이션이다. 


### **Bean Validation 소개**
검증 기능을 지금처럼 매번 코드로 만들고, 검증기로 분리하는 것은 상당히 번거롭다. 특히 특정 필드에 대한 검증 로직은 대부분 빈 값이 아닌지, 특정 크기를 넘는지 아닌지와 같이 매우 일반적인 로직이 대부분이다.
다음 코드를 보자.
```java
public class Item {
    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min=1000, max=1000000)
    private Integer price;

    @NotNull
    @Max(9999)
    private Integer quantity;
}
```
이런 검증 로직을 모든 프로젝트에 적용할 수 있도록 공통화하고, 표준화한 것이 바로 `Bean Validation`이다. 

`Bean Validation`을 사용하기 위해서는 다음 의존관계를 추가해야 한다.
```
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

#### Bean Validation의 검증 어노테이션

|어노테이션|설명|
|--:|--:|
|@NotNull|속성값이 null이 아닌지|
|@AssertTrue|속성값이 참인지|
|@Size|속성값이 최소와 최대 사이의 크기를 가지는지 <br> String, Collection, Map 및 Array에 적용 가능|
|@Min|속성값이 지정된 값보다 작은 값을 가지는지|
|@Max|속성값이 지정된 값보다 큰 값을 가지는지|
|@Email|속성이 이메일 형식을 가지는지|
|@NotNull|속성이 null 또는 비어있지 않은지 <br> String, Collection, Map 및 Array에 적용 가능|
|@NotBlank|텍스트값에만 적용가능하며, 속성이 null 또는 공백인지|
|@Positive<br>@PositiveOrZero|숫자값이 양수인지|
|@Negative<br>@NegativeOrZero|숫자값이 음수인지|
|@Past<br>@PastOrPresent|현재를 포함하여 날짜값이 과거인지|
|@Future<br>@FutureOrPresent|현재를 포함하여 날짜가 미래인지|


**@NotNull, @NotNull, @NotBlank의 차이점**  
- `@NotNull`  
    - `null`만 허용하지 않음
    - `""`이나 `" "`은 허용
- `@NotEmpty`  
    - `null`과 `""`를 허용하지 않음
    - `" "`는 허용
- `@NotBlank`  
    - `null`, `""`, `" "`를 모두 허용하지 않음


#### 스프링 부트는 어떻게 Bean Validator를 사용할까?
스프링 부트가 `spring-boot-starter-validation`라이브러리를 넣으면 자동으로 `Bean Validator`를 인지하고 스프링에 통합한다. 

그 후 `LocalValidatorFactoryBean`을 글로벌 `Validator`로 등록한다. 이 `Validator`는 `@NotNull`같은 어노테이션을 보고 검증을 수행한다. 이렇게 글로벌 `Validator`가 
적용되어 있기 때문에 컨트롤러에서는 검증이 필요한 객체 앞에 `@Valid`, `@Validated`만 적용하면 된다.

이 때 검증 오류가 발생하면, `FieldError`나 `ObjectError`를 생성해서 `BindingResult`에 담아준다.

> 검증 시 @Valid와 @Validated 둘 다 서용 가능하다.  
> @Validated는 스프링 전용 검증 어노테이션이고, @Valid는 자바 표준 검증 어노테이션이다. 둘 중 아무거나 사용해도 동일하게 동작하지만, @Validated는 groups 기능을 포함하고 있다.

참고로 `Bean Validation`이 제공하는 오류 메시지를 좀 더 자세히 변경하고 싶다면, 해당 어노테이션의 오류 코드를 기반으로 `MessageCodesResolver`를 통해 생성되는 메시지를 등록하면 된다.

`@NotNull`
```properties
NotBlank.item.itemName = 아이템 이름을 입력해주세요.
NotBlank.itemName = 아이템 이름을 입력해주세요.
NotBlank.java.lang.String = 문자를 입력해주세요.
NotBlank = 공백일 수 없습니다.
```

#### Bean Validation - 오브젝트 오류
`Bean Validation`에서 하나의 필드를 검증하는 것이 아니라 오브젝트 관련 오류는 `@ScriptAssert()`라는 어노테이션으로 처리할 수 있다.

그런데 실제로 사용해보면 제약이 많고 복잡해서 잘 쓰이지 않는다. 따라서 오브젝트 오류의 경우, 직접 자바 코드로 작성하는 것을 권장한다.

```java
if (item.getPrice() != null & item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
        errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
    }
}
```

#### @ModelAttribute vs @RequestBody
HTTP 요청 파라미터를 처리하는 `@ModelAttribute`는 각각의 필드 단위로 세밀하게 적용된다. 그래서 특정 필드의 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리할 수 있다.
그에비해 `HttpMessageConverter`는 각각의 필드 단위로 적용되지 않고, 전체 객체 단위로 적용된다. 따라서 메시지 컨버터의 작동이 성공해서 `Item`객체를 만들어야 `@Valid`, `@Validated`가 적용된다.



## 참조
<https://www.baeldung.com/java-bean-validation-not-null-empty-blank>  
<https://sanghye.tistory.com/36>