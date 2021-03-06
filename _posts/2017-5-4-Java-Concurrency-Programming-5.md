---
layout: post
title: 자바 병렬 프로그래밍 5장 구성 단위
---

# 5장 구성 단위

## 5.1 동기화된 컬렉션 클래스

* 동기화되어 있는 컬렉션 클래스의 대표 주자는 바로 Vector와 Hashtable이다.

### 5.1.1 동기화된 컬렉션 클래스의 문제점

* 여러 개의 연산을 묶어 하나의 단일 연산처럼 활용해야 할 필요성이 항상 발생한다.
    * 반복 (iteration)
    * 이동 (navigation)
    * 없는 경우에만 추가하는

* 여러 스레드가 해당 컬렉션 하나를 놓고 동시에 그 내용을 변경하려 한다면 컬렉션 클래스가 상식적으로 올바른 방법으로 동작하지 않을 수도 있다.
* `동기화된 컬렉션 클래스는 대부분 클라이언트 측 락을 사용할 수 있도록 만들어져있기 때문에 컬렉션 클래스가 사용하는 락을 함께 사용한다면 새로 추가하는 기능을 컬렉션 클래스에 들어있는 다른 메소드와 같은 수준으로 동기화시킬 수 있다.`

#### 반복문
* 반복문이 실행되는 도중에 예외 상황이 발생하는 경우 역시 클라이언트 측 락을 사용하면 예외 상황이 발생하지 않도록 정확하게 동기화 시킬 수 있다.
* 그러나 성능의 측면에서 약간의 손해가 생길 가능성이 있다.
* 반복문을 실행하는 동안에 내부의 값을 변경하는 모든 스레드가 대기 상태에 들어가기 때문이다.

### 5.1.2 Iterator와 ConcurrentModificationException

Collection 클래스에 들어 있는 값을 차례로 반복시켜 읽어내는 가장 표준적인 방법은 Iterator를 사용하는 방법 (특별한 문법 for문도 동일하다)
그러나 동기화된 컬렉션 클래스에서 만들어낸 Iterator를 사용한다 해도 다른 스레드가 같은 시점에 컬렉션 클래스 내부의 값을 변경하는 작업을 처리하지는 못하게 만들어져 있고, 대신 즉시 멈춤(fail-fast)의 형태로 반응하도록 되어 있다.
`즉시 멈춤이란 반복문을 실행하는 도중에 컬렉션 클래스 내부의 값을 변경하는 상황이 포착되면 그 즉시 ConcurrentModificationException 예외를 발생시키고 멈추는 처리 방법이다.`

### 5.1.3 숨겨진 Iterator

* toString, hashCode, equals 메소드 등은 내부적으로 iterator를 사용한다.
* containsAll, removeAll, retainAll 등의 메소드, 컬렉션 클래스를 넘겨받는 생성 메소드 등도 모두 내부적으로 iterator를 사용한다.
* 이렇게 내부적으로 iterator를 사용하는 모든 메소드에서 ConcurrentModificationException이 발생할 가능성이 있다.

## 5.2 병렬 컬렉션

자바 5.0은 여러 가지 병렬 컬렉션 클래스를 제공하면서 컬렉션 동기화 측면에서 많은 발전이 있었다.
동기화된 컬렉션 클래스는 컬렉션의 내부 변수에 접근하는 통로를 일련화해서 스레드 안전성을 확보했다.
하지만 동기화된 컬렉션은 동시 사용성은 상당 부분 손해를 볼 수 밖에 없다.

병렬 컬렉션은 여러 스레드에서 동시에 사용할 수 있도록 설계되어 있다.
새로 추가된 ConcurrentMap 인터페이스를 보면 추가하려는 항목이 기존에 없는 경우에만 새로 추가하는 연산, 대치 연산, 조건부 제거 연산 등을 정의하고 있다.

자바 5.0은 Queue와 BlockingQueue라는 두 가지 형태의 컬렉션 인터페이스를 추가했다.
* ConcurrentLinkedQueue : 전통적인 FIFO
* PriorityQueue는 특정한 우선 순위에 따라 큐에 쌓여 있는 항목이 추출되는 순서가 바뀌는 순서
* BlockingQueue
    * 큐에 항목을 추가하거나 뽑아낼 때 상황에 따라 대기할 수 있도록 구현
    * 큐가 비어있다면 큐에서 항목을 뽑아내는 연산은 새로운 항목이 추가될 때까지 대기
    * 가측 차 있다면, 큐에 새로운 항목을 추가하는 연산은 큐에 빈 자리가 생길 때까지 대기
    * BlockingQueue 클래스는 프로듀서-컨슈머(producer-consumer) 패턴을 구현할 때 굉장히 편리하게 사용할 수 있다.

