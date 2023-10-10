>[!Learning Goals]
>- What “batching” is and how React uses it to process multiple state updates
>- How to apply several updates to the same state variable in a row

> [!빠른 gg 선언...]
> - **React waits until _all_ code in the event handlers has run before processing your state updates.** : 즉 제랜더링은 state setter가 호출된 이후에만 일어남.
> - state setter에 함수가 전달되었을 때 일어나는 일 중
> 	- 1. React queues this function to be processed after all the other code in the event handler has run. : 리액트의 state 업데이트 연산은 이벤트 핸들러의 모든 코드가 실행되고 난 뒤 일어나기 때문에 어찌 보면 당연한 말.
> 	- 2. During the next render, React goes through the queue and gives you the final updated state. : 그런데 '다음 렌더링 중'에 리액트가 큐로 들어가서 최종적으로 update된 state를 돌려준다고?! 😵‍💫❓
> - 그리고 2의 표현을 뒷받침하는 다른 표현들...
> 	- ==When you call `useState` during the next render, React goes through the queue.== The previous `number` state was `0`, so that’s what React passes to the first updater function as the `n` argument. Then React takes the return value of your previous updater function and passes it to the next updater as `n`, and so on. : 2와 근본적으로 동일한 말, 이후 예시를 들어 큐에서 일어나는 일을 더 자세히 설명하고 있음.
> 	- ==After the event handler completes, React will trigger a re-render. During the re-render, React will process the queue. Updater functions run during rendering,== so **updater functions must be [pure](https://react-ko.dev/learn/keeping-components-pure)** and only _return_ the result. : 심지어 재렌더링이 일어나는 중 큐에 예약된 updater function이 동작하기 때문에 updater function이 순수해야 한다는 설명까지 덧붙이는 중..
> - 왜 2가 이상하다고 생각하나요?
> 	- 애초에 상태 업데이트가 되었다고 평가되어야만 '다음 렌더링'이 일어남.
> 		- 즉, 이미 리액트는 내부 queue의 순회를 마쳐서 변경될 state 값 평가까지 끝냈다는 이야기. ('During the re-render, React will process the queue.' 표현이 이상해짐...)
> 		- 그래서 "Setting a state variable will queue another render"..라는 표현이 이해되지 않음... 
> 		- 이 문서의 표현을 따라, 리액트가 queue에 예약된 state 업데이트 작업을 평가하고 state가 업데이트되어야 할 것을 알아채어, 그것을 실제 업데이트 하기 전에 재렌더링 프로세스를 시작하고, 다음 렌더링의 useState를 만난 시점에서야 state를 실제로 업데이트하고 useState에서 반환한다고 보면, react의 queue processing 작업은 re-processing으로 표현되거나 혹은 이미 평가된 결과만 갖다 쓰게 되는 게 맞지 않나?
> 		- 렌더링은 앞 챕터까지의 서술을 보았을 때 '리액트가 컴포넌트 함수를 호출하는 것'임. 그런 의미에서 이벤트 핸들러 안에 존재하는 setter 호출은 렌더링에 포함되지 않는 것이 맞다고 생각...
> 		- 그럼 리액트 내부 queue는 다음 렌더링에서 useState를 만나기 전까지는 비워지지 않는 건가? ㅇㅅㅇ?
> 	- Updater function을 비롯해 setter에 대입된 값들이 렌더링 프로세스 이전에 동작하는 것으로 설명되는 것이 훨씬 깔끔한 것 같은데 왜 굳이 이렇게..? 이렇게 설명하지 않으면 안되는 이유가 있나? (설마 순수성...)
> - Rendering Components - State as a Snapshot - Queueing a Series of State Updates 챕터를 아우를 수 있는 만족스런 설명이 필요합니다😭

## 그나마...
- state setter의 내부 큐 예약과 렌더링 예약을 분리해서 생각하기
1. State의 변경이 렌더링을 trigger한다 (updating component's state = queues a new render - 'state as a snapshot')
	- setter 함수가 호출될 때마다 new render가 queue 된다고 볼 수도 있다. 이렇게 되면 setter 함수의 호출이 곧 component state update라는 말
	- 하지만 렌더링이 컴포넌트의 호출이라는 앞선 서술들이 반복되는 점이나, 컴포넌트가 실제로 호출되는 것은 모든 setter 함수가 실행되어 그 결과값으로 update될 State 값이 도출된 이후여서, setter 함수의 호출은 update state가 아니라고 보았음
2. 렌더링이 queue 되는 것은 queue에 등록된 모든 setter의 값/함수 평가가 끝나고 나서이다. 이 때 아직 실제 state update는 이뤄지지 않았다.
3. react는 queue된 리렌더링을 시작한다. 이 과정에서 useState를 만나면 다시 queue로 들어가 변경될 state값을 가지고 나와서, useState에 새로운 State값을 리턴한다. (이 시점에서 실제 state 업데이트가 이뤄진다).
4. 업데이트된 state를 기준으로 props, event handler, local variable 재연산되고 JSX에 해당 연산들이 반영되어 UI Snapshot이 리턴된다.
- state가 언제 update되는지만 좀 더 정확히 알려줬어도....;ㅅ;

## React는 state 업데이트를 일괄 처리한다
- 일괄처리 (batching): 리액트는 이벤트 핸들러 내의 모든 코드가 실행될 때까지 기다렸다가 state 업데이트를 처리한다.
	- 즉 핸들러 내의 코드가 모두 실행되기 전까지는 UI가 변화하지 않는다.
	- 리렌더링을 줄이고 동작을 빠르게 해주는 효과
	- State의 변화가 모두 반영되지 않거나 Interaction 관련 처리가 끝나지 않은 '중간 단계의 렌더링' (불완전한 렌더링)을 처리할 필요가 없음
- 단, 이벤트 자체를 일괄처리하진 않는다.
	- 클릭이 여럿 붙어있거나 같은 클릭을 여러번 하더라도, 한 클릭 당의 결과를 기준으로 독립적으로 동작 (ex: 클릭시 스위치 On / off)

## State의 replace와 update
- `setNumber(number + 1)` : number + 1은 '값'으로 평가되어 'replace'로 표현 - setter 함수가 호출되는 즉시 평가된다.
- `setNumber(prevNumber => prevNumber + 1)` : 업데이터 함수의 경우 함수가 실행될 때 연산되어 평가되고, 이전 상태를 사용하므로 update로 표현

## Naming Conventions
- 업데이터 함수 인수의 경우 state 변수의 첫 글자로 지정
	- `enabled → setEnabled(e => !e)`
	- `lastName → setLastName(ln => ...)`
- 좀 더 자세하게 써주고 싶은 경우 전체 state 이름을 반복하여 써주거나 prev를 접두어로 붙여 나타내주기