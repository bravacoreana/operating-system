# Thread & concurrency

## 4.1 Overview

So far, we assumed that

- a process was an excuting program \_with a single thread of control.
- however, a process is able to contain _multiple threads of control_.

지금까지 우리가 배워 온 바에 의하면, 메모리와 CPU가 있는 아키텍쳐에서 실행중인 프로그램, 그러니까 어떤 인스트럭션들을 메모리에 로드하고 CPU가 fetch하고 execute 하는 상태를 `process`라고 한다. 그리고 메모리에 프로세스가 여러개 로드되어 `multiprogrammed` 되어 있는 상태에서 이 프로세스들이 CPU와 타임쉐어링을 해 context-switch를 하며 동시에 실행되는 구조를 알아봤다. 프로세스끼리 서로 IPC를 통해서 통신도 하고 말야. 그런데 우리가 지금까지 알아본 프로세스는 single thread, 즉 흐름이 하나 밖에 없는 프로세스다. 그런데 이 프로세스 하나가 여러개의 threads of control 을 가질 수가 있겠지?<br/>
무슨 말이냐면, 프로세스가 여러개 동작 할 수 있는 이유가 CPU에 프로그램 카운터가 있고, 이 프로그램 카운터가 메모리 내 프로세스의 인스트럭션 정보를 가지고 와 실행을 하다가, 다른 프로세스에서 context-switch가 일어나면 프로그램 카운터만 바꿔주고 또 쭉 실행한다. 그런데 이 프로그램 카운터(레지스터 셋) 정보만 별도로 유지한다면 하나의 프로그램(프로세스) 안에서 굳이 이걸 포크해서 복제를 할 필요 없이, 프로그램 안에서 실행 thread만 달리할 수 있겠지?

<br/>
<br/>

### A thread is

- a lightweight process.
- a basic unit of CPU utilization.
- comprises a _thread ID_ ,a _program counter_ , a _register set_ , and a _stack_ .

이 thread는 LWP(lightweight process)다. CPU입장에서는 실제로 프로세스 단위가 아니라 multi-threading이 제공된다면 가장 기본적인 CPU를 점유하는 단위가 thread가 된다. 그래서 프로세스 ID 대신 (우리가 이때까지 PID로 프로세스를 identify 했다면) 이제는 프로세스가 CPU를 점유하는게 아니라 TID(thread ID)가 CPU를 점유한다고 보면 되겠지? 어떤 프로세스 안에 여러개의 threads 가 있다면 말이야!

그러면 프로그램 카운터는 thread 별로 달라져야 할거야. register 정보도 그렇구. stack을 한번 생각해보자. 프로그램의 함수 호출 stack도 thread 별로 어디까지 호출되었는지(?) 달라지게 된다. 그래서 stack 도 별도로 관리해줘야 한다.

![](ref/fig-4-1.png)

<br/>
<br/>

### Motivation for multithreading

- Let us consider the case of client-server system, e.g., a wb server.

그러면 `multithreading` 을 하면 뭐가 좋냐!

우리가 소켓, 즉 IP가 있는 컴퓨터의 어떤 포트 번호가 열려 있다고 가정하자. 그리고 이 컴퓨터(서버)가 클라이언트를 기다리고 있다(혹은 듣고 있다).

- Single thread: 클라이언트가 접속하면 새로운 소켓을 연결해서 데이터를 주고 받을 것이다. 이 때 데이터를 주고 받는 단계에서 서버에 새로운 연결이 들어오면 어떻게 될까? 기존의 소켓에 줘야 할 데이터를 다 주고 나서야, 새로운 연결을 대응해야 한다. 그런데 기존의 소켓에서 blocking 되어버리면 여기서 멈춰있겠지? 그러면 새로운 연결에도 대응할 수 없을거야.
- Multithread: 클라이언트에게 새로운 요청이 들어왔을 때 서버가 일을 처리하는 것이 아니라, 서버가 새로 들어 온 클라이언트 마다 thread를 새로 생성해서 일을 넘겨주고, 그 일을 thread가 해주는거지. 그러는 동안 서버는 계속해서 non-blocking 으로 새로운 클라이언트들을 맞을 준비를 하고 말야. 이렇게 되면 single thread에서처럼 기존 클라이언트 때문에 새로운 클라이언트가 데이터를 받지 못하는 상황을 막을 수 있겠지!

![](ref/fig-4-2-multithread.png)

<br/>
<br/>

### The benefits of multithreaded programming:

- Responsiveness: may allow continued execution if part of process is blocked, especially important for user interfaces.
- Resource Sharing: threads share resources of process,
  - easier than shared-memory or message-passing.
- Economy: cheaper than process creation
  - thread switching lower overhead than context switching.
- Scalability: process can tak advantage of multiprocessor architectures

그렇다면 multithreaded programming의 장점은 뭘까?<br/>

1. 위에서 얘기 했던 것처럼 유저 인터페이스 같은 걸 처리할 때 blocking 되어 있을 필요 없이, non-blocking으로 계속 실행할 수 있다.
2. 그리고 리소스 공유 차원에서도 생각해보자. 프로세스와 프로세스가 통신을 할 때(프로세스 간 IPC를 할 떄) 중간에 shared-memory를 사용하거나, 아니면 OS에 있는 message passing 을 사용해서 데이터를 주고 받아야 한다. 그런데 이거는 프로세스 단위에서의 데이터 통신 방식이고, threads는 코드와 데이터 영역을 공유하지?(fig 4.1 참고!) 그래서 굳이 shared-memory나 message-passing 같은 걸 만들지 않아도 thread1과 thread2는 마치 shared-memory 처럼 사용할 수 있는 거다! 훨씬 리소스 사용의 제약이 덜하다. 리소스를 공유해야 할 경우 threads가 답임 😍
3. 경제성도 좋음. 배틀그라운드 같은 게임을 생각해보자.(난 안하는딩.. 교수님이 이걸로 상상해보랬음) multi-programming(멀티프로세스)의 경우 그 큰 코드 영역을 복사 해서 써야 하잖아. 즉, 프로세스를 하나 생성하려면 코드 영역을 어마무시하게 잡아먹는다는거지. 하지만 이걸 multithreading 한다면 그 코드 영역을 thread 끼리는 공유하니까 복사할 필요가 없고 따라서 경제적이라는 거야. 그리고 context switching을 할 때도 PCB를 switching하는 것보다 thread를 switching 하는게 훨씬 간단하다.
4. 확장성도 좋다. 멀티 프로세스 아키텍쳐(나중에 보겠지만 코어가 여러개 일 경우에 그 코어의 각각의 thread를 붙여서 병렬처리도 할 수 있다고 함)

## 4.4 Thread Library

### Thread in Java

- In a Java program threads are the fundamental model of program execution.
- Java provides a rich set of features for the creation and management of threads.

### Three techniques for explicitly creating threads in Java.

- _Inheritance_ from the Thread class
  - create a new class that is derived from the `Thread` class and override its `pulic void run()` method.
- _Implementing_ the Runnable interface.
  - define a new class that implements the Runnable interface and override its `public void run()` method.
- Using the _Lambda_ expression(beginning with Java Version 1.8)
  - rather than defining a new class, use a _lambda expresson_ of Runnable instead.
