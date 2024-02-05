# 객체 생성과 파괴 (1) - 아이템 6 ~ 9

## 목차
[Item 6. 불필요한 객체 생성 피하라](#item-6-불필요한-객체을-생성-피하라)

[Item 7. 다 쓴 객체 참조 해제하라](#item-7-다-쓴-객체-참조-해제하라)

[Item 8. finalizer와 cleaner 사용을 피하라](#item-8-finalizer와-cleaner-사용을-피하라)

[Item 9. try-finally대신 try-with=resources를 사용하라](#item-9-try-finally대신-try-withresources를-사용하라)

## Item 6. 불필요한 객체을 생성 피하라

기능적으로 동일한 객체를 새로 만드는 대신 객체 하나를 재사용하는 것이 대부분 적절

**문자열 객체 생성 예시**
```java
String s = new String("bikini"); // 실행될 때마다 객체 생성
String s = "bikini" // 하나의 인스턴스만 사용
```
new 키워드를 사용하면 새로운 객체가 생성되고 "...." 리터럴 문자열을 사용하면 하나의 String 인스턴스만 사용하고 상수 풀에 저장이 된다
이후 똑같은 문자열을 사용하는 코드는 같은 객체를 참조하게 된다.

```java
// 해당 코드에 출력 결과는 true
String s1 = "bikini";
String s2 = "bikini";
System.out.println(s1 == s2);
```

재사용성 보장!

**Boxing type 대신 Primitive(기본자료형) Type 권장**

Boxing 타입은 null값을 사용가능해야 하는 상황이나, 컬렉션 프레임워크에 기본 자료형 값을 저장해야하는 경우에는 사용해야하지만 꼭 필요한 상황이 아니라면 기본 자료형으로 사용하는게 성능과 메모리 면에서 유리하다.

```java
public class Sum {
    private static long sum() {
        //long 이 아니라 Long으로 선언되어 느리다.
        Long sum = 0L; 
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }
}
```
책에서는 위 코드에서 Long을 -> long으로 바꾸는 것만으로 10배 정도의 차이가 난다고 한다. 직접 실험한 결과도 6배 정도 성능 차이가 났다.

Boxing type을 남발하거나 의도치 않은 Auto Boxing을 조심하자.

**주의할 내장 Method**

String의 matches 함수를 사용하면 정규표현식을 통해 문자열 특정 패턴이 일치하는지 간단하게 확인할 수 있다.

내부 구조를 잠깐 살펴보자

```java
// 닉네임을 검증하는 함수
static boolean isNicknameVaild(String s) {
    return s.matches("^[a-z0-9]{8,12}$");
}

// String.matches
public boolean matches(String regex) {
    return Pattern.matches(regex, this);
}

// Pattern.matches
public static boolean matches(String regex, CharSequence input) {
    Pattern p = Pattern.compile(regex);
    Matcher m = p.matcher(input);
    return m.matches();
}

// Pattern.compile
public static Pattern compile(String regex) {
    return new Pattern(regex, 0);
}
```

matches를 통해 비교만 하고 끝나는줄 알았으나 코드를 계속 타고 내려가다 보면 `isNicknameVaild()` 함수가 호출될때마다 새로운 Pattern 객체가 생성되고 있는걸 알 수 있다.

```java
// static boolean isNicknameVaild(String s) {
//     return s.matches("^[a-z0-9]{8,12}$");
// }
//->->
public class NicknameUtil {
    private static final Pattern NICKNAME = Pattern.compile("^[a-z0-9]{8,12}$");
    static boolean isNicknameValid(String s) {
        return NICKNAME.matcher(s).matches();
    }
}
```

이렇게 수정하면 Pattern 객체가 초기화때 한번만 생성되고 이후로는 재사용하게 된다.

## Item 7. 다 쓴 객체 참조 해제하라
## Item 8. finalizer와 cleaner 사용을 피하라 
## Item 9. try-finally대신 try-with=resources를 사용하라
