# Overview

일반적으로 실행 대기 중인 프로세스가 여러 개 있을 때 동시성과 병렬성에 대한 이해가 필요하다. 

- Multiprocessing: 하나의 컴퓨터에서 두개 이상의 CPU(Central Processing Units)를 사용하는 것을 멀티 프로세싱이라고 한다.
- Multithreading: 단일 프로세스가 스레드와 같은 여러 코드 세그먼트를 가질 수 있다. 이러한 세그먼트는 해당 프로세스의 컨텍스트 내에서 동시에 실행된다.
- **Distributed Computing:** 분산 컴퓨팅 시스템은 단일 시스템으로 실행되는 여러 컴퓨터 시스템으로 구성된다. 시스템의 컴퓨터들은 물리적으로 가깝고 로컬 네트워크에 의해 연결될 수 있고 멀리 떨어져 있고 광역 네트워크에 의해 연결될 수 있다.
- **Multicore processor:** 여러 코어 프로세싱 유닛을 포함하는 단일 통합 프로세서입니다. 칩 멀티프로세서(CMP)라고도 한다.
- Pipelinig: 실행 중에 여러 명령어가 겹쳐지는 기술이다.

# 동시성(Concurrency)

동시성은 실제로 여러 작업을 중복된 시간 내에 실행할 수 있음을 의미한다. 작업 중 하나는 이전 작업이 완료되기 전에 시작할 수 있지만 동시에 실행되지는 않는다. CPU는 작업당 시간 슬라이스를 조정하고 컨텍스트를 적절히 전환한다. 그렇기 때문에 이 개념은 구현하거나 디버깅하기가 상당히 복잡하다.

