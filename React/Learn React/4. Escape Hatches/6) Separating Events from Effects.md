>[!Learning Goals]
>- How to choose between an event handler and an Effect
>- Why Effects are reactive, and event handlers are not
>- What to do when you want a part of your Effect's code to not be reactive
>- What Effect Events are, and how to extract them from your Effects
>- How to read the latest props and state from Effects using Effect Events

- Event handler → 동일한 interaction을 직접 수행해야지만 실행 (manual)
- Effect → prop, state variable 등 reactive한 값들이 이전 렌더링과 다를 때 자동으로 sync한다
- ❓ 그렇다면 Effect 내부에서 사용자에 의한 Interaction이 생기고, 그에 의해 reactive한 값들이 변화하는 경우, 해당 값들의 변화에도 불구하고 Effect를 다시 실행시키고 싶지 않다면 어떻게 해야하나?
	- (아직 API 개발중이지만) Effect Event를 사용해라!

## 1. Event handler와 Effect의 구분

>[!(ex) Chat Room]
> 1. Room 선택 : 컴포넌트는 언제나 선택한 room에 연결되어야 함
> 2. 메시지 전송 : '전송' 버튼을 클릭했을 때 채팅에 메시지를 전송해야 함

- 🔥 **"consider 'why' the code needs to run"**
- Event handler : 특정한 Interaction에 대한 반응으로 실행 
	- manually, '~로 인하여 촉발'
	- 메시지 전송 → 사용자가 '버튼을 누를 때에만' 전송되어야 함
- Effect : 동기화가 필요할 때마다 실행 (의존성으로 설정해놓은 state variable, prop 등이 과거 렌더와 달랐을 때)
	- automatically, 자동으로 sync 유지, '~의 존재로 인해 당연하게'
	- Room에 연결 → 사용자가 '왜' 해당 방을 선택하였는지는 중요하지 않음, 사용자가 어떠한 Interaction을 취하지 않았지만 chat room 컴포넌트의 기본 기능을 수행하기 위해선 <b><u>연결 및 재연결이 자동으로</u></b> 되어야 함
		- ![[6) Separating Events from Effects#🔅 Effect에서의 동작 및 중지 설정]]
## 2. Reactive values & Reactive logic
### Reactive values (반응형 값)
- data flow를 렌더링하는 데 참여하는 요소들
	- props
	- state
	- variables inside a component's body
- 🚩 리렌더링에 의해 변할 수 있는 값 ("Reactive values ... can change due to a re-render")
	- <span style="color:skyblue;">'반응형 값이 리렌더링에 의해 변할 수 있다'는 것은 UI에 초점을 맞춰서 이야기하려다 보니 그렇게 된 걸까? 리액트의 작동 관점에서 보았을 때는 반응형 값의 변화가 리렌더링의 결과가 아니라 원인으로 생각하는 게 맞는 것 같은데....</span>
- ![[6) Separating Events from Effects#🔅 반응형 값 vs 비 반응형 값]]
### Reactive Logic (반응형 로직)
- 데이터의 변화에 자동으로 반응하여 실행되는 로직
	- 이벤트 핸들러의 로직 != reactive
		- 사용자가 동일한 상호작용을 수행하지 않으면 다시 실행되지 않는다.
		- 반응형 값이 변화한다 할지라도 로직은 그에 영향을 받지 않는다. 따라서 반응형 값의 변화와 상관 없이 값을 읽을 수 있다.
		- ex) 메시지 내용이 변경되었다고 해서 반드시 자동으로 메시지가 다시 보내져야 하는 것은 아님
	- Effect 내의 로직 = reactive
		- Effect의 로직은 반응형 값의 변화에 따라 자동으로 다시 실행된다. (단, effect 내의 반응형 값을 모두 의존성으로 지정해줬을 경우)
		- 의존성으로 지정한 반응형 값이 이전 렌더링에 비해 변화했다면, 그 변화한 값을 기반으로 effect 로직이 다시 실행된다.
		- ex) roomId가 변경되었다 = 자동으로, 변경된 다른 room에 연결하고 싶다
