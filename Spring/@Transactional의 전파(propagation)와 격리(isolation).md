# @Transactional이란?

- @Transactional은 데이터베이스의 트랜잭션을 메소드 단위로 묶을 때(wrap) 사용할 수 있다.
- @Transactional을 사용하면서 전파, isolation, timeout, read-only, rollback 조건 등을 사용할 수 있다.
- 우리는 이를 Transaction Manager를 통해 명시할 수 있다.

## Transactional의 구현체

- 스프링은 트랜잭션을 만들거나 commit, rollback을 하기 위해 프록시 혹은 byte-code를 만들어낸다.
- proxy를 사용하는 경우 @Transacitonal을 내부 메소드 호출하면 무시된다.
- @Transactional을 붙인 메서드를 호출한다면 스프링은 몇몇 트랜잭션 매니징 코드를 @Transactional 메서드를 호출할 때 둘러싼다.

```java
createTransactionIfNecessary();
try {
    callMethod();
    commitTransactionAfterReturning();
} catch (exception) {
    completeTransactionAfterThrowing();
    throw exception;
}
```

## @Transactional 사용 방법

- 어노테이션을 인터페이스, 클래스, 메서드에 달아주면 사용할 수 있다.
- 트랜잭션은 우선순위에 따라서 오버라이드가 된다.
   - 가장 우선순위가 낮은 순으로 interface, superclass, class, interface method, superclass method, class method이다.
- class 레벨의 어노테이션은 해당 클래스의 모든 public 메서드에 적용된다.
- 하지만 만약 private or protected 메서드에 어노테이션을 붙이면 스프링은 이것을 에러없이(컴파일, 런타임) 무시한다.
- 보통 @Transactional은 인터페이스에 사용하길 추천한다.

```java
@Transactional
public interface TransferService {
    void transfer(String user1, String user2, double val);
}
```

```java
@Service
@Transactional
public class TransferServiceImpl implements TransferService {
    @Override
    public void transfer(String user1, String user2, double val) {
        // ...
    }
}
```

```java
@Transactional
public void transfer(String user1, String user2, double val) {
    // ...
}
```

# 트랜잭션 전파 옵션(Transaction Propagation)

- 전파(Propagation)은 우리의 비즈니스 로직의 트랜잭션 범위이다.
- 스프링은 propagation 셋팅에 따라서 트랜잭션의 시작과 중지를 관리한다.
- 스프링이 TransactionManager::getTransaction를 호출하면 트랜잭션을 시작하거나 가져온다.
- 이것은 트랜잭션 매니저의 모든 전파 옵션을 지원한다.
- 하지만 TransactionManager의 구현체가 지원하는 것은 몇 개 없다.

## REQUIRED
- default propagation 이다.
- 만약 트랜잭션이 활성화되있거나 존재하지 않으면 새로운 것을 만들거나 현재 활성화 된 트랜잭션에 비즈니스 로직을 append 한다.

```
@Transactional(propagation = Propagation.REQUIRED)
public void requiredExample(String user) { 
    // ... 
}
```

아래와 같은 로직이라고 볼 수 있다.

pseudo-code
```java
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
return createNewTransaction();
```


##  SUPPORTS
- 스프링은 존재하는 트랜잭션이 있는지 확인하고 있다면 그것을 사용한다. 만약 없다면 non-transactional하게 사용한다.

pseudo-code
```java
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
return emptyTransaction;
```

## MANDATORY 
- 활성화된 트랜잭션이 있으면 그것을 사용한다. 활성화된 트랜잭션이 없다면 에러를 발생시킨다.

pseudo-code
```java
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
throw IllegalTransactionStateException;
```

## NEVER 

- 만약 활성화된 트랜잭션이 있다면 에러를 발생시킨다.

```java
if (isExistingTransaction()) {
    throw IllegalTransactionStateException;
}
return emptyTransaction;
```

## NOT_SUPPORTED 
- 만약 트랜잭션이 존재한다면 스프링은 그것을 중단시키고 그런 다음 비즈니스 로직을 트랜잭션이 없이 동작시킨다.

> The JTATransactionManager supports real transaction suspension out-of-the-box. Others simulate the suspension by holding a reference to the existing one and then clearing it from the thread context


## REQUIRES_NEW 
- 만약 트랜잭션이 현재 존재하면 중단시키고 새로운 것을 만든다.

> Similar to NOT_SUPPORTED, we need the JTATransactionManager for actual transaction suspension.

```java 
if (isExistingTransaction()) {
    suspend(existing);
    try {
        return createNewTransaction();
    } catch (exception) {
        resumeAfterBeginException();
        throw exception;
    }
}
return createNewTransaction();
```

## NESTED 
- 만약 트랜잭션이 존재하면 이것은 저장 지점을 마킹한다. 만약 우리의 비즈니스 로직이 예외를 던졌을 때 그 트랜잭션은 이 저장포인트로 롤백한다.
- 만약 활성화된 트랜잭션이 없으면 REQUIRED처럼 동작한다.

