---
title: "객체지향에서의 상속과 다형성"
author: "yeoji21"
date: 2022-07-13
categories: [Backend, OOP]
tags: [java, oop, programming]
---

## 들어가면서

- 해당 포스트는 [조영호님의 오브젝트](http://www.yes24.com/Product/Goods/74219491)를 바탕으로 정리한 글입니다.


## 객체지향과 상속
객체지향 언어에서 상속은 뗼레야 뗄 수 없는 관계다.  
필자도 학교에서 자바를 처음 배울 때 상속을 강의 초반에 배웠고 굉장히 재밌다고 느꼈던 기억이 있다.  
(처음 배운 자바와 C언어와의 직접적인 차이점이었던 것 같다 🤔)

그만큼 상속은 객체지향에서 굉장히 중요하고 기본적인 개념이기에 모두가 확장에 대해서 알고는 있지만,  
모두가 상속을 올바르게 이해하고 있지는 않을 것이다. 

불과 얼마 전까지 누군가 나에게 상속을 한 마디로 정의해보라고 한다면  
"자식 클래스가 부모 클래스를 상속받아서 부모 클래스의 코드를 재사용하고 확장하는 기능이요!"  
라고 했을 것 같다.

틀린 말은 아니지만 이번 포스트에서는 상속의 진정한 목적에대해 전달하려 한다.  

**상속의 진정한 목적은 코드 재사용이 아니라 다형성을 위한 서브타입 계층을 구축하는 것이다.**

다형성을 위해 상속을 사용하는 이유는 상속이 클래스들을 계층을 구성하고, 상황에따라 적절한 메소드를 선택할 수 
있는 메커니즘을 제공하기 때문이다.

여기서 주의해야 할 점은 클래스의 계층을 구성한다는 말 때문에 클래스의 상속을 계층도나 조직도처럼 구성
하는 실수를 하면 안된다는 점이다.

<details>
<summary>상속 계층 더보기 / 접기</summary>
<div markdown="1">

<img src="https://velog.velcdn.com/images%2Fjsj3282%2Fpost%2F352fe936-4593-418c-b08e-30d6bf126dcf%2F010.jpg" width=500>

객체지향의 상속은 이런 개념이 아니다. 코드로 이해해보자.  

```java
삼촌 영희누나 = new 사촌누나();
아버지 민지 = new 딸();
```

사촌누나는 삼촌이고 영희누나라는 이름을 가진다??  
딸은 아버지이고 민지라고 부른다..? 기괴하다.  

<img src="https://velog.velcdn.com/images%2Fjsj3282%2Fpost%2F9d3c8909-e912-4016-aec6-7e2655c6b34c%2F011.jpg" width=500>

위와 같이 객체지향의 상속 계층은 분류도와 같이 구성해야 한다. 이렇게 구성하면 상위 클래스로 갈수록 개념이 추상화되고 하위 클래스로 갈수록 세분화되는 것을 느낄 수 있을 것이다. 

```java
포유류 흰수염고래 = new 고래();
동물 남극펭귄 = new 펭귄();
```
코드로 보아도 이치에 맞고 잘 이해되는 것을 확인할 수 있다. 그렇다면 다시 상속과 다형성으로 돌아가자
</div>
</details>
   
<br>

상속의 어떤 메커니즘을 통해 다형성을 구현하는 것일까?  
사실 자바를 사용해본 사람들은 대부분 아래 코드처럼 상속을 통한 다형성을 경험해보았을 것이다.

```java
public class Test {
    public static void main(String[] args) {
        Digimon agumon = new FireDigimon("XS-1", 100, "꼬마 불꽃");
        agumon.attack();
        agumon.getName();
    }
}

class Digimon{
    String name;
    int hp;

    public void attack() {
        System.out.println("attack!");
    }

    public void getName() {
        System.out.println("my name is " + name);
    }

    public Digimon(String name, int hp) {
        this.name = name;
        this.hp = hp;
    }
}

class FireDigimon extends Digimon {
    String skill;

    @Override
    public void attack() {
        System.out.println(skill + " attack!");
    }

    public String getSkill() {
        return skill;
    }

    public FireDigimon(String name, int hp, String skill) {
        super(name, hp);
        this.skill = skill;
    }
}
```

위 코드를 실행하면 다음과 같은 결과가 나오는 것을 대부분 쉽게 예측할 수 있었을 것이다.

```
꼬마 불꽃 attack!
my name is XS-1
```

그렇다면 이런 결과가 나온 원리를 설명할 수 있겠는가?  

FireDigimon 클래스의 인스턴스를 형변환 없이 Digimon 변수에 넣어도 컴파일 에러가 발생하지 않고  

Digimon 클래스로 선언된 변수인 agumon에 attack() 메소드를 실행했지만 Agumon 클래스에 선언된 attack() 메소드가 작동한 점 

FireDigimon 클래스에는 getName()이라는 메소드가 없지만 문제없이 결과가 출력되는 것은 모두

상속을 사용했기 때문에 코드에 선언된 참조 타입과 무관하게 실제 메시지를 수신하는 객체의 타입에 따라 실행되는 메소드가 달라지기 때문이다.  

부모 클래스(Digimon) 타입으로 선언된 변수에 자식 클래스(FireDigimon)의 인스턴스를 할당하는 것은 **업캐스팅**이 작용했기 때문이고  

선언된 변수의 타입이 아니라 메시지를 수신하는 객체의 타입에 따라 실행되는 메시지가 결정되는 것은 **동적 바인딩** 매커니즘이 작용했기 때문이다. 


### 업캐스팅

```java
Digimon agumon = new FireDigimon()
```

FireDigimon 클래스의 인스턴스를 명시적으로 Digimon 클래스로 변환하지 않아도 에러가 발생하지 않는 이유는 무엇일까?

상속을 이용하면 자식 클래스는 부모 클래스의 속성을 상속받는다. 이때 속성이란 부모 클래스에 속한 변수뿐 아니라 부모 클래스의 퍼블릭 인터페이스까지 포함한다.

이로인해 부모 클래스의 인스턴스 대신 자식 클래스의 인스턴스를 사용하더라도 부모 클래스가 처리하는 메시지를 자식 클래스도 문제없이 처리할 수 있다. 

따라서 협력 내에서 자식 클래스는 부모 클래스의 역할을 할 수 있기때문에 컴파일러는 명시적인 형변환 없이도 자식 클래스가 부모 클래스를 대체할 수 있게 허용하는 것이다. 

반대로 부모 클래스가 자식 클래스의 역할을 하려는 경우는 어떨까?

부모 클래스는 자식 클래스의 퍼블릭 인터페이스를 알지 못한다. 위의 예에서 Digimon 클래스는 getSkill()이라는 메시지를 이해할 수 없다. 

따라서 같은 협력 내에서 부모 클래스는 자식 클래스의 역할을 할 수 없기때문에 컴파일러는 자동적인 다운캐스팅은 허용하지 않으며, 원할 경우 명시적으로 형변환을 해야만 한다.

### 동적 바인딩

객체지향 언어에서는 메시지를 수신했을 때 실행될 메소드가 **런타임**에 결정된다. 

attack() 메소드를 실행했을 때 Digimon 클래스의 attack()이 실행될지, FireDigimon 클래스의 attack()이 실행될지는 클래스의 정의를 살펴봐서는 알 수 없다

실행 시점에 어떤 클래스의 인스턴스에 메시지를 전송했는지를 알아야만 실제로 실행되는 메소드를 알 수 있는데

이렇게 실행될 메소드를 런타임에 결정하는 방식을 동적 바인딩 또는 지연 바인딩이라고 부른다

그렇다면 런타임에 실행될 메소드를 어떻게 결정하는지 그 원리에대해 알아보자 


### 동적 메소드 탐색과 다형성

객체지향에서는 다음 규칙에따라 실행할 메소드를 선택한다 

메시지를 수신한 객체는 먼저 자신을 생성한 클래스에 해당 메소드가 있는지 검사한다.  
존재한다면 메소드를 실행하고 탐색을 종료한다.  
⬇️  
메소드를 찾지 못했다면 부모 클래스에서 메소드 탐색을 계속한다.  
이 과정은 메소드를 찾을 때까지 상속 계층을 따라 올라가며 반복한다.  
⬇️  
최상위 클래스에 이르렀지만 해당 메소드를 찾지 못한 경우 예외를 발생시킨다.

이렇게 **메소드 탐색은 자식 클래스에서 부모 클래스 방향으로 진행된다**.  

따라서 항상 자식 클래스의 메소드가 부모 클래스의 메소드보다 먼저 탐색되기 때문에 자식 클래스에 선언된 메소드가 부모 클래스의 메소드보다 우선순위가 높았던 것이다.

```java
Digimon agumon = new FireDigimon("XS-1", 100, "꼬마 불꽃");
agumon.attack();
```

위 코드에서 FireDigimon 클래스의 attack() 메소드가 실행된 이유가 바로 동적 메소드 탐색 순서때문이었던 것이다. 

Digimon 클래스와 FireDigimon 클래스 모두 attack() 메소드를 가지고 있지만

Digimon 타입인 agumon 변수는 FireDigimon 인스턴스를 업캐스팅한 것이기 때문에 실제로는 FireDigimon의 인스턴스이다.

**메소드 오버라이딩**은 이렇게 자식 클래스에서 메소드가 동일한 시그니처를 가진 부모 클래스의 메소드보다 먼저 탐색되기 때문에 벌어지는 현상인 것이다.

반면에 아래 코드를 보면

```java
Digimon agumon = new FireDigimon("XS-1", 100, "꼬마 불꽃");
agumon.getName();
```

FireDigimon 클래스에는 getName() 메소드가 없다. 

따라서 상속 계층을 따라 부모 클래스에서 해당 메소드를 찾는데 Digimon 클래스에서 getName() 메소드를 찾게되었고 이를 실행한다. 


### 자동적인 메시지 위임

위에서 알아본 것처럼 상속을 이용할 경우 개발자가 메시지 위임과 관련된 코드를 명시적으로 작성할 필요가 없다.

메시지는 상속 계층을 따라 부모 클래스에게 자동으로 위임되기 때문에 상속 계층을 정의하는 것은 메소드 탐색 경로를 정의하는 것과 동일하고 

상속의 진정한 목적은 다형성을 위한 서브타입 계층을 구축하는 것이라는 말은 이를 말하는 것이다.

<br>

오브젝트 10장과 11장에서 살펴본 것처럼 상속을 코드의 재사용을 목적을 위해 사용하는 경우, 코드를 변경하기 어렵고 유연하지 못한 설계에 이르기때문에 

상속은 코드 재사용이 아닌 타입 계층을 구조화하기 위해 사용해야 한다는 것을 알게 되었고

이번 포스트에서는 상속의 관점에서 다형성이 구현되는 메커니즘에 대해 알아보았다.

이를 통해 다형성이란 런타임에 메시지를 처리하기 위해 적합한 메소드를 동적으로 탐색하는 과정을 말하며

상속이 이렇게 메소드를 찾기 위한 탐색 경로를 클래스 계층으로 구현하기 위한 방법이라는 사실을 이해할 수 있게 되었다.

함수형 언어의 함수 호출과 달리, 

객체 지향에서의 메소드 호출은 특정 객체의 메소드를 실행시키는 것이 아닌 메시지를 전송하는 것을 말하며

이 메시지를 수신한 객체가 적절한 메소드를 찾아서 메시지에 응답하는 과정이라는 것을 기억해야겠다.
