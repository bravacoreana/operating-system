# CPU Scheduling

## 5.1 Basic Concepts

### CPU scheduling is

- the basis of multiprogrammed operating systems.
- The objective of multiprogramming is

  - to have some processes running at all times.
  - to maximise CPU utilisation.

CPU 스케줄링은 멀티프로그램 OS 에서 필수가 된다. 이 멀티프로그래밍, 즉, CPU 하나의 메모리에 여러개의 프로세스가 동시에 메모리에 로드되어 있고, CPU가 이 프로세스들을 선점해 concurrent 하게 실행하는 것을 멀티 프로그래밍, 멀티 프로세싱, 멀티 태스킹이라고 한다.

![](img/multiprocessing.png)

> concurrent 하게 실행하려는 목적은?
> cpu의 속도가 상당히 빠르기 때문에 context-switch를 통해 시간을 나눠서 사이사이에 cpu 자원을 넣어 사용해도 아무런 문제가 없다 :)

즉, CPU utilisation을 높이기 위해서 CPU Scheduling 이 필요하다.

프로세스 입장에서 생각해보자. 프로세스는 연산을 할 때, read/write file을 할 때 cpu를 쓴다. 이때 I/O를 기다리고 있는데, 이 기다리는 시간을 I/O burst time 이라고 하고, CPU를 막상 쓰는 시간을 CPU burst time 이라고 한다.

![](ref/fig-5-1-bursts.png)

### CPU scheduler

- selects a process from the processes in memeory that are **ready** to execute and **allocates** the CPU to that process.

메모리에 로드되어 있는 프로세스들 중에 어떤 놈에게 cpu를 할당해 줄거냐! 다시 말해, wait 상태에 있는 애들은 보내 줄 필요가 없잖아? 따라서 ready 상테에 있는 프로세스들 중에서 cpu를 할당해 줄 수 있는 프로세스를 선택하는게 cpu 스케줄러!

### Then, how can we select a next process?

- Linked List? or Binary Tree?
- _FIFO Queue_: First-in, First-out
- _Priority Queue_: How can we determine the priority of a process?

대기중인 queue를 linked list로 만들거냐, binary tree로 만들거냐! 그리고, FIFO Queue를 따를 것이냐! 아니면 Priority 를 지정해줘서 priority에 따라 queue를 지정할 것이냐, 그렇다면 priority는 어떻게 결정할 것이냐!

### Preemptive(선점형) kernel vs Non-preemptive(비선점형) kernel

- Preemtive: 강제로 쫓아낼 수 있다
  - Preemtive scheduling
    - a process can be preempted by the scheduler.
- Non-preemtive: 쫓아낼 수 없다, 자발적으로 나오게 한다
  - Non-preemtive scheduling
    - a process keeps the CPU until it releases it, either by terminating or by switching to the waiting state.

Non-preemtive scheduling는 cpu를 프로세스가 선점하면 그 프로세스가 자발적으로 terminating 하거나 switching 해서 release 할 때까지는 그 프로세스가 cpu를 쓰도록 내버려 두는 것이다.
Preemtive는 스케줄러가 어떤 이유에 의해 cpu를 선점하고 있는 프로세스를 쫓아낼 수 있는 것이다.

### Decision making for CPU-scheduling

cpu-scheduling을 위한 의사 결정을 살펴보자. 4가지 경우가 있다.

1. When a process switches from the _running_ to _waiting_ state.
2. When a process switches from the _running_ to _ready_ state.
   - running 상태에 있다가 "내가 잠깐 쉬어야겠다!" 해서 ready 상태로 가는 경우도 있음
3. When a process switches from the _waiting_ to _ready_ state.
   - waiting 상태에서 I/O가 할 일(read/write..)이 다 끝났다면 cpu를 점유하러 ready로 가겠지!
4. When a process terminates.

- 1번 & 4번 : non-preemptive -> 고민할 이유가 없어!
- 2번 & 3번 : choices - preemtive or non-preemtive
