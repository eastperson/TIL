# 플라이 웨이트 패턴(Flyweight Pattern)

플라이웨이트 패턴은 비용이 큰 자원을 공통으로 사용할 수 있도록 만드는 패턴이다. 자원에 대한 비용은 크게 두가지로 나눠 볼 수 있다.

1. 중복 생성될 가능성이 높은 경우
    - 중복 생성될 가능성이 높다는 것은 동일한 자원이 자주 사용될 가능성이 매우 높다는 것을 의미한다. 이런 자원은 공통 자원 형태로 관리해주는 편이 좋다.
2. 자원 생성 비용은 큰데 사용 빈도가 낮은 경우
    - 이런 자원을 항상 미리 생성해 두는 것은 낭비이다. 따라서 요청이 있을 때에 생성해서 제공해주는 편이 좋다.

이 두가지 목적을 위해서 플라이웨이트 패턴은 자원 생성과 제공을 책임진다. 자원의 생성을 담당하는 Factory 역할과 관리 역할을 분리하는 것이 좋을 수 있으나, 일반적으로는 두 역할의 크기가 그리 크지 않아서 하나의 클래스가 담당하도록 구현한다.

게임상에서 나무를 표현할 때 기본적인 위치정보와 함께 나무의 3차원 구조(mesh), 껍질(bark), 잎사귀(leaves)들의 요소를 가지고 구성할 수 있다. 그럴 때 아래처럼 각각의 나무에 대해 mesh, bark, leaves, 위치정보에 대한 객체를 각각 모두 가지고 있다고 표현을 할 수 있다. 이 경우 메모리가 많이 소비된다.

![image](https://user-images.githubusercontent.com/66561524/189467084-6d86d4c0-67c0-4993-b34a-c6ddc00ec1b0.png)

클래스로 나타내면 다음과 같다. Tree 클래스로 만들어지는 객체는 mesh, bark, leaves 객체를 각각 다 가지게 된다.

![image](https://user-images.githubusercontent.com/66561524/189467095-7f9c8d71-539f-431c-a35d-1776b439e4bd.png)
앞의 방식에서 아래처럼 Mesh, Bark, Leaves 객체를 공유하는 방식으로 객체를 다시 구성할 수 있다.

![image](https://user-images.githubusercontent.com/66561524/189467107-9ceeb0c8-9faa-4efb-aa1c-3e5e206a8c99.png)
클래스로 나타내면 다음과 같다. Tree 클래스로 만들어지는 객체는 mesh, bark, leaves 객체를 직접 만들지 않고 미리 만들어진 Tree Model을 사용하게 된다.

![image](https://user-images.githubusercontent.com/66561524/189467114-0e825b1e-e3cd-4eed-8d06-b9dd049d5dee.png)
static 변수로 선언된 객체를 하나 만들어 모든 Tree 객체의 내부에서 사용되는 TreeModel에서 공유한다.

![image](https://user-images.githubusercontent.com/66561524/189467125-448de2d5-78a1-4e5a-90f6-019506c3f494.png)
position, height, thickness는 각각 다른 값이 입력이 되지만, TreeModel은 처음 Factory가 생성될 때 딱 1번만 생성이 된다.

앞의 방식에서 아래처럼 Mesh, Bark, Leaves 객체를 공유하는 방식으로 객체를 다시 구성할 수 있다.

![image](https://user-images.githubusercontent.com/66561524/189467133-cdb0002f-1fee-44e4-b82f-a58db6556f60.png)

**장점**

- 많은 객체를 만들 때 성능을 향상시킬 수 있다.
- 많은 객체를 만들 때 메모리를 줄일 수 있다.
- state pattern과 쉽게 결합될 수 있다.

**단점**

- 특정 인스턴스의 공유 컴포넌트를 다르게 행동하게 하는 것이 불가능하다. (개별 설정이 불가능하다.)

**고전적인 사용 예**

- 워드 프로세서(word processor)에서 문자들의 그래픽적 표현에 대한 자료구조(Designe Patterns: Elements of Reusable Object-Oriented Software 책에 따르면, 플라이웨이트 패턴은 1990년에 Paul Calder와 Mark Linton이 WYSIWYG 문서 편집기의 글자모양 정보를 효율적으로 다루기 위해 처음 도입되고 널리 연구되어졌다.)

**JDK 예제**

- java.lang.String
- java.lang.Integer#valueOf(int)
- java.lang.Boolean#valueOf(boolean)
- java.lang.Byte#valueOf(byte)
- java.lang.Character#valueOf(char)

얕은 복사도 Fly weight Pattern이 적용된 예시이다.

예제)

```java
public class FlyweightFactory {
    private static Map<String,Subject> map = new HashMap<>();

    public Subject getSubject(String key) {
        Subject subject = map.get(key);
        if(subject == null){
            subject = new Subject(key);
            map.put(key,subject);

            System.out.println("새로 생성 " + key);
        }else {
            System.out.println("재사용 " + key);
        }
        return subject;
    }
}
```

```java
public class Subject {
    private String name;

    public Subject(String name) {
        this.name = name;
    }
}
```

```java
public class Application {
    public static void main(String[] args) {

        FlyweightFactory flyweightFactory = new FlyweightFactory();
        flyweightFactory.getSubject("Good");
        flyweightFactory.getSubject("Good");
        flyweightFactory.getSubject("ABC");
        flyweightFactory.getSubject("BBB");
        flyweightFactory.getSubject("BBB");

    }
}
```

# Reference
- [디자인 패턴 with JAVA (GoF)](https://www.inflearn.com/course/Design-pattern-java#curriculum)
