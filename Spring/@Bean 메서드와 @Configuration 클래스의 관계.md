# @Bean

---

스프링 프레임워크에서 수동으로 빈을 등록하기 위해서는 `@Bean` 어노테이션을 사용해야 합니다. `@Bean` 어노테이션에 작성되어있는 java doc 내용을 확인해보겠습니다.

<aside>
💡 ***`@Bean` Methods in `@Configuration` Classes***

`@Bean` Methods in @Configuration Classes
Typically, `@Bean` methods are declared within `@Configuration` classes. In this case, bean methods may reference other `@Bean` methods in the same class by calling them directly. This ensures that references between beans are strongly typed and navigable. Such so-called 'inter-bean references' are guaranteed to respect scoping and AOP semantics, just like getBean() lookups would. These are the semantics known from the original 'Spring JavaConfig' project which require CGLIB subclassing of each such configuration class at runtime. As a consequence, `@Configuration` classes and their factory methods must not be marked as final or private in this mode. For example:

```java
@Configuration
public class AppConfig {
   @Bean
   public FooService fooService() {
       return new FooService(fooRepository());
   }

   @Bean
   public FooRepository fooRepository() {
       return new JdbcFooRepository(dataSource());
   }

   // ...
}
```

</aside>

내용을 보면 `@Bean`으로 등록하는 객체는 일반적으로 `@Configuration` 이 선언되어있는 클래스안에서 선언한다고 나와있습니다. 이 경우 Bean이 선언되어있는 메서드는 동일한 클래스에서 다른 Bean 메서드를 직접 호출하여 참조할 수 있습니다. 이렇게 ‘inter-bean references’ 불리는 빈이 빈을 참조하는 경우 AOP semantics와 respect scoping으로 보장됩니다. 이는 CGLIB의 프록시 패턴으로 적용이 되므로 `@Configuration` 클래스와 팩토리 메서드는 final 혹은 private으로 선언되서는 안됩니다.

<aside>
💡 ***`@Bean` Lite Mode***

`@Bean` methods may also be declared within classes that are not annotated with `@Configuration`. For example, bean methods may be declared in a `@Component` class or even in a plain old class. In such cases, a `@Bean` method will get processed in a so-called 'lite' mode.

Bean methods in lite mode will be treated as plain factory methods by the container (similar to factory-method declarations in XML), with scoping and lifecycle callbacks properly applied. The containing class remains unmodified in this case, and there are no unusual constraints for the containing class or the factory methods.
In contrast to the semantics for bean methods in `@Configuration` classes, 'inter-bean references' are not supported in lite mode. Instead, when one @Bean-method invokes another @Bean-method in lite mode, the invocation is a standard Java method invocation; Spring does not intercept the invocation via a CGLIB proxy. This is analogous to inter-`@Transactional` method calls where in proxy mode, Spring does not intercept the invocation — Spring does so only in AspectJ mode.

For example:

```java
@Component
public class Calculator {

	public int sum(int a, int b) {
		return a+b;
	}

   @Bean
   public MyBean myBean() {
       return new MyBean();
   }
}
```

</aside>

`@Configuration` 외부에서 `@Bean` 메서드로 등록하는 경우는 라이트(lite) 모드라고 불립니다. 라이트 모드의 빈 메서드는 일반 팩토리 메서드로 취급되며 스코핑 및 라이프사이클 콜백이 적절하게 적용됩니다.  `@Configuration` 클래스와 달리 라이트 모드에서는 빈 간 참조가 지원되지 않습니다. 하나의 `@Bean` 메서드가 라이트모드에서 다른 `@Bean` 메서드를 호출할 때 Java 메서드 호출일 뿐이며 CGLIB 프록시를 통해 호출을 가로채지 않습니다. 

# @Configuration이 @Bean을 관리하는 방식

---

```java
public class ConfigurationTest {
    @Test
    void configuration() {
        // configuration 클래스의 특징은 bean라고 붙은 메서드를 가지고 있고 각각의 메서드들이 빈을 생성하는 역할을 한다.
    }

    // 스프링 빈은 모두 싱글톤 레지스트리다. 따라서 의존하고 있는 객체는 동일한 객체여야 한다.
    // Bean1 <- Common
    // Bean2 <- Common
    @Configuration
    static class MyConfig {
        @Bean
        Common common() {
            return new Common();
        }
        
        @Bean
        Bean1 bean1() {
            return new Bean1(common());
        }
        
        @Bean
        Bean2 bean2() {
            return new Bean2(common());
        }
    }
    
    static class Bean1 {
        private final Common common;
        
        Bean1(Common common) {
            this.common = common;
        }
    }
    
    static class Bean2 {
        private final Common common;
        
        Bean2(Common common) {
            this.common = common;
        }
    }
    
    static class Common {
        
    }
}
```

이렇게 선언된 코드가 있다. `Bean1`과 `Bean2` 모두 `Common`을 생성자로 받는 빈 객체이다. `MyConfig`는 이 두개의 빈 객체를 설정정보로 등록하는 역할을 하고 있다. 만일 스프링에 의존하지 않고 MyConfig의 두 빈을 등록하면 다음과 같은 결과가 나온다.

