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

책을 읽으면서 느낀건데 정말 필요한 상황이 아니라면 재정의하지 않는게 좋겠다..

## Item 11. equals를 재정의하려거든 hashcode도 재정의하라



## Item 12. toString을 항상 재정의하라



## Item 13. clone 재정의는 주의해서 진행해라



## Item 14. comparable을 구현할지 고민하라

