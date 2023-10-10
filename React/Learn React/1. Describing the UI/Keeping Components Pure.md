>[!Learning Goals]
> - What purity is and how it helps you avoid bugs
> 	- A component must be pure, meaning:
> 		- It minds its own business: It should not change any objects or variables that existed before rendering.
> 		- Same inputs, same output: Given the same inputs, a component should always return the same JSX.
> 	- Rendering can happen at any time, so components should not depend on each others’ rendering sequence.
> - How to keep components pure by keeping changes out of the render phase
> 	- You should not mutate any of the inputs that your components use for rendering. That includes props, state, and context. To update the screen, [“set” state](https://react-ko.dev/learn/state-a-components-memory) instead of mutating preexisting objects.
> 	- Strive to express your component’s logic in the JSX you return. When you need to “change things”, you’ll usually want to do it in an event handler. As a last resort, you can `useEffect`.
> 	- Writing pure functions takes a bit of practice, but it unlocks the power of React’s paradigm.
> - How to use Strict Mode to find mistakes in your component

## 요약
### 리액트와 순수성
 - 리액트는 [[순수 함수 (Pure function)]] 개념을 중심으로 설계됨
	 - ![[순수 함수 (Pure function)]]
	 - 모든 컴포넌트를 순수 함수로 가정 (동일한 입력 하에서 동일한 JSX를 반환)
 - 따라서 리액트 컴포넌트는 순수성을 지키기 위해(keeping their purity) 다음과 같은 규칙을 가지게 됨
	 - 컴포넌트는 JSX만을 반환
	 - 컴포넌트가 렌더링 되기 이전에 존재했던 객체나 변수를 변경해서는 안 됨
		 - 단, [[#🔅 지역 변이 예제|지역 변이(렌더링 도중에 생성한 객체와 변수의 변경)]]는 관계 없음
	 - [[#🔅 불순한 컴포넌트 예제 들여다보기]], [[#🔅 다시 순수한 컴포넌트로 만들기 - Props 사용]]
 - 순수성을 지키게 되면 생기는 이점
	 - 순수 함수는 언제나 동일한 결과를 반환
		 - 동일한 입력에 대해 동일한 결과를 반환하므로 많은 사용자 요청 처리 가능
		 - 입력이 변경되지 않을 시 렌더링을 건너뛰어 성능 향상이 가능
		 - 순수성으로 인해 연산의 중단과 재시작에서 자유로우므로 렌더링을 언제든 재개할 수 있음
	- 자연스럽게 컴포넌트 간 의존성이 약해지고, 이에 따라 컴포넌트의 재사용이 용이해짐
		 - 컴포넌트의 독립성을 지키는 것이 핵심이므로 컴포넌트가 특정 순서로 렌더링 될 것을 기대하여 로직을 짜는 건 ❌
 - React가 rendering 과정에서 읽을 수 있는 input은 props, state, context의 세 가지 뿐으로, 이들은 언제나 read-only로 취급해야 한다.
	 - 리액트의 [[React/StrictMode]]를 사용하여 불순한 계산을 감지할 수 있음 (컴포넌트를 두 번 호출)
### 리액트에서의 side effect
- side effect : 화면 업데이트, 애니메이션 시작, 데이터 변경 등 '변경'과 관련된 것
	- 렌더링 도중에 일어나지 않으므로 불순해도 okay
		- 사용자가 어떤 동작을 했을때 실행되는 event handler처럼, 렌더링 중에는 실행되지 않음
	- 적당한 이벤트 핸들러를 찾을 수 없을 때는 useEffect를 사용
		- 단, 최소한도로!

#### 🔅 불순한 컴포넌트 예제 들여다보기
```javascript
let guest = 0;

function Cup() {
  // Bad: changing a preexisting variable!
  // 나쁨: 기존 변수를 변경합니다!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```
- 코드의 흐름을 살펴보면 다음과 같다.
	1. 전역 변수 guest가 0으로 초기화된다.
	2. Cup 컴포넌트는 호출될 때마다 guest 변수의 값을 1씩 증가시키고, 변경된 guest 변수값을 포함한 JSX를 반환한다.
		1. Cup 컴포넌트가 자신의 호출 이전에 선언된 전역변수 guest의 값을 변경하고 있음 → 순수성 원칙에 위배
	3. TeaSet 컴포넌트는 Cup 컴포넌트를 연속 세 번 호출한다.
- Cup 컴포넌트가 호출될 때마다 guest 변수의 값이 1씩 증가하고 있으므로 각 Cup 함수의 호출은 이전 호출보다 guest 값이 1 큰 JSX를 생성하게 됨
	- 첫 번째 호출 : guest = 1, `<h2>Tea cup for guest #1</h2>` 반환
	- 두 번째 호출 : guest = 2, `<h2>Tea cup for guest #2</h2>` 반환
	- 세 번째 호출 : guest = 3, `<h2>Tea cup for guest #3</h2>` 반환
- guest가 전역 변수이므로 모든 컴포넌트에서 접근 가능하고, Cup 컴포넌트에서는 guest의 값을 변경하고 있으므로, 어떤 컴포넌트가 guest를 읽는다고 했을 때 해당 컴포넌트의 렌더링 시점에 따라 guest의 값이 유동적임
	- A 컴포넌트가 guest를 읽는다고 할 때
		- TeaSet 컴포넌트 내에서 Cup을 호출하기 전에 A 컴포넌트가 guest를 읽음 : guest = 0
		- TeaSet 컴포넌트가 Cup을 호출한 이후 A 컴포넌트가 guest를 읽음 : guest = 3
- 이로 인해 컴포넌트 렌더링 결과를 예측하기 어려워짐 (unpredictable)
- 결과에서 guest가 각각 2, 4, 6으로 나왔던 것은 StrictMode의 동작으로 인함 (컴포넌트를 2회씩 호출)

#### 🔅 다시 순수한 컴포넌트로 만들기 - [[Props]] 사용
- 예제가 불순해지는 원인이 자신의 호출 이전에 선언된 전역변수 guest의 값을 변경하고 있는 점이므로, guest를 전역변수로 사용하지 않고 Cup 컴포넌트의 prop으로 사용하여 순수성을 지킬 수 있다.
```javascript
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

#### 🔅 지역 변이 예제
```javascript
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaGathering() {
  let cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```