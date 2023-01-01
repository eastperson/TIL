# 데크(Deque)

데크(Double-ended Queue, Deque)는 양쪽 끝에서 삽입과 삭제를 허용하는 자료구조이다. 스택과 큐 자료구조를 혼합한 자료구조라고 할 수 있다.

![image](https://user-images.githubusercontent.com/66561524/210160908-28e0c1bc-3766-440b-8c05-525d5036fa88.png)

데크의 구현은 배열이나 연결 리스트 모두 가능하지만 이중 연결 리스트(Doubly Linked List)로 구현하는 편이 가장 잘 어울린다.

![image](https://user-images.githubusercontent.com/66561524/210160911-45f3864f-090e-4a31-80f1-4c15af3aeec5.png)

**수행시간**

데크를 배열이나 이중연결리스트로 구현한 경우 스택과 큐의 시간은 같다. 하지만 양 끝에서 삽입과 삭제가 가능하므로 프로그램이 다소 복잡하며 이중연결리스트로 구현한 경우는 더 복잡해진다.

### 용례

- 스크롤
- 문서 편집기의 undo 연산
- 웹 브라우저의 방문 기록
