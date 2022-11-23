# Kotlin 공식 문서

- 코틀린 1.5 이후 등장
    
    [](https://kotlinlang.org/docs/whatsnew15.html#package-wide-sealed-class-hierarchies)
    
- 대게 우리는 원시 타입의 변수를 비즈니스 로직을 위해 래퍼 클래스로 감싸는 경우가 많다. 그런데 원시타입에 비해 래퍼 클래스는 런타임시 성능 최적화를 하지 못하며 래퍼 클래스를 제공하는 것을 되려 망설일 수 있다. 이 문제를 해결하기 위해 코틀린은 *inline class 라고 불리는 것을 제공한다. inline class 는* ****Value Classes**** 의 부분집합이다. 이들은 식별자 없이 value만 갖고 있다.

```kotlin
value class Password(private val s: String)
```

- 우리는 value 예약어를 클래스 앞에 붙인다.
- 인라인 클래스는 JVM 백엔드를 위해서 `@JvmInline` 어노테이션을 클래스 선언 앞에 사용해준다.
- inline 예약어를 사용하는 것은 deprecated 되었다.
- inline 클래스는 하나의 프로퍼티가 주 생성자로 인해 초기화 되어야 한다. 런타임시간에 인라인 클래스의 인스턴스는 이 단일 프로퍼티를 사용하기 위해 표현될 것입니다.

```kotlin
// No actual instantiation of class 'Password' happens
// At runtime 'securePassword' contains just 'String'
val securePassword = Password("Don't try this in production")
```

- 인라인 클래스의 주된 기능은 인라인 함수와 유사하게 단일 프로퍼티를 사용하는 것과 같이 표현됩니다.
    
    [](https://kotlinlang.org/docs/inline-classes.html#members)
    

### Members

- 인라인 클래스는 일반 클래스의 기능을 지원한다.
- 프로퍼티와 함수를 선언할 수 있으며 init 블록이 있다

```kotlin
@JvmInline
value class Name(val s: String) {
    init {
        require(s.length > 0) { }
    }

    val length: Int
        get() = s.length

    fun greet() {
        println("Hello, $s")
    }
}

fun main() {
    val name = Name("Kotlin")
    name.greet() // method `greet` is called as a static method
    println(name.length) // property getter is called as a static method
}
```

- 인라인 클래스의 프로퍼티들은 backing fields를 가질 수 없다.
- 그들은 오직 간단한 계산가능한 프로퍼티만 가질 수 있다. (no lateinit /delegated properties)

### **Inheritance**

- 인라인 클래스는 인터페이스 상속만 허용된다.

```kotlin
interface Printable {
    fun prettyPrint(): String
}

@JvmInline
value class Name(val s: String) : Printable {
    override fun prettyPrint(): String = "Let's $s!"
}

fun main() {
    val name = Name("Kotlin")
    println(name.prettyPrint()) // Still called as a static method
}
```

- 이것은 인라인 클래스가 클래스 계층에 참여하는 것을 금지한다. 이것은 인라인 클래스들은 다른 클래스로 확장할 수 없으며 언제나 `final` 클래스 임을 의미한다.

### Representation

- 코틀린 컴파일러는 코드를 생성해내어 각 인라인 클래스들에 대해서 wrapper를 유지하게 만듭니다.
- 인라인 클래스는 러타임시에는 wrapper(ex. Integer, Long) 또는 primitive(int, long) 타입으로 표현됩니다.
- 코틀린 컴파일러는 가장 성능이 좋고 최적화된 코드를 만들기 위해 wrapper 대신 primitive 타입을 사용하는 것을 선호한다.
- 하지만 때때로 래퍼로 유지해야하는 경우가 있다. 이의 예시는 아래와 같다.

```kotlin
interface I

@JvmInline
value class Foo(val i: Int) : I

fun asInline(f: Foo) {}
fun <T> asGeneric(x: T) {}
fun asInterface(i: I) {}
fun asNullable(i: Foo?) {}

fun <T> id(x: T): T = x

fun main() {
    val f = Foo(42)

    asInline(f)    // unboxed: used as Foo itself
    asGeneric(f)   // boxed: used as generic type T
    asInterface(f) // boxed: used as type I
    asNullable(f)  // boxed: used as Foo?, which is different from Foo

    // below, 'f' first is boxed (while being passed to 'id') and then unboxed (when returned from 'id')
    // In the end, 'c' contains unboxed representation (just '42'), as 'f'
    val c = id(f)
}
```

- 인라인 클래스는 기본타입과 래퍼로 표현되기 때문에 참조 동등성이 의미가 없다. 즉, `===` 를 사용할 수 없다.
- 인라인 클래스는 또한 generic type parameter를 내부 value(unerlying type)으로 가질 수 있다. 이러한 상황해서 컴파일러는 이것을 일반적으로 Any?로 변환하거나 타입 파라미터의 upper bound로 변환한다.

```kotlin
@JvmInline
value class UserId<T>(val value: T)

fun compute(s: UserId<String>) {} // compiler generates fun compute-<hashcode>(s: Any?)
```

> Generic inline classes is an [Experimental](https://kotlinlang.org/docs/components-stability.html) feature. It may be dropped or changed at any time. Opt-in is required with the `-language-version 1.8`
 compiler option.

[](https://kotlinlang.org/docs/inline-classes.html#representation)

### Mangling

- 인라인 클래스는 그들의 underlying type 으로 컴파일되기 때문에 이것은 잘 알려지지 않는 에러를 만들어낼 수 있다. 예쌍못한 platform signature 충돌 같은 에러 등이 있다.

```kotlin
@JvmInline
value class UInt(val x: Int)

// Represented as 'public final void compute(int x)' on the JVM
fun compute(x: Int) { }

// Also represented as 'public final void compute(int x)' on the JVM!
fun compute(x: UInt) { }
```

- 이런 문제를 방지하기 위해 인라인 클래스는 안정적인 해시코드를 추가해서 maggled 하게 만든다. 따라서 fun compute(x: UInt) 는 `public final void compute-<hashcode>(int x)` 로 represented 하게 되어 충돌 문제를 해결한다.

> The mangling scheme has been changed in Kotlin 1.4.30. Use the `-Xuse-14-inline-classes-mangling-scheme` compiler flag to force the compiler to use the old 1.4.0 mangling scheme and preserve binary compatibility.

### **Calling from Java code**

- 인라인 클래스를 자바코드에서 호출할 수 있다. 이를 위해서는 직접 mangling 사용을 막아야 한다. add@JvmName 어노테이션을 함수 호출전에 명시해줘야 한다.

```kotlin
@JvmInline
value class UInt(val x: Int)

fun compute(x: Int) { }

@JvmName("computeUInt")
fun compute(x: UInt) { }
```

### **Inline classes vs type aliases**

- 인라인 클래스는 type aliases와 매우 유사해 보인다. 실제로 둘 다 새로운 유형을 도입하는 것 같아 보이며 실행시 기본 유형으로 표현이 되보인다.
- 그러나 중요한 차이점은 type aliasese는 그들의 기저타입으로 assignment-compatible되는 반면에 인라인 클래스는 그렇지 않다.
- 인라인 클래스는 물론 새로운 타입을 보여주지만 반대로 type aliases는 오직 이미 존재하는 타입의 대안적인 이름만을 보여준다.

```kotlin
typealias NameTypeAlias = String

@JvmInline
value class NameInlineClass(val s: String)

fun acceptString(s: String) {}
fun acceptNameTypeAlias(n: NameTypeAlias) {}
fun acceptNameInlineClass(p: NameInlineClass) {}

fun main() {
    val nameAlias: NameTypeAlias = ""
    val nameInlineClass: NameInlineClass = NameInlineClass("")
    val string: String = ""

    acceptString(nameAlias) // OK: pass alias instead of underlying type
    acceptString(nameInlineClass) // Not OK: can't pass inline class instead of underlying type

    // And vice versa:
    acceptNameTypeAlias(string) // OK: pass underlying type instead of alias
    acceptNameInlineClass(string) // Not OK: can't pass underlying type instead of inline class
}
```

즉, String을 typealias 하면 String과 같은 타입으로 호환이 가능하다. 하지만 value class를 사용하면 같은 타입으로 호환을 할 수 없다.

### **Inline classes and delegation**

- by delegation의 구현은 인터페이스 계층에서 허용된다.

```kotlin
interface MyInterface {
    fun bar()
    fun foo() = "foo"
}

@JvmInline
value class MyInterfaceWrapper(val myInterface: MyInterface) : MyInterface by myInterface

fun main() {
    val my = MyInterfaceWrapper(object : MyInterface {
        override fun bar() {
            // body
        }
    })
    println(my.foo()) // prints "foo"
}
```

### **Referential equality**

- Referential equality 는 `===` ,`!==`  연산자로 확인할 수 있다. 만약 a, b가 같은 객체이면 ===는 true로 평가받는다. 런타임시에 원시타입을 represented 하기 위해서 === 동일성 확인은 == 동등성확인과 같다.

[](https://kotlinlang.org/docs/equality.html#referential-equality)

# [이펙티브 자바] 아이템47 - 인라인 클래스의 사용을 고려하라

## Overview

---

- 하나의 값을 보유하는 객체도 inline으로 만들 수 있다.
- 코틀린 1.3부터 도입된 기능이다.
- 기본 생성자 프로퍼티가 하나인 클래스 앞에 inline을 붙이면 해당 객체를 사용하는 위치가 모두 해당 프로퍼티로 교체된다.

```kotlin
inline class Name(private val value: String) {
	// ...

	fun greet() {
		print("Hello, I am ${value}")
	}
}
```

```kotlin
// 코드
val name: Name = Name("ep")
name.greet()

// 컴파일 때 다음과 같은 형태로 변경된다.
val name: String = "ep"
Name.'greet-impl'(name)
```

- inline 클래스는 타입만 맞으면 아래와 같이 값을 곧바로 집어도 넣을 수 있다.
- inline 클래스의 메서드는 모두 정적 메서드로 만들어진다.
- 다른 자료형을 래핑해서 새로운 자료형을 만들 때 많이 사용된다.
    - 어떠한 오버헤드도 발생하지 않는다.
        - 아이템 45 참고

인라인 클래스는 다음과 같은 상황에서 많이 사용된다.

1. 측정 단위를 표현할 때
2. 타입 오용으로 발생하는 문제를 막을 때

## 측정 단위를 표현할 때

---

### 문제

```kotlin
interface Timer {
	fun callAfter(time: Int, callback: () -> Unit)
}
```

- 위와 같이 타이머 클래스를 가정했을 때, time 인자로 들어가는 타입은 어떤 단위인지 명확하지 않는다. ms(밀리초), s(초), min(분) 중에서 어떤 단위인지 모른다.

### 이름에 단위 명시

```kotlin
interface Timer {
	fun callAfter(timeMillis: Int, callback: () -> Unit)
}

interface User {
	fun decideAboutTime(): Int
	fun wakeUp()
}

fun setUpUserWakeUpUser(user: User, timer: Timer) {
	val time: Int = user.decideAboutTime()
	timer.callAfter(time) {
		user.wakeUp()
	}
}
```

- **파라미터 이름**에 측정 단위를 붙여서 표현할 수 있다.
    - 하지만 어떤 단위로 리턴해주는지 알려줄 수 없다.
- 또는 **함수에 이름**을 붙여서 어떤 단위로 리턴할지 알려줄 수 있다.
    - 하지만 이러한 해결 방법은 함수를 길게 만들고 필요 없는 정보까지도 전달해줄 수 있다.

### 타입에 제한

```kotlin
inline class Minutes(val minutes: Int) {
	fun toMillis(): Millis = Millis(minutes * 60 * 1000)
}

inline class Millis(val milliseconds: Int) {

}

interface User {
	fun decideAboutTime(): Minues
	fun wakeUp()
}

interface Timer {
	fun callAfter(timeMillis: Millis, callback: () -> Unit)
}

fun setUpUserWakeUpUser(user: User, timer: Timer) {
	val time: Minutes = user.decideAboutTime()
	// compile error 타입이 올바르지 않아 오류가 발생한다. Millis 타입을 인자로 넣어줘야 한다.
	timer.callAfter(time) {
		user.wakeUp()
	}
}
```

- inline class를 사용하면 타입 제한을 걸어줄 수 있다.

## 타입 오용으로 발생하는 문제를 막을 때

- SQL 데이터베이스는 일반적으로 ID를 사용해서 요소를 식별한다.

```kotlin
@Entity(tableName = "grades")
class Grades(
	@ColumnInfo(name = "studentId")
	val studentId: Int,

	@ColumnInfo(name = "teacherId")
	val teacherId: Int,

	@ColumnInfo(name = "schoolId")
	val schoolId: Int
)
```

- Int 자료형이므로 실수로 잘못된 값을 넣을 수 있다.
- 이런 문제를 막기 위해 inline 클래스를 활용할 수 있다.

```kotlin
inline class StudentId(val studentId: Int)
inline class TeacherId(val teacherId: Int)
inline class SchoolId(val schoolId: Int)

@Entity(tableName = "grades")
class Grades(
	@ColumnInfo(name = "studentId")
	val studentId: StudentId,

	@ColumnInfo(name = "teacherId")
	val teacherId: TeacherId,

	@ColumnInfo(name = "schoolId")
	val schoolId: SchoolId
)
```

- ID의 사용이 안전해지며 컴파일할 때 타입이 Int로 대체되므로 코드를 바꿔도 별도의 문제가 발생하지 않는다.

## 인라인 클래스와 인터페이스

- 인라인 클래스도 인터페이스를 구현할 수 있다.
- 하지만 인터페이스로 표현을하면 클래스가 inline으로 동작하지 않는다. 인터페이스를 통해서 타입을 나타내려면 객체를 래핑해서 사용해야 하기 때문이다.
- 따라서 인터페이스를 구현하는 인라인 클래스는 아무런 의미도 장점도 없다.

## typealias

tyoealias를 사용하면 타입에 새로운 이름을 붙여줄 수 있다.

```kotlin
typealias NewName = Int
val n: NewName = 10
```

- typealias는 길고 반복적으로 사용할 떄 많이 유용하다.

```kotlin
typealias ClickListener = (view: View, event: Event) -> Unit

class View {
	fun addClickListener(listener: ClickListener) {}
	fun removeClickListner(listener: ClickListener) {}
}
```

- 하지만 typealias는 안전하지 않다.
    - 타입이 같은 다른 함수로 사용해도 아무런 문제가 없다
- 이름이 명확하게 붙여있어 안전할거라는 착각을 하게 만든다. 따라서 문제가 발생했을 대 찾기가 더 어렵다.
- 단위 등을 표현하려면 파라미터 이름 또는 클래스를 사용해야한다.
    - 이름은 비용이 적게 들고 클래스는 안전하다.
- 인라인 클래스를 사용하면 비용과 안전이라는 두 마리 토끼를 모두 잡을 수 있다.

## 정리

- 인라인 클래스를 사용하면 성능적인 오버헤드 없이 타입을 래핑할 수 있다.
- 인라인 클래스는 타입 시스템을 통해 실수로 코드를 잘못 작성하는 것을 막아주므로 코드의 안정성을 향상시켜 줍니다.
- 의미가 명확하지 않은 타입 특히 여러 측정 단위들을 함께 사용하는 경우에는 인라인 클래스를 꼭 활용하세요.
