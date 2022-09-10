
> ❗ *First, we'll start with CascadeType.REMOVE which is a way to **delete a child entity or entities when the deletion of its parent happens**. Then we'll take a look at the orphanRemoval attribute, which was introduced in JPA 2.0. This provides us with a way to **delete orphaned entities from the database***

JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속상태여야 한다. 부모/자식 모두 영속상태로 만들어야 하는데 이럴 경우 영속성 전이를 사용하면 부모가 영속화 될 때 자식도 영속객체로 만들 수 있다.

## cascade

**영속성 전이란, 자신의 엔티티를 영속시킬 때, 자신과 연관돼있는 엔티티 또한 영속시키는 것**을 말한다.

또한, 사람들이 흔히 cascade와 fetch가 함께 쓰여 둘이 상관관계가 있을 것이라 오해하는데, **영속성 전이(cascade)는 지연로딩, 즉시로딩과 같은 연관관계 세팅과 전혀 상관이 없다**고 한다.

- `CascadeType.PERSIST` – 엔티티를 생성하고, 연관 엔티티를 추가하였을 때 `persist()` 를 수행하면 연관 엔티티도 함께 `persist()`가 수행된다. 만약 연관 엔티티가 DB에 등록된 키값을 가지고 있다면 detached entity passed to persist Exception이 발생한다.
- `CascadeType.MERGE` – 트랜잭션이 종료되고 detach 상태에서 연관 엔티티를 추가하거나 변경된 이후에 부모 엔티티가 merge()를 수행하게 되면 변경사항이 적용된다.(연관 엔티티의 추가 및 수정 모두 반영됨)
- `CascadeType.REMOVE` – 삭제 시 연관된 엔티티도 같이 삭제된다.
- `CascadeType.DETACH` – 부모 엔티티가 detach()를 수행하게 되면, 연관된 엔티티도 detach() 상태가 되어 변경사항이 반영되지 않는다.
- `CascadeType.ALL` – 모든 Cascade 적용

**cascade가 사용되는 시점**

1. 연관 엔티티가 완전 부모의 개인 소유인 경우에 사용할 수 있다.

2. DDD의 Aggregate Root와 어울린다.

3. 애매하면 사용하지 않는다.

[cascade 옵션 질문 - 인프런 | 질문 & 답변](https://www.inflearn.com/questions/31969)

## **orphanRemoval (고아 객체)**

**고아 객체란, 부모 엔티티와 연관관계가 끊어진 자식 엔티티**를 말한다.

이러한 고아 객체를 자동으로 삭제되게 할 수 있는 방법이 있는데, 바로 `orphanRemoval=true`이다.

만약 `orphanRemoval=false`로 설정하거나, 아예 `orphanRemoval`을 설정하지 않으면 아예 삭제가 되지 않는다.

## Test Code

```java
@Entity @Data
public class Parent {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> children = new ArrayList<>();
}
```

```java
@Entity @Data
public class Child {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Parent parent;

}
```

```java
@Test
void cascadeTest(){
    Parent parent = parentRepository.save(new Parent());
    Child child = childRepository.save(new Child());

    // 관계를 맺어준다.
    parent.setChildren(Arrays.asList(child));

    // 부모를 삭제
    parentRepository.delete(parent);

    // result
    // cascade = CascadeType.ALL, REMOVE, orphanRemoval = true 각각에 모두 해당
    List<Parent> parents = parentRepository.findAll();
    List<Child> children = childRepository.findAll();
    assertThat(parents).isEmpty();
    assertThat(children).isEmpty();
}
```

```java
@Test
void orphanRemovalTest(){

    // 부모 생성
    Parent parent = new Parent();

    // 자식들 생성
    Child child1 = childRepository.save(new Child());
    Child child2 = childRepository.save(new Child());
    Child child3 = childRepository.save(new Child());

    // 관계를 맺어준다. child 1,2
    parent.getChildren().add(child1);
    parent.getChildren().add(child2);
    parent = parentRepository.save(parent);

    // 관계를 끊어준다. child 1,2
    parent.getChildren().remove(child1);
    parent.getChildren().remove(child2);
    parent = parentRepository.save(parent); // 고아객체는 commit하는 시점에 처리가 된다.
    assertThat(parent.getChildren().size()).isEqualTo(0);

    // result
    // orphanRemoval = true 에만 해당
    List<Parent> parents = parentRepository.findAll();
    List<Child> children = childRepository.findAll();
    assertThat(parents).isNotEmpty();

    //  child3만 db에 존재. 나머지 child 1,2는 삭제
    assertThat(children.size()).isEqualTo(1);
}
```

## Conclusion

1. `CascadeType.REMOVE`는 부모 자식의 참조관계가 있는 경우 부모가 삭제되었을 때, 자식들을 삭제한다.
2. `orphanRemoval = true`는 부모 자식의 참조관계가 있는 경우 부모와 자식과 관계가 끊어졌을 때, 자식들을 삭제한다.

즉, 둘은 모두 부모 관계가 있는 경우 부모가 삭제되면 자식들이 전부 삭제되고 부모와 관계가 없는 자식들은 영향을 주지 않는다. 하지만 `CascadeType.REMOVE`는 관계를 끊는다고 자식들을 제거하지 않는다. `orphanRemoval = true`는 관계를 끊는 순간 자식들도 제거한다.

그래서 자식 생명주기를 을 부모 엔티티에게 완전히 영속화시키려고 하면 `CascadeType.All, orphanRemoval = true`를 동시에 사용하여 관리한다. 이는  DDD(DomainDrivenDesign) 도메인 주도 설계의 AggregateRoot개념을 구현할 때, 유용하다. 단, Aggregate하나당 Repository하나인 것을 추천한다.

## Reference

[](https://www.baeldung.com/jpa-cascade-remove-vs-orphanremoval)

[[JPA] 실전예제 5_ 영속성 전이(Cascade), 고아 객체(orphanRemoval=true) 적용](https://kth990303.tistory.com/54)

[[JPA팁,JPa강좌]orphanRemoval = true vs DDL의 On Delete Cascade vs CascadeType.REMOVE 차이점](http://ojc.asia/bbs/board.php?bo_table=LecJpa&wr_id=205)

[JPA,영속성전이, 패치전략(EAGER, LAZY),orphanRemoval,CASCADE](http://ojc.asia/bbs/board.php?bo_table=LecJpa&wr_id=265)
