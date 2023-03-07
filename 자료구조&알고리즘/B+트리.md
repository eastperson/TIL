![image](https://user-images.githubusercontent.com/66561524/223460668-1b837d24-e55c-4219-9758-45a395c327c6.png)
![image](https://user-images.githubusercontent.com/66561524/223460694-c81fcea8-0d5b-44b7-90b7-32e5bdf3ca11.png)

B+ 트리는 B 트리의 일종으로, 데이터베이스 인덱스를 구현하는 데 자주 사용되는 자료 구조입니다. B+ 트리는 B 트리와 유사하지만, 몇 가지 차이점이 있습니다.

B+ 트리에서는 내부 노드와 리프 노드가 구분됩니다. 내부 노드는 다른 노드를 가리키는 포인터만을 가지고 있고, 리프 노드는 데이터를 저장하며, 이 데이터는 내부 노드에서도 중복되지 않습니다. 또한 리프 노드들은 오름차순으로 연결 리스트 형태로 연결되어 있습니다.

B+ 트리는 많은 양의 데이터를 저장할 때 효율적인 검색과 범위 검색을 가능하게 해줍니다. B+ 트리는 높이가 낮아 검색 시간이 빠르고, 리프 노드들이 연결 리스트 형태로 연결되어 있기 때문에 범위 검색을 할 때 유용합니다. 또한 B+ 트리는 데이터의 삽입과 삭제가 빈번하게 일어날 때도 성능이 좋습니다.

- BST는 편향이 발생하면 O(N)에 가까워진다.
![image](https://user-images.githubusercontent.com/66561524/223460999-8ec5d581-5511-4c02-ba47-014dd547c22a.png)
- 균형을 맞추는 AVL tree와 Red-Black Tree는 왜 index로 사용되지 않을까?
![image](https://user-images.githubusercontent.com/66561524/223461050-cb6f1435-b2d6-49b9-8779-23fa87418987.png)
- 컴퓨터를 껐다가 켜도 저장되는게 영구적(비휘발성). 메인 메모리는 휘발성.
- 메인 메모리는 용량이 작아 실행 중인 프로그램의 데이터 중 잘 사용되지 않는 데이터는 Secondary storage에 저장. 이것을 swap이라고 함. 사용이 필요할 때 올려놓음.
![image](https://user-images.githubusercontent.com/66561524/223461116-9f55853f-fafb-4577-97b6-6b336e798936.png)
- 데이터를 저장하는 용량은 가장 크다.
- block 단위로 데이터를 읽고 쓴다.
    - 불필요한 데이터까지 읽어올 가능성이 있다.
![image](https://user-images.githubusercontent.com/66561524/223461176-2e43445b-b049-40bb-9e42-06f58a001a85.png)
- DB는 secondary storage 에 저장된다.
    - 많이 쓰이는 데이터는 Main Memory에 저장한다.
- DB에서 데이터를 조회할 때 secondary storage에 최대한 적게 접근하는 것이 성능 면에서 좋다.
- block 단위로 읽고쓰기 때문에 연관된 데이터를 모아서 저장하면 더 효율적으로 읽고 쓸 수 있다.

AVL tree index(b)
![image](https://user-images.githubusercontent.com/66561524/223461247-b62b58d3-0d7c-4ad8-b210-182016189e45.png)
- secondary storage 4번 접근
- 자녀 노드수 1~2개
- 노드의 데이터 수 1개
- 차수를 내려갈 때 마다 secondary storage를 조회

5차 B tree index(b)
![image](https://user-images.githubusercontent.com/66561524/223461316-3af2aaf5-6049-49c5-a1a4-e99b875cf30e.png)
- secondary storage 2번 접근
- 자녀 노드수 3~5개
- 노드의 데이터 수 2~4개
- 데이터를 찾을 때 탐색 범위를 빠르게 좁힐 수 있다. 리프노드에 도달하는 속도가 빠르다.
![image](https://user-images.githubusercontent.com/66561524/223461367-1f57c977-91e2-497a-8e6f-1c4389ec1eb2.png)
- block 단위에 대한 저장 공간 활용도가 더 좋다.
![image](https://user-images.githubusercontent.com/66561524/223461409-4d622834-867f-4dac-9b67-799f4388c901.png)
![image](https://user-images.githubusercontent.com/66561524/223461429-cec18202-e033-46ad-8229-cf443b488d84.png)
- 높이 3으로 1억개의 데이터를 저장한다.
![image](https://user-images.githubusercontent.com/66561524/223461523-6d64fa6f-f8e1-40f6-bbf7-2bb9c3c1f731.png)
- worst case여도 265301개를 저장가능
![image](https://user-images.githubusercontent.com/66561524/223461605-6dfe3283-5b92-419a-9858-fdb24640c9ac.png)




**B tree 계열을 DB 인덱스로 사용하는 이유**

- DB는 기본적으로 secondary storage에 저장
- B tree index는 self-balancing BST에 비해 secondary storage 접근을 적게 한다.
- B tree 노드는 block 단위의 저장공간을 알차게 사용할 수 있다.

[(3부) B tree가 왜 DB 인덱스(index)로 사용되는지를 설명합니다](https://www.youtube.com/watch?v=liPSnc6Wzfk&t=1712s)
