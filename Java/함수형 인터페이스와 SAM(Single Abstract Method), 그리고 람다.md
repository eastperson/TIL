# 함수형 인터페이스(Functional Interface)

- 추상 메서드를 딱 하나만 가지고 있는 인터페이스
- SAM(Single Abstract Method) 인터페이스
- [@FuncationaInterface](https://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html) 애노테이션을 가지고 있는 인터페이스
    - Java가 직접 제공하는 Java Standard Library안에 있다.
    - 이 애너테이션이 달려있으면 2개 이상의 추상 메서드를 작성할 수 없다.
    - 함수형 인터페이스를 정의할 목적이면 애너테이션을 꼭 달아주자!
    

```kotlin
package com.example;

public interface RunSomething {

    void doIt();

    static void printName(){
        System.out.println("Dongin");;
    }

    default void pringAge(){
        System.out.println("29");
    }
}
```

하지만 위의 인터페이스는 함수형 인터페이스가 될 수 있을까? 될 수 있다. 추상 메서드가 1개 있기 때문이다. 이러한 인터페이스 위에는 @FunctionalInterface 애너테이션을 달아준다.

**람다 표현식(Lambda Expressions)**

- 함수형 인터페이스의 인스턴스를 만드는 방법으로 쓰일 수 있다.
- 코드를 줄일 수 있다.
- 메소드 매개변수, 리턴 타입, 변수로 만들어 사용할 수도 있다.

함수형 인터페이스를 사용한 코드(익명 클래스 사용)

```kotlin
public class Foo {
    public static void main(String[] args) {
        RunSomething runSomething = new RunSomething() {
            @Override
            public void doIt() {
                System.out.println("Hello Wolrd");
            }
        };
    }
}
```

람다 코드

```kotlin
public class Foo {
    public static void main(String[] args) {
        RunSomething runSomething = () -> System.out.println("Hello Wolrd");
    }
}
```

이렇게 사용을 할 수 있다. 코드도 줄고 직관적이다.

**자바에서 함수형 프로그래밍**

- 함수를 First class object로 사용할 수 있다.
- 순수 함수(Pure function)
    - 사이드 이펙트가 없다. (함수 밖에 있는 값을 변경하지 않는다.)
    - 상태가 없다.(함수 밖에 있는 값을 사용하지 않는다.)
- 고차 함수(Higher-Order Function)
    - 함수가 함수를 매개변수로 받을 수 있고 함수를 리턴할 수도 있다.
- 불변성

함수는 자바에서 특수한 형태의 Object이다. 따라서 반환을 할 수 있다.

```java
public class Foo {
    public static void main(String[] args) {
        RunSomething runSomething = () -> System.out.println("Hello Wolrd");
        runSomething.doIt();
    }
}
```

순수 함수로서 입력값이 동일한 경우 결과값이 같아야 하는 룰이 있다. 따라서 입력값이 똑같으면 꼭 결과값이 같아야한다. 이것을 보장해주지 못한다면 함수형 프로그래밍이라고 부르기 어렵다.

# 자바에서 제공하는 함수형 인터페이스

Java가 기본으로 제공하는 함수형 인터페이스

- [java.lang.funcation 패키지](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
- 자바에서 미리 정의해둔 자주 사용할만한 함수 인터페이스
- `Function<T,R>`
- `BiFunction<T,U,R>`
- `Consumer<T>`
- `Supplier<T>`
- `Predicate<T>`
- `UnaryOperator<T>`
- `BinaryOperator<T>`

```java
public class Foo {
    public static void main(String[] args) {
        Function<Integer,Integer> plus10 = number -> number + 10;
        System.out.println(plus10.apply(1));
    }
}
```

함수형 인터페이스는 기본적으로 이렇게 사용한다. 또한 default 메서드를 이용해서 함수를 조합해보자.

```java
public class Foo {
    public static void main(String[] args) {
        Function<Integer,Integer> plus10 = number -> number + 10;
        Function<Integer,Integer> multiply2 = number -> number * 2;

        // 10을 더하기 전에 2를 곱하겠다.
        // post.compose(primary)
        // 함수형 인터페이스를 매개변수로 사용하는 고차객체
        Function<Integer,Integer> multiplyTwoPlusTen = plus10.compose(multiply2);
        System.out.println(multiplyTwoPlusTen.apply(2));
    }
}
```

**Function**

- T 타입을 받아서 R 타입을 리턴하는 함수 인터페이스
    - R apply(T t)
        - apply는 정의된 함수를 적용
- 함수 조합용 메소드
    - andThen
        - A 함수 이후에 B 함수 적용
    - compose
        - B 함수 이후에 A 함수 적용

**BiFunction**

- 두 개의 값(T, U)를 받아서 R 타입을 리턴하는 함수 인터페이스
    - R apply(T t, U u)

**Consumer**

- T 타입을 받아서 아무값도 리턴하지 않는 함수 인터페이스
    - void Accept(T t)
    - 반환값이 없는 인터페이스
- 함수 조합용 메소드
    - andThen

```java
public class Foo {
    public static void main(String[] args) {
        Consumer<Integer> printT = (i) -> System.out.println(i);
        printT.accept(10);
    }
}
```

Supplier

- T 타입의 값을 제공하는 함수 인터페이스
    - T get()
    - 반환값이 정해져있는 인터페이스

```java
public class Foo {
    public static void main(String[] args) {
        Supplier<Integer> ten = () -> 10;
        System.out.println(ten);
    }
}
```

Predicate

- T 타입을 받아서 boolean을 리턴하는 함수 인터페이스
    - boolean test(T t)
- 함수 조합용 메소드
    - And
    - Or
    - Negate

```java
public class Foo {
    public static void main(String[] args) {
        Predicate<String> startWithKeesun = s -> s.startsWith("keesun");
        Predicate<String> isLengthUnder10 = s -> s.length() < 10;
        boolean answer1 = startWithKeesun.and(isLengthUnder10).test("keesun");
        boolean answer2 = startWithKeesun.and(isLengthUnder10).test("keesunkedsdsesun");
        System.out.println(answer1);
        System.out.println(answer2);
    }
}
```

UnaryOperator

- Function의 특수한 형태로, 입력값 하나를 받아서 동일한 타입을 리턴하는 함수 인터페이스

```java
public class Foo {
    public static void main(String[] args) {
        UnaryOperator<Integer> plus10 = i -> i+10;
        System.out.println(plus10.apply(1));
    }
}
```

BinaryOperator

- BiFunction의 특수한 형태로, 동일한 타입의 입렵값 두개를 받아 리턴하는 함수 인터페이스

```java
public class Foo {
    public static void main(String[] args) {
        BinaryOperator<Integer> plus10 = (a,b) -> a+b+10;
        System.out.println(plus10.apply(1,2));
    }
}
```

# 람다 표현식

**람다**

- (인자 리스트) -> {바디}

**인자 리스트**

- 인자가 없을 때 : ()
- 인자가 한개일 때 : (one) 또는 one
- 인자가 여러개 일 때 : (one,two)
- 인자의 타입은 생략 가능, 컴파일러가 추론(infer)하지만 명시할 수도 있다. (Integer one, Integer two)

**바디**

- 화살표 오른쪽에 함수 본문을 정의한다.
- 여러 줄인 경우에 {}를 사용해서 묶는다.
- 한 줄인 경우에 생략 가능, return도 생략 가능.

**변수 캡쳐(Variable Capture)**

- 로컬 변수 캡쳐
    - final이거나 effective final인 경우에만 참조할 수 있다.
    - 그렇지 않을 경우 concurrency 문제가 생길 수 있어서 컴파일러가 방지한다.
- effective final
    - 이것도 역시 자바 9부터 지원하는 기능으로 '사실상' final인 변수
    - final 키워드 사용하지 않은 변수를 익명 클래스 구현체 또는 람다에서 참조할 수 있다.
- 익명 클래스 구현체와 달리 '쉐도잉'하지 않는다.
    - 익명 클래스는 새로 스콥을 만들지만, 람다는 람다를 감싸고 있는 스콥과 같다.

![image](https://user-images.githubusercontent.com/66561524/193818845-f8e41f2d-2bd8-4105-885c-71a2abb135c7.png)

```java
public class Foo {
    public static void main(String[] args) {
        Foo foo = new Foo();
        foo.run();
    }

    private void run() {
        final int baseNumber = 10;

        // final로 선언되면 변경을 못한다.
        //baseNumber++;

        // final 키워드는 없어도 변수를 변경하지 않는 경우를 '사실상(effective) final'이라고 한다.
        // effective final은 로컬,익명, 클래스, 람다에서 참조가 가능하다.

        // 로컬 클래스
        class LocalClass{
            void locaPrintInt() {
                int baseNumber = 11;
                System.out.println(baseNumber); // 11
            };
        }

        // 익명 클래스
        Consumer<Integer> integerConsumer = new Consumer<Integer>() {
            @Override
            public void accept(Integer baseNumber) {
                System.out.println(baseNumber); // 11
            }
        };

        // 람다
        IntConsumer printInt = (Integer baseNumber) -> {
            System.out.println(baseNumber); // 컴파일 에러
        };

        printInt.accept(10);
    }
}
```

로컬클래스와 익명 클래스는 클래스 안에 내부의 스콥이 생성되지만, 람다는 메서드와 같은 스콥을 사용한다. 따라서 변수를 재정의 할 수 없다.

참고

- [https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html#shadowing](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html#shadowing)
- [https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)

# 메소드 레퍼런스

람다가 하는 일이 기존 메소드 또는 생성자를 호출하는 거라면, 메소드 레퍼런스를 사용해서 매우 간결하게 표현할 수 있다.

메소드 참조하는 방법

![image](https://user-images.githubusercontent.com/66561524/193818891-c68eba1c-d8cb-4521-a58e-9046a1944e73.png)

- 메소드 또는 생성자의 매개변수로 람다의 입력값을 받는다.
- 리턴값 또는 생성한 객체는 람다의 리턴값이다.

```java
package com.example;

public class Greeting {

    private String name;

    public Greeting(){

    }
    public Greeting(String name) {
        this.name = name;
    }

    public String hello(String name) {
        return "hello " + name;
    }

    public static String hi(String name) {
        return "hi " + name;
    }

}
```

static 메서드 참조

```java
public class Foo {
    public static void main(String[] args) {
        UnaryOperator<String> hi = (s) -> "hi " + s;
        UnaryOperator<String> hi2 = Greeting::hi;

        System.out.println(hi.apply("ep"));
        System.out.println(hi2.apply("ep2"));
    }
}
```

인스턴스 메서드 참조

```java
public class Foo {
    public static void main(String[] args) {
        Greeting greeting = new Greeting();
        UnaryOperator<String> hello = greeting::hello;
    }
}
```

생성자 참조

```java
public class Foo {
    public static void main(String[] args) {
        Supplier<Greeting> greetingSupplier = Greeting::new;
        Greeting greeting = greetingSupplier.get();
    }
}
```

```java
public class Foo {
    public static void main(String[] args) {
        Function<String,Greeting> epGreeting = Greeting::new;
        Supplier<Greeting> newGreeting = Greeting::new;
        System.out.println(epGreeting.apply("ep").getName());
        System.out.println(newGreeting.get().getName());
    }
}
```

임의 객체의 인스턴스 메서드 참조(람다 -> 메서드 참조)

```java
public class Foo {
    public static void main(String[] args) {
        String[] names = {"ep","wayne","dongin"};
        Arrays.sort(names, String::compareToIgnoreCase);
        System.out.println(Arrays.toString(names));
    }
}
```

참고

- [https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)