![image](https://user-images.githubusercontent.com/66561524/208928849-187e27f7-8d07-4340-878f-5b44412892cb.png)

만약 두개 이상의 작업이 단일 코어 또는 다중 코어 CPU의 동일한 코어에서 두 개 이상의 작업이 실행 중인 경우 동일한 리소스에 동시에 엑세스할 수 있다. 데이터 읽기 작업이 병렬로 수행되고 안전하더라도 쓰기 엑세스동안 개발자는 데이터 무결성을 유지해야 한다.

효율적인 [프로세스 스케줄링](https://www.baeldung.com/cs/process-scheduling#3-round-robin-rr)은 동시 시스템에서 중요한 역할을 한다. [선입선출(FIFO)](https://www.baeldung.com/cs/process-scheduling#1-first-in-first-out-fifo), [최단 작업 우선(SJF)](https://www.baeldung.com/cs/process-scheduling#2-shortest-job-first-sjf) 및 [라운드 로빈(RR)](https://www.baeldung.com/cs/process-scheduling#3-round-robin-rr)은 널리 사용되는 task scheduling algorithms 이다.

# 병렬성(Parallelism)

병렬성은 프로그램의 독립적인 작업을 같은 시간에 실행할 수 있는 기능이다. 동시성 작업과 달리 병렬 작업은 다른 프로세스 코어, 다른 프로세서 또는 분산 시스템이 될 수 있는 완전히 다른 컴퓨터에 동시에 실행될 수 있다. 실제 애플리케이션의 컴퓨팅 속도에 대한 수요가 증가함에 따라 병렬처리가 더 일반적이게 되었다.

## 병렬작업의 원리

![image](https://user-images.githubusercontent.com/66561524/208928882-ae4c01b0-cf8d-49c5-b4be-7d6d77277a10.png)

분산 컴퓨팅 시스템은 여러 컴퓨터 시스템으로 구성되지만 단일 시스템으로 실행된다. 시스템에 있는 컴퓨터는 물리적으로 서로 가깝고 로컬 네트워크에 연결될 수 있고 멀리 떨어져 있고 광역 네트워크에 의해 연결될 수 있다.

성능향상을 위해서는 병렬화가 필수이다. 

- 분산 시스템은 병렬 시스템의 가장 중요한 예 중 하나다. 기본적으로 그들만의 메모리와 IO를 가진 독립적인 컴퓨터다.
- 우리는 하나의 명령어 집합에 의해 관리되는 여러 개의 덧셈기와 곱셈기와 같은 여러개의 기능 단위를 가질 수 있다.
- 프로세스 파이프라인은 병렬 처리의 또 다른 예시입니다.
- 칩 수준에서도 병렬 처리는 작업의 동시성을 증가시킬 수 있다.
- 우리는 또한 같은 컴퓨터에서 여러 개의 코어를 사용해서 병렬 처리를 활용할 수 있다.

# 동시성 vs 병렬성

두 개의 코어와 두 개의 작업이 있다할 때, 동시성 작업은 각 코어가 시간이 지남에 따라 두 작업 사이를 전환하여 두 작업을 실행합니다. 대조적으로 병렬 접근 방식은 작업 간에 전환되지 않고 시간이 지남에 따라 병렬로 실행된다.

![image](https://user-images.githubusercontent.com/66561524/208928934-8352e556-2040-4b10-b87a-a7237c4272d7.png)

동시처리에 대한 이 예는 텍스트 편집기와 같은 user-interactive 프로그램일 수 있다. 이 프로그램에서 CPU 사이클을 낭비하는 일부 IO 작업이 있을 수 있다. 파일을 저장하거나 인쇄할 때 사용자는 동시에 입력할 수 있다. 주 스레드는 타이핑, 저장 및 유사한 작업을 동시에 수행하기 위해 많은 스레드를 실행한다. 동일한 시간 동안 실행될 수 있지만 실제로 병렬로 실행되지 않는다.

대조적으로 병렬 시스템을 위한 하둡 기반 분산 데이터 처리의 예로 많은 클러스터에서 대규모 데이터 처리를 수반하며 병렬 프로세서를 사용한다. 개발자들은 전체 시스템을 하나의 데이터베이스로 본다.

## 동시성과 병렬성의 잠재적인 함정

동시성과 병렬성은 복잡하며 고급 개발 기술을 필요로 한다. 그렇지 않으면 시스템의 신뢰성을 위태롭게 하는 잠재적 위험이 있을 수 있다. 예를 들어 동시성 환경을 신중하게 설계하지 않으면 [교착 상태(deadlocks](https://www.baeldung.com/cs/deadlock-livelock-starvation#1-what-is-a-deadlock)), [경쟁상태(race conditions)](https://www.baeldung.com/cs/race-conditions) 또는 [기아 상태(starvation)](https://www.baeldung.com/cs/deadlock-livelock-starvation#starvation)가 발생할 수 있다.

마찬가지로 병렬 프로그래밍을 할 때도 주의해야 한다. 어디서 멈추고 무엇을 공유하는지 알아야 한다. 그렇지 않으면 [메모리 손상, 누출 또는 오류](https://www.baeldung.com/java-memory-leaks)에 직면할 수 있다.

## 동시성과 병렬성을 지원하는 프로그래밍 언어

프로세스와 스레드를 동시에 실행하는 것은 동시 프로그래밍 언어가 사용하는 주요 아이디어이다. 반면에 병렬 처리를 지원하는 언어는 프로그래밍 구조를 둘 이상의 컴퓨터에 실행할 수 있게 한다. 명령 및 데이터 스트림은 병렬 분류법의 핵심 용어다.

- **Systems programming:** It’s basic OS and hardware management that can include system call implementation and writing a new scheduler for an OS.
- **Distributed computing:** As we mentioned earlier, it’s a must for parallel CPUs to be utilized.
- **Performance computing:** This concept is necessary for CPU resource optimization.

Now let’s categorize the different languages, frameworks, and APIs:

- **Shared memory languages:** Orca, Java, C (with some additional libraries)
- **Object-oriented parallelism:** Java, C++, Nexus
- **Distributed memory:** MPI, Concurrent C, Ada
- **Message passing:** Go, Rust
- **Parallel functional languages:** LISP
- **Frameworks and APIs:** Spark, Hadoop

# Reference

[Concurrency vs Parallelism](https://www.baeldung.com/cs/concurrency-vs-parallelism)
