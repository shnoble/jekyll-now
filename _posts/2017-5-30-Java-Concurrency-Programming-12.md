---
layout: post
title: 자바 병렬 프로그래밍 12장 병렬 프로그래밍 테스트
---
# 12장 병렬 프로그래밍 테스트

병렬 프로그램을 테스트한 결과는 전통적으로 사용해왔던 문제 상황인 안정성(safety)과 활동성(liveness)의 문제로 귀결된다.
* 안정성 : 안 좋은 일이 발생하지 않는 상황
* 활동성 : 결국 좋은 일이 발생하는 상황

## 12.1 정확성 테스트

올바른 값을 정확하게 알고 있는 변수가 어떤 것이 있는지, 그 변수가 최종적으로 어떤 값을 가져야 하는지 등의 내용을 확인해야 한다.

### 12.1.1 가장 기본적인 단위 테스트

### 12.1.2 블로킹 메소드 테스트

대기 상태 테스트
* 대기 상태에 들어가는 메소드를 테스트하는 것은 반드시 예외 상황이 발생하는 메소드를 테스트하는 것과 비슷하다.
* 만약, 대상 메소드가 리턴돼 버린다면 테스트는 실패한 것이다.
* 대기 상태 테스트는 어떤 방법으로건 대기 상태를 풀어서 대기 상태에 들어갔었음을 확인해야 한다. (가장 확실한 방법은 바로 `인터럽트`)

예제 12.3)
```java
Thread taker = new Thread() {
    public void run() {
        try {
            int unused = bb.take();
            fail(); // 오류 발생 (대기 상태에 들어가지 못했다.)
        } catch (InterruptedException success) {
            // 성공
        }
    }
}
```
* taker 스레드가 호출한 take 메소드가 리턴된다면 taker 스레드는 오류가 발생했다는 사실을 기록해둔다.

```java
try {
    taker.start();
    Thread.sleep(LOCKUP_DETECT_TIMEOUT);            // 스레드를 실행하고 적당량 이상 오래 기다린다.
    taker.interrupt();                                                       // taker 스레드에 인터럽트를 건다.
    taker.join(LOCKUP_DETECT_TIMEOUT);                 // taker 스레드가 종료될 때까지 join 메소드로 기다린다. 
    assertFalse(taker.isAlive());                                       // join 메소드가 정상적으로 종료됐는지 확인한다.
} catch (Exception unexception) {
    fail();     // 실패
}
```

Thread.getState
* Thread.getState 메소드를 사용하는 방법은 그다지 믿을만하지 못하다.
* 특정 스레드가 대기 상태에 들어갔다고 해서 항상 스레드가 WAITING 또는 TIMED_WAITING 상태에 놓여 있다고 볼 수 없다.

### 12.1.3 안전성 테스트

병렬 처리 환경에서 동작하는 클래스의 기능을 동시 다발적으로 호출할 때 발생하는 문제를 제대로 테스트하려면, put 메소드나 take 메소드를 호출하는 여러 개의 스레드를 충분한 시간 동안 동작시킨 다음 테스트 대상 클래스의 상태가 올바른지, 잘못된 값이 들어 있지는 않은지 확인해야 한다.

* 제2의 리스트 사용
* 큐에 들어가고 나오는 항목의 체크섬을 구한 다음 순서를 유지하는 체크섬의 형태로 관리하고, 쌓인 체크섬을 비교해 확인하는 방법
(프로듀서가 하나만 동작하고 하나의 컨슈머를 사용하는 경우에 가장 효과가 큰 테스트 방법이다. - 순서까지 테스트 가능)
* 다수의 프로듀서와 다수의 컨슈머가 연결돼 있는 구조에서는 항목이 추가되는 순서에 상관없는 체크섬 방법을 사용해야 한다.

체크섬 연산을 컴파일러가 예측할 수 없는 연산인지도 확인해야한다. 예를 들어 테스트용 데이터로 일련번호를 사용하면 일단 결과가 항상 동일할 것이다. 컴파일러가 최적화를 충분히 할 수 있는 능력이 있다면 결과를 미리 계산해 버릴 수도 있다.
따라서, 일련번호 대신 임의이 숫자를 생성해 사용해야 한다.

```
테스트 프로그램은 스레드가 교차 실행되는 경우의 수를 최대한 많이 확보할 수 있도록 CPU가 여러 개 장착된 시스템에서 돌려보는 것이 좋다.
그렇다고 CPU가 수십 개 달렸다고해서 서너 개의 CPU가 장착된 시스템에 비해 테스트 효율이 좋아진다고 보기 어렵다.
절묘한 타이밍에 공유된 데이터를 사용하다 나타나는 오류를 찾으려면 CPU가 많이 있는 것보다 스레드를 더 많이 돌리는 편이 낫다.
스레드가 많아지면 실행 중인 스레드와 대기 상태에 들어간 스레드가 서로 교차하면서 스레드 간의 상호 작용이 발생하는 경우의 수가 많아지기 때문이다.
```

### 12.1.4 자원 관리 테스트

테스트 프로그램으로 테스트하고자 하는 두 번째 측면이 있는데, 바로 하지 말아야 할 일을 실제로 하지 않는지 테스트하는 일이다.
예를 들어 자원을 유출하는 등의 일을 해서는 안된다.

모든 객체는 더 이상 필요하지 않은 객체에 대한 참조를 필요 이상으로 긴 시간동안 갖고 있어서는 안된다.
이처럼 데이터를 갖고 있는 객체의 참조를 해제하지 않고 유출되면 가비지 컬렉터가 메모리를 확보할 수 없다.

### 12.1.5 콜백 사용

클라이언트가 제공하는 코드에 콜백 구조를 적용하면 테스트 케이스를 구현하는데 도우미이 된다.
콜백 함수는 객체를 사용하는 동안 중요한 시점마다 그 내부의 값을 확인시켜주는 좋은 기회로 사용할 수 있다.

### 12.1.6 스레드 교차 실행량 확대

스레드의 교차 실행 정도를 크게 높이고 그와 동시에 테스트할 대상 공간을 크게 확대시킬 수 있는 트릭이 있는데, 바로 공유된 자원을 사용하는 부분에서 `Thread.yield` 메소드를 호출해 컨텍스트 스위치가 많이 발생하도록 유도할 수 있다.

## 12.2 성능 테스트

성능 테스트는 특정한 사용 환경 시나리오를 정해두고, 해당 시나리오를 통과하는 데 얼마만큼의 시간이 걸리는지를 측정하고자 하는 데 목적이 있다.
두 번째 목적은 바로 성능과 관련된 스레드의 개수, 버퍼의 크기 등과 같은 각종 수치를 뽑아내고자 함이다.

### 12.2.1 PutTakeTest에 시간 측정 부분 추가

### 12.2.2 다양한 알고리즘 비교

### 12.2.3 응답성 측정

## 12.3 성능 측정의 함정 피하기

### 12.3.1 가비지 컬렉션

### 12.3.2 동적 컴파일

### 12.3.3 비현실적인 코드 경로 샘플링

### 12.3.4 비현실적인 경쟁 수준

### 12.3.5 의미 없는 코드 제거

## 12.4 보조적인 테스트 방법

테스팅의 목적은 '오류를 찾는 일'이 아니라 대상 프로그램이 처음 작성할 때 설계했던 대로 동작한다는, 즉 '신뢰성을 높이는 작업'이라고 봐야 한다.

### 12.4.1 코드 리뷰

### 12.4.2 정적 분석 도구

### 12.4.3 관점 지향 테스트 방법

### 12.4.4 프로파일러와 모니터링 도구

## 요약
