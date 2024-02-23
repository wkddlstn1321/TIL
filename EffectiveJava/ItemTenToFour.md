# 모든 객체의 공통 메서드 - 아이템 10 ~ 14

자바의 클래스를 선언할 때 extends 키워드를 생략하면 컴파일러는 자동으로 extends Object를 추가한다.

고로 Object는 모든 Java 클래스의 최상위 부모 클래스이다.

Object 클래스는 필드를 가지지 않으며 메소드들만으로 구성되어 있다.

이번 챕터는 재정의를 염두에 두고 설계된 Object의 메서드들을 어떻게 재정의해야하는지를 살펴보자 

## 목차
[Item 10. equals는 일반 규약을 지켜 재정의하라](#item-10-equals는-일반-규약을-지켜-재정의하라)

[Item 11. equals를 재정의하려거든 hashcode도 재정의하라](#item-11-equals를-재정의하려거든-hashcode도-재정의하라)

[Item 12. toString을 항상 재정의하라](#item-12-tostring을-항상-재정의하라)

[Item 13. clone 재정의는 주의해서 진행해라](#item-13-clone-재정의는-주의해서-진행해라)

[Item 14. comparable을 구현할지 고민하라](#item-14-comparable을-구현할지-고민하라)

## Item 10. equals는 일반 규약을 지켜 재정의하라

equals를 재정의 하지 않으면 오직 자기 자신일때만 같게 된다.

책에서는 equals는 꼭 필요한 경우가 아니라면 재정의 하지 않는걸 추천하는데

논리적 동치성을 확인하고 싶은데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되어있지 않을때만 재정의하는게 좋겠다.

어떻게 보면 당연한 소리이기는 하다. 객체가 똑같은지를 확인하고 싶은거면 재정의를 안해도 되거니와,
특수한 상황이 아니고서야 equals를 선언하는 경우는 객체가 가지고 있는 값이 같은지를 확인하고 싶을때일터

Integer, String 같은 값을 표현하는 클래스들이 그러하듯 같이 가지고 있는 **값** 만을 비교하고 싶을 때 재정의 해주면 되겠다.

equals 메서드를 재정의할 때 따라야하는 일반 규약

null이 아닌 참조값일 때를 기준으로 아래 조건들이 성립해야한다.

* 반사성 : x에 대해 x.equals(x)는 true
* 대칭성 : x, y에 대해 x.equals(y) 가 true면 y.equals(x) 도 true
* 추이성 : x, y, z에 대해 x.equals(y)가 true면 y.equals(z)도 true이면 x.equals(z)도 true 
* 일관성 : x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
* not null : x.equals(null)은 false

위 조건들을 성립하기 위한 검사 패턴은 아래와 같다.
1. == 을 통해 input이 자기 자신의 참조인지
2. instanceof를 통해 input 타입이 명확한지
3. 2를 통해 검사한 객체를 올바른 타입으로 형변환
4. 핵심 필드들이 모두 일치하는지
5. not null 인지

```java
class Test{
	public int a;

	Test(int num) {
		this.a = num;
	}

	@Override
	public boolean equals(Object obj) {
		if (obj instanceof Test) {
			Test p = (Test)obj;
			return a == p.a;
		}
		return false;
	}
}
```
간단한 Test 클래스를 만들어서 equals를 재정의해보았다. 원래라면 hashcode도 재정의 해야하지만 다음 아이템이기 때문에 여기서는 생략

**정리**

책을 읽으면서 느낀건데 정말 필요한 상황이 아니라면 재정의하지 않는게 좋겠다..
equals를 구현하기 위해 고려해야 하는 5가지 규약을 확실히 지키기가 생각보다 까다로워 보인다.

## Item 11. equals를 재정의하려거든 hashcode도 재정의하라

equals를 재정의한 클래스 모두에서 hashcode도 재정의해야 한다.

그렇지 않으면 "equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야한다." 는 일반 규약을 어기게 된다.

논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

아래에 코드를 보자

```java
public static void main(String[] args) {
	Map<Test, String> map = new HashMap<>();
	map.put(new Test(10), "홍길동");
	// null이 출력된다.
	System.out.println(map.get(new Test(10)));
}
```
홍길동을 의도했지만 null이 반환된다.

Item 10 에서 equals를 재정의한 Test클래스는 멤버변수인 a만 같다면 논리적으로 같은 객체임을 의미하지만 
hashcode는 재정의 되어있지 않기때문에 둘을 다르다고 판단하여 서로 다른 값을 반환하게 된다.
이런 경우 hash관련된 자료구조는 사용할 수 없게 되기 때문에 hashcode를 재정의 해줄 필요가 있다.

~~만약 설계상 hash 기능을 안쓰는 클래스라면? 그렇다 해도 미래는 알 수 없다. 안전하게 가야지~~

hashcode를 작성 요령

1. 아주 간단한 방식
```java
	@Override
	public int hashCode() {
		// 인자는 모든 핵심 필드
		return Objects.hash(a);
	}
```
Object 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 hash 메서드를 제공해준다.
있는거 그대로 사용하는 아주 간단한 방식!, 그러나 성능은 좋지 못하다고 한다.
입력 인수를 담기 위한 배열 생성, 입력 중 기본 타입이 있다면 박싱, 언박싱 과정을 거치게 되니,
성능이 중요한 경우에는 좋지 못한 방법이라고 볼 수 있겠다.

2. 정석
int result를 아래의 방법으로 초기화
첫번째 핵심필드(a)의 해시 코드를 계산 
	* 기본 타입 : Type.hashcode(a)
	* 참조 타입 :
		equals가 재귀적 호출 : hashcode도 재귀적 호출 or
		표준형을 만들고 그 표준형의 hashcode를 호출
	* 배열 타입 : 핵심원소 각각을 별도 필드처럼 다룬다,

나머지 모든 핵심필드에 대해서 아래 과정 반복
int result = 31 * result + c;

위 규칙을 멤버변수 b를 추가한 Test 클래스에 적용한 결과
```java
@Override
public int hashCode() {
	int result = Integer.hashCode(a);
	result = 31 * Integer.hashCode(b);
	return result;
}
```

아주 정석적인 방식이라고 볼 수 있지만 핵심 필드에 참조 타입과 배열 타입이 포함된다면 머리가 아프지 않을 수 없다.

그럴땐 3번 방법도 고려해봄직 하다.

3. Lombok 활용

``@EqualsAndHashCode(exclude = {"비교에서 제외할 필드"})``

@EqualsAndHashCode 어노테이션을 사용한다면 비교를 원하지않는 필드를 제외하면서 equals와 hashcode를 재정의를 자동으로 해준다.
성능을 챙기면서 편하게 한줄로 재정의를 해줄 수 있는 마법

물론 진짜 마법은 아니기 때문에 사용할일이 생긴다면 해당 어노테이션의 내부 동작원리는 이해하고 사용하는것이 좋겠다.


**정리**

이번기회에 hash동작 방식에 대해서 더 자세히 이해하는 기회가 됐고 equals는 진짜 웬만하면 재정의하지 말자는 교훈도 얻게됐다.

꼭 필요한 경우라면 구현하는게 맞고 그걸 위해서 Item 11을 다루기도 했지만 재정의를 할 때 놓치기 쉬운 실수들이 많아서
정말 어쩔 수 없을 때만 규칙을 놓치지 말고 잘지켜서 재정의해보자

## Item 12. toString을 항상 재정의하라

toString에 default value 는 "className@ 16진수로 표현한 hashcode" 이다.

재정의 되지 않은 toString의 기능으로 개발자가 이해할 수 있는 부분은 해당 객체가 어떤 클래스명을 가지고 있는지 까지이다. + hashcode

toString의 일반 규약인 "간결하고 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다" 를 따르자면 재정의를 하는게 맞다.

toString을 직접사용하지 않더라도 오류 메시지를 로깅할 때 자동으로 호출되는 경우도 있기 때문에 overriding을 해주자

**toString 재정의**

객체가 가진 주요 정보는 전부 반환해 주는게 좋다

```java
//toString을 재정의할 Person 클래스
public class Person {
	private String name;
	private int age;
	private String address;

	public Person(String name, int age, String address) {
		this.name = name;
		this.age = age;
		this.address = address;
	}
}
```

방법 1. 직접 구현
```java
	// 나타내고 싶은 필드들 (핵심 정보들)을 직접 만들어서 리턴해준다.
	@Override
	public String toString() {
		return "Person{" +
				"name='" + name + '\'' +
				", age=" + age +
				", address='" + address + '\'' +
				'}';
	}
```

방법 2. Lombok 의 @ToString 이용

```java
// 클래스위에 ToStinrg 어노테이션 붙여주면 끝
@ToString()
public class Person {
	... 생략
}
```
Lombok... 아주 편리하다고 볼 수 있다.

대신 Lombok 이용 시 모든 필드 정보를 전부 리턴하기 때문에 필요없는 정보를 빼고 싶다면 exclude를 이용해서 제외해주자

어노테이션의 인자로 추가

``@ToString(exclude = {"address"})``

또는

변수 선언시 추가

``@ToString.Exclude private String address;``

**정리**

toString 구현은 까다롭지 않기 때문에 귀찮더라도 직접 로그를 찍을 일이 있다면 반드시 재정의를 해주자.

Lombok을 사용한다면 어노테이션만 추가하면 구현이 되기 때문에 안 할 이유가 없다.

## Item 13. clone 재정의는 주의해서 진행해라

clone은 객체의 모든 필드를 새로운 객체에 복사하여 반환하는 동작을 수행한다.

기본형 타입을 clone하는 경우는 상관없지만 참조타입을 clone하는 경우 원본 객체와 복사된 객체가 같은 주소를 참조하기 때문에 값을 공유해서 문제가 생길 수 있다.

완벽한 복사를 원한다면 clone을 재정의 해야하지만 몇가지 주의점이 있다.

**Cloneable 구현하기**

clone을 재정의 하려면 Cloneable을 구현해야한다.

cloneable은 clone으로 복사 가능하다라는 뜻을 표시하는 인터페이스이다.

cloneable을 구현하지 않고 clone을 재정의하면 clone을 호출할 때 CloneNotSuppotedException 예외를 던진다.

이부분 좀 신기하다. cloneable 인터페이스는 아무런 메서드도 포함하지 않은 깡통인데 이걸 implements 하냐 안하냐 따라서 예외를 던지고 말고를 정한다. 어케한거냐?

**반환타입**

재정의된 clone은 public 접근 지정자로 선언해야 하고 자신의 타입을 리턴해야 한다.

이게 무슨 소리냐

clone의 리턴값은 Object다. 또는 자식 클래스였다면 상속받은 부모 클래스 타입이 리턴된다. 

값을 반환받으며 적절한 형변환을 해주자.


**정리**

상세하게 정리하지는 않았는데 책에서도 강의에서도 clone 재정의를 그렇게 추천하지는 않는거 같다. 실제로 나같은 경우도 값을 복사하길 원하면 내부에서 deepCopy메서드 하나 만들어서 사용했던 기억이 있는데 그게 clone 재정의 하는것보다는 훨씬 쉽다. 

메서드 정의 말고도 clone 대체할만한 추천 방식으로 생성자나 팩터리 메서드를 정의하는건데 clone과 똑같은 기능을 수행해줄 수 있기 떄문에 힘들게 clone을 재정의 할 필요는 없을거같다.(객체를 생성해서 리턴한다는 구조 때문에 clone보다 더 좋아 보이기도 한다)

아직은 clone에 필요성을 크게 못느껴서 나중에 기회가 되면 추가해보겠다.

## Item 14. comparable을 구현할지 고민하라

