---
layout: post
title:  "[React] 공부 시작"
date:   2021-05-26 14:19:35 +0900
categories: React
---

연구실 인턴 하면서 UI를 만들 기회가 생겨서 react 공부 시작하게 되었습니다. [react 문서](https://ko.reactjs.org/docs/hello-world.html) 및 [react 튜토리얼](https://ko.reactjs.org/tutorial/tutorial.html) 등을 보면서 메모합니다. Javascript 및 Typescript도 잘 모르기 때문에 이번 기회에 같이 공부합니다. 

# Javascript 기초
## Number
- 놀라운 사실: 정수타입이 없다. Number는 모두 IEEE 754 64bit double precision으로 저장된다.(단 BigInt는 있다)
- 문자열->정수: `parseInt('123', 10); // 123` 2번째 argument에 따라 N진법으로 해석됨.
- 문자열->실수: `parseFloat('1234.5');`
- NaN과 그걸 검사하는 함수. `isNaN(NaN); // true`
- isFinite(): Infinity, -Infinity, NaN 구별 가능

## String
- 문자열도 객체로 취급
- 길이: `'hello'.length;`
- 다른 기본적인 함수들
	```
	'hello'.charAt(0); // "h"
	'hello, world'.replace('hello', 'goodbye'); // "goodbye, world"
	'hello'.toUpperCase(); // "HELLO"
	```

## Variables
- let
- const
- var

-----

# React 기초
여기서부터는 React.
- 지금까지 배운 것
	- 레이아웃 만들기
	- 컴포넌트 사용법. 컴포넌트 만들어서 한 컴포넌트 안에 다른 컴포넌트 넣고...
	- material-ui 가져다 쓰기


## hello world
```
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```

## JSX


## component