```java
@Test
void configuration() {
    Common common = new Common();
    assertThat(common).isSameAs(common);

    MyConfig myConfig = new MyConfig();
    Bean1 bean1 = myConfig.bean1();
    Bean2 bean2 = myConfig.bean2();

    assertThat(bean1.common).isSameAs(bean2.common); // error
}
```

두 빈 객체는 동일하지 않는 객체이다. 이유는 두 빈 객체의 생성시점에 Common 객체도 생성해주기 때문이다. 그런데 놀랍게도 스프링 빈으로 등록하고 비교하면 동일한 객체로 나온다.

```java
@Test
void configuration() {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext();
    ac.register(MyConfig.class);
    ac.refresh();

    Bean1 bean1 = ac.getBean(Bean1.class);
    Bean2 bean2 = ac.getBean(Bean2.class);

    assertThat(bean1.common).isSameAs(bean2.common); // true
}
```

직관적으로 개발자가 기대하는 흐름과 다르게 나온다. 이유는 `@Configuration`으로 빈을 등록할 때는 직접 빈을 등록하는 것이 아니라 프록시 객체를 빈으로 등록하기 때문이다. 구현 방식의 예시는 다음과 같다.

```java
static class MyConfigProxy extends MyConfig {
        
    private Common common;
    
    @Override
    Common common() {
        if (this.common == null) {
            this.common = super.common();
        }
        return this.common;
    }
}
```

```java
@Test
void proxyCommonMethod() {
    MyConfigProxy myConfigProxy = new MyConfigProxy();
    Bean1 bean1 = myConfigProxy.bean1();
    Bean2 bean2 = myConfigProxy.bean2();
    assertThat(bean1.common).isSameAs(bean2.common); // true
}
```

위의 방식과 같이 proxyObject를 통해서 생성하는 빈 객체를 하나로만 만들어서 등록할 수 있다. 만약 이 방식을 사용하고 싶지 않으면 어떻게 해야할까? 스프링 `@Configuration`을 사용할 때 `proxyBeanMethods` 설정을 바꿔주면 된다. 

```java
public @interface Configuration {

.
.
.
	 * Specify whether {@code @Bean} methods should get proxied in order to enforce
	 * bean lifecycle behavior, e.g. to return shared singleton bean instances even
	 * in case of direct {@code @Bean} method calls in user code. This feature
	 * requires method interception, implemented through a runtime-generated CGLIB
	 * subclass which comes with limitations such as the configuration class and
	 * its methods not being allowed to declare {@code final}.
	 * <p>The default is {@code true}, allowing for 'inter-bean references' via direct
	 * method calls within the configuration class as well as for external calls to
	 * this configuration's {@code @Bean} methods, e.g. from another configuration class.
	 * If this is not needed since each of this particular configuration's {@code @Bean}
	 * methods is self-contained and designed as a plain factory method for container use,
	 * switch this flag to {@code false} in order to avoid CGLIB subclass processing.
	 * <p>Turning off bean method interception effectively processes {@code @Bean}
	 * methods individually like when declared on non-{@code @Configuration} classes,
	 * a.k.a. "@Bean Lite Mode" (see {@link Bean @Bean's javadoc}). It is therefore
	 * behaviorally equivalent to removing the {@code @Configuration} stereotype.
	 * @since 5.2
	 */
	boolean proxyBeanMethods() default true;
```

proxyBeanMethods는 디폴트로 true는 proxy 객체를 생성해서 만든다. 이 경우 `@Configuration`이 붙은 클래스는CGLib을 이용해서 프록시 클래스로 확장을 해서 `@Bean`이 붙은 메소드의 동작 방식을 변경한다. `@Bean` 메소드를 직접 호출해서 다른 빈의 의존 관계를 설정할 때 여러번 호출되더라도 싱
글톤 빈처럼 참조할 수 있도록 매번 같은 오브젝트를 리턴하게 한다.

만약 `@Bean` 메소드 직접 호출로 빈 의존관계 주입을 하지 않는다면 굳이 복잡한 프록시 생성
을 할 필요가 없다. 이 경우 proxyBeanMethods를 false로 지정해서도 된다. `@Bean` 메소드는
평범한 팩토리 메소드처럼 동작한다.

proxyBeanMethods는 스프링 5.2 버전부터 지원되기 시작했고 지금은 스프링과 스프링 부트
의 상당히 많은 `@Configuration` 클래스 설정에 적용되고 있다. `@Bean`은 `@Configuration`이 아닌 빈 클래스에서도 사용될 수 있다. 이를 `@Bean`라이트 모드(lite mode)라고 부른다. 빈으로 등록되는 단순 팩토리 메소드로 사용된다.

스프링이 @Configuration을 따로 만든 이유는 CGLib으로 프록시 패턴을 적용해 수동으로 등록하는 스프링 빈이 반드시 싱글톤으로 생성됨을 보장하기 위해서이다.

[토비의 스프링 부트 - 이해와 원리 - 인프런 | 강의](https://www.inflearn.com/course/토비-스프링부트-이해와원리)

[[Spring] @Configuration 안에 @Bean을 사용해야 하는 이유, proxyBeanMethods - (2/2)](https://mangkyu.tistory.com/234)

[[Spring] Spring의 AOP 프록시 구현 방법(JDK 동적 프록시,  CGLib 프록시)과 @EnableAspectJAutoProxy의 proxyTargetClass - (3/3)](https://mangkyu.tistory.com/175)
