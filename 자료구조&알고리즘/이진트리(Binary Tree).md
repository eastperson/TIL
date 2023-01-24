### 이진트리(Binary Tree)

- 트리중에서 널리 사용되는 트리 자료구조는 이진 트리와 이진 탐색 트리(Binary Search Tree, BST)다.
- 각 노드가 m개 이하의 자식을 갖고 있으면 m-ary 트리(다항트리, 다진 트리)라고 한다.

이진트리는 각 노드의 자식 수가 2 이하인 트리이다. 이진트리는 데이터의 구조적인 관계를 잘 반영하고 효율적인 삽입과 탐색을 가능하게 하며 이진트리의 서브트리를 다른 이진트리의 서브트리와 교환하는 것이 쉽기 때문이다.

![image](https://user-images.githubusercontent.com/66561524/212500360-a59a093f-cdd9-46d5-8f70-067a2475d451.png)

> 이진트리는 empty이거나, empty가 아니면 루트노드와 2개의 이진트리인 왼쪽 서브트리와 오른쪽 서브트리로 구성된다.

이진트리에는 두 종류의 특별한 형태를 가진 트리가 존재한다. 하나는 포화이진트리(Full Binary Tree)이고 다른 하나는 완전이진트리(Complete Binary Tree)이다. 포화이진트리는 모든 이파리노드의 깊이가 같고 각 내부노드가 2개의 자식노드를 가지는 트리이다. 완전이진트리는 마지막 레벨을 제외한 각 레벨이 노드들로 꽉 차있고 마지막 레벨에는 노드들이 왼쪽부터 빠짐없이 트리이다.

**포화이진트리(Full Binary Tree)**
![image](https://user-images.githubusercontent.com/66561524/212500373-4ff542e6-45a8-4877-9340-3927be37d769.png)

모든 내부 노드가 자식 노드를 가지며 모든 이파리 노드가 같은 깊이를 가지고 있는 것이다.

**완전이진트리(Complete Binary Tree)**

![image](https://user-images.githubusercontent.com/66561524/212500381-37107c4b-4e6f-4a5d-9693-4d1387a86368.png)

마지막 레벨을 제외하고 모든 레벨이 완전히 채워져 있는 트리이다. 마지막 레벨은 꽉 차 있지 않아도 되지만 노드가 왼쪽에서 오른쪽으로 채워져야 한다.

**이진트리 연산들의 수행시간**

- 레벨 k에 있는 최대 노드 수는 $2^{k-1}$이다. 단, k=1,2,3, ... 이다.
- 높이가 h인 포화이진트리에 있는 노드 수는 $2^h-1$이다.
- N개의 노드를 가진 완전이진트리의 높이는 $[log_2(N+1)]$이다.

![image](https://user-images.githubusercontent.com/66561524/212500385-db8f30a4-4a0e-4e54-99e0-8d9d3dab603f.png)

이진트리는 1차원 배열이나 단순연결리스트를 확장하여 각 노드에 2개의 레퍼런스를 사용하여 저장할 수 있다.1차원 배열 a를 사용하는 경우에는 a[0]은 사용하지 않고 트리의 레벨 1부터 내려가며 각 레벨에서는 좌에서 우로 트리의 노드들을 a[1]부터 차례로 저장한다.

![image](https://user-images.githubusercontent.com/66561524/212500390-0904efe5-a160-4bc7-97d7-ee3551bbf05b.png)

이렇게 이진트리를 배열에 저장하면 노드의 부모노드와 자식노드가 배열의 어디에 저장되어있는지를 알 수 있는 규칙이 있다.

**배열로 구현한 이진트리 노드 탐색 방법**

- a[i]의 부모노드는 a[i/2]에 있다. 단, i > 1이다.
- a[i]의 왼쪽 자식노드는 a[2i]에 있다. 단, 2i ≤ N이다.
- a[i]의 오른쪽 자식노드는 a[2i+1]에 있다. 단, 2i + 1 ≤N이다.

완전트리를 저장하기 위해 배열을 사용하는 경우 레퍼런스를 저장할 메모리공간이 필요 없기 때문에 효율적이다.

![image](https://user-images.githubusercontent.com/66561524/212500394-5659ae2b-cac9-4e32-a0a3-06fcbc695db6.png)

하지만 편향(Skewed) 이진트리를 배열에 저장하는 경우 트리의 높이가 커질수록 메모리 낭비가 커진다.

![image](https://user-images.githubusercontent.com/66561524/212500403-bece93a0-8546-41b4-9a46-186ee9dfcdb6.png)

일반적인 경우 이진트리의 노드는 키와 2개의 레퍼런스 필드를 가진다.

![image](https://user-images.githubusercontent.com/66561524/212500406-2a38066f-3fe8-479b-b4aa-acf9bae5caf0.png)

### 이진트리의 연산

---

이진트리에서 수행되는 기본 연산들은 트리를 순회(Traversal)하며 이루어진다. 이진트리를 순회하는 방식은 아래의 네 가지다. 방식은 각각 다르지만 순회는 항상 트리의 루트노드로부터 시작한다.  루트노드로부터 순회를 시작하여 트리의 모든 노드들을 반드시 1번씩 방문해야 순회가 종료된다.

**노드**

```java
package List.src.main.java;

public class NodeForTree<Key extends Comparable<Key>> {
    
    private Key item;
    private Node<Key> left;
    private Node<Key> right;
    public NodeForTree(Key newItem, Node lt, Node rt) {
        this.item = newItem;
        this.left = lt;
        this.right = rt;
    }
    public Key getKey() {return item;}
    public Node<Key> getLeft() {return left;}
    public Node<Key> getRight() {return right;}
    public void setKey(Key newItem) {item = newItem;}
    public void setLeft(Node<Key> lt){left = lt;}
    public void setRight(Node<Key> rt) {right = rt;}
    
}
```

- 전위순회(Preorder Traversal)
- 중위순회(Inorder Traversal)
- 후위순회(Postorder Traversal)
- 레벨순회(Levelorder Traversal)

![image](https://user-images.githubusercontent.com/66561524/212500424-572ad982-5775-45b5-88cb-a3847abf69c3.png)

```
전위 순회 Preorder Traversal 
root -> left -> right 
부모노드 -> 왼쪽 자식 노드 -> 오른쪽 자식 노드
8 9 4 10 11 5 2 12 13 6 14 15 7 3 1
```

```java
    public void preorder(NodeForTree n) { // 전위순회
        if(n != null) {
            System.out.print(n.getKey() + " "); // 노드 n 방문
            preorder(n.getLeft()); // n의 왼쪽 서브트리를 순회하기 위해
            preorder(n.getRight()); // n의 오른쪽 서브트리를 순회하기 위해
        }
    }
```
![image](https://user-images.githubusercontent.com/66561524/212500434-3206c686-3d60-409a-a433-3c6ec5dc522b.png)

```
중위 순회 Inorder Traversal
left -> root -> right
왼쪽 자식 노드  -> 부모노드 -> 오른쪽 자식 노드
8 9 4 10 11 5 2 12 13 6 14 15 7 3 1
```

```java
public void inorder(NodeForTree n) { // 중위순회
        if(n != null) {
            inorder(n.getLeft()); // n의 왼쪽 서브트리를 순회하기 위해
            System.out.print(n.getKey() + " "); // 노드 n 방문
            inorder(n.getRight()); // n의 오른쪽 서브트리를 순회하기 위해
        }
    }
```

![image](https://user-images.githubusercontent.com/66561524/212500439-7fe1b7c1-d655-4112-a740-b2108aa1ecbe.png)

```
후위 순회 Postorder Traversal
left -> right -> root
왼쪽 자식 노드 -> 오른쪽 자식 노드 -> 부모노드
8 9 4 10 11 5 2 12 13 6 14 15 7 3 1
```

```java
public void postorder(NodeForTree n) { // 후위순회
        if(n != null) {
            postorder(n.getLeft()); // n의 왼쪽 서브트리를 순회하기 위해
            postorder(n.getRight()); // n의 오른쪽 서브트리를 순회하기 위해
            System.out.print(n.getKey() + " "); // 노드 n 방문
        }
    }
```
![image](https://user-images.githubusercontent.com/66561524/212500445-d245f9c1-006b-488a-95c5-99c5b9ab0439.png)

레벨순회는 루트노드가 있는 최상위 레벨부터 각 레벨마다 좌에서 우로 노드를 방문한다. 레벨순회는 큐자료구조를 활용하여 LinkedList를 사용해 구현한 Queue를 사용한다. 

```java
public void levelorder(NodeForTree root) { // 레벨 순회
        Queue<NodeForTree> q = new LinkedList<NodeForTree>(); // 큐 자료구조 이용, bfs
        NodeForTree t;
        q.add(root); // 루트 노드 큐에 삽입
        while(!q.isEmpty()){
            t = q.remove(); // 큐에서 가장 앞에 있는 노드 제거
            System.out.print(t.getKey() + " "); // 제거된 노드 출력(방문)
            if(t.getLeft() != null) { // 제거된 왼쪽 자식이 null이 아니면
                q.add(t.getLeft()); // 큐에 왼쪽 자식 삽입
            }
            if(t.getRight() != null) { // 제거된 오른쪽 자식이 null이 아니면
                q.add(t.getRight()); // 큐에 오른쪽 자식 삽입
            }
        }
    }
```

이외의 연산

`size()`와 `height()`는 후위순회에 기반하고 `isEqual()`은 전위순회에 기반한다. 2개의 이진트리를 비교하는 것은 다른 부분을 발견하는 즉시 비교연산을 멈추기 위해 전위순회 방법을 사용한다.

**트리의 노드 수**

> 트리의 노드 수 = 1 + (루트노드의 왼쪽 서브트리에 있는 노드 수) + (루트노드의 오른쪽 서브트리에 있는 노드 수)이다. 여기서 1은 루트노드 자신을 계산에 반영하는 것이다.

```java
public int size(NodeForTree n) { // n을 루트로 하는 (서브)트리에 있는 노드 수
    if(n==null) return 0;
    else return (1 + size(n.getLeft())) + size(n.getRight());
}
```

**트리의 높이**

> 트리의 높이 = 1+ max(루트노드의 왼쪽 서브트리의 높이, 루트노드의 오른쪽 서브트리의 높이)이다. 여기서 1은 루트노드 자신을 계산에 반영하는 것이다.

```java
public int height(NodeForTree n) { // n을 루트로하는 (서브)트리의 높이
    if (n == null) return 0;
    else return (1 + Math.max(height(n.getLeft()), height(n.getRight())));
}
```

**이진트리 비교**

> 전위순회 과정에서 다른 점이 발견되는 순간 false를 리턴한다.

```java
public static boolean isEqual(NodeForTree n, NodeForTree m) { // 두 트리의 동일성 검사
    if(n == null || m == null) return n == m; // 둘 다 null이면 true, 아니면 false

    if(n.getKey().compareTo(m.getKey()) != 0) return false; // 둘다 null이 아니면 item 비교

    // item이 같으면 왼쪽/오른쪽 자식으로 재귀호출
    return (isEqual(n.getLeft(),m.getLeft()) && isEqual(m.getRight(),m.getRight()));
}
```

**수행시간**

앞서 설명된 각 연산은 트리의 각 노드를 한 번씩만 방문하므로 $O(N)$ 시간이 소요된다.

이진 트리

```java
package List.src.main.java;

import java.util.LinkedList;
import java.util.Queue;

public class MyBinaryTree<Key extends Comparable<Key>> {
    private NodeForTree root;
    public MyBinaryTree() {root = null;} // 트리 생성자
    public NodeForTree getRoot() {return root;}
    public void setRoot(NodeForTree newRoot){root = newRoot;}
    public boolean isEmpty(){return root == null;}

    public void preorder(NodeForTree n) { // 전위순회
        if(n != null) {
            System.out.print(n.getKey() + " "); // 노드 n 방문
            preorder(n.getLeft()); // n의 왼쪽 서브트리를 순회하기 위해
            preorder(n.getRight()); // n의 오른쪽 서브트리를 순회하기 위해
        }
    }

    public void inorder(NodeForTree n) { // 중위순회
        if(n != null) {
            inorder(n.getLeft()); // n의 왼쪽 서브트리를 순회하기 위해
            System.out.print(n.getKey() + " "); // 노드 n 방문
            inorder(n.getRight()); // n의 오른쪽 서브트리를 순회하기 위해
        }
    }

    public void postorder(NodeForTree n) { // 후위순회
        if(n != null) {
            postorder(n.getLeft()); // n의 왼쪽 서브트리를 순회하기 위해
            postorder(n.getRight()); // n의 오른쪽 서브트리를 순회하기 위해
            System.out.print(n.getKey() + " "); // 노드 n 방문
        }
    }

    public void levelorder(NodeForTree root) { // 레벨 순회
        Queue<NodeForTree> q = new LinkedList<NodeForTree>(); // 큐 자료구조 이용, bfs
        NodeForTree t;
        q.add(root); // 루트 노드 큐에 삽입
        while(!q.isEmpty()){
            t = q.remove(); // 큐에서 가장 앞에 있는 노드 제거
            System.out.print(t.getKey() + " "); // 제거된 노드 출력(방문)
            if(t.getLeft() != null) { // 제거된 왼쪽 자식이 null이 아니면
                q.add(t.getLeft()); // 큐에 왼쪽 자식 삽입
            }
            if(t.getRight() != null) { // 제거된 오른쪽 자식이 null이 아니면
                q.add(t.getRight()); // 큐에 오른쪽 자식 삽입
            }
        }
    }

    public int size(NodeForTree n) { // n을 루트로 하는 (서브)트리에 있는 노드 수
        if(n==null) return 0;
        else return (1 + size(n.getLeft())) + size(n.getRight());
    }

    public int height(NodeForTree n) { // n을 루트로하는 (서브)트리의 높이
        if (n == null) return 0;
        else return (1 + Math.max(height(n.getLeft()), height(n.getRight())));
    }

    public static boolean isEqual(NodeForTree n, NodeForTree m) { // 두 트리의 동일성 검사
        if(n == null || m == null) return n == m; // 둘 다 null이면 true, 아니면 false

        if(n.getKey().compareTo(m.getKey()) != 0) return false; // 둘다 null이 아니면 item 비교

        // item이 같으면 왼쪽/오른쪽 자식으로 재귀호출
        return (isEqual(n.getLeft(),m.getLeft()) && isEqual(m.getRight(),m.getRight()));
    }
}
```

테스트 코드

```java
public class Main{
    public static void main(String[] args) {
        NodeForTree n1 = new NodeForTree(100,null,null);
        NodeForTree n2 = new NodeForTree(200,null,null);
        NodeForTree n3 = new NodeForTree(300,null,null);
        NodeForTree n4 = new NodeForTree(400,null,null);
        NodeForTree n5 = new NodeForTree(500,null,null);
        NodeForTree n6 = new NodeForTree(600,null,null);
        NodeForTree n7 = new NodeForTree(700,null,null);
        NodeForTree n8 = new NodeForTree(800,null,null);

        n1.setLeft(n2);
        n1.setRight(n3);
        n2.setLeft(n4);
        n2.setRight(n5);
        n3.setLeft(n6);
        n3.setLeft(n7);
        n4.setLeft(n8);

        MyBinaryTree t = new MyBinaryTree();
        t.setRoot(n1);

        System.out.println("트리 노드 수 : " + t.size(t.getRoot()));
        System.out.println("트리 높이 수 : " + t.height(t.getRoot()));
        System.out.print("전위 순회 : ");
        t.preorder(t.getRoot());
        System.out.println();

        System.out.print("중위 순회 : ");
        t.inorder(t.getRoot());
        System.out.println();

        System.out.print("후위 순회 : ");
        t.postorder(t.getRoot());
        System.out.println();

        System.out.print("레벨 순회 : ");
        t.levelorder(t.getRoot());
        System.out.println();

        // 두번째 이진트리를 만들어 isEqual() 테스트하기 위해
        NodeForTree n10 = new NodeForTree(100,null,null);
        NodeForTree n20 = new NodeForTree(200,null,null);
        NodeForTree n30 = new NodeForTree(300,null,null);
        NodeForTree n40 = new NodeForTree(400,null,null);
        NodeForTree n50 = new NodeForTree(500,null,null);
        NodeForTree n60 = new NodeForTree(600,null,null);
        NodeForTree n70 = new NodeForTree(700,null,null);
        NodeForTree n80 = new NodeForTree(800,null,null);

        n10.setLeft(n20);
        n10.setRight(n30);
        n20.setLeft(n40);
        n20.setRight(n50);
        n30.setLeft(n60);
        n30.setRight(n70);
        n40.setLeft(n80);

        MyBinaryTree t2 = new MyBinaryTree();
        t2.setRoot(n10);

        System.out.println("동일성 검사 : " + MyBinaryTree.isEqual(t.getRoot(),t2.getRoot()));

    }
}
```

```java
트리 노드 수 : 7
트리 높이 수 : 4
전위 순회 : 100 200 400 800 500 300 700 
중위 순회 : 800 400 200 500 100 700 300 
후위 순회 : 800 400 500 200 700 300 100 
레벨 순회 : 100 200 300 400 500 700 800 
동일성 검사 : true
```

**스레드 이진트리**

앞서 소개한 모든 이진트리는 레벨순회를 제외하고 모두 스택 자료구조를 사용한다. 메소드의 재귀호출은 시스템 스택을 사용하므로 스택 자료구조를 사용한 것으로 간주한다. 스택에 사용되는 메모리 공간의 크기는 트리의 높이에 비례한다.

스택없이 이진트리의 연산을 구현하는 방법에는 두 가지가 있다. 하나는 Node에 부모노드를 가리키는 레퍼런스 필드를 추가로 선언하여 순회에 사용하는 방법이다. 다른 하나는 노드의 null 레퍼런스를 활용 하는 것이다. null 레퍼런스 공간에 다음에 방문할 노드의 레퍼런스를 저장하는 것이다. 이렇게 만든 이진트리를 **스레드 이진트리(Threaded Binary Tree)**라고 한다.

![image](https://user-images.githubusercontent.com/66561524/212500457-8f9543fb-3fc2-46f7-937f-f30d90a70c6c.png)

N개의 노드가 있는 이진트리에는 각 노드마다 2개의 레퍼런스 필드(left와 right)가 있으므로 총 2N개의 레퍼런스 필드가 존재한다. 이중에서 부모 자식을 연결하는 레퍼런스는 N-1개이기 때문에 N+1개의 null 레퍼런스 필드가 있다. 레퍼런스가 N-1개인 이유는 루트 노드를 제외한 각 노드가 1개의 부모노드를 갖기 떄문이다.

 스레드 이진트리는 N+1개의 null 레퍼런스를 활용하여 이전에 방문한 노드와 다음에 방문할 노드를 가리키도록 만들어 순회 연산이 스택 없이도 수행될 수 있도록 만든 트리앋. 스레드 이진트리는 대부분의 경우 중위순회에 기반하여 구현된다. 하지만 전위순회나 후위순회에 기반하여 스레드 트리를 구현할 수 있다.

### 상호배타적 집합을 위한 트리 연산

합집합(union) 연산과 주어진 원소에 대해 어느 집합에 속해 있는지를 계산하는 find 연산을 다룬다. 특히 어느 두 집합도 중복된 원소를 갖지 않는 경우 이러한 집합들을 **상호배타적 집합(Disjoint Set)**이라고 한다. 상호배타적 집합의 `union`과 `find` 연산은 Kruskal의 최소신장트리 알고리즘을 구현하는데 사용된다.

`union` 연산은 2개의 집합을 하나의 집합으로 만드는 연산이다. `find` 연산은 인자로 주어지는 x가 속한 집합의 루트노드를 찾는 연산이다.

![image](https://user-images.githubusercontent.com/66561524/212500482-1710d317-7ea1-49d4-b091-42c059ada72e.png)

> 먼저 union 연산은 rank에 기반하여(union-by-rank) rank가 높은 루트노드가 union 후에도 승자(합쳐진 트리의 루트노드)가 되도록한다.

- rank : 트리의 높이

rank가 높은 루트노드를 승자로 만드는 이유는 합쳐진 트리가 더 커지지 않도록 하기 위함이다. 만일 두 트리의 높이가 같은 경우에는 둘 중 하나의 루트노드가 승자가 되고 합쳐진 트리의 높이는 1 증가한다.

![image](https://user-images.githubusercontent.com/66561524/212500492-82cc3188-ed92-4893-ba4d-77cc66ff00ec.png)

rank에 기반한 union 연산의 목적은 두 트리가 하나로 합쳐진 후에 트리의 높이가 커지는 것을 방지하기 위함이다. find 연산을 수행할 때 루트노드까지 올라가야 하므로 트리의 높이가 낮을수록 find의 수행시간을 줄일 수 있다.

- Union 연산

![image](https://user-images.githubusercontent.com/66561524/212500497-798dd5fd-0a1c-4474-8e0f-c798ec0d345d.png)
![image](https://user-images.githubusercontent.com/66561524/212500502-3a79e18b-d371-4ed4-ba71-a84c0788372d.png)

- Find 연산

**수행시간**

union 연산은 두 루트노드를 각각 찾는 find 연산을 수행하고 rank를 비교하여 승자가 합쳐진 트리의 루트노드로 남는다. rank가 같은 경우엔 둘 중에 하나의 승자가 되고 승자의 rank를 1 증가시킨다. find 연산을 제외한 순수 union 연산의 수행시간은 $O(1)$이다.

find 연산은 최대 트리의 높이만큼 올라가야 하므로 트리의 높이에 비례한다. find 연산을 수행하며 경로압축을 하므로 경로 상의 노드에 대해 수행되는 find 연산의 수행시간은 트리의 높이보다는 적게 걸린다. 상각분석의 결과는 $O(logN)$이다.
