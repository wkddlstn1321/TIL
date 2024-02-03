# 객체 생성과 파괴 (1) - 아이템 1 ~ 5

## 목차
[Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라](#item-1-생성자-대신-정적-팩터리-메서드를-고려하라)

[Item 2. 매개변수가 많다면 빌더를 고려하라](#item-2-생성자에-매개변수가-많다면-builder를-고려하라)

[Item 3. 생성자나 열거 타입으로 싱글턴임을 보증하라](#item-3-생성자나-열거-타입으로-싱글턴임을-보증하라)

[Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라](#item-4-인스턴스화를-막으려거든-private-생성자를-사용하라)

[Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라](#item-5-자원을-직접-명시하지-말고-의존-객체-주입을-사용하라)

## Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클래스의 인스턴스를 public 생성자로만 얻을 필요는 없다.

상황에 따라 정적 팩터리 메서드를 제공하는게 더 유용할 수 있다.

비교
```java
public class Laptop {
  private String model;
  private String company;

  // Constructor
  public Laptop(String model, String company) {
    this.model = model;
    this.company = company;
  }

  // 이름을 가진 Static Factory Method
  public static Laptop ofModelNameAndCompany(String model, String company) {
    Laptop laptop = new Laptop();
    laptop.model = model;
    laptop.company = company;
  }
}
```

똑같은 기능을 가진 생성자와 정적 팩터리 메서드 코드를 비교해봤다.

코드를 보면 생성자가 훨씬 간단하고, 만약 멤버변수가 많다고 가정해도 Lombok에서 제공하는 @AllArgsConstruct를 사용하면 편하게 구현할 수 있을거 같은데 정적 팩터리 메서드는 언제 사용?

```java
public class LaptopForm {
  private String model;
  private String company;
}

@PostMapping(value = "/add")
public LaptopDto addLapTop(@RequestBody LaptopForm laptopForm) {
// 생략
}

public class Laptop {
  private String model;
  private String company;
  // laptopForm 객체를 Laptop으로 convert
  public Laptop From(LaptopForm laptopForm) {
    Laptop laptop = new Laptop();
    laptop.model = laptopForm.getModel;
    laptop.company = laptopForm.getCompany;
  }
}

```
LaptopForm 객체를 받는 정적 팩토리 메서드로 Laptop 객체를 만드는 예시

| **장점과 단점**

**장점**
1. 이름을 가질 수 있다.
2. 인스턴스를 매번 생성하지 않아도 된다
3. 반환 타입의 하위 타입 객체를 반환할 수 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
5. 반환할 객체의 클래스가 존재하지 않아도 된다

**단점**
1. 생성자 없이 정적 팩터리 메서드만 제공하면 상속할 수 없다.
2. 프로그래머에게 인지가 잘 되지 않을 수 있다.


**정리**

정적 팩터리 메서드와 Public 생성자의 상대적인 장단점을 이해하고 상황에 맞게 골라서 사용하는 것이 좋다

지금까지는 목적이 무엇이던 객체를 만들 때 정적 팩터리 메서드는 고려해본적이 없없는데,
미리 알고있었다면 유용하게 사용할만한 기회가 없지 않았던거 같다.

다음부터는 예시처럼 DTO 를 VO 로 컨번터하는 것 같이 객체를 다루는 경우는 정적 팩터리 메서드를, 단순히 변수들을 주입하는 경우에는 생성자를 사용한다던지 본인만에 적절한 기준으로 판단해서 사용하면 될 것 같다.

하지만, 생성자로도 충분히 가능한거 같긴해서 이름을 가져서 기능을 명확하게 표현해 줄 수 있는거 말고는 아직 장점이 애매하다...

나중에 경험이 쌓이면 추가해보겠다.


## Item 2. 생성자에 매개변수가 많다면 Builder를 고려하라

```Java
public class NutritionFacts {
    private final int servingSize;  
    private final int servings;     
    private final int calories;     
    private final int fat;          
    private final int sodium;       
    private final int carbohydrate; 
}
```

위 코드 처럼 생성자에 필요한 매개변수 많을 때 사용할 수 있는 2가지 패턴이있다.
  * Pattern 1 : 점층적 생성자 패턴
	
	필수 매개변수와 선택 매개변수 멤버변수 개수만큼 조합하여 미리 생성자를 여러개 만들어서 인스턴스를 만들 때 필요한 생성자를 호출해서 객체를 생성하는 방식이다.

	단점으로는,
	매개변수가 많아지면 코드를 작성하거나 읽기 어렵고
	클래스의 멤버 변수가 추가되면 미리 만들어놨던 모든 생성자를 수정해야 된다는 불편함이 있다.

  * Pattern 2 : Java beans Pattern (Setter)
	매개변수가 없는 생성자로 객체를 만든 후 Setter 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.

	점층적 생성자 패턴보단 비교적 만들기 쉽고 읽기도 쉬운 코드이지만,
	갹채 하나를 만들기 위해서 여러 메서드를 호출해야 해서 코드라인이 많이 길어지게 되고 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태이다.
	프로그래머에게 전적으로 의존해야하는 형태

두 패턴의 장점만을 섞은 방식이 Builder 패턴이다.
  > Builder 구현
  ```Java
  public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
  }
  ```

  Lombok에서 제공하는 기본적인 Builder를 그대로 사용해도되고 "hiddenBuilder" 등의 속성을 이용해서 Custom 해서 사용할 수 도 있다.

**장점**

상속받은 Class의 Buidder가 정의한 build 메서드가
상위 메서드 타입을 return 하는 것이 아닌 자신의 타입을 return한다,

**단점**

1. Builder를 항상 만들어야 하기 때문에 생성 비용이 생긴다.
2. 점층적 생성자 패턴 보다 장황하여 매개변수가 적으면 오히려 불리할 수 있다.

**정리**

매개변수가 적은 경우에는 Builder를 사용하지 않고 생성자를 사용하여 구현하는게 좋지만, API는 시간이 지날수록 매개변수가 많아지는 경향이 있기때문에 기능을 잘 고려하여 Builder 사용여부를 판단해야겠다.


## Item 3. 생성자나 열거 타입으로 싱글턴임을 보증하라 

public static member

INSTANCE 가 초기화 되고 나면 고정이 된다.
```java
// publlic static final 필드 방식
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
}
```

```java
//정적 팩터리 방식
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }
}
```
static 멤버를 private 으로 막은 방식 get함수만 추가된거 같지만 캡슐화를 해준거기 때문에 좀 더 유연하게 사용가능하다.
예를들어 api를 수정하지 않고도 내부 구조 수정 가능


아래는 책에서 추천하는 방식의 싱글턴
```java
// 열거 타입 방식의 싱글턴
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        ...
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```
enum 방식의 장점으로 간단한 구현, 복잡한 직렬화 상황이나 리플렉션 공격에서도 보호가 된다. 확장성도 좋다 새로운 싱글턴 인스턴스를 추가할 때 코드를 크게 변경할 필요가 없다.

하지만 단점이 없진않다.

enum 은 상속이 불가능하다. 고로 기존 클래스의 기능을 활용하여 싱글턴을 구현할 수 없다. 

enum의 사용의도와 맞지 않을 수 있다. enum을 싱글턴으로 활용할 수 있다는 내부 구성원끼리 합의도 필요한데 이건 강의에서도 실용성에 대해서는 의문을 갖는 부분

**정리**

Singleton pattern에 여러 구현방법에 대해서 잘 숙지하고,
상황에 맞게 선택해서 사용할 수 있도록 하자.

Spring 컨테이너도 싱글톤을 사용하니 알아두면 많이 도움이 될 것

## Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

util class처럼 인스턴스화 하지 않고 사용하려고 클래스를 설계할경우 생성자를 생략하면 컴파일러가 기본 생성자를 만들어준다.

그럴 경우 사용자는 이 생성자가 자동으로 만들어진건지 의도된건지 알 수 없다.

명시적으로 private 생성자를 만들어줌으로서 컴파일러의 생성자 자동 생성을 막고, 설계 의도와 다르게 클래스를 인스턴스화 해서 사용하는 실수를 방지할 수 있다.

## Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

클래스 내부에서 사용하는 자원을 직접 명시하는것 보다 의존객체주입을 사용하는것이 더 바람직하다.

예시를 보자

```java
// 정적 자원을 사용하는 테스트 클래스
public class Player {
  private static final String job  = "Hunter";

  public static void printJob() {....}
}
```
Player의 직업이 hunter로 고정이 되어있다. 하나밖에 사용하지 않으니 아직은 문제가 없다.

하지만 다음 업데이트에서 여러 직업이 추가되었다. 하지만 job이 상수로 박혀있어서 골라서 선택할 수 가 없다.


```java
public class Player {
  private final String job;
   // 의존성 주입
  public Player(String job) {
    this.job = job;
  }
  public static void printJob() {....}
}
```

값을 외부에서 주입 받음으로 원하는 직업을 인자로 넘겨줘서 해당 직업을 가진 Player 클래스를 생성할 수 있다.

**정리**

의존 객체 주입은 유연성과 테스트 용이성을 제공해준다.

다만, 의존성이 아주 많아진다면? 하나씩 일일히 넣어주는게 개발자 입장에서 피곤할 수 있는데 이때는 여러 프레임워크를 통한 역전제어를 이용해 프레임워크가 의존성을 주입할 수 있게해줄 수 있다.

지금은 이펙티브자바니까 역전제어의 대한 자세한 설명은 생략하고 **클래스 내부에서 사용하는 자원은 고정값보단 외부에서 주입받아 정의하는게 좋다**로 마무리