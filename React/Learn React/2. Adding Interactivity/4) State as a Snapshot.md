>[!Learning Goals]
>- How setting state triggers re-renders
>- When and how state updates
>- Why state does not update immediately after you set it
>- How event handlers access a “snapshot” of the state

## 포인트 1: state는 React가 직접 관리한다
- state는 함수(컴포넌트)가 호출되고 리턴값을 넘겼을 때 사라질 일반 변수와는 달리 함수가 종료되고서도 남아있다.
	- 즉, state는 함수 내에서 선언되지만 함수에 종속적이지 않고 React 자체에서 관리한다.
- React는 컴포넌트 함수를 호출할 때 (=렌더링할 때) 자신이 관리하고 있던 state를 컴포넌트 함수에 제공하여, 함수에서 props, event handler, 지역 변수등이 연산된 새로운 UI snapshot(=JSX)이 만들어지도록 한다.
## 포인트 2: setter 함수와 event handler의 동작을 분리해서 생각하기
- React는 이벤트 핸들러(또는 상황에 따라 전달된 인라인 익명 함수)의 모든 코드가 실행되길 기다린 뒤 State의 업데이트를 시작한다(React waits until all code in the event handlers has run before processing your state updates).
	- 큐 안의 모든 명령이 실행되고 난 뒤의 예비 state 값을, 이전 렌더링의 결과로 얻은 state와(snapshot state)과 비교해서, 그 값이 동일하면 state를 업데이트하지 않고 다른 경우에만 업데이트한다.
	- 리렌더링은 이 state의 업데이트가 일어나는 경우에만 이루어진다.
- state setter function은 state를 변경하지 않고 리렌더링을 촉발하는 역할만 한다.
	- '리렌더링의 촉발(triggering a re-render)' : 리렌더링이 일어난다는 뜻이 아니라, 리렌더링을 초래할 수 있는 동작을 예약한다는 의미
	- setter 함수는 호출된 순서대로 [[큐(Queue)와 스택(Stack)#큐|큐]]에 저장된다.
		- ![[큐(Queue)와 스택(Stack)#특징|큐]]
		- '리렌더링의 촉발' 대신 'setter의 호출은 해당 setter에 주어진 state 변경 관련 로직을 큐에 올리고, 결과 연산 및 state 업데이트를 대기한다' 정도로 했다면..?
- 🌟 종합하면 setter 함수는 state를 변경하지 않고, 새롭게 state가 되어야 할 연산의 결과값이 snapshot state와 다른 경우, event handler의 실행이 state를 변경한다고 할 수 있음

## 포인트 3: 이벤트 핸들러(및 인라인 익명 함수)에 제공된 state는 변경되지 않는다
- state setter가 state를 변경하지 않으므로, 핸들러 내에서 사용하는 state는 모두 snapshot state가 될 수 밖에 없다.
	- 이는 핸들러 내부의 동작이 비동기적이더라도 일괄적으로 적용된다. (ex: setTimeOut으로 아무리 오래 기다려도 새 state 값 대신 snapshot state를 얻게 됨)
- 이로 인해 동일한 setter로 state를 여러번 변경하더라도 연산 결과가 일반적인 js 함수에서와 달라질 수 있다.

## 🌟 초기 렌더링부터의 동작 정리
### 🔅 예시 코드
```javascript
// 클릭시 switch on / off를 변경하도록 고안된 컴포넌트
const Switch = () => {
	const [isSwitchOn, setIsSwitchOn] = useState(false)
	const onClickHandler = () => {
		setIsSwitchOn(!isSwitchOn)
		setIsSwitchOn(!isSwitchOn)
	}
	return (
		<>
			<button onClick={onClickHandler}>turn switch</button>
			<p>Switch {isSwitchOn ? "On" : "Off"}</p>
		</>
	)
}
```

1. 초기 렌더링시의 snapshot state: `isSwitchOn` = false
2. 유저가 UI 요소와 인터랙션 → 'turn switch' 버튼을 클릭
3. 이벤트 핸들러(또는 상황에 따라 JSX에 전달된 인라인 익명 함수)가 호출된다.
	- `setIsSwitchOn(!isSwitchOn)`이 2번 들어갔으나, ==이벤트 핸들러에 전달된 isSwitchOn은 snapshot state(false)==이다.
	- ==state setter가 호출된 순서대로 React 내부 큐에 저장되어 실행 대기상태==가 된다.
	- 핸들러에서 사용되는 state가 false로 동일하기 때문에 setter에서 값 반전이 두 번 사용되었어도 핸들러의 state 연산값은 true가 된다.
	- ![[4) State as a Snapshot#🔅 React 내부 Queue 예시]]
4. 이벤트 핸들러의 모든 코드가 실행되고 난 뒤 React가 내부 queue의 작업들을 처리하며 state의 업데이트를 시작한다.
	- snapshot state는 false, 핸들러 실행 이후 state는 true이기 때문에 state 업데이트가 일어난다.
5. 업데이트된 state에 따라 react가 컴포넌트 함수 호출(=재렌더링)을 시작한다.
	- 재렌더링이 일어나므로 snapshot state가 변경된다.
	- 변경된 state를 기반으로 props, event handler, 지역 변수가 재연산되고, 이는 JSX에 담겨 리턴된다(=UI snapshot).
		- ![[4) State as a Snapshot#🔅 이 시점의 예시 코드 mental state]]
	- React는 컴포넌트 함수의 리턴(JSX)을 토대로 DOM 트리를 업데이트한다.
		- Virtual DOM으로 컴포넌트 변화 파악 → 실제 DOM과 비교해 변경 필요 부분 결정 → 변경이 필요하다고 여겨진 부분을 DOM에 커밋
		-  Rendering takes a snapshot in time : ==모든 state 변경과 UI 변경이 확정된 단계에서 해당 렌더링의 스냅샷을 찍는다고 볼 수 있음==


------ example list ------
#### 🔅 React 내부 Queue 예시
| Queued update | isSwitchOn | returns |
| ------------- | ---------- | ------- |
| !isSwitchOn   | false      | true    |
| !isSwitchOn   | false      | true    |
#### 🔅 이 시점의 예시 코드 mental state
```javascript
const Switch = () => { // 모든 isSwitchOn이 true로 변경되어 리턴된다고 보면..!
	const [isSwitchOn, setIsSwitchOn] = useState(false) // 현재 isSwitchOn = true로 변경된 상태
	const onClickHandler = () => { // 다음번에 유저 인터랙션이 일어나면 이 핸들러가 전달된다.
		setIsSwitchOn(!true)
		setIsSwitchOn(!true)
	}
	return (
		<>
			<button onClick={onClickHandler}>turn switch</button>
			<p>Switch {true ? "On" : "Off"}</p>
		</>
	)
}
```
