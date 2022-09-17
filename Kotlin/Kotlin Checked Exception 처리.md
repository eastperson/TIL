자바에서는 메서드 내 발생할 수 있는 Checked Exception을 throws하면 그 메서드를 호출하는 쪽에서 예외처리를 강제한다. 처리를 하려면 또 그 메서드를 호출하는 쪽으로 throws를 하거나 try-catch 등을 해줘야 한다.

| | |
| ----- | ------ |
| ![image](https://user-images.githubusercontent.com/66561524/190850465-9791b318-2c1c-46c9-8f80-a68fc27092e0.png) | ![image](https://user-images.githubusercontent.com/66561524/190850474-eca383c4-b056-4235-861e-86f491b9a7e2.png) |
하위 메서드에서 throws한 예외는 상위메서드에서 안잡으면 컴파일 에러가 발생한다. | 하위 메서드에서 넘어온 checked 예외는 또 다른 상위 메서드로 넘기거나
| ![image](https://user-images.githubusercontent.com/66561524/190850482-13a7b32e-59a1-4458-8ed3-8f272a1b1613.png) | ![image](https://user-images.githubusercontent.com/66561524/190850489-5178ee74-b3fe-4964-ba72-568ac857195e.png) |
try-catch를 해줘야 한다. | 물론 하위 메서드에서도 throws나 try-catch를 하지 않으면 에러가 발생한다.

하지만 코틀린에서는 Checked Exception 처리를 강제하지 않는다.

![image](https://user-images.githubusercontent.com/66561524/190850501-a0859315-f36b-4044-a853-1f2fc2e40cbe.png)

코틀린은 exception을 자바에게 상속받았음에도 불구하고 자바와 동작을 달리한다. 다른 최신 언어들처럼 코틀린 개발자도 Checked Exception을 포함하지 않기로 했다. (코틀린인액션)

![image](https://user-images.githubusercontent.com/66561524/190850949-c8a9e5a4-2321-4195-a22f-494730149492.png)

[Java's checked exceptions were a mistake (and here's what I would like to do about it)](https://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html)

하지만 반대로 자바와의 상호운용성을 위해 코틀린 메서드에서 발생한 Checked 예외를 thorws해주는 애너테이션을 제공해준다.

`@Throws`

[](https://www.baeldung.com/kotlin/throws-annotation)

- 코틀린에서 발생하는 checked exception을 자바 코드에 throws할 수 있도록 만들어주는 어노테이션

# Reference

[Exceptions | Kotlin](https://kotlinlang.org/docs/exceptions.html#checked-exceptions)
