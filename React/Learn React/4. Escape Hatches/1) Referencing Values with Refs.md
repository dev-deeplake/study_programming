>[!Learning Goals]
>- How to add a ref to your component
>- How to update a ref's value
>- How refs are different from state
>- How to use refs safely

## 1. Ref?
- component가 기억은 하되, 값의 변경에 따른 리렌더링을 촉발하지 않는 데이터(정보)
- 내부 구조 : `{ current: initialValue }`
	- 일반 Javascript 객체 형태
	- current로 어떤 형태의 데이터든 할당할 수 있음
	- 값 변경시 setter를 통해 우회적으로 변경해야 하는 state와 달리 \[someRef].current로 직접적인 변형이 가능
		- state와는 달리 mutation에 대해 걱정할 필요가 없다!
- ### Ref의 추가 및 사용
	- useRef 훅에 초기값을 주어 선언하고,  \[someRef].current를 통해 활용 및 조작
	- ![[1) Referencing Values with Refs#🔅 간단한 ref의 추가 및 사용]]
		- 이 예시에선 handler에서 직접적으로 ref.current값을 변경하고 있고, 그것이 alert라는 외부 메시지로 표출되고 있다. 간단한 예제이므로 이 경우에는 문제가 없지만 side effect가 일어나는 곳에서 ref 값을 변경하는 것은 지양되어야 함
	- ![[1) Referencing Values with Refs#🔅 ref의 추가 및 사용 Stopwatch]]
		- 본 예제의 interval id는 렌더링에 이용되지 않으므로 ref의 사용에 더 적합한 예시
## 2. useRef 훅의 작동
- useRef는 useState를 기반으로 동작하고 있다고도 볼 수 있음
- ![[1) Referencing Values with Refs#🔅 useRef 훅의 작동]]
## 3. Ref vs State
| 항목            | ref                                         |            state            |
| --------------- | ------------------------------------------- |:---------------------------:|
| hook의 리턴     | { current: initialValue }                   |      \[value, setter]       |
| 리렌더링 촉발   | X                                           |              O              |
| value 직접 수정 | O                                           | X (setter를 통한 간접 수정) |
| 렌더링 중 활용  | X (렌더링 중에는 읽거나 사용하지 말아야 함) |              O (언제든 state를 읽고 쓸 수 있지만, 각 렌더는 변하지 않는 고유한 snapshot을 가짐)              |
- ### '렌더링 중 활용' +
	- 왜 굳이 '각 렌더는 변하지 않는 고유한 snapshot을 가진다'는 언급을 했을까?
		- state가 한 렌더링 주기 안에서 하나의 Snapshot을 사용한다는 것은 그 렌더링 주기 안에서는 state가 바뀌는 일이 없다는 것이다. (state가 변경되면 재렌더링이 일어남!)
		- 하지만 ref는 렌더링과 독립적이다. 즉, 렌더링 주기와 ref 값은 동기화되지 않는다.
			- 이에 따라 렌더링이 완료되기 전에 값이 변화할 수 있다.
			- 그리고 React는 ref.current가 언제 변경되었는지 추적하지 않는다 - 컴포넌트가 예측하기 어렵게 됨
			- 만일 ref를 이용해 DOM 트리를 직접 변경할 경우, react에서 관리하는 가상 DOM 트리와 실제 DOM 트리가 불일치하여 UI 업데이트 및 렌더링에 문제가 일어날 소지가 있다.
## 4. How to use ref well
- ref = just an 'escape hatch'
	- ref는 React와 바깥을 이어주는 수단
	- 외부 시스템과의 연결 및 렌더링과 관련 없는 단순한 데이터를 저장하기엔 좋지만, 앱의 로직과 데이터 플로우 상당 부분이 Ref에 의존하고 있다면 재고 필요!
- 렌더링 중에는 ref.current를 읽거나 쓰지 말기
	- 렌더링 중 데이터가 필요한 경우 state를 사용할 것
## 5. Use cases
- 컴포넌트의 UI 렌더링과 관계가 없어야 함 (필수) + React 외부의 요소와 연결되어야 할 때
	- timeout ID 저장 (sm. stopwatch)
	- 실제 DOM element의 저장 및 조작 🌟
	- JSX 연산에 필요하지 않은 객체 저장

## ------ Example List ------
#### 🔅 간단한 ref의 추가 및 사용
```javascript
import { useRef } from 'react';

export default function Counter() {
  const ref = useRef(0); // 초기값으로 0을 할당

  function handleClick() {
    ref.current = ref.current + 1; // 클릭시마다 현재의 ref 값에 1을 추가
    alert('You clicked ' + ref.current + ' times!'); // 업데이트된 ref를 alert창에서 표출
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}

```
#### 🔅 ref의 추가 및 사용 : Stopwatch
- start 버튼을 눌러 시간 측정을 시작하고 stop을 클릭했을 때는 멈추는 타이머 기능
```javascript
import { useState, useRef } from 'react';

export default function Stopwatch() {
// state와 ref를 함께 사용
  const [startTime, setStartTime] = useState(null); // 시작 시간 set
  const [now, setNow] = useState(null); // 현재 시간 set
  const intervalRef = useRef(null); // setInterval이 반환하는 ID 저장용

  function handleStart() {
    setStartTime(Date.now()); // 시작 시간으로 현재 시각을 setting
    setNow(Date.now()); // 지금 시간 set

    clearInterval(intervalRef.current); // start 버튼 클릭시 타이머가 리셋되어야하므로 존재하던 타이머를 지워줌
    intervalRef.current = setInterval(() => { // ref에 다음 interval id를 저장, 이 id는 해당 Interval이 중지될 때까지 동일하게 유지됨
      setNow(Date.now());
    }, 10); // 10ms 단위로 지금 시간을 업데이트
  }

  function handleStop() {
	clearInterval(intervalRef.current); // stop 버튼 클릭시 ref에 저장되어있던 Id를 이용하여 interval 해제
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```
#### 🔅 useRef 훅의 작동
```javascript
function useRef(initialValue) {
	const [ ref, _ ] = useState({ current: initialValue })
	return ref // 즉, useRef는 useState를 사용하되 setter 함수를 사용하지 않고 current key를 가진 js 객체를 state value로 리턴하는 함수와 유사
	// ref.current의 값이 계속 변경된다 → 새로운 객체를 만들지 않고 동일한 객체 내에서 값이 계속 업데이트!
}
```