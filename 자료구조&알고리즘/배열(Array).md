## 배열(Array)

배열은 동일한 타입의 원소들이 연속적인 메모리 공간에 할당되어 각 항목이 하나의 원소에 저장되는 기본적인 자료구조이다. 특정 원소에 접근할 때에는 배열의 인덱스를 이용해서 $O(1)$ 시간에 접근할 수 있다. 그러나 새 항목이 배열 중간에 삽입되거나 중간에 있는 항목이 삭제되면 항목들을 한 칸씩 뒤로 또는 앞으로 이동시켜야 하므로 삽입, 삭제 연산은 항상 $O(1)$ 시간에 수행할 수 없다.

배열은 미리 정해진 크기의 메모리 공간을 할당 받은 뒤 사용한다. 빈자리가 없어 새 항목을 삽입할 수 없는 상황(Overflow)에 직면할 수 있다.

> 배열의 경우 overflow가 발생할 때 크기를 2배로 확장하는 방식의 로직을 사용한다. 또한 배열의 3/4가 비어있으면 배열 크기를 1/2로 축소하는 로직도 같이 사용한다.

할당된 메모리 공간을 확장 또는 축소하는 배열을 동적배열(Dynamic Array)이라고 한다. 스택과 큐 자료구조에서도 프로그램의 안정성 향상과 메모리 공간 효율을 위해 동적 배열을 사용한다.

자바 배열의 메모리 구조는 아래의 그림과 같다. score[i]는 인덱스 i를 가지는 원소를 가리키는 레퍼런스이다. 각 원소의 레퍼런스는 별도로 저장하지 않고 a가 가지고 있는 레퍼런스에 원소의 크기(byte) * i를 더하여 a[i]의 레퍼런스를 계산한다. 즉 score + (원소의 크기 * i)이다.

![image](https://user-images.githubusercontent.com/66561524/208939832-b5da4b24-478e-4f10-a5c9-d68e52037d9d.png)

리스트를 배열로 구현한 것이 ArrayList이다. 

```java
package List.src.main.java;

import java.util.Arrays;
import java.util.NoSuchElementException;

public class MyArrayList <E> {
    private E a[]; // 리스트의 항목들을 저장할 배열
    private int size; // 리스트의 항목 수

    public E peek(int k) { // k번째 항목을 리턴, 단순히 읽기만 한다.
				// underflow 인 경우 프로그램 정지
        if(size == 0) throw new NoSuchElementException("underflow 발생!");
        return a[k];
    }

    public void insertLast(E newItem) { // 가장 뒤에 새 항목 삽입
        if (size == a.length) // 배열에 빈  공간이 없으면
            resize(2*a.length); // 배열 크기 2배로 확장
        a[size++] = newItem; // 새 항목 삽입
    }

    public void insert(E newItem, int k) { // 새 항목을 k-1번째 항목 다음에 삽입
        if(size == a.length) // 배열에 비공간이 없으면
            resize(2*a.length); // 배열 크기 2배로 확장

        for (int i = size - 1; i>= k; i--) // 한 칸씩 뒤로 이동
            a[i+1] = a[i];
        a[k] = newItem;
        size++;

    }

    public E delete(int k) { // k번째 항목 삭제
        if(isEmpty()) throw new NoSuchElementException(); // underflow 경우에 플로우 중지
        E item = a[k];
        for(int i = k; i < size; i++) a[i] = a[i+1]; // 한 칸씩 앞으로 이동
        size--;
        if (size > 0 && size == a.length/4) // 배열에 항목들이 1/4만 차지한다면
            resize(a.length/2);     // 배열을 1/2 크기로 축소
        return item;
    }

    private boolean isEmpty(){
        if (size == 0)
            return true;
        return false;
    }

    private void resize(int newSize){ // 배열의 크기 조절
        Object[] t = new Object[newSize]; // newSize 크기의 새로운 배열 t 생성
        for (int i = 0; i < size; i++)
            t[i] = a[i]; // 배열 s를 배열 t로 복사
        a = (E[]) t; // 배열 t를 배열 s로
    }

    public MyArrayList(){
        a = (E[]) new Object[1]; // 최초에 1개의 원소를 가진 배열 생성
        size = 0; // 항목 수를 0으로 초기화
    }

    public E[] getA() {
        return a;
    }

    public int getSize() {
        return size;
    }

    public void setA(E[] a) {
        this.a = a;
    }

    public void setSize(int size) {
        this.size = size;
    }

    @Override
    public String toString() {
        return "MyArrayList{" +
                "a=" + Arrays.toString(a) +
                ", size=" + size +
                '}';
    }
}
```

**수행시간**

탐색 연산을 수행하는 peek() 메소드는 인덱스를 이용하여 배열 원소를 직접 접근하므로 $O(1)$ 시간에 수행 가능하다. 그러나 삽입이나 삭제는 중간에 삽입하거나 중간에 있는 항목을 삭제한 후 뒤따르는 항목들을 한 칸씩 앞이나 뒤로 이동해야 한다. 이 때 각각 최악의 경우는 $O(N)$ 시간이 소요된다. 즉, 새 항목을 리스트의 가장 앞에 삽입하거나 첫 항목을 삭제하는 경우가 바로 최악의 경우이다. 단, 새 항목을 가장 뒤에 삽입하는 경우는 $O(1)$ 시간에 수행이 가능하다.

> 그러나 상각분석에 따르면 삽입이나 삭제의 평균 수행시간은 $O(1)$ 이다.