## 3. useEffectEvent의 필요와 사용
- 🔥 <b><u>reactive logic과 non-reactive logic을 함께 사용하고 싶을 때 useEffectEvent를 사용하여 Effect Event를 분리</u></b>
	- ex) 사용자가 채팅을 연결했을 때, Props에서 theme을 읽어 그에 맞게 알림을 표시
		- theme = a prop = reactive value → Effect의 의존성(dependency)이 되어야 함
		- ![[6) Separating Events from Effects#🔅 useEffectEvent를 사용해야하는 경우 - 분리 전]]
			- theme이 dependency로 설정되어있으므로 theme이 바뀔 때 마다 useEffect 내부의 로직(채팅 연결, 알림 표시)이 항상 함께 재실행됨
			- theme은 reactive value이지만, 그럼에도 불구하고 의존성에서는 제외되어야 함
- ### Effect Event
	- Effect 로직의 일부이지만 event handler처럼 동작하는 것
		- effect 내의 effect event(사용자에 의해 trigger되는, event의 성질을 가진 반응형 값들)는 `useEffectEvent(...)`를 사용하여 useEffect에서 분리하고 의존성에서 제거
		- ![[6) Separating Events from Effects#🔅 useEffectEvent를 사용해야하는 경우 - 분리 후]]
		- ![[6) Separating Events from Effects#🔅 useEffectEvent에서 최근 props와 state 읽기]]
			- closure의 특성을 이용해 Hook으로 감싼 함수를 매개변수 없이 사용하는 대신 명시화하는 이유
				- 사용자 관점에서 어떤 요소가 '이벤트'인지를 나타내는 표식
				- useEffect 내에서 함수 사용시 매개변수가 존재하지 않아 의존성 배열에서 해당 반응형 값을 실수로 삭제하게 될 가능성을 차단
				- ⭐️ useEffect 내에 비동기 로직이 있는 경우
					- ![[6) Separating Events from Effects#🔅 Effect event & Effect - 비동기 로직이 있는 경우]]
			- 
- useEffectEvent가 안정화되면 의존성 linter를 억제하지 말 것 - 예상치 못한 동작과 에러가 발생할 수 있음!
	- ![[6) Separating Events from Effects#🔅 Linter 억제로 인해 예상치 못한 동작이 발생하는 경우]]
## 4. Effect event의 제한사항
- Effect 내부에서만 <u><b>호출</b></u>할 수 있다.
- 다른 component나 hook에 전달해선 안된다!
	- 전달하는 대신 늘 사용할 Effect와 동일한 컴포넌트 안에 선언하고 사용하기!
## ------ Example List ------
#### 🔅 Effect에서의 동작 및 중지 설정
```javascript
...
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  
		connection.connect();  
		return () => {  
			connection.disconnect(); ⭐️
		};  
	}, [roomId]);
...
```

#### 🔅 반응형 값 vs 비 반응형 값
```javascript
const serverUrl = 'https://localhost:1234';  // 비-반응형 (렌더링이 변화하더라도 변하지 않음)

function ChatRoom({ 👉🏻 roomId }) {  
	const [👉🏻 message, setMessage] = useState(''); 
	...
}
// ...
```
#### 🔅 useEffectEvent를 사용해야하는 경우 - 분리 전
```javascript
function chatRoom({ roomId, theme }) {
	useEffect(() => {
		const connection = createConnection(serverUrl, roomId)
		/* connect 되면, theme을 읽어서 notification을 보여준다 */
		connection.on("connected", () => {
			showNotification("Connected", theme)
		})
		connection.connect()
		return () => {
			connection.disconnect()
		}
	}, [roomId, theme])
}
```
#### 🔅 useEffectEvent를 사용해야하는 경우 - 분리 후
```javascript
function ChatRoom({ roomId, theme }) {  
	/* theme이 관여하는 로직을 useEffectEvent로 감싸, Effect event로 만들고 useEffect와 분리 */
	const onConnected = useEffectEvent(() => {
		showNotification('Connected!', theme);
		// theme의 변경은 useEffect 안에서 사용자의 interaction에 의해 trigger 될 수 있음 : event의 성격
	});  
	
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  
		connection.on('connected', () => {  
			onConnected();  
		});  
	
		connection.connect();  
		return () => connection.disconnect();  

	}, [roomId]); // 의존성 목록에서 theme 제거
...
}
```
#### 🔅 useEffectEvent에서 최근 props와 state 읽기
>[!예시 조건]
> - 페이지 방문을 기록하기 위한 함수 logVisit
> 	- logVisit은 방문한 url과 해당 url 방문 시점의 장바구니 물건 개수를 매개변수로 받아 기록
> - url은 Page 컴포넌트에서 prop으로 전달받음
```javascript
function Page({ url }) {  
	const { items } = useContext(ShoppingCartContext);  
	const numberOfItems = items.length;  
	
	const onVisit = useEffectEvent(visitedUrl => {  
		logVisit(visitedUrl, numberOfItems);  // logVisit에서 url과 numberOfItems를 매개변수로 사용하지만
	});  
	
	useEffect(() => {  
		onVisit(url);  // effect 내 로직의 onVisit는 매개변수로 url만을 요구하고, 의존성에 영향을 주는 것도 그 url뿐임
	}, [url]);
	// ... 
}
```

#### 🔅 Effect event & Effect - 비동기 로직이 있는 경우
```javascript
...
	const onVisit = useEffectEvent(visitedUrl => {  
		logVisit(visitedUrl, numberOfItems);  
	});  
	
	useEffect(() => {  
		setTimeout(() => {  
			onVisit(url);  
		}, 5000); // 방문 기록을 지연시킴: url이 변경되었더라도, onVisit 내부의 visitedUrl은 호출된 시점의 값으로 남아있음
	}, [url]);
...
```
#### 🔅 Linter 억제로 인해 예상치 못한 동작이 발생하는 경우
```javascript
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  function handleMove(e) {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  }

  useEffect(() => {
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, []);
  /* 본래 handleMove 함수가 의존성에 포함되어야 하지만,
   * 의존성이 존재하지 않는다고 Linter를 속였기 때문에 handleMove는 언제나 첫 렌더링시 전달된 초기상태로만 동작한다.
   * 이에 따라 canMove는 언제나 true가 되는 문제가 생긴다.
   */

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        The dot is allowed to move
      </label>
      <hr />
      <div style={{
        position: 'absolute',
        backgroundColor: 'pink',
        borderRadius: '50%',
        opacity: 0.6,
        transform: `translate(${position.x}px, ${position.y}px)`,
        pointerEvents: 'none',
        left: -20,
        top: -20,
        width: 40,
        height: 40,
      }} />
    </>
  );
}
```