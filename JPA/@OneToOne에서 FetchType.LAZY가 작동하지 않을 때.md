### @OneToOne의 FetchType 기본전략
- @OneToOne의 기본전략은 EAGER이다.

```java
@Target({METHOD, FIELD})
@Retention(RUNTIME)
public @interface OneToOne {

    Class targetEntity() default void.class;

    CascadeType[] cascade() default {};

    /**
     * (Optional) Whether the association should be lazily
     * loaded or must be eagerly fetched. The EAGER
     * strategy is a requirement on the persistence provider runtime that
     * the associated entity must be eagerly fetched. The LAZY
     * strategy is a hint to the persistence provider runtime.
     */
    FetchType fetch() default EAGER;

    boolean optional() default true;

    String mappedBy() default "";

    boolean orphanRemoval() default false;
}
```

### `@OneToOne`에 FetchType을 Lazy로 명시해도 Eager로 동작하는 예외
- nullable이 허용되지 않는 1:1 관계. 즉, 참조 객체가 optional = false 로 지정할 수 있는 관계여야 한다.
- 양방향이 아닌 단방향 1:1 관계여야한다. 연관관계의 주인이 아닌 경우는 Lazy로 호출할 수 없다.
- `@PrimaryKeyJoin`은 허용되지 않는다. 부모와 자식 엔티티간의 조인컬럼이 모두 PK의 경우를 의미한다.


### 원인
1. JPA의 객체 참조는 프록시 기반이다.
2. 연관관계가 있는 객체는 참조를 할때 기본적으로 null이 아닌 프록시 객체를 반환
3. 1:1관계에서는 null이 허용되는 경우, 프록시형태로 null객체를 반환할 수가 없기 때문
4. 1:N관계는 이미 배열의 형태로 이미 참조할 프록시 객체를 싸고 있기 때문에 그 객체가 null이라도 참조할때는 문제가 되지 않음
5. 프록시 객체의 id가 없으면 해당 객체를 식별할 수 있는 값이 없어 프록시 객체를 null로 만들 수 없어서 쿼리를 한 번 더 보내기 때문에 eager로 동작하게 된다.


[JPA 도입 — OneToOne 관계에서의 LazyLoading 이슈](https://yongkyu-jang.medium.com/jpa-%EB%8F%84%EC%9E%85-onetoone-%EA%B4%80%EA%B3%84%EC%97%90%EC%84%9C%EC%9D%98-lazyloading-%EC%9D%B4%EC%8A%88-1-6d19edf5f4d3)
