---
title: "Consuming a RESTful Web Service"
author: "yeoji21"
date: 2021-08-07
categories: [Backend, Spring]
tags: [java, spring, guides]
# pin: true
---

## 스프링 문서 링크
아래 게시물은 spring.io guide의 내용입니다.  
환경 설정 및 원본 문서 링크 [link](https://spring.io/guides/gs/consuming-rest/) 
<br>
<br>

## Fetching a REST Resource
Restful 서비스는 https://quoters.apps.pcfone.io/api/random 에서 임의의 JSON 데이터를 받아온다.   

### JSON 형식   
```java
    {
        type: "success",
        value: {
            id: 10,
            quote: "Really loving Spring Boot, makes stand alone Spring apps easy."
        }
    }
```
<br>

### Quote.java
```java
package com.example.consumingrest;
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Quote {

  private String type;
  private Value value;

  public Quote() {
  }

  public String getType() {
    return type;
  }

  public void setType(String type) {
    this.type = type;
  }

  public Value getValue() {
    return value;
  }

  public void setValue(Value value) {
    this.value = value;
  }

  @Override
  public String toString() {
    return "Quote{" +
        "type='" + type + '\'' +
        ", value=" + value +
        '}';
  }
}
```

**@JsonIgnoreProperties(ignoreUnknown = true)** 어노테이션은 JSON 데이터를 받아올 때 해당 객체에 바인딩되지 않는 속성은 무시하도록 한다.  

JSON 데이터를 사용자 정의 class에 직접 바인딩하기 위해서는 속성의 이름이 API에서 반환된 JSON 문서의 key와 일치해야 하는데 

그렇지 않은 경우, **@JsonProperty** 어노테이션을 사용해 속성에 대한 키 이름을 지정할 수 있다.  

<br>

### Value class
```java
package com.example.consumingrest;
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Value {

  private Long id;
  private String quote;

  public Value() {
  }

  public Long getId() {
    return this.id;
  }

  public String getQuote() {
    return this.quote;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public void setQuote(String quote) {
    this.quote = quote;
  }

  @Override
  public String toString() {
    return "Value{" +
        "id=" + id +
        ", quote='" + quote + '\'' +
        '}';
  }
}
```

<br>

## Finishing the Application
### main() method  

이제 ConsumingRestApplication class에 몇 가지 항목을 추가하여 RESTful source의 데이터를 표시해야 한다.  
1. 콘솔에 로그를 표시하기 위한 logger
2. Jackson JSON 라이브러리를 사용하여 데이터를 처리하는 RestTemplate
3. 실행 시 RestTemplate를 실행하고 데이터를 가져오는 CommandLineRunner  

<br>

```java
package com.example.consumingrest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class ConsumingRestApplication {

	private static final Logger log = LoggerFactory.getLogger(ConsumingRestApplication.class);

	public static void main(String[] args) {
		SpringApplication.run(ConsumingRestApplication.class, args);
	}

	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		return builder.build();
	}

	@Bean
	public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
		return args -> {
			Quote quote = restTemplate.getForObject(
					"https://quoters.apps.pcfone.io/api/random", Quote.class);
			log.info(quote.toString());
		};
	}
}
```

### RestTemplate
RestTemplate는 spring 3.0 부터 스프링에서 제공하는 http 통신에 유용하게 쓸 수 있는 템플릿으로, HTTP 서버와의 통신을 단순화하고 RESTful 원칙을 지킨다.(json, xml을 쉽게 응답 받음)  

REST API 호출이후 응답을 받을 때까지 기다리는 동기방식이다.  
<br>

RestTemplate의 주요 메소드는 아래와 같다.  
<img src="https://media.vlpt.us/images/soosungp33/post/d02efacc-56e6-4abc-a390-12ecbb565c6b/image.png" width=500>

<br>

### CommandLineRunner
CommandLineRunner 인터페이스는 구동 시점에 실행되는 코드가 자바 문자열 아규먼트 배열에 접근해야할 필요가 있는 경우에 사용한다. CommandLineRunner 인터페이스를 구현한 클래스에 @Component 어노테이션을 선언해두면 컴포넌트 스캔이되고 구동 시점에 run 메소드의 코드가 실행된다.  
이 예제에서는 따로 구현 클래스를 두지 않고 ConsumingRestApplication 내부에서 **@Bean** 어노테이션을 사용하였다.  
따라서 어플리케이션 구동 시 CommandLineRunner에 의해 RestTemplate의 getForObject 메소드를 호출하여 JSON 데이터를 받아와 Quote class로 파싱하게 된다.


## 실행 결과

```console
2019-08-22 14:06:46.506  INFO 42940 --- [main] c.e.c.ConsumingRestApplication : Quote{type='success', value=Value{id=1, quote='Working with Spring Boot is like pair-programming with the Spring developers.'}}
```
JSON 데이터가 Quote class의 객체로 파싱되어 콘솔 로그에 출력되었다.  
<br>
<br>

## 참조 
- <https://sjh836.tistory.com/141>
- <https://madplay.github.io/post/run-code-on-application-startup-in-springboot>
- <https://www.daleseo.com/spring-boot-runners/>
- <https://jeong-pro.tistory.com/206>