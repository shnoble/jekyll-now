---
layout: post
title: 자바 병렬 프로그래밍 1장 개요
---

# 1장 개요
## 1.1 작업을 동시에 실행하는 일에 대한 간략한 역사

* 스레드는 메모리, 파일 핸들과 같이 **프로세스에 할당된 자원을 공유**한다.
* 각 스레드는 각기 별도의 **프로그램 카운터, 스택, 지역 변수**를 갖는다.
* 프로그램을 스레드로 분리하면 멀티프로세스 시스템에서 자연스럽게 하드웨어 병렬성을 이용할 수 있다.
즉, 한 프로그램 내 여러 스레드를 동시에 여러 개의 CPU에 할당해 실행 시킬 수 있다.
* 현대 운영체제의 대부분은 프로세스가 아니라 **스레드를 기본 단위로 CPU 자원의 스케줄**을 정한다.
* 스레드는 자신이 포함된 프로세스의 메모리 주소 공간을 공유하기 때문에 한 프로세스 내 모든 스레드는 같은 변수에 접근하고 같은 힙(heap)에 객체를 할당한다.

## 1.2 스레드의 이점

### 1.2.1 멀티프로세서 활용

* 프로세서 스케줄링의 기본 단위는 스레드이기 때문에 **스레드 하나로 동작하는 프로그램은 한 번에 최대 하나의 프로세서**만 사용한다.
* 프로세서가 두개인 시스템에서 스레드가 하나뿐인 프로그램을 실행하면 CPU 자원의 50%를 낭비하는 셈이다.

### 1.2.2 단순한 모델링

* 복잡하고 비동기적인 작업 흐름을 각기 별도 스레드에서 수행되는 더 단순하고 동기적인 작업 흐름 몇 개로 나눌 수 있다.
* 작업 흐름에서는 특정한 동기화 시점에서만 상호 작용이 발생한다.

## 1.3 스레드 사용의 위험성

### 1.3.1 안정성 위해 요소

* 동기화를 하지 않으면 컴파일러, 하드웨어, 실행 환경 각각에서 명령어의 실행 시점이나 실행 순서를 상당히 자유롭게 조정할 수 있다.
* 레지스터나 다른 스레드에 일시적으로 보이지 않는 프로세서별 캐시 메모리에 변수를 캐시해 둘 수도 있다.

### 1.3.2 활동성 위험

* 멀티 스레드를 사용하면 단일 스레드 프로그램에서는 발생하지 않는 추가적인 안정성 위험에 노출될 수 있다.
* 비슷하게 스레드를 사용할 때는 단일 스레드 프로그램에서는 나타나지 않는 추가적인 형태의 활동성(liveness) 장애가 생길 수 있다.
* 데드락(deadlock), 소모상태(starvation), 라이브락(livelock)

### 1.3.3 성능 위험

* 멀티스레드 프로그램은 단일 스레드 프로그램에서 발생할 수 있는 모든 성능 위험뿐만 아니라, 스레드를 사용하기 때문에 생기는 추가 위험에도 노출된다.
* 스레드가 많은 프로그램에서는 **컨텍스트 스위칭이 더 빈번**하고, 그 때문에 상당한 부담이 생긴다.
* 즉, 실행중인 컨텍스르를 저장하고 다시 읽어들여야 하며, 메모리를 읽고 쓰는 데 있어 지역성(locality)이 손실되고, 스레드를 실행하기도 버거운  CPU 시간을 스케줄링하는 데 소모해야 한다.
* 스레드가 데이터를 공유할 때 동기화 수단도 사용해야 한다. 이런 동기화는 컴파일러 최적화를 방해하고, 메모리 캐시를 지우거나 무효화하기도 한다.
