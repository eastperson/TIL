# Overview

`@Query`과 `@Modifying`를 같이 사용할 때 영속성 컨텍스트 캐시와 데이터베이스 데이터가 동기화가 되지 않는 문제가 생길 수 있다. `@Modifying`을 사용하면 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리가 나가는 방식이다. 따라서 영속성 컨텍스트에 캐싱된 데이터와 실제 DB에 저장된 데이터가 동기화되지 않는 문제가 발생할 수 있다. 우리는 FlushMode를 설정하기 때문에 기본적으로 이 상황을 방지하지만 그럼에도 유의해야하는 점이 있다.

`@Modifying`의 Attribute인 `clearAutomatically`와 `flushAutomatically`에 대해 학습하고 사용해야 한다.

# @Modifying의 동일성 보장과 데이터베이스 동기화 문제

우선 각 개념을 정리하자.

## @Query

- Spring Data JPA는 @Query를 사용하면 JPQL, 네이티브 쿼리로 직접 쿼리를 작성하여 데이터베이스에 사용할 수 있다.

## **@Modifying**

[스프링 공식문서](https://docs.spring.io/spring-data/data-jpa/docs/current/api/org/springframework/data/jpa/repository/Modifying.html) 

- `@Query` 애너테이션으로 작성된 변경, 삭제 쿼리 메서드를 사용할 때 필요하다.
- 조회 쿼리를 제외한 INSERT, UPDATE, DELETE, DDL에서 사용된다.
- 주로 벌크 연산시 사용된다.
- **JPA Entity LifeCycle을 무시하고 쿼리가 샐행**되어 영속성 컨텍스트 관리에 주의해야한다.

> Queries that require a `@Modifying` annotation include `INSERT`, `UPDATE`, `DELETE`, and DDL statements.
> 

Attribute

> **`clearAutomatically`**
Defines whether we should clear the underlying persistence context after executing the modifying query.
> 

> `flushAutomatically`
****Defines whether we should flush the underlying persistence context before executing the modifying query.
> 

## 벌크연산

- 다건의 UPDATE, DELTE 연산을 하나의 쿼리로 하는 것을 의미한다.
- 단건 UPDATE의 경우 Dirty Checking을 통해서 수행되거나 save()로도 가능하다.
- DELETE는 다건, 단건 모두 쿼리 메서드로 제공된다.
- 따라서 `@Modifying`을 명시적으로 가장 많이 사용되는 것은 `@Query`를 사용한 벌크 Update 연산이다.
    - 그래서 이름이 modifying인 듯
- [@Query에 벌크 연산 쿼리를 작성하고 @Modifying을 붙이지 않으면 InvalidDataAccessApiUsage이 발생](https://www.baeldung.com/spring-data-jpa-modifying-annotation)한다.

## **clearAutomatically**

- `@Modifying`이 붙은 해당 쿼리 메서드 직 후 영속성 컨텍스트를 clear할지 정하는 attribute이다.
- default 값은 false
- deafult 값을 사용할 시 영속성 컨텍스트의 1차 캐시와 관련한 문제가 발생할 수 있다.

## ****flushAutomatically****

- `@Query`와 `@Modifying`을 사용할 떄 해당 쿼리를 실행하기 전 영속성 컨텍스트의 변경 사항을 DB에 flush할지를 결정하는 attribute이다.
- default 값은 false이다.

하지만 이상하게도 false로 설정을 했는데도  자동으로 flush가 발생하는 경우가 있습니다. 이유는 Spring Data JPA의 구현체인 Hibernate에 있습니다.

## Hibernate의 FlushModeType

- Spring Data JPA는 구현체로 Hibernate를 사용한다.
- Hibernate는 FlushModeType으로 enum class가 있다. 그 값은 AUTO와 COMMIT이며 default는 AUTO

> AUTO(Default) : flush가 쿼리 실행 시 발생한다.
> 
> 
> COMMIT : flush가 트랜잭션이 commit 될 때 발생한다.
> 
- 따라서 쿼리 실행 전 flush가 나가게 됩니다. 참고로 `@Modifying`을 사용하면 영속성 컨텍스트를 무시하고 쿼리가 발생하기 때문에 flush가 됩니다. 이런 이유로 기본 설정으로는 `flushAutomatically`를 설정하지 않아도 flush가 됩니다.
- 다만  `application.properties`에 `spring.jpa.properties.org.hibernate.flushMode=COMMIT`으로 설정되어 있는 경우 `flushAutomatically`를 `true`로 하지않으면 쿼리전 flush가 되지 않아 데이터 동기화문제가 발생할 수 있습니다.

# 문제점

`@Modifying` 쿼리가 영속성 컨텍스트를 거치지 않고 데이터 베이스에 접근하기 전에 쿼리 전, 후에 문제가 발생할 수 있다.

1. `@Modifying` 쿼리 직전에 flush가 되지 않으면 영속성 컨텍스트의 1차캐시의 내용이 반영되지 않은 데이터베이스의 값을 대상으로 쿼리가 나갈 수 있다.
2. `@Modifying` 쿼리 직후에 영속성 컨텍스트의 캐시를 비워내지 않으면 조회 연산 수행시 `@Modifying` 쿼리의 내용이 반영되지 않은 캐시 데이터를 사용할 수 있다.

# 해결

1. 쿼리 직전에 영속성 컨텍스트의 있는 1차 캐시 데이터를 비워내기 위해 flush를 해줘야한다. `flushAutomatically`는 default로 false이지만 Hibernate 구현체의 FlushMode가 `AUTO`를 기본으로 설정되어 있어 `@Modifying` 쿼리 직전에 flush가 발생해서 문제가 발생하지 않는다. 하지만 만일 FlushMode를 수동으로 `COMMIT`으로 설정했을 경우 `flushAutomatically`를 true로 설정하지 않으면 flush가 되지 않아 데이터 동기화가 되지 않을 수 있다. 따라서 `flushAutomatically`를 true해줘야 한다.
2. 쿼리 직후에는 영속성 컨텍스트를 비워내지 않으면 현재 캐싱된 데이터와 데이터베이스의 데이터가 동기화되어있지 않을 수 있다. `clearAutomatically`를 `true`로 바꿔주면 쿼리 직후에 영속성 컨텍스트를 clear해주어서 조회 연산시 데이터베이스에 조회 쿼리를 실행하게 된다.

> 조회 연산시 영속성 컨텍스트에 캐시가 있으면 캐싱된 데이터를 가져다 사용하고 캐시가 없을시에 데이터베이스에 쿼리를 직접 날려 조회한다.
> 

# Reference

[Spring Data JPA @Modifying (1) - clearAutomatically](https://devhyogeon.tistory.com/4?category=878035)

[Spring Data JPA @Modifying (2) - flushAutomatically](https://devhyogeon.tistory.com/5)

[JPA의 동일성 보장으로 인해 발생하는 데이터 동기화 문제](https://devhyogeon.tistory.com/6?category=878035)
