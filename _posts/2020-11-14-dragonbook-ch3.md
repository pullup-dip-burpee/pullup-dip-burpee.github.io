---
layout: post
title:  "용책 읽기 - Ch.3"
date:   2020-03-10 20:19:35 +0900
categories: [textbooks/compiler]
---


## Recognition of Tokens

### 3.4.1 Translation Diagrams
- translation diagrams
- states
- Edges


### 3.4.2 Recognition of Reserved Words and Identifiers
Keyword와 Identifier를 어떻게 알아볼 수 있을까? 두 가지 방법이 있다.


1. reserved words(keywords)를 처음에 symbol table에 install한다. 
2. 각 keyword에 대해 분리된 transition diagram을 만든다. 

### 3.4.3 Completion of the Running Examples


### 3.4.4 Architecture of a Transition-Diagram-Based Lexical Analyzer
Translation diagrams를 가지고 lexical analyzer를 만들기. *state*라는 변수를 통해 translation diagram의 state를 나타내게 하고, 

### 3.5 The Lexical-Analyzer Generator *Lex*



### 3.6 Finite Automata
유한 오토마타. Input string은 우선 유한 오토마타로 바뀐 뒤 다시 syntax analyzer가 다룰 수 있는 형태가 된다.  NFA(Non-deterministic Automata)와 DFA(Deterministic Automata)가 있다. 

Regular expression -> NFA -> DFA