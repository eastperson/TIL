함수를 호출할 때 파라미터를 전달하는 방식은 두 가지가 있다.

# call by value

> 값에 의한 호출

- 함수가 호출될 때 메모리 공간 안에서는 함수를 위한 별도의 임시공간이 생성된다. 함수 종료시 해당 공간은 사라진다.
- call by value 호출 방식은 함수 호출시 전달되는 변수 값을 복사해서 함수 인자로 전달한다. 이 때 복사된 인자는 함수 안에서 지역적으로 사용되기 때문에 local value 속성을 가진다.
- 따라서 함수 안에서 인자 값이 변경되더라도 외부 변수 값은 변경이 안된다.

# call by reference

> 참조에 의한 호출

- call by reference 호출 방식은 함수 호출 시 인자로 전달되는 변수의 레퍼런스를 전달한다.
- 따라서 함수 안에서 인자 값이 변경되면, 아규먼트로 전달된 객체의 값도 변경된다.


# Java에는 call by value만 있다.

메서드로 객체를 넘기고 값을 수정해본 경험이 있을 것이다. 그래서 call by reference라고 오해할 수 있지만 java는 오직 call by value로만 동작한다.


## JVM 메모리에 변수가 저장되는 위치

java에서 변수를 선언하면 stack 영역에 할당된다. 원시 타입(primitive type)은 stack 영역에 변수와 함께 저장되며 참조 타입(reference type) 객체는 heap 영역에 저장되고 stack 영역에 있는 변수가 객체의 주소값을 갖고 있다.

