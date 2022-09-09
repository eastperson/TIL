# String Literal과 String Constructor의 차이

![image](https://user-images.githubusercontent.com/66561524/189258100-9b325e22-9718-46bc-8cd4-c50435ac9a16.png)

키워드

- Flyweight 패턴
- String Constant Pool
- JVM Heap
- String equals(), hashcode()

```java
String cat = "cat"
String cat2 = cat;
```

- String Literal로 문자열을 호출하면 String Constant Pool에 해당 문자열을 추가하고 그 참조값(메모리주소)를 반환한다.
- 만약 String Literal을 호출했을 때 Pool에 이미 해당 문자열이 있으면 이전에 추가했던 참조값을 반환한다. (`String.intern()`)
- 그런데 `new String()`으로 문자열을 만들게 되면 String Pool와 별개로 생성이 되어 풀에 추가되지 않고 새로운 객체를 생성하여 참조값을 반환한다.
- 따라서 `StringBuilder`나 `new String()`으로 만든 문자열은 새로운 객체가 된다.

![image](https://user-images.githubusercontent.com/66561524/189258333-0b1d02a5-43d6-4a4f-91e7-65a271421689.png)

또한 String 객체는 내부적으로 `hashcode()`와 `equals()`를 오버라이딩해서 사용한다.

`String`의 `hashcode()` 메모리 주소값을 의미하는게 아니므로 비교하는 것은 의미가 없고 주소값을 비교하기 위해서는 '`==`'를 사용하는 것이 옳다.결론적으로 리터럴 `"cat"`과 `new String("cat")`은 다른 객체이며 처음 스트링 리터럴을 호출하는 것은 객체를 생성한다고 보기 힘들기 때문에 한 번만 객체 생성이 된다.

```java
    @Test
    void test(){
        String cat = "cat";
        String cat2 = new String("cat");
        StringBuilder stringBuilder = new StringBuilder();
        String cat3 = stringBuilder.append("cat").toString();

        assertTrue(cat == cat.intern() && cat == "cat");
        assertTrue(cat != cat2 && cat != cat3);
        assertTrue(cat2 != cat3);
        assertTrue(cat.equals(cat2) && cat.equals(cat3));
    }
```

![image](https://user-images.githubusercontent.com/66561524/189258353-56a39461-4e7b-44e4-ae4a-a6ad6ec95a01.png)

```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

```java
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

# String Constant Pool의 장점
- Java Runtime시 Heap 영역을 절약할 수 있다.
- 불변객체로서 관리가 되기 때문에 Thread-safe하다.
- String의 hashcode를 생성단계에서 캐싱하므로 String의 hashcode를 계산하지 않아도 된다. HashMap을 사용할 때 기능상 이점이 있다.


[[Java] 핵심정리 (1) - String 객체를 중심으로](https://victorydntmd.tistory.com/133)

[자바의 String 객체와 String 리터럴](https://madplay.github.io/post/java-string-literal-vs-string-object)
