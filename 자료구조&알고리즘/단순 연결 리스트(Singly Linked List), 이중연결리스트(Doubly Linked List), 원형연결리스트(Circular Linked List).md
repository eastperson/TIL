## Overview
> 연결 리스트는 데이터 요소의 선형 집합이다. 하지만 데이터의 순서가 메모리에 물리적인 순서대로 저장되지는 않는다.
- 연결 구조를 통해 물리 메모리를 연속적으로 사용하지 않아도 되기 떄문에 관리가 쉽다. 데이터를 구조체로 묶어서 포인터로 연결하는 개념은 다양하게 활용이 가능하다.

## 단순 연결 리스트(Singly Linked List)

동적 메모리 할당을 이용해 리스트를 구현하는 자료구조이다. 즉, 동적 메모리 할당을 받아 노드(Node)를 저장하고 노드는 레퍼런스를 이용하여 다음 노드를 가리키도록 만들어 노드들을 한 줄로 연결시킨다.

![image](https://user-images.githubusercontent.com/66561524/209427522-4436759a-f498-4c3c-9f0e-35e22ca0643a.png)

배열의 경우 길이가 정해져있기 때문에 삽입이나 삭제를 하는 경우 뒤따르는 항목들을 이동해야 한다. 하지만 연결리스트에서는 삽입이나 삭제시 항목들의 이동이 필요없다. 또한 빈 공간이 존재하지 않는다.

반면에 연결리스트에서는 항목을 탐색하려면 항상 첫 노드부터 원하는 노드를 찾을 때까지 차례로 방문하는 순차탐색(Sequential Search)를 해야만한다. 또한 배열은 각 원소에 레퍼런스를 저장할 필요가 없지만 연결리스트는 각 노드마다 레퍼런스를 저장할 공간이 필요하다. 단순연결리스트는 스택과 큐, 해싱의 체이닝에 사용되며 트리는 단순 연결리스트를 확장시킨 개념이다.

```java
package List.src.main.java;

import java.util.NoSuchElementException;

public class MySinglyLinkedList<E> {

    protected Node head;
    private int size;
    public MySinglyLinkedList(){
        head = null;
        size = 0;
    }

    // 탐색은 순차적으로 진행된다.
    public int search(E target) {
        // head부터 시작
        Node p = head;
        // target과 일치하는 노드(객체)가 나올 때 까지 순차적으로 탐색 (O(N))
        for(int k = 0; k < size; k++){
            if(target == p.getItem()) return k;
            p = p.getNext();
        }

        return -1; // 탐색을 실패했을 경우 -1 리턴
    }

    // 연결 리스트 맨 앞쪽에 노드를 삽입
    // 첫 번째 노드에만 방문 하므로 O(1)
    public void insertFront(E newItem){
        head = new Node(newItem, head); // 기존의 head를 next에 담고 head는 새로운 node를 가르킨다.
        size++;
    }

    // 노드 p 바로 다음에 새 노드 삽입
    // 특정 노드 p의 참조값이 주어지지 않은 경우 search()를 수행하기 때문에 O(n)
    public void insertAfter(E newItem, Node p) {
				// 기존의 p의 next 노드를 new item의 next 노드에 넣어준다.
        p.setNext(new Node(newItem,p.getNext()));
        size++;
    }

    // delete연산은 실질적인 삭제 연산이 들어가는 것이 아니라 참조변경을 한다.(gc가 알아서 처리)
    // 리스트의 첫 노드 삭제
    // 첫 번째 노드에만 방문 하므로 O(1)
    public void deleteFront(){
        if(isEmpty()) throw new NoSuchElementException();
        head = head.getNext(); // head의 다음 노드를 head로 저장한다.
        size--;
    }

    // 해당 노드의 다음 노드를 삭제한다.
    // 특정 노드 p의 참조값이 주어지지 않은 경우 search()를 수행하기 때문에 O(n)
    public void deleteAfter(Node p){
        if(p == null) throw new NoSuchElementException(); // p가 존재하지 않으면 예외 발생
        Node t = p.getNext(); // p의 다음 노드를 trget으로 설정한다.
        p.setNext(t.getNext()); // p의 다음 노드를 t의 다음노드(p의 다다음 노드)로 설정한다.
        t.setNext(null); // t와 다음 노드의 연결을 끊어준다.(t는 참조 카운트(reference count)가 0이 되면서 삭제
       size--;
    }

    private boolean isEmpty(){
        if (size == 0)
            return true;
        return false;
    }

}

class Node<E> {
    private E item;
    private Node<E> next;

    public Node(E newItem, Node<E> node) {
        item = newItem;
        next = node;
    }
    // get과 set 메서드
    public E getItem(){return item;}
    public Node<E> getNext(){return next;}
    public void setItem(E newItem){item = newItem;}
    public void setNext(Node<E> newNext){next=newNext;}
}
```

### insert 연산

![image](https://user-images.githubusercontent.com/66561524/209427626-e3895c5c-5aa6-4e09-a07f-a28339b07ba1.png)
![image](https://user-images.githubusercontent.com/66561524/209427634-20b8a361-86fd-4615-a82d-ac810928cfda.png)
![image](https://user-images.githubusercontent.com/66561524/209427635-357d5b04-65f7-41f9-ba53-0c1f25a8a3b4.png)

### delete 연산

![image](https://user-images.githubusercontent.com/66561524/209427654-c4d3bbbf-c743-4eb1-9df8-4177c0c473d9.png)


**수행시간**

search() 연산은 탐색을 위해 연결리스트의 노드들을 첫 노드부터 순차적으로 방문해야 하므로 $O(N)$ 시간이 소요된다. 그러나 삽입이나 삭제 연산은 각각 상수 개의 레퍼런스를 갱신하므로 $O(1)$ 시간이 소요된다. 단, insertAfter()나 deleteAfter()의 경우에 특정 노드 p의 레퍼런스가 주어지지 않으면 head로부터 p를 찾기 위해 search()를 수행해야 하므로 $O(N)$ 시간이 소요될 수 있다.

## 이중연결리스트(Doubly Linked List)

각 노드가 두 개의 레퍼런스를 가지고 각각 이전 노드와 다음 노드를 가리키는 연결리스트이다. 단순연결리스트는 다음 노드의 레퍼런스만 노드들이 연결되어 이전 노드를 가리키는 레퍼런스를 추가로 알아야 한다. 이중연결리스트는 각 노드마다 추가로 한 개의 레퍼런스를 저장해야 한다는 단점을 가진다.

![image](https://user-images.githubusercontent.com/66561524/209427691-d545394d-c694-4711-84f1-054f0dec25d0.png)
![image](https://user-images.githubusercontent.com/66561524/209427693-c9c2b3b9-2b56-4292-9723-92bf3ede84fa.png)

따라서 head는 prev가 null이 되고 tail은 next가 null이 된다.

```java
package List.src.main.java;

import java.util.NoSuchElementException;

// 이중 연결 리스트
// 이중 연결 리스트는 previous 노드와 next를 따로 가지고 있다.
public class MyDoublyLinkedList<E>{

    protected DNode head, tail;
    protected int size;
    public MyDoublyLinkedList(){ //

        head = new DNode(null,null,null); // 하나의 노드에는 3가지의 객체를 가지고 있다
        tail = new DNode(null,head,null); // tail의 이전 노드를 head로 만든다.
        head.setNext(tail); // head의 다음 노드를 tail로 만든다.
        size = 0;
    }

    // p가 가리키는 노드 앞에 삽입
    public void insertBefore(DNode p, E newItem) {
        DNode t = p.getPrevious(); // 노드의 이전 노드를 기억한다.
        DNode newNode = new DNode(newItem,t,p); // 새로운 객체(아이템)으로 사이에 껴 넣는다.
        p.setPrevious(newNode); // 새로운 node를 p의 prev로 저장
        t.setNext(newNode); // 새로운 node를 t의 next로 저장
        size++; // 사이즈 추가
    }

    // p가 가리키는 노드 뒤에 삽입
    public void insertAfter(DNode p, E newItem) {
        DNode t = p.getNext();
        DNode newNode = new DNode(newItem,p,t);
        t.setPrevious(newNode);
        p.setNext(newNode);
        size++;
    }

    // 삭제 연산
    public void delete(DNode x) {
        if(x == null) throw  new NoSuchElementException();
        DNode f = x.getPrevious();
        DNode r = x.getNext();
        f.setNext(r);
        r.setPrevious(f);
        size--;

    }
}

class DNode <E> {
    private E item;
    private DNode previous;
    private DNode next;
    public DNode(E newItem, DNode p, DNode q) { // 노드 생성자
        item = newItem;
        previous = p;
        next = q;
    }

    public void setItem(E item) {
        this.item = item;
    }

    public void setPrevious(DNode previous) {
        this.previous = previous;
    }

    public void setNext(DNode next) {
        this.next = next;
    }

    public E getItem() {
        return item;
    }

    public DNode getPrevious() {
        return previous;
    }

    public DNode getNext() {
        return next;
    }
}
```

**수행시간**

이중연결리스트에서 수행되는 삽입이나 삭제 연산은 단순연결리스트의 삽입이나 삭제 연산보다 복잡하기는 하나 각각 상수 개의 레퍼런스만을 갱신하므로 $O(1)$ 시간에 수행된다. 탐색연산은 단순연결리스트의 탐색과 같이 head 또는 tail로부터 노드들을 순차적으로 탐색해야 하므로 $O(N)$ 시간이 걸린다.

## 원형연결리스트(Circular Linked List)

---

원현연결리스트는 마지막 노드가 첫 노드와 연결된 **단순연결리스트**이다. 원형연결리스트에서는 마지막 노드의 레퍼런스가 저장된 last가 단순 연결리스트의 head와 유사한 역할을 한다. 따라서 마지막 노드와 첫 노드를 $O(1)$ 시간에 방문할 수 있는 장점을 가진다. 또한 리스트가 empty가 아니면 어떤 노드도 null 레퍼런스를 가지고 있지 않으므로 프로그램에서 null 조건을 검사하지 않아도 된다.

반면에 원형연결리스트에서는 반대 방향으로 노드들을 방문하기 쉽지 않다. 무한 루프가 발생할 수 있음에 유의할 필요가 있다. 원형연결리스트는 여러 사람이 차례로 돌아가며 하는 게임을 구현하는데 적합한 자료구조이다.

```java
package List.src.main.java;

import java.util.NoSuchElementException;

public class MyCircularLinkedList<E> {
    private Node last; //리스트의 마지막 노드(항목)을 가리킨다.
    private int size;
    public MyCircularLinkedList(){
        last = null;
        size = 0;
    }

    // last가 가리키는 노드의 다음에 새노드 삽입
    public void insert(E newItem) {
        // 새 노드 생성
        Node newNode = new Node(newItem, null);
        // 리스트가 empty일 때
        if(last == null){
            newNode.setNext(newNode);
            last = newNode;
        }
        else {
            // new node의 다음 노드가 last가 가리키는 노드의 다음 노드가 되도록
            newNode.setNext(last.getNext());
            // last가 가리키는 노드의 다음 노드가 new node가 되도록
            last.setNext(newNode);
        }
        size++;
    }

    // 삭제 연산
    public Node delete() {
        if(isEmpty()) throw new NoSuchElementException();

        Node x = last.getNext();
        if(x == last) last = null;
        else {
            last.setNext(x.getNext());
            x.setNext(null);
        }
        size--;
        return x;
    }

    private boolean isEmpty(){
        if (size == 0)
            return true;
        return false;
    }
}
```

**수행시간**

원형연결리스트에서 삽입이나 삭제 연산 각각 상수 개의 레퍼런스를 갱신하므로 $O(1)$ 시간에 수행된다. 탐색 연산은 단순연결리스트의 탐색과 같이 last로부터 노드들을 순차적으로 탐색해야 하므로 $O(N)$ 시간이 소요된다.

[리스트의 연산에 대한 최악경우 수행시간 비교](https://www.notion.so/9c06b75d38b7445b83092bfdba3ec4db)
