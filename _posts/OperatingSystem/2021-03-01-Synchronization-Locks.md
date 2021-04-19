---
layout: post
title:  "[OS]Concurrency(2): Synchronization - Locks"
date:   2021-03-01 13:19:35 +0900
categories: [OperatingSystem]
---

OSTEP을 읽고 강의를 들으면서 메모합니다.

- references: [book](), [code](https://github.com/remzi-arpacidusseau/ostep-code/tree/master/threads-locks)

# Synchronization이란
Concurrent하게 여러 개의 프로세스나 스레드를 실행할 때 오작동하지 않게 하기 위해 해주는 일입니다. 대표적인 오작동으로는 A, B 두 개의 프로세스가 있을 때 A의 어떤 작업을 하던 도중 instruction들의 순서가 섞여버려서, 서로 같은 shared data에 접근해 잘못된 데이터를 읽거나 쓰는 등의 race condition이 있고, 또 A 프로세스만 계속 실행되고 B가 실행이 안 되는 starvation 상황이 있을 수 있습니다. 

- 아래 코드에서 어떻게 하면 panic이 일어날 수 있을까요? 
    ```C
    //Thread 1
    p = someComputation();
    pInitialized = true;

    //Thread 2
    while(!pInitialized)
        ;
    q = anotherComputation(p);
    if (q != anotheromputation(p));
        panic
    ```
    컴파일러는 Thread 1에서 변수를 initialize 하는 부분을 앞에다가 배치해버릴 수가 있습니다. `pInitialized = true`가 먼저 실행되어 버리면, Thread 2에서 `q = anotherComputation(p);` 이 실행된 후 `p = someComputation();`가 실행되고 다시 `if (q != anotheromputation(p));`로 들어가서 panic까지 갈 수 있습니다. 

# 문제상황: race condtion, starvation

- Race Condition: concurrent program의 결과가 스레드 간의 operation의 순서에 따라 달라지는 버그입니다. 
- 가령 아래의 코드를 컴파일해서 실행시키면 balance가 0이 되지 않고 제멋대로인 값이 나옵니다. 멀티스레딩에서 생기는 문제. `++balance`가 atomic하지 않고, 어셈블리 레벨로 가면 3개의 어셈블리 operation이 합쳐진 것이기 때문에 그렇습니다. 

```assembly
//++balance
mov &balance, %eax
add $0x1, %eax
mov %eax, &balance
```

그럼 아래의 코드를 실행시킬 때 결과가 0이 아니라 제멋대로 나오게 됩니다. 

```C
#include <stdio.h>
#include <pthread>

static int balance = 0;

void* deposit(void *arg){
    int i;
    for (i=0; i<1e7; ++i)
        ++balance;
}

void* withdraw(void *arg) {
    int i;
    for (i=0; i<1e7; ++i)
        --balance;
}

int main(void){
    pthread_t t1, t2;
    pthread_create(&t1, NULL, deposit, (void*)1);
    pthread_create(&t2, NULL, deposit, (void*)2);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    printf("balance: %d\n", balance);
    return 0;
}
```


# 해결방법 - Lock 걸기
- shared data를 사용하기 전 Lock을 걸어서 다른 프로세스가 해당 데이터에 접근하는 것을 막습니다.  
- 잘못된 방법: race condition 유발
  - 아래의 lock 함수 안에서 while 문이 끝난 뒤에 interrupt 발생해서 다른 thread로 switch되고, lock이 넘어간다면? 


```C
typedef struct __lock_t {
    int flag;
} lock_t;

void init(lock_t *mutex) {
    lock->flag = 0;
}

void lock(lock_t *mutex) {
    while (mutex->flag == 1) 
        ; //spin-wait (do nothing)
    mutex->flag = 1;
}

void unlock(lock_t *mutex) {
    lock->flag = 0;
}
```

- SW만으로는 온전히 atomicity 지원하기 어려워서 HW의 도움이 필요합니다. 대표적으로 TestAndSet과 CompareAndSwap이 있습니다.  

```C
int TestAndSet (int *old_ptr, int new) {
    int old = *old_ptr;
    *old_ptr = new;
    return old;
}
```

- 위 코드를 이용해서 아래와 같은 lock 사용 가능하빈다. 

```C
typedef struct __lock_t {
    int flag;
} lock_t;

void init(lock_t *lock) {
    lock->flag = 0;
}

void lock(lock_t *lock) {
    while (TestAndSet(&lock->flag)) 
        ;
}

void unlock(lock_t *lock) {
    lock->flag = 0;
}
```



## CompareAndSwap
- `TestAndSet` 대신 아래와 같이 `CompareAndSwap`을 사용할 수도 있는데, 이 방식은 "lock-free synchronization"등에서 나오지만, 더 강력합니다.

```C
int CompareAndSwap(int *ptr, int expected, int new) {
    int original = *ptr;
    if (original == expected)
        *ptr = new;
    return original;
}
```

```C
void lock(lock_t *lock) {
    while (CompareAndSwap(&lock->flag, 0, 1) == 1)
        ; //spin
}
```

- Too much spinning? 
  - spinning lock 대신 sleeping mutex 사용 가능. 
  - sleeping(yield) 하면 그만큼 CPU resource 절약되고 performance 향상됨. 다른 이유로는 priority inversion이 있는데, 가령 스레드 T1 T2 T3이 있고, 현재 T2가 lock을 가지고 작업하는 상황. T3 작업 중 T2에 의해 lock이 선점당한다면, T1이 T2보다 우선순위가 높다고 하더라도 T2가 먼저 처리됨 -> correctness에 문제 발생. 해결을 위해 priority inheritance 라는 기법이 생김. 
  - 이런 이유로 대부분의 userspace lock 구현은 sleeping mutex임. 
  - 단, OS의 경우 yield할 대상이 없으므로 kernel에서는 항상 spinning lock 사용함. OS가 spin lock을 얻으면, lock이 있는 동안은 disable interrupts 할 수 있어야 함. 다른 interrupt
  - spinning lock은 되도록 빨리 끝내는 게 좋음

```C
void init() {
    flag = 0;
}

void lock() {
    while (TestAndSet(&flag, 1) == 1)
        yield(); // give up the CPU
}

void unlock() {
    flag = 0;
}
```

- 아래는 queues를 사용하는 방법. 
  - spin만 쓰거나 즉시 yield CPU 하는 두 방법 모두 리소스 낭비 및 starvation이 있을 수 있어서, queue를 사용해서 해결함. 
  - `park()`는 스레드를 재우고 `unpark(threadID)`는 깨움. 
  - lock_t 구조체가 queue와 guard를 가지고 있음. guard는 spin lock으로 쓰이는데, spinning에 쓰이는 시간을 제한함. 우선 thread가 guard lock을 얻으면, 실제 lock(flag)을 얻을 수 있는지 확인. 얻을 수 있으면 lock을 얻고 guard lock는 0으로 돌림. 그렇지 않으면 queue에 thread id를 넣고, guard lock을 0으로 돌리고, park()로 스레드를 재움. 
  

```C
typedef struct __lock_t {
    int flag;
    int guard;
    queue_t *q;
} lock_t

void lock_init(lock_t *m) {
    m->flag = 0;
    m->guard = 0;
    queue_init(m->q);
}

void lock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1)
        ; //acquire guard lock by spinning
    
    if(m->flag == 0) {
        m->flag = 1;
        m->guard = 0;
    }
    else {
        queue_add(m->q, gettid());
        m->guard = 0;
        park();
    }
}

void unlock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1)
        ; // acquire guard lock by spinning
    if (queue_empty(m->q)) 
        m->flag = 0;
    else
        unpark(queue_remove(m->q)); // hold lock for next thread

    m->guard = 0;
}
```