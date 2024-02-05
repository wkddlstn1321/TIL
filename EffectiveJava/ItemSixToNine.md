# 객체 생성과 파괴 (1) - 아이템 6 ~ 9

## 목차
[Item 6. 불필요한 객체 생성 피하라](#item-6-불필요한-객체을-생성-피하라)

[Item 7. 다 쓴 객체 참조 해제하라](#item-7-다-쓴-객체-참조-해제하라)

[Item 8. finalizer와 cleaner 사용을 피하라](#item-8-finalizer와-cleaner-사용을-피하라)

[Item 9. try-finally대신 try-with=resources를 사용하라](#item-9-try-finally대신-try-withresources를-사용하라)

## Item 6. 불필요한 객체을 생성 피하라

기능적으로 동일한 객체를 새로 만드는 대신 객체 하나를 재사용하는 것이 대부분 적절

* 문자열 객체 생성
```java
String s = new String("bikini"); // 실행될 때마다 객체 생성
String s = "bikini" // 하나의 인스턴스만 사용
				    //똑같은 문자열은 상수 풀에서 동일한 참조를 제공
```

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
## Item 7. 다 쓴 객체 참조 해제하라
## Item 8. finalizer와 cleaner 사용을 피하라 
## Item 9. try-finally대신 try-with=resources를 사용하라
