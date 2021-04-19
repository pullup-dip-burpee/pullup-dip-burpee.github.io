---
layout: post
title:  "[OS]Concurrency(1): Introduction"
date:   2021-03-26 13:19:35 +0900
categories: [OperatingSystem]
---

# Concurrency?
주요 주제는 multi-threading입니다. 우선 thread란 어떤 running process에 대한 abstraction이라고 할 수 있는데, 기존 single thread 프로그램과 멀티스레드 프로그램의 차이점은 멀티스레드는 동시에 여러 개의 point of execution을 가질 수 있다는 점입니다. 


# thread와 process의 차이점
1. address space가 같습니다. 즉, context switch에서 사용 중인 page table을 바꿀 필요가 없습니다. 
2. stack에 있어서 싱글스레드 프로그램에서는 address space에 스택이 하나만 있지만, 멀티스레드 프로그램에는 스레드마다 하나씩 해서 여러 개의 스택이 있습니다.

![single_threaded_and-multi_threaded_address_space](../../images/OS/single_threaded_and-multi_threaded_address_space.png
)


# Thread Context Switch
한 스레드를 실행하다가 다른 스레드를 실행할 때 일어나는 일입니다. 현재 running thread의 실행을 멈추고 다른 스레드의 실행을 resume하는데, 현재 running thread의 registers를 TCB에 저장하고 stack도 저장합니다. 그 다음 다른 스레드의 register를 그 스레드의 TCB에서 불러오고 stack 도 불러옵니다. 이 과정에서 다음 질문들에 답을 할 수 있어야 합니다. 
- What triggers a context switch?
- How does a voluntary context switch(e.g. a call to thread_yield) work?
- How does an involuntary context switch differ from a voluntary one?
- What thread should the scheduler choose to run next?

## What triggers a context switch?


# Condition Variables
