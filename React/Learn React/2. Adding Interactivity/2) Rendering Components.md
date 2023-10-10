>[!Learning Goals]
>- What rendering means in React
>- When and why React renders a component
>- The steps involved in displaying a component on screen
>- Why rendering does not always produce a DOM update

- 컴포넌트가 화면에 표시되기 전에 React에 의해 '렌더링'을 거친다.
- React에서 유저의 요청을 받고, 그 요청을 처리하여 다시 되돌려주는 과정은 다음의 3단계를 거친다.
	1. Triggering (렌더링의 촉발)
	2. Rendering (React가 컴포넌트를 렌더링함)
	3. Committing (React가 DOM에 렌더링을 커밋함)

## 1. Triggering
- 렌더링이 촉발되는 것은 다음의 2가지 경우이다.
	- 컴포넌트의 Initial rendering시
		- ![[2) Rendering Components#🔅 initial render triggering 관련 예시 코드]]
			- `createRoot`를 target인 root DOM 노드와 함께 호출
				- `createRoot(...)`는 render와 unmount 메소드가 있는 객체를 반환
			- 호출한 대상(예시의 root) 객체가 가지고 있는 `render` 메소드를 이용해 root 안에 렌더링할 대상을 명시
				- 대상으로 명시될 수 있는 것은 React Node(JSX / 리액트 컴포넌트), createElement로 생성된 리액트 엘리먼트, string, number, null, undefined 등 다양
				- `root.render(...)`를 처음 호출했을 때, 리액트는 기존 React root 내의 모든 HTML 콘텐츠를 모두 삭제한 뒤 react 컴포넌트를 렌더링함
	- 컴포넌트의 state가 변화했을 시
		- 컴포넌트의 첫 렌더링이 완료되면 state의 setter 함수로 state를 업데이트하여 추가적인 렌더링을 발생시킬 수 있음
## 2. Rendering (React가 컴포넌트를 렌더링함)
- React의 렌더링 = "리액트가 컴포넌트를 호출함" : React는 이 과정에서 실제로 스크린에 무엇을 표시해야하는지 알아낸다.
	- 컴포넌트의 initial rendering시 : React가 root 컴포넌트를 호출
	- initial 외의 컴포넌트 렌더링 : React는 state가 업데이트된 React 컴포넌트 함수를 호출
	- react는 virtual DOM을 이용해 컴포넌트의 변화를 파악하고, 실제 DOM과 비교해 변경이 필요한 부분을 결정한다. ==즉, 이 단계에서 react는 자신의 [[Virtual DOM]] tree를 구축하는 것!==
- ==컴포넌트 렌더링 과정은 recursive하게 반복==
	- A > B > C 순으로 컴포넌트가 중첩되어있다고 할 때,
		- initial rendering에서 React는 루트 컴포넌트인 A를 렌더링하고
		- A 컴포넌트가 B 컴포넌트를 반환하면, react는 그 때 B 컴포넌트를 렌더링한다.
		- B 컴포넌트가 C 컴포넌트를 반환하면, react는 그 때 C 컴포넌트를 렌더링한다.
		- C 컴포넌트가 더 이상의 자식 컴포넌트를 반환하지 않는다면, react는 렌더링 과정을 끝내고 이 시점에서 무엇을 화면에 그려야 할지 결정하게 된다.
- 렌더링은 항상 '순수한' 계산이어야 함 : 리액트가 컴포넌트를 호출했을 때, 같은 input이 들어갔다면 output 역시 항상 같아야 한다.
- ![[2) Rendering Components#🔅 rendering 관련 예시 코드]]
	- ==initial rendering: react는 `<section>`, `<h1>`, 그리고 3개의 `<img>` 태그를 읽고 `document.createElement()`를 이용해 해당 DOM 노드를 생성한다.==
	- ==subsequent rendering: react는 이전 렌더링에 비해 달라진 요소가 있는지 계산'만' 한다! (실제 작업은 commit 페이즈에서 실행한다)==
### ==virtual DOM vs DOM==
- virtual DOM은 실제 DOM을 추상화한 JavaScript 객체
![[Virtual DOM#예시]]

## 3. Committing (React가 DOM에 렌더링을 커밋)
 - initial rendering : screen에 그려져야 할 모든 node를 `appendChild()` DOM API를 이용해 그린다.
 - subsequent rendering : 렌더링 과정에서 변경이 필요하다고 계산된 부분(이전 렌더링과 새 렌더링 간에 차이가 있는 부분)만 변경
 - 🌟 ==DOM 트리의 변경은 웹 페이지상에 그려지는 UI 변경과는 다르다! 웹 페이지의 구조는 변경되지만, 사용자는 커밋 단계에서 아직 이 변경을 보지 못함.==

## cf) 브라우저의 화면 렌더링 (=페인팅)
 - react의 렌더링 작업이 완료되고, commit 작업까지 끝난 뒤 브라우저가 화면을 다시 그리는 단계.
***
------ example list ------
#### 🔅 initial render triggering 관련 예시 코드
```javascript
const Image = () => {
	const imageSrc = "https://example.img.source/ex"
	const alt = "explanation of the image"
	return <img src={imageSrc} src={alt} />
}

import { createRoot } from "react-dom/client"

const root = createRoot(document.getElementById("root"))
root.render(<Image />)
```

#### 🔅 rendering 관련 예시 코드
```javascript
// Gallery.js
export default function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

```javascript
// index.js
import Gallery from './Gallery.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Gallery />);
```