# Overview

스택은 콜 스택(Call Stack)이라 하여 컴퓨터 프로그램의 서브루틴에 대한 정보를 저장하는 자료구조에도 널리 활용된다. 컴파일러가 출력하는 에러도 스택처럼 맨 마지막 에러가 가장 먼저 출력되는 순서를 보인다. 또한 메모리 영역에서 LIFO 형태로 할당하고 접근하는 구조인 아키텍처 레벨의 하드웨어 스택의 이름으로도 널리 사용된다. 꽉 찬 스택에 요소를 삽입하고자 할 때 스택에 요소가 넘쳐서 에러가 발생하는 것을 스택 버퍼 오버플로(Stack Buffer Overflow)라고 부른다.

# 스택 원리

스택은 한 쪽 끝에서만 item을 삭제하거나 새로운 item을 저장하는 자료구조이다. 스택에서 새로운 item을 저장하는 연산을 push라고 하고, item을 삭제하는 연산을 pop이라고 한다. 후입선출(LIFO) 형태의 자료구조라고 할 수 있다.

![image](https://user-images.githubusercontent.com/66561524/209827843-e99ccfcd-7e73-4663-9900-73a5e67cb9a0.png)

- `push()`: 요소를 컬렉션에 추가한다.
- `pop()`: 아직 제거되지 않은 가장 최근에 삽입된 요소를 제거한다.

# 스택 구현 (Java)

**배열로 구현한 스택**

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

**리스트로 구현한 스택**

```java
package List.src.main.java;

import java.util.EmptyStackException;

public class MyListStack<E> {
    private Node<E> top; // 스택 top 항목을 가진 Node를 가리키기 위해
    private int size; // 스택의 항목 수
    public MyListStack() {
        top = null;
        size = 0;
    }
    public int size() {return size;} // 스택의 항목수를 리턴
    public boolean isEmpty() {return size == 0;} // 스택이 empty이면 true 리턴
    
    public E peek() { // 스택 top 항목만을 리턴
        if(isEmpty()) throw new EmptyStackException();
        return top.getItem();
    }
    
    public void push(E newItem) { // 스택 push 연산
        Node newNode = new Node(newItem,top); // 리스트 앞부분에 삽입, 현재 top 노드를 참조
        top = newNode; // top이 새 노드를 가리킴
        size++; // 스택 항목 수 1 증가
    }
    
    public E pop() { // 스택 pop 연산
        if(isEmpty()) throw new EmptyStackException();
        E topItem = top.getItem(); // 스택 top 항목을 top item에 저장
        top = top.getNext(); // top이 top 아래의 항목을 가리킴
        size--;
        return topItem;
    }
}
```

스택 자료구조는 미로 찾기, 트리의 방문 그래프의 깊이 우선 탐색(DFS)를 수행하는데 기본이 되는 자료구조이다. 또한 함수(메소드)를 호출 및 재귀호출도 스택을 바탕으로 구현된다.

# 수행시간 

배열로 구현한 스택의 push와 pop 연산은 각각 $O(1)$ 시간이 소요된다. 그러나 배열 크기를 확대 또는 축소시키는 경우에 스택의 모든 item들을 새 배열로 복사해야 하므로 $O(N)$ 시간이 소요된다.하지만 상각분석에 따르면 $O(1)$ 시간이다. 단순 연결리스트로 구현한 스택의 push와 pop 연산은 각각 $O(1)$ 시간이 걸리는데 연결리스트의 앞 부분에서 노드를 삽입하거나 삭제하기 때문이다.

# 용례

- 문서편집기 되돌리기(undo)
- 컴파일러의 괄호 짝 맞추기
- 회문 검사하기
- 후위표기법 수식 계산하기
- 중위표기법 수식을 후위표기법 수식으로 변환하기
- 미로찾기
- 트리의 노드 방문
- 그래프 깊이우선탐색


## 시스템 스택(Stack) 함수 호출)
- 시스템 스택(system stack) : 운영체제가 사용하는 스택

 

복귀주소 기억
: 함수의 실행이 끝나면 자신을 호출한 함수로 되돌아갑니다. 이 때 스택은 복귀할 주소를 기억하는 데 사용됩니다. 함수는 호출된 역순으로 되돌아가야 하기 때문입니다.

![image](https://user-images.githubusercontent.com/66561524/209828882-46be6261-361f-424f-9646-a7a411a13b7b.png)

활성 레코드
: 시스템 스택에는 함수가 호출될 때마다 활성 레코드(activation record)가 만들어지며, 이곳에 복귀 주소가 저장됩니다. 활성 레코드에 생성되는 정보는 아래와 같습니다.
    - 복귀 주소
    - 프로그램 카운터
    - 매개변수
    - 함수 안에서 선언된 지역변수
 

시스템 스택에는 아래 그림과 같은 순서로 활성 레코드가 만들어졌다가 없어지게 됩니다. 

![image](https://user-images.githubusercontent.com/66561524/209828978-5be11e69-665f-40f4-a619-e7751cb54d85.png)

