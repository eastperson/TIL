# Overview

트랜잭션은 기본적으로 DataSource의 Connection을 기준으로 생성이 된다. 하나의 DB 커넥션 안에서 만들어지는 트랜잭션을 로컬 트랜잭션(local transaction)이라고 한다. Persistence Layer와 Business Layer가 분리되어있는 환경에서 트랜잭션의 경계 설정을 Business Layer에서 해야하는 상황이 있다.

하지만 기존의 Connection을 관리하는 코드가 Business Layer에 침투하면 데이터 액세스 기술에 대해 독립적일 수 없다. 또한 멀티스레드 환경에서 공유하는 인스턴스 변수에 스레드별로 생성하는 정보를 저장하다가 서로 덮어쓰는 일이 발생할 수 있다.

스프링은 이러한 딜레마를 해결하기 위한 **트랜잭션 동기화(Transaction Synchronization)** 기술을 제공한다.

# 트랜잭션 동기화

트랜잭션 동기화는 트랜잭션을 시작하기 위한 Connection 객체를 특별한 저장소에 보관해두고 필요할 때 꺼내쓸 수 있도록 하는 기술이다. 트랜잭션 동기화 저장소는 작업 쓰레드마다 Connection 객체를 독립적으로 관리하기 때문에 멀티쓰레드 환경에서도 충돌이 발생할 여지가 없다.

