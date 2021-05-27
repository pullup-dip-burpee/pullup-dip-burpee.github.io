---
layout: post
title:  "[OS]File System"
date:   2021-05-19 13:19:35 +0900
categories: [OperatingSystem]
---

파일 시스템 강의를 듣고 메모합니다. 

# open files
`open`이 있는 이유는 name resolution의 불필요한 반복을 막기 위하여. 
- Name resolution을 통해서 디렉토리에서 파일 이름을 찾고 permission확인.
- metadata는 open file table(file ddescriptor table)에 저장됨. 
- Control path, data path가 분리됨. name resolution 등은 control path 통해서. 

# descriptor
- (descriptor table)--(open file table)--(v-node table)
- descriptor table: process 당 하나씩
- open file table: 모든 process가 공유
- v-node table: 모든 process가 공유

# v-node란? 

# i-node란? 
파일 시스템을 위해 UNIX에서 사용하는 구조. 모든 파일/디렉토리는 i-node로 표현됨 

	
## Problem 2: Unorganized Free List
- 해결 방법?
	- 주기적인 compact/defragment disk. 단점: 
	- 인접한 free block들을 free list에 같이 두기
	- Bitmap of free blocks: 좋음.
	
## Problem 3: Data Locality
- Locality techniques
- Cylinder Group
	- 같은 cyliner group 안에 있는 건 빠르게 seek할 수 있음. 
	- superblock, ... 

# ext(EXTended file system)
- 리눅스를 위해 만들어진 파일 시스템
- ext1
    - 문제점: linked list사용해서 free block, inode를 트래킹하기 때문에 fragmentation 발생? 
- ext2: prj4에서 다룸. 
	- 
     - 블록 사이즈(1 KB ~ 8 KB)에 따라 파일 크기 제한 및 파일 시스템 크기 제한이 달라짐
    - `struct ext2_inode`: on-disk 정보
    - `struct ext2_inode_info`: in-memory 정보
- ext3
	- ext2에서 새로운 개념 추가됨.
	- File System Reliability
	- 하나의 logical file operation이 여러개의 물리적인 disk block들을 업데이트 할 수 있다. 
- ext4
	- 오늘날 리눅스가 주로 쓰는 파일시스템

## Possible Crash Scenarios
1. B I D : No problem
2. B' I D: 
3. 

## Inconsistency 해결책: fsck
- reboot할 때 FS를 consistent하도록 전체 disk를 스캔함.
	- 장: FS 코드가 단순해짐, 단지 crashed FS 처리 말고도 repair 가능
	- 단: 
## 다른 해결책: Journaling
- Write-ahead logging from database community
- 
