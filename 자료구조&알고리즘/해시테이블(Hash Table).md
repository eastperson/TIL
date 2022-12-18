# Overview

Java에서 데이터를 조회하는 가장 빠른 자료구조로 배열(Array)을 사용한다. 배열는 특정 원소에 접근할 때 배열의 인덱스를 이용하여 `O(1)` 시간에 접근할 수 있다. 이 방식을 통해 원소를 찾기 위해 데이터를 저장하기 위해 key를 인덱스로 사용하면 key를 통한 원소 접근을 `O(1)` 시간에 접근할 수 있다.  이때 key를 인덱스로 사용하면 사이에 메모리 공간이 남게되므로 값을 특정한 인덱스로 적절히 변환해서 관리하며 key에 맞는 value를 저장한다. 이 자료구조를 해시 테이블(Hash Table)이라고 한다.

![image](https://user-images.githubusercontent.com/66561524/208306258-05a1974c-eaee-4d68-ab0b-70309bbff50e.png)

- key를 특정한 인덱스로 적절히 변환하는 방법을 **해싱(Hashing)**이라고 한다.
- 해싱에 사용되는 함수를 **해시함수(Hash Function)**이라고 한다.
- 해시함수가 계산한 값을 **해시값(Hash value)** 또는 **해시주소(Hash Address)**라고 한다.
- 항목이 해시값에 따라 저장되는 데이터 구조를 **해시테이블(Hash Table)**이라고 한다.
- 해시테이블에서 하나의 데이터가 저장되는 공간을 **버킷(Bucket)** 또는 **슬롯(Slot)**이라고 한다.

# Direct-address table

![image](https://user-images.githubusercontent.com/66561524/208306270-c7d036c3-6231-4dc6-89cb-f406f98f1fa3.png)

- 키의 전체 개수와 동일한 크기의 버킷을 가진 해시테이블을 뜻한다.
- 키 개수와 해시테이블 크기가 동일하기 때문에 해시충돌문제가 발생하지 않는다.
- 하지만 전체 키(unverse of key)가 실제 사용하는 키(actual key)보다 많은 경우 사용하지 않는 키들을 위한 공간까지 마련해야 하는 탓에 메모리 효율성이 크게 떨어진다.
- 보통의 경우 해시테이블 크기(m)가 실제 사용하는 키 개수(n)보다 적은 해시테이블을 운용한다.
- 이 때 n/m을 **load factor**라고 한다.
    - 해시 테이블의 한 버킷에 평균 몇 개의 키가 매핑되는가를 나타내는 지표이다.
    - Direct-address table의 load factor는 1 이하이며, 1보다 큰 경우 해시충돌 문제가 발생

# 해시함수(Hash Function)

가장 이상적인 해시함수는 키들을 균등하게(Uniformly) 해시테이블의 인덱스로 변환하는 함수이다.

일반적으로 실세계의 키들은 부여된 의미나 특성을 가지고 있다. 따라서 키의 부분을 이용해서 해시값으로 사용하는 단순한 방식의 해시함수는 많은 충돌을 야기할 수 있다.

따라서 의미가 있는 키를 뒤죽박죽 만들어 해시테이블의 크기에 맞도록 해시값을 계산한다. 하지만 균등한 결과를 보장하는 해시함수라도 함수 계산 자체에 긴 시간이 소요된다면 해싱의 장점인 신속성을 상실한다. 단순하면서 동시에 키들을 균하게 변환하는 함수가 햄시함수로 바람직하다.

**해시함수**

- 중간제곱(Mid-square) 함수: 키를 제곱한 후 적절한 크기의 중간부분을 해시값으로 사용.
- 접기(Folding) 함수: 큰 자릿수를 갖는 십진수를 키로 사용하는 경우 몇자리씩 일정하게 끊어서 만든 숫자들의 합을 이용해 해시값을 만든다.
- 곱셈(Multiplicative) 함수: 1보다 작은 실수를 키에 곱하여 얻은 숫자의 소수부분을 테이블 크기와 곱한다. 이렇게 나온 값의 정수 부분을 해시값으로 사용한다.

위 함수의 공통점은 키의 모든 자리의 숫자들이 함수 계산으로 참여함으로써 계산 결과에서는 원래의 키에 부여된 의미나 특성을 찾아볼 수 없게 된다는 점을 들 수 있다.

- 나눗셈(Division) 함수: 나눗셈 함수는 키를 소수(Prime)로 나눈 뒤 나머지를 해시값으로 사용한다. 제수로 소수를 사용하는 이유는 나눗셈 연산을 했을 때 소수가 키들을 균등하게 인덱스로 변환시키는 성질을 갖는다.

실세계에서 가장 널리 사용되는 해시함수다.

# 해시충돌(Hash Collision)

우수한 해시함수를 사용하더라도 두 개 이상의 항목을 해시테이블의 동일한 원소에 저장해야하는 경우가 발생한다. 이는 ‘비둘기집 원리’(*n*+1개의 물건을 n개의 상자에 넣은 경우, 최소한 한 상자에는 그 물건이 반드시 두 개 이상 들어있다는 원리, Pigeonhole Principle)로 인해 무조건적으로 발생할 수 밖에 없는 현상이다.

![image](https://user-images.githubusercontent.com/66561524/208306289-1f8b0728-7a44-46e9-869c-117a14f865fa.png)

이를 해결하기 위한방법은 크게 **개방주소방식(Open Hashing)**과 **폐쇄주소방식(Closed Addressing)**으로 나누어진다.

## 개방주소방식(Open Addressing)

![image](https://user-images.githubusercontent.com/66561524/208306303-f8ecfcdc-4c80-45ab-a8ee-deafcc967119.png)

해시테이블 전체를 열린 공간으로 가정하고 해시 충돌이 일어날 경우 비어있는 hash를 찾아 데이터를 저장하는 기법이다. 따라서 개방주소방식에는 1개의 해시에는 1개의 value만 저장을 하게 된다. 이때 비어있는 hash를 찾는 방법은 대표적으로 선형조사(Linear Probing), 이차조사(Quadratic Probing), 랜덤조사(Random Probing), 이중해성(Double Hashing) 등이 있다.

### 선형조사(Linear Probing)

![image](https://user-images.githubusercontent.com/66561524/208306310-551c3442-2c85-41b0-bd78-8ab8614da4c9.png)

- 충돌이 일어난 원소에서부터 순차적으로 검색하여 처음 발견한 empty 원소에 충돌이 일어난 키를 저장한다.
    - 순차탐색으로 empty 원소를 찾아 충돌된 키를 저장하면 해시테이블의 키들이 빈틈없이 뭉쳐지는 현상이 발생한다. 이를 1차 군집화(Primary Clustering)이라고 한다. 해시테이블에 empty 원소 수가 적을수록 심화되어 해시성능을 저하시킨다.
- 해당 해시값에 대해 고정폭(ex, 1칸)으로 옮겨 다음 해시값에 해당하는 버킷에 액세스한다. 여기에 데이터가 있으면 또 옮겨 액세스한다.

### 이차조사(Quadratic Probing)

![image](https://user-images.githubusercontent.com/66561524/208306321-6ee644d3-aa07-478a-bbad-9e2e53500748.png)

- 이차조사는 선형조사와 근본적으로 동일한 충돌해결 방법이다. 다만 고정 폭으로 이동하는 선형탐사와 달리 그 폭이 제곱수로 늘어간다는 특징이 있다.
- 1차 군집화 문제는 해겨하지만 같은 해시값을 갖는 서로 다른 키들인 동의어(Synonym)들이 똑같은 점프 시퀀스(Jump Sequence, 처음 충돌이 발생한 위치로부터 연속적으로 그 다음 위치를 찾아가는 순서)를 따라 원소를 찾아 저장하므로 또 다른 형태의 군집화인 2차 군집화(Secondary Clustering)에 취약하다.

### 랜덤조사(Random Probing)

![image](https://user-images.githubusercontent.com/66561524/208306329-082cb610-057f-4cb7-8c9c-46d6f35fd8a7.png)

- 랜덤조사는 선형조사와 이차조사의 규칙적인 점프 시퀀스와는 달리 점프 시퀀스를 무작위화하여 empty 원소를 찾는 충돌해결방법이다.
- 랜덤조사는 의사 난수 생성기(Pseudo Random Number Generator)를 사용하여 다음 위치를 찾는다.
- 하지만 랜덤조사 방식도 동의어들이 똑같은 점프 시퀀스에 따라 empty 원소를 찾아 키를 저장하게 되고 이 때문에 2차 군집화와 3차 군집화(Tertiary Clustering)가 발생한다.

### 이중해싱(Double Hashing)

![image](https://user-images.githubusercontent.com/66561524/208306339-722561b5-2c61-49a2-93ed-9d3d4be3a318.png)

- 이중해싱(Double Hashing)은 두 개의 해시함수를 사용하는 충돌해결방법이다.
- 두 해시 함수 중 하나는 기본적인 해시함수로 키를 해시테이블의 인덱스로 변환하고 제2의 함수는 충돌 발생시 다음위치를 위한 점프 크기를 제2의 해시함수를 사용한다.
- 선형조사는 1차 군집화, 이중조사는 2차 군집화, 랜덤조사는 3차 군집화를 야기한다. 이중해싱은 동의어들이 저마다 제2의 해시함수를 갖기 때문에 점프 시퀀스가 일정하지 않는다. 따라서 모든 군집화 문제를 해결하는 충돌해결방법이다.

## 폐쇄주소방식(Closed Addressing)

키에 대한 해시값에 대응되는 곳에만 키를 저장한다.

### 체이닝(Chainig)

![image](https://user-images.githubusercontent.com/66561524/208306354-f7b8b39f-ef16-48c4-9dbc-6e521713cc9b.png)

- 연결리스트(linked list)를 이용하는 방법인 Chaining 방법을 이용해서 해결하는 방식이다.
- 해시테이블 크기인 M개의 연결리스트를 가지며 키를 해시값에 대응되는 연결리스트에 저장하는 해시방식이다.
- 개방주소방식처럼 해시테이블의 empty 원소를 찾는 오버헤드가 없고 군집화 현상이 없고 구현이 간결하다.

![image](https://user-images.githubusercontent.com/66561524/208306364-1287b12f-cf39-439c-900a-94fb18e4c4ff.png)

- 단, 각 버킷의 자료구조 형태는 연결리스트(Linked List)이기 때문에 데이터를 탐색하기 위한 시간복잡도는 최대 `O(n)`까지 늘어날 수 있다.
- 따라서 해시성능은 연결리스트의 평균 길이가 10정도일 때 좋은 성능이 보인다.

# Reference

---

[Hash에 대해서](https://lee33398.tistory.com/33)

[자료구조(Data Structure) 톺아보기 - 해시(Hash)](https://ai-rtistic.com/2022/01/29/data-structure-hash/)

[해싱, 해시함수, 해시테이블](https://ratsgo.github.io/data%20structure&algorithm/2017/10/25/hash/)

[Today-I-Learn/Java HashMap 동작원리.md at master · wjdrbs96/Today-I-Learn](https://github.com/wjdrbs96/Today-I-Learn/blob/master/Java/Collection/Map/Java%20HashMap%20%EB%8F%99%EC%9E%91%EC%9B%90%EB%A6%AC.md)

[NAVER D2](https://d2.naver.com/helloworld/831311)
