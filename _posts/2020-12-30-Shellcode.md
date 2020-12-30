---
layout: post
title:  "Shellcode란?"
date:   2020-12-30 20:19:35 +0900
categories: [ComputerArchitecture]
---

# Shellcode란?
## Shellcode 튜토리얼
- [CS6265 튜토리얼](https://tc.gts3.org/cs6265/2020/tut/tut02-warmup2.html#shellcode) 참고  
Shellcode란 [exploitation(취약점 공격)](https://ko.wikipedia.org/wiki/%EC%B7%A8%EC%95%BD%EC%A0%90_%EA%B3%B5%EA%B2%A9)을 위한 generic [payload](https://ko.wikipedia.org/wiki/%ED%8E%98%EC%9D%B4%EB%A1%9C%EB%93%9C_(%EC%BB%B4%ED%93%A8%ED%8C%85))이다. 


## execve란

`int execve(const char *filename, char *const argv[], char *const envp[]);`

이 함수는 현재 실행중인 프로세스의 실행코드를 실행가능한 파일인 filename로 바꾼다. 현재 실행 중인 프로세스의 기능이 filename의 것으로 완전히 바뀐다는 점에서, 기존 프로세스는 있으면서 새로운 프로세스를 만드는 fork()와 구별된다. 

## shellcode에서 execve

시스템 해킹의 shellcode에서도 사용되는데, 

