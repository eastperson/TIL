# Overview

---

[***ThreadLocal***](https://www.baeldung.com/java-threadlocal)은 오직 한 쓰레드에 의해 읽고 쓰여질 수 있는 쓰레드만의 독립적인 로컬 변수이다. 

보통 프로세스 내부 자원을 쓰레드로 공유하게 되는데 아무처리를 하지 않으면 쓰레드들은 순서없이 프로세스의 내부자원에 접근하게 된다. 여기서 변경이 생기면 그 뒤의 쓰레드들은 변경된 데이터로 작업을 하게 되는데 이 때 생각지 못한 동작을 하게 될 가능성이 많다.

이 점을 해결하기 위해 세마포어와 뮤텍스(이진세마포어)라는 개념이 있지만 공유변수가 아닌 오직 쓰레드만의 로컬 변수를 사용하고 싶을 때는 `ThreadLocal`을 사용하게 된다.

![image](https://user-images.githubusercontent.com/66561524/197363526-bee5ba49-1048-4e97-b04f-05a70fdf38e2.png)

# ThreadLocal 용례

---

`ThreadLocal`의 생성자는 오직 하나의 쓰레드만 접근할 수 있도록 허용되어 있다. 쓰레드와 관련된 코드에서 파라미터를 사용하지 않고 객체를 전파하기 위한 용도로 사용된다. 주요 용도는 다음과 같다.

- 사용자 인증정보 전파 - Spring Security에서 ThreadLocal을 이용해서 사용자 인증정보를 전파한다.
    - 스프링에서 사용하는 빈(bean)은 싱글톤 레지스트리로 관리가 되기 때문에 무상태(stateless)로 설계되어야 한다. 따라서 특정 클라이언트에 의존적인 필드가 있으면 안된다. 그래서 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.
    - 무수한 HTTP 요청으로 쓰레드가 많이 생겨도 각각의 ThreadLocal에서 꺼낸 인증정보는 꼬이지 않는다.

`ThreadLocalSecurityContextHolderStrategy`

```java
final class ThreadLocalSecurityContextHolderStrategy implements SecurityContextHolderStrategy {

	private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();

	@Override
	public void clearContext() {
		contextHolder.remove();
	}

	@Override
	public SecurityContext getContext() {
		SecurityContext ctx = contextHolder.get();
		if (ctx == null) {
			ctx = createEmptyContext();
			contextHolder.set(ctx);
		}
		return ctx;
	}

	@Override
	public void setContext(SecurityContext context) {
		Assert.notNull(context, "Only non-null SecurityContext instances are permitted");
		contextHolder.set(context);
	}

	@Override
	public SecurityContext createEmptyContext() {
		return new SecurityContextImpl();
	}

}
```

- 트랜잭션 컨텍스트 전파 - 트랜잭션 매니저는 트랜잭션 컨텍스트를 전파하는데 `ThreadLocal`을 사용한다.

`TransactionSynchronizationManager`

```java
public abstract class TransactionSynchronizationManager {
		private static final ThreadLocal<Map<Object, Object>> resources = new NamedThreadLocal("Transactional resources");
		private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations = new NamedThreadLocal("Transaction synchronizations");

...
public static boolean isSynchronizationActive() {
    return synchronizations.get() != null;
}

public static void initSynchronization() throws IllegalStateException {
    if (isSynchronizationActive()) {
        throw new IllegalStateException("Cannot activate transaction synchronization - already active");
    } else {
        synchronizations.set(new LinkedHashSet());
    }
}
```

- 쓰레드에 안전해야하는 데이터 보관

- `InheritableThreadLocal`이라는 `ThreadLocal`을 상속받아 구현한 클래스를 사용한다면 단일 쓰레드 뿐만 아니라 그 쓰레드에서 생성한 하위 쓰레드 까지도 데이터를 공유해서 사용할 수가 있다.

# ThreadLocal 동작방식

---

![image](https://user-images.githubusercontent.com/66561524/197363534-0bd2ced5-f7e8-4956-b0e7-1eafe2d73d14.png)

```java
public class ThreadLocal<T> {

	...

		/**
		 * Returns the value in the current thread's copy of this
		 * thread-local variable.  If the variable has no value for the
		 * current thread, it is first initialized to the value returned
		 * by an invocation of the {@link #initialValue} method.
		 *
		 * @return the current thread's value of this thread-local
		 */
		public T get() {
		    Thread t = Thread.currentThread();
		    ThreadLocalMap map = getMap(t);
		    if (map != null) {
		        ThreadLocalMap.Entry e = map.getEntry(this);
		        if (e != null) {
		            @SuppressWarnings("unchecked")
		            T result = (T)e.value;
		            return result;
		        }
		    }
		    return setInitialValue();
		}

	...

		/**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
    }

	...

		/**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

	...

}
```

- `ThreadLocal`은 현재의 쓰레드를 가져와ㅏ서 `ThreadLocalMap`을 가져온다.

```java
public class Thread implements Runnable {
	...

		/* ThreadLocal values pertaining to this thread. This map is maintained
		 * by the ThreadLocal class. */
		ThreadLocal.ThreadLocalMap threadLocals = null;

	...
}
```

- 쓰레드가 코드를 실행하면서 만나게 되는 모든 ThreadLocal 변수를 threadLocals 객체에 저장한다.

![image](https://user-images.githubusercontent.com/66561524/197363542-e6c23b2b-9c58-4972-9eae-83a4ff117daf.png)

- 해당 쓰레드 안에서 별도로 관리되고 있는 threadLocals에 변수를 저장하는 것이다.

# ThreadLocal 사용 방법

---

- 직접 `ThreadLocal`을 사용할 때는 보통 인터셉터에서 사용을 하면 비즈니스 로직에서 사용할 수 있다.

```java
/**
 * 스레드 로컬 선언
 */
public class MadContext {
	public static final ThreadLocal<String> THREAD_LOCAL = ThreadLocal.withInitial(() -> "");
}

/**
 * 인터셉터 정의
 */
public class MadContextInterceptor implements HandlerInterceptor {

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		// `id` 파라미터 값 추출
		final String id = request.getParameter("id");

		// 스레드 로컬에 값 저장
		MadContextHolder.THREAD_LOCAL.set(id);
		return true;
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
		// 스레드 로컬 정보 제거
		MadContextHolder.THREAD_LOCAL.remove();
	}
}
```

# ThreadLocal 주의 사항

---

Thread pool 환경에서 ThreadLocal 사용하는 경우 보관된 데이터 사용이 끝나면 해당 데이터를 반드시 삭제해줘야 한다. 그렇지 않을 경우 재사용되는 쓰레드가 올바르지 않은 데이터를 참조할 수 있다.

# Reference

---

[](https://www.baeldung.com/java-threadlocal)