![image](https://user-images.githubusercontent.com/66561524/192078776-9c741e91-2058-4163-8bad-b6a2b73b7e19.png)

원시 타입, 참조 타입을 생성할 때 마다 동일한 방식으로 메모리에 할당된다.

## 원시타입 전달

원시 타입은 Stack 영역에 위치한다.

```java
public class PrimitiveTypeTest {

    @Test
    @DisplayName("Primitive Type 은 Stack 메모리에 저장되어서 변경해도 원본 변수에 영향이 없다")
    void test() {
        int a = 1;
        int b = 2;

        // Before
        assertEquals(a, 1);
        assertEquals(b, 2);

        modify(a, b);

        // After: modify(a, b) 호출 후에도 값이 변하지 않음
        assertEquals(a, 1);
        assertEquals(b, 2);
    }

    private void modify(int a, int b) {
        // 여기 있는 파라미터 a, b 는 이름만 같을 뿐 test() 에 있는 a, b 와 다른 변수
        a = 5;
        b = 10;
    }
}
```

![image](https://user-images.githubusercontent.com/66561524/192078886-938d21d6-54db-4c5c-a8a8-048b720b23f3.png)

원시 타입의 전달은 값만 전달하는 call by value로 동작한다.

## 참조 타입 전달

참조 타입은 변수 자체가 stack 영역에 생성되지만 실제 객체는 heap 영역에 위치한다.
stack에 있는 변수가 heap에 있는 객체를 바라보고 있는 형태다.

```java
class User {
    public int age;

    public User(int age) {
        this.age = age;
    }
}

public class ReferenceTypeTest {

    @Test
    @DisplayName("Reference Type 은 주소값을 넘겨 받아서 같은 객체를 바라본다" +
                 "그래서 변경하면 원본 변수에도 영향이 있다")
    void test() {
        User a = new User(10);
        User b = new User(20);

        // Before
        assertEquals(a.age, 10);
        assertEquals(b.age, 20);

        modify(a, b);

        // After
        assertEquals(a.age, 11);
        assertEquals(b.age, 20);
    }

    private void modify(User a, User b) {
        // a, b 와 이름이 같고 같은 객체를 바라본다.
        // 하지만 test 에 있는 변수와 확실히 다른 변수다.

        // modify 의 a 와 test 의 a 는 같은 객체를 바라봐서 영향이 있음
        a.age++;

        // b 에 새로운 객체를 할당하면 가리키는 객체가 달라지고 원본에는 영향 없음
        b = new User(30);
        b.age++;
    }
}
```

Reference 자체를 전달하는 게 아니라 주소값만 전달해주고 modify() 에서 생긴 변수들이 주소값을 보고 객체를 같이 참조하고 있다.

- 처음 변수 선언시

![image](https://user-images.githubusercontent.com/66561524/192078944-dd6d2afc-1237-4ee4-a961-9d9258ed7a2b.png)

- modify(a,b) 호출 시점

![image](https://user-images.githubusercontent.com/66561524/192078950-71e02e56-23b3-4537-ba1f-d0948cc1ac41.png)

- modify(a, b) 수행 직후

![image](https://user-images.githubusercontent.com/66561524/192078958-fd3aef95-ad8d-43ce-b0a4-73e098af4903.png)

- test() 끝난 후

![image](https://user-images.githubusercontent.com/66561524/192078971-764d2226-f312-45e4-b7e4-2a667eb3eb25.png)

User03은 참조되고 있지 않아 Garbage Collector에 의해 제거

- "결국 주소값을 넘기는 게 결국 call by reference 아닌가?" 라는 생각을 할 수도 있다.
- 하지만 call by reference 는 참조 자체를 넘기기 때문에 새로운 객체를 할당하면 원본 변수도 영향을 받는다.
- 가장 큰 핵심은 호출자 변수와 수신자 파라미터는 Stack 영역 내에서 각각 존재하는 다른 변수인 것이다.


# Java의 call by reference가 있다는 오해

Java는 by reference로 함수를 호출하거나 파라미터를 전달하지 않는다. 제임스 고슬링이 쓴 책에서 call by reference 에 대한 이야기가 나온다.

제임스 고슬링
> "Some people will say incorrectly that objects are passed “by reference.” In programming language design, the term pass by reference properly means that when an argument is passed to a function, the invoked function gets a reference to the original value, not a copy of its value. If the function modifies its parameter, the value in the calling code will be changed because the argument and parameter use the same slot in memory.
> ...
> The Java programming language does not pass objects by reference; it passes object references by value."
> - The Java Programming (David Holmes, Ken Arnold, James Gosling)

- 자바의 "reference"는 "reference value"를 줄인 말이다. 레퍼런스 값(value)이다.
- 자바에서의 reference 용어와 프로그래밍 언어의 'by reference'와 혼동에서 생기는 문제이다.
- 다시 말하지만 자바에서는 call by reference가 없다. call by reference value라고 불릴 수는 있다. 하지만 결국 call by value이다.

![image](https://user-images.githubusercontent.com/66561524/192079504-07bfdc66-f069-463d-a8fb-b8b07e0e78d4.png)

[토비 이사님 post](https://www.facebook.com/tobyilee/posts/10222585502760852) 

# 추가([토비의 자바 채널 문의내용](https://discord.com/channels/687618003717587011/1012262477016354888/1022874204682600459))

![image](https://user-images.githubusercontent.com/66561524/192079363-b429a674-0574-4b54-b872-cd166c1a5c47.png)
![image](https://user-images.githubusercontent.com/66561524/192079369-84c7ddaa-ebeb-4f69-b746-96aea08e21d1.png)
![image](https://user-images.githubusercontent.com/66561524/192079370-366ed77e-e497-44ec-8a7c-14ccc2e8b540.png)
![image](https://user-images.githubusercontent.com/66561524/192079373-2e195d01-c7f0-41a1-866d-19d706215a25.png)
![image](https://user-images.githubusercontent.com/66561524/192079247-e03c3bd7-e8b8-46ad-abb8-2a2b81d8f89e.png)
![image](https://user-images.githubusercontent.com/66561524/192079256-fdb18314-9bcc-4e24-94ca-8b0792bab3cf.png)

# Reference

- [call by value](https://coco-log.tistory.com/197)
- [Java 의 Call by Value, Call by Reference](https://bcp0109.tistory.com/360)
- [Pass-By-Value as a Parameter Passing Mechanism in Java](https://www.baeldung.com/java-pass-by-value-or-pass-by-reference)
