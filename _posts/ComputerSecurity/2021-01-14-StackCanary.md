---
layout: post
title:  "Stack Canary"
date:   2021-01-14 20:19:35 +0900
categories: [ComputerSecurity]
---

# stack overflow 방어

Stack Canary는 스택이 변조되었는지를 검사하는 보안 기법입니다. Stack overflow 공격을 방어합니다. canary라는 특정 값을 stack 상의 특정 위치에 넣어서, overflow가 있었는지를 검사합니다. 

# 컴파일시 유무 설정

cc로 컴파일 시 `-fstack-protector`, `-fnostack-protector` 중 무엇을 주냐에 따라 stack canary를 줄지 말지 결정할 수 있습니다. 

```
    mov    ebp,esp
    push   esi
    push   ebx
    sub    esp,0x18
    call   0x80484d0 <__x86.get_pc_thunk.bx>
    add    ebx,0x1a5d
    mov    eax,DWORD PTR [ebp+0xc]
    mov    DWORD PTR [ebp-0x20],eax
    mov    eax,gs:0x14
    mov    DWORD PTR [ebp-0xc],eax
    xor    eax,eax
    call   0x8048420 <geteuid@plt>
    mov    esi,eax
    
    ...

    mov    edx,DWORD PTR [ebp-0xc]
    xor    edx,DWORD PTR gs:0x14
    call   0x80486d0 <__stack_chk_fail_local>
    pop    ebx
    pop    esi
    pop    ebp
```

이 코드는 stack canary가 적용된 모습인데, 각각 prologue와 epilogue입니다. prologue 의 `mov    eax,gs:0x14`가 gs:0x14에 저장된 stack canary를 eax로 옮기고, epilogue의 `xor    edx,DWORD PTR gs:0x14`가 `DWORD PTR [ebp-0xc]`의 stack canary 값을 원래의 값과 비교합니다. `__stack_chk_fail_local`가 canary 값이 바뀌었는지를 검사합니다. 

ssp가 적용되면 stack 내부의 값들의 순서가 아래와 같이 바뀝니다. return address를 덮어쓰기 위해서는 canary 값을 바꾸어야 하는데, canary 값이 변조된 것이 감지되기 때문에 return address를 덮어쓸 수 없습니다. 

![스택까나리](/images/computersecurity/stack-canary.png)  
(이미지출처: https://access.redhat.com/blogs/766093/posts/3548631)

실습에서는 이 canary 값을 알아내기 쉽게 만들고 해당 canary 부분에 맞는 canary 값까지 입력값으로 넣거나, canary를 덮어쓰지 않고 우회하는 방법 등을 배웁니다. 

# 예시

ASLR이 없다는 가정 아래에서 파훼법입니다. 

1. 카나리 값이 고정되어 있는 경우  
    그 값을 카나리 위치에 넣어서 buffer overflow 공격을 합니다.
2. 카나리 값은 변경되지만 우회할 수 있는 경우  
    가령 input을 읽어들이는 read 등의 함수가, canary보다 destination을 설정하는 값을 먼저 건드린다면, 해당 값을 잘 살펴서 임의의 메모리 주소에 입력을 넣을 수 있습니다. return address 주소부터 덮어쓰는 것도 가능합니다. 