![image](https://user-images.githubusercontent.com/66561524/196822907-9dbc14f7-44d9-40e6-9f4b-416fbe935f28.png)

트랜잭션이 시작되면 트랜잭션 매니저가 커넥션을 트랜잭션 동기화 매니저라는 곳에 보관을 한다. Persistence Layer에서는 데이터에 접근해야할 로직이 발생하면 트랜잭션 동기화 매니저를 통해 커넥션을 획득한다. 

따라서 비즈니스 로직 플로우에서 트랜잭션이 종료가 되는 시점까지 같은 커넥션의 트랜잭션을 사용할 수 있는 것이다. Connection의 commit/rollback이 발생한 시점에 이를 제거하게 된다.

# 트랜잭션 동기화 작업 코드

멀티 스레드 환경에서 트랜잭션 동기화를 구현하기 위해서 스프링은 간단한 유틸리티 메소드를 제공한다. 실제 트랜잭션 매니저와 `DataSourceUtils`를 사용하는 코드는 아래와 같다.

![image](https://user-images.githubusercontent.com/66561524/196822922-98f03e7f-c997-4dfa-ad15-2826bf0b6ef8.png)

스프링은 `TransactionSynchronizationManager` 를 통해 동기화를 관리한다. `DataSourceUtils` 의 `getConnection(dataSource)`은 DataSource에서 Connection 오브젝트를 생성해줄뿐만 아니라 트랜잭션 동기화에 사용하도록 저장소에 바인딩해준다. 

## 동기화 작업 초기화(`initSynchronization`)

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

## DB 커넥션 생성(`getConnection`)

`DataSourceUtils`

```java
public abstract class DataSourceUtils {

...

public static Connection getConnection(DataSource dataSource) throws CannotGetJdbcConnectionException {
    try {
        return doGetConnection(dataSource);
    } catch (SQLException var2) {
        throw new CannotGetJdbcConnectionException("Failed to obtain JDBC Connection", var2);
    } catch (IllegalStateException var3) {
        throw new CannotGetJdbcConnectionException("Failed to obtain JDBC Connection", var3);
    }
}

public static Connection doGetConnection(DataSource dataSource) throws SQLException {
    Assert.notNull(dataSource, "No DataSource specified");
    ConnectionHolder conHolder = (ConnectionHolder)TransactionSynchronizationManager.getResource(dataSource);
    if (conHolder == null || !conHolder.hasConnection() && !conHolder.isSynchronizedWithTransaction()) {
        logger.debug("Fetching JDBC Connection from DataSource");
        Connection con = fetchConnection(dataSource);
        if (TransactionSynchronizationManager.isSynchronizationActive()) {
            try {
                ConnectionHolder holderToUse = conHolder;
                if (conHolder == null) {
                    holderToUse = new ConnectionHolder(con);
                } else {
                    conHolder.setConnection(con);
                }

                holderToUse.requested();
                TransactionSynchronizationManager.registerSynchronization(new DataSourceUtils.ConnectionSynchronization(holderToUse, dataSource));
                holderToUse.setSynchronizedWithTransaction(true);
                if (holderToUse != conHolder) {
                    TransactionSynchronizationManager.bindResource(dataSource, holderToUse);
                }
            } catch (RuntimeException var4) {
                releaseConnection(con, dataSource);
                throw var4;
            }
        }

        return con;
    } else {
        conHolder.requested();
        if (!conHolder.hasConnection()) {
            logger.debug("Fetching resumed JDBC Connection from DataSource");
            conHolder.setConnection(fetchConnection(dataSource));
        }

        return conHolder.getConnection();
    }
}
```

동기화 매니저에 커넥션을 가져오기 위해 접근하고 만일 커넥션이 없는 경우에 새로운 커넥션을 생성해서 해당 커넥션을 사용한다.

커넥션이 없는지 확인 있으면 생성

```java

if (conHolder == null || !conHolder.hasConnection() && !conHolder.isSynchronizedWithTransaction())
```

```java
TransactionSynchronizationManager.bindResource(dataSource, holderToUse);
```

커넥션이 있으면 기존 커넥션 요청

```java
else {
    conHolder.requested();
    if (!conHolder.hasConnection()) {
        logger.debug("Fetching resumed JDBC Connection from DataSource");
        conHolder.setConnection(fetchConnection(dataSource));
    }

    return conHolder.getConnection();
}
```

## 동기화 저장소 닫기(`releaseConnection`)

트랜잭션을 사용하기 위해 동기화된 커넥션은 바로 닫지 않고 그대로 유지한다. 반면에 트랜잭션 동기화 매니저가 관리하는 커넥션이 없을 경우 해당 커넥션은 바로 닫아버린다.

```java
public static void releaseConnection(@Nullable Connection con, @Nullable DataSource dataSource) {
    try {
        doReleaseConnection(con, dataSource);
    } catch (SQLException var3) {
        logger.debug("Could not close JDBC Connection", var3);
    } catch (Throwable var4) {
        logger.debug("Unexpected exception on closing JDBC Connection", var4);
    }

}

public static void doReleaseConnection(@Nullable Connection con, @Nullable DataSource dataSource) throws SQLException {
    if (con != null) {
        if (dataSource != null) {
            ConnectionHolder conHolder = (ConnectionHolder)TransactionSynchronizationManager.getResource(dataSource);
            if (conHolder != null && connectionEquals(conHolder, con)) {
                conHolder.released();
                return;
            }
        }

        doCloseConnection(con, dataSource);
    }
}
```

Business Layer에서 트래잭션 매니저를 사용하지 않았으면 커넥션을 유지시키지 않고 바로 닫는 것이다.

## 동기화 저장소 작업 종료 및 정리

`TransactionSynchronizationManager`

```java
....

public static Object unbindResource(Object key) throws IllegalStateException {
    Object actualKey = TransactionSynchronizationUtils.unwrapResourceIfNecessary(key);
    Object value = doUnbindResource(actualKey);
    if (value == null) {
        throw new IllegalStateException("No value for key [" + actualKey + "] bound to thread");
    } else {
        return value;
    }
}

...

public static void clearSynchronization() throws IllegalStateException {
    if (!isSynchronizationActive()) {
        throw new IllegalStateException("Cannot deactivate transaction synchronization - not active");
    } else {
        synchronizations.remove();
    }
}
```

# Reference

[[Spring] 트랜잭션에 대한 이해와 Spring이 제공하는 Transaction(트랜잭션) 핵심 기술 - (1/3)](https://mangkyu.tistory.com/154)

[10.4 Synchronizing resources with transactions](https://docs.spring.io/spring-framework/docs/3.0.0.M4/reference/html/ch10s04.html)

[[Spring] 트랜잭션 문제 해결 - 트랜잭션 동기화 기법](https://withseungryu.tistory.com/98)