> DataSourceTransactionManager supports this propagation out-of-the-box. Some implementations of JTATransactionManager may also support this.

> paTransactionManager supports NESTED only for JDBC connections. However, if we set the nestedTransactionAllowed flag to true, it also works for JDBC access code in JPA transactions if our JDBC driver supports save points.

# Transaction Isolation
- Isolation은 트랜잭션의 특징 ACID(Atomicity, COnsistency, Isolation, Durability) 중 하나이다.
- 동시성 트랜잭션들(concurrent transactions)가 다른 것들에게 어떻게 보여주는 지 변화를 드러낸다.
- 각 isolation level은 트랜잭션의 동시성 문제를 예방해준다.

1. Dirty read
  - concurrent transaction의 커밋되지 않은 변화를 읽는다.
2. Nonrepeatable read
  - 만약 concurrent transation이 같은 row와 commits을 update한 경우 다른 value를 다시 읽어들인다.
3. Phantom read
  - 만약 또 다른 트랜잭션이 범위안에 있는 row나 commits을 추가하거나 삭제한 경우 우리는 range query를 재실행시킨 후에 다른 rows를 가져온다.

- 우리는 @Transactional::isolation 를 통해 트랜잭션의 고립 레벨을 설정할 수 있다.
- 스프링에서는  DEFAULT, READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE로 5가지 enum으로 관리한다.

## Isolation Management in Spring
- isolation level의 default는 DEFAULT이다.
- 스프링은 새로운 트랜잭션을 만들 때 그 isolation level은 우리의 RDBMS default 전략을 따른다. 만약 database를 바꿀 때면 이 설정을 유심히 봐야한다.
- 우리가 다른 isolation을 가진 메서드를 호출할 때 심사숙고 해야한다.
- 기본적으로 새로운 트랜잭션을 만들 때에만 isolation을 적용한다.
- 만약 우리가 다른 isolation을 사용하길 원치 않을 때 우리는 ransactionManager::setValidateExistingTransaction 를 true로 설정해야 한다.

```java
if (isolationLevel != ISOLATION_DEFAULT) {
    if (currentTransactionIsolationLevel() != isolationLevel) {
        throw IllegalTransactionStateException
    }
}
```

## READ_UNCOMMITTED
- 가장 낮은 레벨이며 가장 많이 허용하는 concurrent 접근 방식이다.
- 이것은 아까 언급한 3가지의 동시성 문제가 발생할 수 있다.

- 다른 concurrent transations의 커밋되지 않은 데이터가 의해 읽힌다.
- 또한 두개의 non-repeatable and phantom 읽기가 일어날 수 있다.
- 우리는 row의 재읽기, range query의 재실행에 다른 결과를 가질 수 있다.

> Postgres does not support READ_UNCOMMITTED isolation and falls back to READ_COMMITED instead. Also, Oracle does not support or allow READ_UNCOMMITTED.

## READ_COMMITTED
- 두번제 레벨이다. dirty reads를 예방해준다. 다른 동시성 문제는 일어날 것이다.
- 커밋되지 않은 변화는 영향을 끼치지 않는다. (dirty read)
- 하지만 트랜잭션 커밋이 변화되면 우리의 결과는 re-querying되어 바뀔 것이다.

> READ_COMMITTED is the default level with Postgres, SQL Server, and Oracle.

## REPEATABLE_READ
- 세 번째 단계이다. dirty와 non-repeatable한 읽기를 예방한다. 그래서 우리는 concurrent transations 중 uncommited 변화에대해 영향을 받지 않는다.
- 또한 우리가 row에 대해 re-query를 할 때, 우리는 다른 결과를 받지 않는다. 그러나 range-queries의 재실행할 때 우리는 새 row를 얻거나 제거해야할 것이다.
- 게다가 이것은 update 예방을 최소로 요구하는 수준이다. 그 lost update는 두 개 이상의 concurrent transaition이 같은 row를 읽고 업데이트하는 경우이다.
- REPEATABLE_READ는 모든 로우의 접근을 simultaneous 하게 허용하지 않는다. 그 lost update가 잃어나지 않는다.

> REPEATABLE_READ is the default level in Mysql. Oracle does not support REPEATABLE_READ.

## SERIALIZABLE 
- 가장 높은 격리 수준이다. 이것은 언급한 모든 동시성 문제를 예방한다.
- 하지만 최저의 concurrent access rate이기도 하다 이유는 이것은 순차적으로 concurrent가 실행되기 때문이다.
- 다시말해 직렬화된 트랜잭션의 그룹의 동시적 실행은 이것을 직렬화한 실행과 같다.

# Reference
[Transaction Propagation and Isolation in Spring @Transactional](https://www.baeldung.com/spring-transactional-propagation-isolation)

