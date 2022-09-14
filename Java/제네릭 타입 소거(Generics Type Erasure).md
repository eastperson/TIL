# 제네릭 타입 소거(Generics Type Erasure)

제네릭 타입은 자바와의 하위 호환성을 위해 제네릭 타입을 런타임시 소거를 합니다. 이유는 오직 이전 버전의 Java와의 호환성을 유지하기 위해서입니다. 제네릭 타입은 바이트코드로 컴파일 될 때 코드상에 존재하지 않습니다.

[오라클 공식 문서](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html)

> Generics were introduced to the Java language to provide tighter type checks at compile time and to support generic programming. To implement generics, the Java compiler applies type erasure to: Replace all type parameters in generic types with their bounds or Object if the type parameters are unbounded. The produced bytecode, therefore, contains only ordinary classes, interfaces, and methods.

제네릭은 컴파일 타임에만 제약을 가하고 런타임시 타입에 대한 정보를 버립니다. 정보를 버린다는 것은 아래와 같이 타입이 치환이 되는 것입니다.

- bounded type → bond type
    
    바운디드 타입
    
    `T = <T extends Cat>`

Cat으로 변환
    
- unbounded type → Object
    
    언바운디드 타입
    
    `T = <T super Cat>`

Object로 변환

제네릭 타입은 JVM에서 컴파일시 아래와 같은 절차를 수행합니다.

> 1. Replace generic types with objects
> 2. Replace bounded types (More on these in a later question) with the first bound class
> 3. Insert the equivalent of casts when retrieving generic objects.

따라서 우리가 작성한 제네릭 타입 인자는 바이트 코드로 컴파일 된 이후에는 제네릭 타입이 제거되고 특정 타입으로 치환됩니다. 따라서 우리가 컴파일 전에 작성한 코드와 다를 수 있으며 런타임시 사용할 수 있는 코드를 작성할 수 없는 것입니다.

런타임 시점에 해당 타입 인자가 어떤 타입인지 검사할 수 없다는 단점이 있습니다. 하지만 저장해야 하는 타입 정보의 크기가 줄어들어서 전반적인 메모리 사용량이 줄어든다는 장점도 있습니다.

### Java에서 런타임시 제네릭 타입을 사용하는 방법

자바에서 런타임시 제네릭 타입을 사용하려면 아래와 같이 사용합니다.

```java
public class CatCage implements Cage<Cat>
```

리플렉션을 사용해서 우리는 타입 인자를 얻어낼 수 있습니다.

```java
(Class<T>) ((ParameterizedType) getClass()
  .getGenericSuperclass()).getActualTypeArguments()[0];
```

하지만 역시 [권장하는 방법이 아닙니다.](https://www.baeldung.com/java-generics-interview-questions#q14-are-there-any-situations-where-generic-type-information-is-available-at-runtime)


# Reference

[Generic Type Erasure는 어떻게 타입캐스팅을 하는건가요?](https://woodcock.tistory.com/37)
