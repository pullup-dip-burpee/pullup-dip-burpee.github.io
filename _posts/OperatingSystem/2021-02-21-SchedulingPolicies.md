---
layout: post
title:  "Scheduling Policies"
date:   2021-02-21 13:19:35 +0900
categories: [OperatingSystem]
---

OS scheduler는 CPU resource를 여러 프로세스에 잘 할당하기 위해 scheduling을 하는데 종류가 있습니다. 크게는 preemptive(not cooperative)와 non-preemptive(cooperative)로 나눌 수 있는데, 전자는 OS가 process를 강제로 suspend하고 다른 process를 실행시킬 수 있고, 후자는 그냥 현재 process가 끝날 때까지 기다립니다. 다음 여러 가지 간단한 policies가 있습니다. 

# FIFO(First-In-First-Out)
단순히 먼저 온 게 먼저 되는데 convey effect 문제가 있을 수 있고 turnaround time이 대개는 길어져서 안 좋습니다. 

# SJF(Shortest-Job-First)

process 들이 동시에 arrive한 경우 짧은 걸 먼저 합니다. 하지만 non-preemptive라서, 긴 게 먼저 도착하는 경우 여전히 짧은 것들이 뒤에서 막히는 문제가 있습니다. 
# STCF(Shortest Time to Completion First) == SRTF(Shortest Remaining Time First)
SJF의 보완인데 preemptive 합니다. 

# RR(Round Robin)

프로세스마다 시간을 할당합니다. 할당된 시간을 다 썼는데도 안 끝난 프로세스는 큐의 마지막으로 갑니다. 

# MLQ(Multi Level Queue)
큐가 우선순위에 따라 여러 개 있습니다. 우선순위 0번 큐의 process들이 먼저 실행되고, 같은 우선순위끼리는 round robin 적용하고..

# MLFQ(Multi Level Feedback Queue)

Linux에서 사용하는데, MLQ와 대개 비슷하지만 process가 여러 queue 사이를 이동할 수 있습니다. 우선순위 처음에 너무 오래 있으면 하나 내리고, 마지막에 너무 오래 있으면 다시 높게 올려주고 하는 식으로 aging을 통해 기아 상태(starvation)를 막습니다. 

