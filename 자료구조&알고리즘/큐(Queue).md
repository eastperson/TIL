# 큐(Queue)

큐는 삽입과 삭제가 양 끝에서 각각 수행되는 자료구조이다. 자료구조는 선입선출(FIFO) 원칙하에 item의 삽입과 삭제를 수행한다.

![image](https://user-images.githubusercontent.com/66561524/210160785-2117161f-3b6b-44c1-9d58-6a625c3f77e1.png)

> 큐(Queue)는 시퀀스의 한쪽 끝에는 엔티티를 추가하고 다른 반대쪽 끝에는 제거할 수 있는 엔티티 컬렉션이다.
> 

FIFO(First-In-First-Out(선입선출)로 처리된다. 상대적으로 스택에 비해서는 쓰임새가 적다. 하지만 데크(Deque)나 우선순위 큐(Priority Queue) 같은 변형들은 여러 분야에서 유용하게 쓰인다. 너비 우선 탐색(Breadth-First Search, BFS)이나 캐시 등을 구현할 때 널리 사용된다.

**배열로 구현한 큐**

```java
package List.src.main.java;

import java.util.EmptyStackException;

public class MyArrayStack<E> {
    private E s[]; // 스택을 위한 배열
    private int top; // 스택의 top 항목의 배열 원소 인덱스
    public MyArrayStack(){ // 스택 생성자
        s = (E[]) new Object[1]; // 초기에 크기가 1인 배열 생성
        top = -1;
    }
    public int size() {return top+1;} // 스택에 있는 항목의 수를 리턴
    public boolean isEmpty() {return (top == -1);} // 스택이 empty이면 true를 리턴

    public E peek() {
        if(isEmpty()) throw new EmptyStackException(); // underflow시 프로그램 정지
        return s[top];
    }

    public void push(E newItem) {
        if(size() == s.length) {
            resize(2*s.length); // 스택을 2배의 크기로 확장
        }
        s[++top] = newItem; // 새 항목을 푸시
    }

    public E pop() {
        if(isEmpty()) throw new EmptyStackException(); // underflow시 프로그램 정지
        E item = s[top];
        s[top--] = null; // top 아이템은 null로 초기화
        if(size() > 0 && size() == s.length/4) resize(s.length/2);
        return item;
    }

    private void resize(int newSize){ // 배열의 크기 조절
        Object[] t = new Object[newSize]; // newSize 크기의 새로운 배열 t 생성
        for (int i = 0; i < size(); i++)
            t[i] = s[i]; // 배열 s를 배열 t로 복사
        s = (E[]) t; // 배열 t를 배열 s로
    }
}
```

**리스트로 구현한 큐**

```java
package List.src.main.java;

import java.util.NoSuchElementException;

public class MyListQueue<E> {
    private Node<E> front, rear;
    private int size;
    public MyListQueue(){
        front = rear = null;
        size = 0;
    }
    public int size() {return size;} // 큐의 항목의 수를 리턴
    public boolean isEmpty(){return size() == 0;} // 큐가 empty이면 true 리턴
    
    public void add(E newItem) {
        Node newNode = new Node(newItem,null); // 새 노드 생성
        if(isEmpty()) front = newNode; // 큐가 empty이었으면 front도 newNode를 가리키게 한다.
        else rear.setNext(newNode); // 그렇지 않으면 rear의 next를 newNode를 가리키게 한다.
        rear = newNode; // 큐 항목 수 1 증가
        size++;
    }
    
    public E remove() {
        if(isEmpty()) throw new NoSuchElementException();
        E frontItem = front.getItem(); // front가 가리키는 노드의 항목을 frontItem에 저장
        front = front.getNext(); // front가 front 다음 노드를 가리키게 한다.
        if(isEmpty()) rear = null; // 큐가 empty이면 rear = null
        size--; // 큐 항목 수 1 감소
        return frontItem;
    }
}
```

큐 자료구조는 CPU의 테스크 스케줄링(Task Scheduling), 네트워크 프린터, 실시간(Real-time) 시스템의 인터럽트(Interrupt) 처리, 다양한 이벤트 구동 방식(Event-driven) 컴퓨터 시뮬레이션, 콜 센터의 전화 서비스 처리 등에 사용된다. 또한 이진트리의 레벨순회(Level-order Traversal), 그래프의 너비우선탐색(Breath-First Search)에 사용된다.

**수행시간**

배열로 구현한 큐의 add와 remove 연산은 각각 $O(1)$ 시간이 소요된다. 하지만 배열 크기를 확대 또는 축소시키는 경우에 큐의 모든 item들을 복사해야 하므로 O(N) 시간이 소요된다. 단순 연결리스트로 구현한 큐의 add와 remove 연산은 각각 $O(1)$ 시간이 걸리는데 삽입 또는 삭제 연산이 rear 와 front로 인해 연결리스트의 다른 노드들을 일일이 방문할 필요없이 각 연산이 수행된다.

# 용례

- CPU의 태스크 스케줄링
- 네트워크 프린터
- 실시간 시스템의 인터럽트 처리
- 다양한 이벤트 구동 방식 컴퓨터 시뮬레이션
- 콜 센터의 전화 서비스 처리
- 이진트리의 레벨순회
- 그래프의 너비우선탐색
