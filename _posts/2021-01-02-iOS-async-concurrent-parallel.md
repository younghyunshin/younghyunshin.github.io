---
layout: post
title: "iOS) 비동기, 동시성, 병렬성"
tags: [iOS]
comments: true
---

> 동시라는 단어에 대하여  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

이전 [GCD관련 포스팅](https://www.notion.so/GCD-Queue-2a17a092d6284a7d839152d974a3a43b)에서는 동시성과 비동기의 차이를 명확히 하고 이 둘의 조합으로 사용 가능한 GCD 큐의 활용법에 대해 알아보았다. 오늘은 동시성과 병렬성의 차이를 알아보고 병렬처리를 통한 성능개선이 항상 가능한 것인지에 대해 공부해보자.

## 동시성(Concurrency) vs. 병렬성(Parallelism)

동시성은 "여러 스레드에게 일을 나누어 주는 것"이라고 했었다. 그래서 Concurrenct 큐를 이용할 때 여러 작업이 여러 스레드에 의해 `동시`에 진행될 수 있었다. 하지만 정말 **동시**에 진행되고 있었을까?

![1](https://user-images.githubusercontent.com/35067611/104592922-4e22c100-56b2-11eb-98c5-785181008b1e.png)

위 그림만 보면 task1, 2, 3은 큐에 들어온 순서에 따라 처리 순서는 다르지만 동시에 처리되고 있는 것 처럼 보인다. 하지만 실제로는 동시에 처리되고 있지 않다. 아래 그림을 보자.

![2](https://user-images.githubusercontent.com/35067611/104592925-4f53ee00-56b2-11eb-8c09-8c9edd0e2dbb.png)

물론 상황에 따라 다르겠지만 물리적으로 병렬처리를 해주지 않았다면 이와 같이 "동시에 실행되는 것 처럼 보일 뿐" 작업을 실행하는 스레드를 매우 빠르게 context switch 하고 있을 가능성이 있다. 그렇다면 병렬성은 뭐길래 실제로 동시에 작업이 처리될 수 있는 걸까? 바로 물리적인 지원이 있기에 가능한 것이다. 아래 두 개념의 차이를 정리한 것을 보자.

### 병렬성(Parallelism)

- 물리적 용어
- 실제로 작업이 동시에 처리되는 것
- 멀티 코어에서 멀티 스레드를 동작시키는 방식
- 한 개 이상의 스레드를 포함하는 각 코어들이 동시에 실행되는 성질
- 병렬성은 데이터 병렬성(Data parallelism)과 작업 병렬성(Task parallelism)으로 구분

### 동시성(Concurrency)

- 논리적인 용어
- 동시에 실행되는 것처럼 보임
- 싱글 코어에서 멀티 스레드를 동작시키기 위한 방식
- 멀티 코어에서도 동시성은 사용 가능
- 멀티 태스킹을 위해 여러 스레드가 번갈아가면서 실행되는 성질
- 동시성을 이용한 싱글 코어의 멀티 태스킹은 각 스레드들이 병렬적으로 실행되는 것처럼 보이지만 사실은 번갈아가면서 조금씩 실행되고 있는 것

## 비동기 + 동시 vs. 비동기 + 병렬

이렇게 비동기, 동시성에 더해 병렬성의 개념까지 들어오니 그럼 내가 지금까지 사용한 GCD는 어떻게 동작했나 혼란스러워지기 시작했다. 나름 곰곰히 생각을해본 결과 아래와 같은 결론을 도출했다. (GCD의 Concurrent 큐를 이용해 비동기 방식으로 프로그래밍 했다는 가정하에 동시성과 병렬성에 대해 비교해본 것이다)

1. GCD를 사용한다고 해서 반드시 병렬, 혹은 동시인지는 단언할 수 없다. 상위 계층에서는 구분하기 어렵고 디바이스에 따라 실제 동작이 달라질 수 있다.
2. Concurrent 큐를 사용했으며 async 키워드를 통해 비동기 방식으로 프로그래밍 했다면 메인 스레드와 서브 스레드로 작업이 나누어졌을 것이고(Concurrent 큐) 메인 스레드는 서브 스레드를 기다리지 않고 다음 작업을 진행할 것(Async)이다. 이 때 메인스레드의 다음 작업과 서브스레드에게 할당된 작업이 멀티 태스킹이 되느냐, 멀티 스레딩이 되느냐의 차이가 곧 비동기 방식의 동시처리/병렬처리로 나뉜다.
3. 병렬성은 물리적 지원이 있어야만 가능하다. 그러므로 디바이스가 멀티코어 환경에서 실제 여러 CPU를 보유한 상황이라면 메인 스레드와 서브 스레드가 "동시에" 각자의 작업을 처리하는 병렬처리가 가능할 것이다.
4. 하지만 디바이스가 싱글코어만 지원한다면 병렬성은 구현될 수 없으므로 메인 스레드와 서브 스레드가 context switching을 통해 빠르게 멀티 태스킹 되고 있을 것이다.

## 비동기 방식의 병렬처리는 언제나 성능을 개선할까?

흠....