### 5.2.1 ConcurrentHashMap

```
ConcurrentHashMap은 HashMap이나 HashTable처럼 key-value 형태의 자료구조다. 
Concurrent뒤에 HashMap이라는 이름을 사용하고 있지만, ConcurrentHashMap은 null을 허용하지 않는다는 점이나 동기화가 되어 있어 thread-safe 하다는 점에서 HashTable과 더 비슷하다. 
HashTable는 내부적으로 모든 쓰기 작업에 lock을 걸어 동기화를 구현하였지만, ConcurrentHashMap은 여러 부분으로 구역을 나누어 처리하여, 좀 더 많은 쓰레드가 동시에 접근할 수 있다.
```
[Java의 ConcurrentHashMap](http://hyeonjae.github.io/java/2015/02/14/concurrent-hash-map.html)

이전에는 모든 연산에서 하나의 락을 사용했기 때문에 특정 시점에 하나의 스레드만이 해당 컬렉션을 사용할 수 있었다.
ConcurrentHashMap은 락 스트라이핑 (lock striping)이라 부르는 굉장히 세밀한 동기화 방법을 사용해 여러 스레드에서 공유하는 상태에 훨씬 잘 대응할 수 있다.

`ConcurrentHashMap이 만들어 낸 Iterator는 ConcurrentModificationException을 발생시키지 않는다'
따라서 ConcurrentHashMap의 항목을 대상으로 반복문을 실행하는 경우에는 따로 락을 걸어 동기화해야 할 필요가 없다.
ConcurrentHashMap에서 만들어 낸 Iterator는 즉시 멈춤 대신 미약한 일관성 전략을 취한다.

### 5.2.2 Map 기반의 또 다른 단일 연산

### 5.2.3 CopyOnWriteArrayList

'변경할 때마다 복사'하는 컬렉션 클래스는 불변 객체를 외부에 공개하면 여러 스레드가 동시에 사용하려는 환경에서도 별다른 동기화 작업이 필요 없다는 개념을 바탕으로 스레드 안전성을 확보하고 있다.
변경할 때마다 복사하는 컬렉션은 변경 작업보다 반복문으로 읽어내는 일이 훨씬 빈번한 경우에 효과적이다.
이벤트 처리 시스템에서 이벤트 리스너를 관리하는 부분에 유용하게 사용할 수 있다.

## 5.3 블로킹 큐와 프로듀서-컨슈머 패턴

큐가 가득 차 있다면 put 메소드는 값을 추가할 공간이 생길 때까지 대기한다.
큐가 비어있는 상태라면 take 메소드는 뽑아낼 값이 들어올 때까지 대기한다.

프로듀서-컨슈머 패턴을 사용하면 작업을 만들어 내는 부분과 작업을 처리하는 부분을 완전히 분리할 수 있기 때문에 개발 과정을 좀더 명확하게 단순화시킬 수 있고, 작업을 생성하는 부분과 처리하는 부분이 각각 감당할 수 있는 부하를 조절할 수 있다는 장점이 있다.

offer 메소드는 큐에 값을 넣을 수 없을 때 대기하지 않고 바로 공간이 모자라 추가할 수 없다는 오류를 알려준다.

* LinkedBlockingQueue
* ArrayBlockingQueue
* PriorityBlockingQueue
* SynchronousQueue

### 5.3.1 예제: 데스크탑 검색

### 5.3.2 직렬 스레드 한정

프로듀서-컨슈머 패턴과 블로킹 큐는 가변객체를 사용할 때 객체의 소유권을 프로듀서에서 컨슈머로 넘기는 과정에서 직렬 스레드 한정(serial thread confinement) 기법을 사용한다.
스레드 한정된 객체는 특정 스레드 하나만이 소유권을 가질수 있는데, 객체를 안전한 방법으로 공개하면 객체에 대한 소유권을 이전(transfer)할 수 있다.

### 5.3.3 덱, 작업 가로채기

Deque은 앞과 뒤 어느 쪽에도 객체를 쉽게 삽입하거나 제거할 수 있도록 준비된 큐

작업 가로채기 (work stealing)라는 패턴을 적용할 때에는 덱을 그대로 가져다 사용할 수 있다.
컨슈머가 자신의 덱에 들어 있던 작업을 모두 처리하고 나면 다른 컨슈머의 덱에 쌓여있는 작업 가운데 맨 뒤에 추가된 작업을 가로채 가져올 수 있다.
스레드가 작업을 진행하는 도중에 새로 처리해야 할 작업이 생기면 자신의 덱에 새로운 작업을 추가한다.
자신의 덱이 비었다면 다른 작업 스레드의덱을 살펴보고 밀린 작업이 있다면 가져다 처리해 자신의 덱이 비었다고 쉬는 스레드가 없도록 관리한다.



## 5.4 블로킹 메소드, 인터럽터블 메소드

스레드가 블록되면 동작이 멈춰진 다음 블록된 상태(BLOCKED, WAITING, TIMED_WAITING) 가운데 하나를 갖게 된다.
블로킹 연산은 단순히 실행 시간이 오래 걸리는 일반 연산과는 달리 멈춘 상태에서 특정한 신호(I/O 작업이 끝나기를 기다리거나, 기다리던 락을 확보했거나, 다른 스레드의 작업 결과를 받아오는 등의 신호)를 받아야 계속해서 실행할 수 있는 연산을 말한다.

* BlockingQueue 인터페이스
    * put 메소드와 take 메소드는 Thread.sleep 메소드와 같이 InterruptedException을 발생시킬 수 있다.
    * InterruptedException을 발생시킬 수 있다는 것은 해당 메소드가 블로킹 메소드라는 의미이다.

Thread 클래스는 해당 스레드를 중단시킬 수 있도록 interrupt 메소드를 제공한다.

* InterrupedException이 발생했을 때 대처할 수 있는 방법
    * InterruptedException 전달 : 받아낸 InterruptedException을 그대로 호출한 메소드에게 넘겨버리는 방법이다.
    * 인터럽트를 무시하고 복구 : Runnable 인터페이스를 구현하는 경우 처럼 특정상황에서는 InterruptedException을 throw 할 수 없다.
이런 경우, InterruptedException을 catch한 다음, 현재 스레드의 interrupt 메소드를 호출해 인터럽트 상태를 설정해 상위 호출 메소드가 인터럽트 상황이 발생했을을 알 수 있도록 해야 한다.


## 5.5 동기화 클래스

상태 정보를 사용해 스레드 간의 작업 흐름을 조절할 수 있도록 만들어진 모든 클래스를 동기화 클래스 (synchronizer) 라고 한다.

### 5.5.1 래치

래치는 스스로가 터미널 (terminal) 상태에 이를 때까지의 스레드가 동작하는 과정을 늦출 수 있도록 해주는 동기화 클래스이다.

* CountDownLatch
[자바 CountDownLatch를 이용한 동시성 높이기..](http://sjava.net/2013/07/자바-countdownlatch를-이용한-동시성-높이기/)
    * 래치의 상태는 양의 정수 값으로 카운터를 초기화하며, 이 값은 대기하는 동안 발생해야 하는 이벤트의 건수를 의미한다.
    * await 메소드는 래치 내부의 카운터가 0이 될 때까지, 즉 대기하던 이벤트가 모두 발생했을 때까지 대기하도록 하는 메소드 이다.
        * await 메소드는 카운터가 0이 되거나, 
        * 대기하던 스레드에 인터럽트가 걸리거나, 
        * 대기 시간이 길어 타임아웃이 걸릴 때까지 대기한다.

### 5.5.2 FutureTask

스레드를 시작해 두고, get() 메소드를 호출할 때 결과를 모두 가져왔다면 즉시 결과를 알려줄 것이고, 아직 결과를 가져오는 중이라면 완료될 때 까지 대기하고 결과를 알려준다.

Callable 인터페이스로 정의되어 있는 작업에서는 예외를 발생시킬 수 있으며, 어디에서든 Error도 발생시킬 수 있다.
Callable의 내부 작업에서 어떤 예외를 발생시키건 간에 그 내용은 Future.get 메소드에서 ExecutionException으로 한 번 감싼 다음 다시 throw 한다.

* ExecutionException이 발생하면 그 원인은 세 가지 가운데 하나여야 한다.
    * Callable이 던지는 예외
    * RuntimeException
    * Error

> [java의 FutureTask를 사용한 간단한 수행 대기 ](http://lacti.me/2012/05/28/simply-wait-using-FutureTask-at-java/)

### 5.5.3 세마포어

세마포어 클래스는 가상의 퍼밋 (permit)을 만들어 내부 상태를 관리하며, 세마포어를 생성할 때 생성 메소드에 최초로 생성할 퍼밋의 수를 넘겨준다.
* 현재 사용할 수 있는 남은 퍼밋이 없는 경우, acquire 메소드는 남는 퍼밋이 생기거나, 인터럽트가 걸리거나, 지정한 시간을 넘겨 타임아웃이 걸리기 전까지 대기한다.
* release 메소드는 확보했던 퍼밋을 다시 세마포어에게 반납하는 기능을 한다.

[이진 세마포어]
* 이진 세마포어는 초기 퍼밋 값이 1로 지정된 카운팅 세마포어이다.
* 이진 세마포어는 비재진입(nonreentrant) 락의 역할을 하는 `뮤텍스(mutex)`로 활용할 수 있다.

### 5.5.4 배리어

```
CountDownLatch 클래스의 인스턴스는 카운트다운밖에 할 수 없다. 즉 한 번 카운트가 0에 도달하면 await 메소드를 호출해도 바로 돌아와버린다. 쓰레드의 동기를 몇 번이고 반복하고 싶을 때에는 java.util.concurrent.CyclicBarrier 클래스를 사용한다.

CyclicBarrier는 주기적으로 (cyclic 하게) 장벽(barrier)을 만든다. 장벽에 부딪힌 쓰레드는 장벽이 사라질 떄까지 다음 단게로 나아갈 수 없다. 장벽이 사라지는 조건은 생성자에서 지정한 개수의 쓰레드가 그 장벽에 도착하는 것이다. 즉, 지정한 개수의 쓰레드가 장벽에 도착하면 장벽이 사라지고 쓰레드들이 모두 실행을 시작하게 된다.
```

[[JAVA:병렬 프로그래밍 - 2] CyclicBarrier 사용하기.](http://dev.re.kr/53)

* 래치와의 차이점은 모든 스레드가 배리어 위치에 동시에 이르러야 관문이 열리고 계속해서 실행할 수 있다는 점이 다르다. 
* 래치는 '이벤트'를 기다리기 위한 동기화 클래스이다.
* 배리어는 '다른 스레드'를 기다리기 위한 동기화 클래스이다.
* 스레드는 각자가 배리어 포인트에 다다르면 await 메소드를 호출한다.
* await 메소드는 모든 스레드가 배리어 포인트에 도달할 때 까지 대기한다.
* 모든 스레드가 배리어 포인트에 도달하면 배리어는 모든 스레드를 통과시키며, await 메소드에서 대기하고 있던 스레드는 대기 상태가 모두 풀려 실행되고, 배리어는 다시 초기 상태로 돌아가 다음 배리어 포인트를 준비한다.
* await 호출 후 타임아웃이나 대기하던 스레드에서 인터럽트가 걸리면 배이러는 깨진것으로 간주하고, await에서 대기하던 모든 스레드에 `BrokenBarrierException`이 발생한다.
* 배리어가 성공하면 await 메소드는 각 스레드별로 배리어 포인트에 도착한 순서를 알려준다.

배리어와 약간 다른 형태 Exchanger 클래스
* Exchanger는 두 개의 스레드가 연결되는 배리어이다.
* 배리어 포인트에 도달하면 양쪽의 스레드가 서로 갖고 있던 값을 교환한다.
* Exchanger 클래스는 양쪽 스레드가 서로 대칭되는 작업을 수행할 때 유용하다.

## 5.6 효율적이고 확장성 있는 결과 캐시 구현

`메모이제이션이란 프로그래밍을 할 때 반복되는 결과를 메모리에 저장해서 다음에 같은 결과가 나올 때 빨리 실행하는 코딩 기법을 말합니다.`

