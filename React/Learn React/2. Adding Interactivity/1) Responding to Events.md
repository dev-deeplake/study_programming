>[!Learning Goals]
>- Different ways to write an event handler
>- How to pass event handling logic from a parent component
>- How events propagate and how to stop them

## Event handler (이벤트 핸들러)
### 1. Event Handler?
- ==상호작용 (interaction - eg. click, hover, focus...)에 반응하여 트리거되는 커스텀 함수==
- 리액트에서는 JSX에 event handler를 넘겨줄 수 있다.
- 이벤트 핸들러는(순수성을 지켜야 하는 렌더링 함수와는 다르게) 상호작용에 반응하여 어떤 일을 수행하는 것이기 때문에 그 안에서 다른 것을 변화시킬 가능성이 높다. 리액트에서는 해당 변경을 state로 관리한다.
### 2. 네이밍 컨벤션
- handle + \[Event Name]
	- ex) handleDelete, handleOnClick ...
### 3. 이벤트 핸들러를 추가하는 방법
1. ==컴포넌트 안에서 이벤트 핸들러를 정의==
	- 🌟 클로저로 인해 이벤트 핸들러가, 자신이 정의된 컴포넌트의 Prop에 자유롭게 접근할 수 있음
	- 이벤트 핸들러를 사용할 컴포넌트 안에서 정의하는 경우
		- ![[1) Responding to Events#🔅 이벤트 핸들러 정의 - 사용될 컴포넌트 안에서 정의하는 예시]]
	- 부모 컴포넌트에서 이벤트 핸들러를 정의하여 자식에 prop으로 넘겨주는 경우
		- ![[1) Responding to Events#🔅 이벤트 핸들러 정의 - 부모 컴포넌트에서 이벤트 핸들러를 정의한 뒤 자식에게 prop으로 넘기는 예시]]
		- ==같은 상호작용에서 핸들러에 따라 컴포넌트가 실행하는 기능에 차이를 두고 싶을 때==
		- 버튼과 같은 기본 컴포넌트에 스타일링을 포함하되 동작을 포함하지 않는다면 부모 컴포넌트에서 그 동작을 제어하게 된다. (참고: [[디자인 시스템 (Design System)]])
		- 이벤트 핸들러 prop의 네이밍 컨벤션
			- Built-in component(HTML 태그와 동일한)의 경우 브라우저 이벤트 이름과 동일하게 설정해주어야 함
				- [참고: 브라우저 이벤트명](https://react-ko.dev/reference/react-dom/components/common#common-props)
			- Custom component의 경우 원하는 방식으로 이벤트 핸들러명 지정 가능
				- on + 대문자로 시작 (ex. onDelete, onClick ...)
2. JSX에서 inline 함수로 정의하여 바로 넘겨주는 방법
	- ==이벤트 핸들러가 짧고 간단하며 재사용되지 않을 경우..!==
	- ![[1) Responding to Events#🔅 이벤트 핸들러 정의 - JSX에서의 Inline 정의]]

## Event Propagation (이벤트 전파)
### 1. [[Event Bubbling]]
>![[Event Bubbling#정의와 특성]]
- 이벤트 중 Scroll 이벤트의 경우 웹 표준에서도 버블링이 일어나지 않기 때문에, 웹 표준을 따른 React에서도 마찬가지.
### 2. 이벤트 전파를 막기
- 이벤트 핸들러는 인수로 이벤트 객체만을 받으며, 이 이벤트 객체를 이용해 이벤트 전파를 막을 수 있다.
1) e.stopPropagation()
	- ==부모로 전파되는 버블링을 막는다.==
	- 이용 예시: `자식 컴포넌트와 부모 컴포넌트에 동일한 이벤트가 부착되어있으나 부모와 자식 컴포넌트의 이벤트가 분리되어 동작해야 할 경우`
2) e.preventDefault()
	- ==브라우저 이벤트에 연결된 기본 동작을 막는다.==
	- 이용 예시: `<form>요소에서 내부 버튼 클릭시 전체 페이지가 재로드되는 기본 동작을 막고 싶을 때`
- 이벤트 전파가 미치는 영향이 커 부착된 핸들러의 실행 및 추적이 어려운 경우, 전파를 막고 핸들러를 넘겨 전파의 대용으로 사용할 수 있다.
	- ![[1) Responding to Events#🔅 전파를 막고 핸들러 passing한 예시]]
### 🔥 '이벤트 전파' 같은 건 왜 만들었을까 🧐 ?
- 이벤트 전파가 있기 때문에 부모에서 자식의 이벤트를 모두 capture할 수 있다.
- ==즉, 상위 요소에서 하위 이벤트 요소를 한꺼번에 관리할 수 있다는 뜻!==
	- ![[1) Responding to Events#🔅 상위 노드에서 하위 노드의 이벤트 요소를 한꺼번에 관리하는 예시]]

------ example lists ------
#### 🔅 이벤트 핸들러 정의 - 사용될 컴포넌트 안에서 정의하는 예시
```javascript
const Button = ({ children }) => {
	const handleOnClick = () => alert("the button is clicked!")
	return <button onClick={handleOnClick}>{children}</button>
}
```

#### 🔅 이벤트 핸들러 정의 - 부모 컴포넌트에서 이벤트 핸들러를 정의한 뒤 자식에게 prop으로 넘기는 예시
```javascript
const Button = ({ buttonText, onClickHandler }) => {
	return <button onClick={onClickHandler}>{buttonText}</button>
}

const SubmitButton = ({ children }) => {
	const handleOnClick = () => alert("Submitted!")
	return <Button buttonText={children} onClickHandler={handleOnClick} />
}
```

#### 🔅 이벤트 핸들러 정의 - JSX에서의 Inline 정의
```javascript
const Button = ({ children }) => {
	return <button onClick={() => alert("the button is clicked!")}>{children}</button>
}
```
#### 🔅 전파를 막고 핸들러 passing한 예시
```javascript
...
const Button = ({ onClick, children }) => {
	return (
		<button onClick={ e => {
			e.stopPropagation() // 이벤트의 전파를 우선적으로 막음
			/**
			 *
			 * 이 부분에 추가 실행하고 싶은 코드를 자유롭게 작성
			 *
			**/
			onClick() // Button 컴포넌트의 prop인 onClick 이벤트 핸들러를 실행시킴
		}}
	)
}

const ButtonMenuBar = () => { // 버튼 1, 버튼 2, 버튼의 부모 div에 모두 onClick이 달려있지만 Button의 e.stopPropagation()으로 인해 이벤트가 div까지 전달되지 않음
	const [showButtons, setShowButtons] = useState(true)
	return (
		<div onClick={() => setShowButtons(!showButtons)}>
			{
				showButtons && (
					<>
						<Button onClick={() => alert("button 1 clicked!")}>button 1</button>
						<Button onClick={() => alert("button 2 clicked!")}>button 2</button>
					</>
				)	
			}
		</div>
	)
}
```
#### 🔅 상위 노드에서 하위 노드의 이벤트 요소를 한꺼번에 관리하는 예시
```javascript
const ButtonSet = () => {
	const handleParentClick = (e) => {
		const buttonId = e.target.id;
		switch (buttonId) {
			case "submit":
				alert("submitted!")
				break
			case "cancel":
				alert("canceled!")
				break
			default:
				break
		}
	}
	return (
		<div onClick={handleParentClick}>
			<button id="submit">submit</button>
			<button id="cancel">cancel</button>
		</div>
	)
}
```