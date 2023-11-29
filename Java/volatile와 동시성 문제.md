# Overview

CPU가 작업을 처리하기 위한 CPU와 RAM의 관계도를 먼저 확인해보자.

![image](https://user-images.githubusercontent.com/66561524/192395561-9c9e167a-d169-4499-ba1b-9e7ec5808712.png)

- CPU는 작업을 처리하기 위한 데이터가 필요할 때 RAM의 일부분을 고속 저장 장치인 CPU Cache Memory로 읽어 들인다.
    - non-volatile 변수는 MultiThread 애플리케이션에서 Task를 수행하는 동안 성능 향상을 위해 Main Memory에서 읽어온 변수 값을 CPU Cache에 저장하다.
- 읽어 들인 데이터로 명령을 수행하고 다시 RAM에 저장하기 위해서는 읽기의 역순을 밟는다.
- 동시성 프로그래밍에서는 CPU와 RAM의 중간에 있는 CPU Cache Memory와 병렬성이라는 특징 때문에 다수의 스레드가 공유 자원에 접근할 때 두가지 문제가 발생한다.

# 동기화와 원자성

- 동기화
    - 동기화는 **배타적 실행**뿐 아니라 스레드 사이의 **안정적인 통신**에 꼭 필요하다
    - 자바에서 메서드나 블록을 한 스레드가 수행하도록 보장하려면 synchronized 키워드를 사용하면 된다.
    - 동기화를 제대로 사용하면 어떤 메서드도 객체의 상태가 일관되지 않은 순간을 볼 수 없다. 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 같게 한다.
- 원자성
    - 한 줄의 프로그램 문장이 컴파일러에 의해 기계어로 변경되면서 순차적으로 처리하기 위한 여러 개의 Machine Instruction이 만들어져 실행된다.
    - 멀티 스레드 환경에서는 한 스레드가 각 기계명령어를 수행하는 동안에 다른 스레드가 개입하여 공유 변수에 접근하여 같은 기계 명령어를 수행할 수 있으므로 값이 꼬인다. (race condition)
    - 자바 언어의 명세상으로 long과 double를 제외한 변수를 읽고 쓰는 것은 원자적이다. 즉, 동기화 없이 여러 스레드가 같은 변수를 수정하더라도 항상 어떤 스레드가 정상적으로 저장한 값을 읽어오는 것을 보장한다는 것이다.
    - 하지만 **스레드가 필드를 읽을 때 항상 ‘수정이 완전히 반영된’ 값을 얻는다 보장**하지만, **한 스레드가 저장한 값이 다른 스레드에게 ‘보이는가’는 보장하지 않는다.**
    - 따라서 원자적 데이터를 쓸 때도 동기화해야 한다.

# 가시성의 문제와 volatile 키워드

```java
public class StopThread {

    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

- volatile 키워드는 java 변수를 Main Memory에 저장하겠다라는 것을 명시한다.
- 변수의 값을 write할 때마다 Main Memory에 작성한다.
- • `CPU Cache`보다 `Main Memory`가 비용이 더 크기 때문에 `변수 값 일치`을 보장해야 하는 경우에만 `volatile` 사용하는 것이 좋다.

![image](https://user-images.githubusercontent.com/66561524/192395547-8f42d948-b3a2-44aa-a788-8a858ea9fe81.png)

- 스레드별로 각 CPU를 사용하여 CPU Cache Memory에서 읽어들이는 값이 다를 수 있다. 이러한 문제를 가시성의 문제라고 한다.
- 이 문제를 해결하기 위해서는 **volatile**로 선언하면된다. **volatile**로 선언한 변수에 대해서는 아래와 같이 CPU Cache Memory를 거치지 않고 RAM으로 직접 읽고 쓰는 작업을 수행한다.

![image](https://user-images.githubusercontent.com/66561524/192395601-749d9399-4588-45b0-859e-f6c0177af14f.png)

- **volatile**은 가시성 문제 (스레드 사이의 안정적인 통신)는 해결하지만 동시 접근 문제는 해결 못한다

# 동시 접근 문제와 synchronized

---

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

- 코드상으로 증가 연산자(++)는 하나지만 실제로는 volatile 필드에 두 번 접근한다. 먼저 값을 읽고, 그다음에 1을 증가한 후 새로운 값을 저장하는 것이다. 따라서 **두 번째 스레드가 첫 번째 스레드의 연산 사이에 들어와 공유 필드를 읽게 되면, 첫 번째 스레드와 같은 값을 보게 될 것**이다. volatile 키워드는 어디까지나 volatile 변수를 메인 메모리로부터 읽을 수 있게 해 주는 것이 전부다.
- 여러 Thread에서 wrtie하는 상황에서는 적합하지 않다.
- 이처럼 잘못된 결과를 계산해내는 오류를 **안전 실패(safety failure)**
라고 한다. 이 문제는 메서드에 synchronized를 붙이고 volatile 키워드를 공유 필드에서 제거하면 해결된다. 다음의 코드는 synchrozied를 이용해 위의 문제를 해결한 수정된 코드이다.

```java
private static int nextSerialNumber = 0;

public static int generateSerialNumber() {
    increment();
    return nextSerialNumber;
}

private static synchronized void increment() {
        ++nextSerialNumber;
}
```

- **또한 synchrozied는 가시성의 문제도 해결한다. synchronized 블록을 들어가기 전에 CPU Cache Memory와 Main Memory를 동기화해주기 때문이다.**
- 추가로 synchrozied와 volatile은 jvm의 최적화 기법을 방지하는 역할을 한다.

# Atomic

---

- atomic 또한 멀티 스레드 환경에서 원자성을 보장하기 위해 나온 개념이다. 동기화(blockin)가 아닌 CAS(Compared And Swap)라는 알고리즘으로 작동하여 원자성을 보장한다.
- CAS 알고리즘이란 volatile에서 설명했던 CPU Cache Memory와 RAM을 비교하여 일치한다면 CPU Cache Memory와 RAM에 적용하고, 일치하지 않는다면 재시도함으로써 어떠한 스레드에서 공유 자원에 읽기/쓰기 작업을 하더라도 원자성을 보장한다.

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

# Reference

---

[Java volatile이란?](https://nesoy.github.io/articles/2018-06/Java-volatile)

[[이펙티브 자바] 아이템 78 공유중인 가변 데이터는 동기화해 사용하라](https://gona.tistory.com/m/82)

- 이펙티브 자바 아이템 78
