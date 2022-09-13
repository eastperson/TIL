# Overview

Java 프로그램이 실행되면 JVM은 시스템으로부터 프로그램을 수행하는 필요한 메모리를 할당받고 이 메모리를 용도에 따라 여러 영역을 나누어서 관리합니다.

- Java Compiler : Java 코드를 ByteCode로 변환한다.
- Java API, Class Files : 컴파일 된 자바 클래스 파일
- **Class Loader System** : 컴파일된 자바 클래스 파일을 메모리(Runtime Data Area)에 적재한다
- Execution Engine : ByteCode를 실행 가능 하도록 해석

![image](https://user-images.githubusercontent.com/66561524/190019961-564b9a15-2203-4938-bc61-d59318d2ee3c.png)

JVM의 Run-Time Data Area는 크게 PC Registers, JVM Stacks, Method Area, Heap, Native Method Stacks가 있습니다. 각 영역에 대한 설명은 아래와 같습니다.

![image](https://user-images.githubusercontent.com/66561524/190019991-5e63a844-2f21-4f0c-b19a-69577ef63632.png)

## PC(Program Counter) Registers

- Java에서 Thread는 각자의 메소드를 실행하게 된다.
- 이 때 Thread 별도로 동시에 실행하는 환경이 보장되어야 한다. JVM에서는 명령어 주소값을 저장할 공간이 필요하다.
- Thread 들은 각각 자신만의 PC Registers를 가지고 있다.
- 만약 실행했던 메소드가 네이티브 하다면 undefined가 기록된다. 네이티브하지 않다면 PC Registers는 JVM에서 사용된 명령의 주소값을 저장하게 된다.

## JVM Stacks

- 각 Thread 별로 따로 할당되는 영역이다.
- Heap 메모리 영역보다 비교적 빠르다.
- 각각의 Thread 별로 메모리를 따로 할당하기 때문에 동시성 문제에서 자유롭다.
- Thread 들은 메소드를 호출할 때마다 Frame 이라는 단위를 추가(push)하게 된다.
- 메서드가 마무리 되면 결과를 반환하면 Frame은 Stack으로부터 제거(pop)이 된다.
- Frame은 메서드에 대한 정보를 가지고 있는 Local Variable, Operand Stack 그리고 Constant Pool Reference 로 구성이 되어있다.
    - Local Variable 메서드 안의 지역 변수들을 가지고 있다.
    - Operand Stack은 메서드 내 연산을 위해서 바이트 코드 명령문이 들어있는 공간이다.
    - Constant Pool Reference는 Constant Pool 참조를 위한 공간이다.
- 이렇게 구성된 Java Stack은 메서드가 호출될 때 마다 Frame이 쌓이게 된다.
- Java Stack 영역이 가득 차게 되면 StackOverflowError가 발생된다.

![image](https://user-images.githubusercontent.com/66561524/190020296-e745a7f4-145a-4476-a025-1ce24211bed0.png)

```java
class Person {
    int id;
    String name;

    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }
}

public class PersonBuilder {
    private static Person buildPerson(int id, String name) {
        return new Person(id, name);
    }

    public static void main(String[] args) {
        int id = 23;
        String name = "John";
        Person person = null;
        person = buildPerson(id, name);
    }
}
```
![image](https://user-images.githubusercontent.com/66561524/190020372-f1752257-8e1c-465a-8c8c-e892bdd780a6.png)

- 각 Frame의 연산이 끝나게 되면 결과값을 호출한 상위 Frame에 반환하게 된다.
    - 네모박스가 하나의 Frame이다.

## Method Area

- Method Area에는 인스턴스 생성을 위한 객체 구조, 생성자, 필드 등이 저장된다.
- Runtime Constant Pool과 static 변수 그리고 메소드 데이터와 같은 Class 데이터들도 이곳에서 관린된다.
- 이 영역은 JVM당 하나만 생성이 된다.
- 인스턴스 생성에 필요한 정보도 존재하기 때문에 JVM의 모든 Thread 들이 Method Area를 공유하게 된다.
- JVM의 다른 메모리 영역에서 해당 정보에 대한 요청이 오면 실제 물리 메모리 주소로 변환해서 전달해준다.
- JVM 구동시 생성이 되며 종료시까지 유지되는 공통 영역이다.

## Heap

- 코드 실행을 위한 Java로 구성된 객체 및 JRE 클래스들이 탑재된다.
- 문자열에 대한 정보를 가진 String Pool 뿐만 아니라 실제 데이터를 가진 인스턴스, 배열 등이 저장된다.
- JVM 당 하나만 생성되고 해당 영역이 가진 데이터는 모두 Java Stack 영역에서 참조되어 Thread 간 공유가 된다.
- Heap 영역이 가득 차게 되면 OutOfMemoryError를 발생시킨다.
- Heap 에서는 참조되지 않은 인스턴스와 배열에 대한 정보 또한 얻을 수 있기 때문에 GC의 주 대상이 된다.
- Thread 별로 메모리를 할당받는 Java Stack 영역과 달리 조금 속도가 느리다. 모든 Thread 들이 해당 영역인 Heap을 공유해서 Java의 동시성 문제가 발생한다. Thread에 의해서 공유가 되기 때문에 Thread Safe 하지 않다. 이 때문에 해당 영역에 있는 객체나 인스턴스를 사용하게 되면 synchronized 블록을 사용하는 방법들을 비롯하여 동시성을 지켜주는 방법을 사용한다.

```
public class Heap {
    public static void main(String[] args) {
        System.out.println("Heap 메모리 오류");
        int num = 1;
        List<Integer> nums = new LinkedList<>();
        try {
            while (true) {
                nums.add(num);
                num = num + 1;
                if (num < 1) {
                    break;
                }
            }
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

```
Heap 메모리 오류
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
        at Heap.main(Heap.java:16)
```

- 인스턴스가 생성된 후 시간에 따라 5가지 부분으로 나눌 수 있다.

![image](https://user-images.githubusercontent.com/66561524/190020995-363ebb8e-6558-49b7-9b45-17b592d3f7bf.png)

- Eden, Survivor0, Survivor1, Old , Perm으로 나누어진다.
- Young Gen 이라고 불리는 비교적 신생 데이터 부분은 Eden , Survivor0 , Survivor1이다.
- Eden 에는 new 를 통해 새롭게 생성된 인스턴스가 위치하며, 이후에는 Survivor 로 이동하게 된다.
- 이곳에서도 참조되지 않는 인스턴스와 배열 대상으로 Minor GC가 일어난다.
- GC 가 일어나는 부분은 그 이후의 부분인 Old 부분이다.
- Perm의 경우 클래스의 메타 정보 및 static 변수를 저장하고 있다.
- Java 8 버전 이후로 Native 영역에 존재하는 Metaspace 라는 영역으로 대체됐다.

## Native Method Stacks

- Java로 작성된 프로그램을 실행하면서 순수 Java로 구성된 코드만 사용할 수 없는 시스템의 자원이나 API가 존재한다.
- 다른 프로그래밍 언어로 작성된 메소드들을 Native Method라고 한다. Native Method Stacks는 Java로 작성되지 않은 메서드를 다루는 영역이다.
- Java Stacks 영역과 비슷하게 Native Method가 실행될 경우 Stack에 해당 메서드가 쌓이게 된다.
- 각각의 Thread 들이 생성되면 Native Method Stacks도 동일하게 생성된다.

# Reference

[JVM의 Runtime Data Area](https://www.holaxprogramming.com/2013/07/16/java-jvm-runtime-data-area/)

[JVM에 관하여 - Part 3, Run-Time Data Area](https://tecoble.techcourse.co.kr/post/2021-08-09-jvm-memory/)

[Stack Memory and Heap Space in Java
](https://www.baeldung.com/java-stack-heap)
