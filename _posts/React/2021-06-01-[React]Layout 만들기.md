---
layout: post
title:  "[React]Layout 만들기"
date:   2021-06-01 14:19:35 +0900
categories: React
---

# 시작
- npm, nodejs 설치
```javascript
npx create-react-app [AppName]
cd AppName
npm start
```

## 소스파일 구조
- 모든 JS파일과 CSS 파일은 `public/` 안에. 
- `src/index.js`를 가장 먼저 봐야 함. JavaScript entry point. 그러면 아래처럼 되어있는데, 
```javascript
ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```
여기서 App이 포함됨. 다시 `src/App`을 보면 아래처럼 되어 있는데 여기에 추가. 이번 과제 같은 경우는 SmartPotManager라는 컴포넌트를 만들고 App에 추가한 것.

```javascript
function App() {
	return (
    <div className="App">
      { <SmartpotManager />}
    </div>
  );
}
```

- 각 component는 `src/components/` 안에 있음. 

## 예쁘게 꾸미기?
- material-ui 다운로드

	```
		npm install @material-ui/core
		npm install @material-ui/icons
	```
	
	
- react 반응형 UI 만들기
	- 가령 어떤 component에 들어가는 게 세 부분(potImange, control, graph) 있다면 아래와 같이 할 수 있음. 
	
```javascript
const useStyles = makeStyles(theme => ({
	root: {
	    margin: 10,
	    padding: 10,
	},
	potImageView: {
	    padding: 10,
	    margin: theme.spacing(10),
	    backgroundColor: "blue",
	},
	controlView: {
	    padding: 10,
	    margin: theme.spacing(10),
	    backgroundColor: "green",
	},
	graphView: {
	    padding: 10,
	    margin: theme.spacing(10),
	    backgroundColor: "red",
	},

}));
```

이런 식으로 함수 하나 만들어놓고, 실제 사용할 때는 `const classes = useStyles();` 같은 식으로 사용. 

## 그래프 그리기

- 그래프 라이브러리: [recharts](https://recharts.org/en-US/guide/getting-started)
- installation: `npm install recharts`
- 습도 그래프 그려보기. 아래처럼 하면 그래프 그리기에 필요한 JSX.Element 정의됨. 
```javascript
const renderLineChart = (
    <LineChart width={400} height={400} data={data}>
        <Line type="monotone" dataKey="moisture" stroke="#8884d8" />
        <CartesianGrid stroke="#ccc" />
        <XAxis dataKey='name' />
        <YAxis />
    </LineChart>
);
```

## Modal 사용하기



