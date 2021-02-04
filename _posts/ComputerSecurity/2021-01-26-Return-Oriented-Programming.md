---
layout: post
title:  "Return Oriented Programming"
date:   2021-01-17 20:19:35 +0900
categories: [ComputerSecurity]
---

# ROP란?
ROP(Return Oriented Programming)란 이름은 프로그래밍이지만 사실은 보안 취약점 공격인데, stack smashing의 업그레이드 버전쯤 되는 것 같습니다. Shellcode를 직접 메모리에 올릴 수 없을 때 사용하는데, 이전의 shellcode 등보다 비교적 최근에 나온 강력한 기법이라고 합니다. NX(DEP)를 우회할 수 있다고 합니다. 

# NX? DEP?
NX(No-eXecute)나 DEP(Data Execution Prevention)는 메모리 상 특정 영역을 non-executable하게 만드는 보안기법입니다. stack, heap 등은 non-executable하므로, 해당 영역에 shellcode를 올렸다 치더라도 그걸 실행할 수 없고, program counter가 그 영역에 들어가면 프로세스가 종료됩니다. 

# ASLR?
ASLR(Address Space Layout Randomization)은 메모리 상의 heap, stack, shared libraries 주소 등을 랜덤화해서 프로세스가 실행될 때마다 다른 주소가 뜨게 합니다. 그래서 고정된 주소를 바탕으로 한 공격을 막습니다. 

# gcc에서 canary, NX, ASLR 등 사용하기
default가 사용인 것 같은데, 아무튼 컴파일 시 
- canary: `-fstack-protector`  - 켜짐, `-fno-stack-protector` - 꺼짐
- NX
- ASLR
# x86 (32bit) 예시 

# gadget
gadget이란 

`ropper --file ./target` 으로 gadget을 볼 수 있습니다. 

# x86-64 (64bit) 예시
`man syscall`커맨드로 syscall의 동작방식을 보면 아래와 같습니다. 

![mansyscall](/images/computersecurity/man_syscall.png)

x86에서는 스택에 쌓아서 파라미터를 넘기지만 x86-64에서는 레지스터를 사용합니다. 따라서 `rdi, rsi, ...` 등에 파라미터를 넘기는 gadget을 사용해야 합니다. 



