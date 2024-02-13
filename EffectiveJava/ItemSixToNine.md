# 객체 생성과 파괴 (2) - 아이템 6 ~ 9

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

이렇게 수정하면 Pattern 객체가 초기화때 한번만 생성되고 이후로는 효율적으로 재사용할 수 있게된다. ~~matcher도 호출할 때 마다 생성되고 있나?~~

## Item 7. 다 쓴 객체 참조 해제하라

Managed 언어인 Java에서는 가비지 컬렉터를 통해 메모리를 관리한다.
개발자가 직접 신경쓰지 않아도 사용하지 않는 객체를 회수하기 떄문에 편리하지만 만능은 아니다

기능적으로는 사용하지 않는 객체여도 참조가 남아있다면 가비지 컬렉터가 메모리를 회수하지 않는다.

```java
// 스택 구현
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

위 스택을 구현한 코드를 보면 size라는 멤버 변수를 통해 스택의 사이즈를 관리하고 있고 요소를 제거하는 pop 메서드에서 size-- 를 통해 해당 원소의 더 이상 접근하지 않는다.

의도대로면 size 보다 큰 elements의 인덱스 객체는 프로그램에서 더 이상 사용하지 않는 객체이지만 기능상 접근만 하지 않을 뿐 참조가 남아있기 때문에 가비지 컬렉터는 저 객체를 해제하지 않는다.

이런경우는 pop을 호출할 때마다 메모리 누수가 발생하고 있다고 볼 수 있다.
```java
// 재구현한 pop
   public Object pop() {
       if (size == 0)
           throw new EmptyStackException();
       Object result = elements[--size];
       elements[size] = null; // 다 쓴 참조 해제
       return result;
   }
```
null 처리를 통해 참조를 끊어줘야 제대로 회수가 된다.

스택같은 경우는 자기메모리를 직접 관리하는 클래스이기 때문에 예외적으로 null 처리를 통해 참조를 끊어줬지만 일반적으로 유호 Scope 범위를 이용하는 것으로 객체 참조를 해제해주는게 효율적이다.

함수가 선언됬을 때 생성된 local 객체는 함수가 종료될 때 해제된다.
이 때 local 객체가 참조하고 있던 heap 영역의 객체들은 만약 사라진 local 객체가 유일한 참조자였다면 더 이상 참조가 없기 때문에 접근할 수 없게된다. 이렇게되면 즉시 해제되진 않지만 가비지컬렉터의 회수대상이다. 

```java
    private void run() {
        // 1. 로컬 변수 test가 heap 영역 생성된 Test 객체를 참조
        Test test = new Test();
    }

    public static void main(String[] args) {
        run();
        // 2. run() 이 끝난 시점에서 로컬변수 test는 해제되고 heap 영역의 Test를 참조하고 있는 변수가 없다.
    }
```

이게 유효Scope를 이용한 참조 해제 방법

**정리**

가비지 컬렉터는 마법이 아니다.
아무생각없이 사용하면 누수가 발생할 수 있고 이걸 방지하기 위해서는
Heap, Stack, Method 등 메모리 구조에 대해서 알고 있어야하며
JVM, gabage Collector 동작원리를 알고 있는게 중요하겠다.


## Item 8. finalizer와 cleaner 사용을 피하라 

* finalizer

    java9에서 부터 권장하지 않는다.
    
    가비지 컬렉터가 동작할 때 실행할 로직을 정의할 수 있다.

    C++의 소멸자 처럼 객체가 사라질 때 동작한다고 생각할 수 도 있는데 엄연히 다르다

    가비지 컬렉터의 동작 시점은 예측할 수 없기 때문에

    중요한 작업은 finalizer 에게 맞기면 안된다.

    가비지 컬렉터를 강제로 실행하는 System.gc 를 사용하면 실행 가능성을 높여주긴 하지만 보장해주지는 않기 때문에 사용하지 않는것이 좋다.

* cleaner
    
    java9 부터 finalizer를 대신해 도입됐다.

    finalizer와는 다르게 GC가 메모리 정리를 수행하기 전에 실행된다. 객체가 메모리에서 정리되기 전에 외부 자원을 해제

    실행 중 예외가 발생해도 메모리 누수가 발생하지 않는다.

    그러나 성능 저하 가능성과 cleaner에 실행 시점을 개발자가 예측할 수 없다는 단점만은 동일하다.

**정리**

Item 8은 원래도 사용해 본 적 없는 것들을 사용 하지말라는 내용이어서 가볍게 읽어보고 넘어갔다. 



## Item 9. try-finally대신 try-with=resources를 사용하라

try-catch-finally 에서 try 블록에서 예외가 발생하는 경우에만 catch블록이 실행됐다면 finally 블록은 try 실행결과와 상관없이 반드시 실행된다.

해당 특성 덕분에 메서드를 호출후 반드시 닫아줘야하는 자원을 사용할 때 유용하게 사용된다.

```java
BufferedReader br = new BufferedReader(new FileReader(path));
//readLine 호출 후 close가 보장됨
try {
    return br.readLine();
} finally {
    br.close();
}
```

단, 자원이 많아지면 관리가 힘들어진다.
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
두 번의 try를 중첩해서 사용한 케이스인데 코드 블록이 가독성이 좋지 않고 에러 추적이 좋지 않다.
두번째 try의 read와 finally의 close가 연속으로 실패한다면 read에서 발생한 에러가 덮어씌워져서 발생시점을 확인할 수 없게 된다. (코드로 해결할 수는 있지만 코드가 지저분해짐)

이러한 문제를 해결하기 위해 try-with-resources가 나왔다.

1. try 블록에서 리소스 객체를 선언한다.
2. try 블록이 끝나면, 예외 발생 여부와 상관없이 리소스 객체가 자동으로 close 된다.

위 구조를 사용하려면 자원이 AutoCloseable 인터페이스를 구현해야 한다.

AutoCloseable 은 void를 리턴하는 close 메서드 하나만 정의한 인터페이스이다.
```java
// AutoCloseable 
public interface AutoCloseable {
    void close() throws Exception;
}

// 닫아줘야하는 자원은 이렇게 처리해줘야 try-with-resources 구조를 사용할 수 있다.
public class Resource implements AutoCloseable{
    @Override
    public void close() throws Exception {
        throw new Exception("inside Resource exception");
    }
}
```

try-finally copy 예제 코드를 try-with-resources 로 구현
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream   in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```
코드가 간결해졌고 예외가 양쪽에서 발생해도 다 확인할 수 있다.

catch절 도 함께 사용이 가능해서 try 중첩없이 다수의 예외 처리 가능하다. 

**정리**

반드시 사용 후 닫아야 하는 자원을 다룰때는 try-finally 대신 try-with-resources를 사용하자
