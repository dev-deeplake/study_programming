>[!Learning Goals]
>- How to fix infinite Effect dependency loops
>- What to do when you want to remove a dependency
>- How to read a value from your Effect without "reacting" to it
>- How and why to avoid object and function dependencies
>- Why suppressing the dependency linter is dangerous, and what to do instead

- 불필요한 의존성을 검토 및 제거하기 위한 가이드!
## 1. Dependencies should match the code
- 의존성은 우리가 '선택'할 수 있는 것이 아니라 코드에 의해 결정됨
	- Effect 안에서 사용된 모든 반응형 값이 강제
		- ![[6) Separating Events from Effects#Reactive values (반응형 값)]]
- 따라서 Effect의 의존성 배열을 변경하고 싶다면 코드를 변경해라!
	- 가장 확실한 방법 중 하나는 해당 값을 비-반응형으로 만드는 것!
		- ![[7) Removing Effect Dependencies#🔅 반응형 값을 비-반응형으로 만들기]]
	- Linter를 억제하는 방법은 추천하지 않음
		- ![[6) Separating Events from Effects#🔅 Linter 억제로 인해 예상치 못한 동작이 발생하는 경우]]
		- ![[7) Removing Effect Dependencies#🔅 Linter 억제로 인해 예상치 못한 동작이 발생하는 경우 (2)]]
			- 예시의 문제들이 여러 컴포넌트에 분산되어있다면? → 🫠
			- 따라서 lint error는 컴파일 오류와 동일하게 보고 억제하지 말 것!
	- 의존성 변경을 위한 코드 수정은 보통 다음과 같은 패턴으로 진행
		- Effect의 코드 또는 반응형 값 선언 방식 변경(위 예시처럼!)
		- 변경한 코드에 맞게 의존성을 조정
		- 의존성 목록이 마음에 들지 않으면 다시 첫 번째 단계로 돌아가서 코드를 변경
## 2. Removing unnecessary dependencies

>[!불필요한 의존성 제거를 위해 해야 할 질문들]
>- 이 코드가 정말로 Effect여야 하는가? (event handler로 처리되어야 할 것은 아닌가?)
>- Effect가 서로 관련 없는 여러가지 작업을 처리하고 있지 않은가?
>- 다음 state를 계산하기 위해 state를 읽고 있지는 않은가?
>- Effect의 로직 안에 non-reactive 로직이 포함되어있지는 않은가?
>- 객체 또는 함수를 의존성으로 지정하고 있지는 않은가?

- 사용자 Interaction의 응답으로 인해 실행되는 코드 및 로직인 경우 Effect가 아닌 이벤트 핸들러에 포함시키기
- Effect가 서로 관련 없는 여러가지 작업을 하고 있을 경우 Effect를 분리하여 작성하기
	- 각 Effect가 독립적인 동기화 프로세스를 관장하도록 분리하기
		- ![[7) Removing Effect Dependencies#🔅 Effect가 서로 관련 없는 여러 작업을 하고 있는 경우 - 수정 전]]
		- ![[7) Removing Effect Dependencies#🔅 Effect가 서로 관련 없는 여러 작업을 하고 있는 경우 - 수정 후]]
	- 중복이 걱정될 시 반복되는 로직을 custom hook으로 추출하기
		- ![[7) Removing Effect Dependencies#🔅 custom hook으로 고치면?]]
- 다음 state를 계산하기 위해 어떤 state를 읽고 있다면, state 대신 업데이터 함수를 전달하기
	- ![[7) Removing Effect Dependencies#🔅 다음 state를 계산하기 위해 어떤 state를 읽고 있는 경우 - 수정 전]]
		- message를 수신할 때마다 useEffect로 인해 업데이트된 메시지 배열을 만들게 됨.
		- messages state가 업데이트되었으므로, 다시 useEffect가 일어남 (불필요한 connection이 한번 더...)
	- ![[7) Removing Effect Dependencies#🔅 다음 state를 계산하기 위해 어떤 state를 읽고 있는 경우 - 수정 후]]
		- update 로직을 담은 함수를 전달하여 setter 안에서 message state variable을 더이상 읽지 않도록 변경하고, 의존성에서도 제거
			- React에서는 updater 함수를 큐에 넣고
			- 다음 렌더링시 이 updater 함수가 전달될 당시의 messages (msgs)를 제공
			- updater 함수는 해당 msgs를 기반으로 새 상태를 계산하여 반환
- Effect 안에 non-reactive 로직이 포함되어있을 경우 useEffectEvent를 사용하여 effect 내의 이벤트를 명시 및 분리
	- case 1 ) Effect의 dependency로 사용자의 interaction으로 인해 trigger 될 수 있는 state 또는 prop이 지정되어있는 경우 (해당 reactive value의 변경은 Effect의 주 로직과 독립적)
		- ![[6) Separating Events from Effects#🔅 useEffectEvent를 사용해야하는 경우 - 분리 전]]
		- ![[6) Separating Events from Effects#🔅 useEffectEvent를 사용해야하는 경우 - 분리 후]]
		- 
	- case 2 ) 컴포넌트가 이벤트 핸들러를 prop으로 받는 경우
		- ![[7) Removing Effect Dependencies#🔅 Effect 안에 non-reactive 로직이 포함되어있을 경우 이벤트 handler를 prop으로 받을 때 - 수정 전]]
		- ![[7) Removing Effect Dependencies#🔅 Effect 안에 non-reactive 로직이 포함되어있을 경우 이벤트 handler를 prop으로 받을 때 - 수정 후]]
		- 
- 컴포넌트 본문에서 생성된 객체 또는 함수를 의존성으로 지정하고 있는 경우 다음의 해결 방법 중 선택
	- Effect와 동일한 컴포넌트의 본문에서 생성한 객체 및 함수는, 해당 컴포넌트가 재렌더 될 때마다 새롭게 생성되므로 Effect를 한 번 더 트리거함
		- ![[7) Removing Effect Dependencies#🔅 컴포넌트 본문에서 생성된 객체 또는 함수를 의존성으로 지정한 경우]]
	- ### 1. (정적-static) 객체와 함수를 컴포넌트 외부로 이동시키기
		- 객체와 함수가 Props 및 state에 의존하지 않을 경우 고려해 볼 수 있음
			- ![[7) Removing Effect Dependencies#🔅 객체와 함수를 컴포넌트 외부로 이동시키기]]
	- ### 2. Effect 내로 (동적-dynamic)객체 및 함수 이동
		- 객체 및 함수가 props나 state에 의존하는 경우 컴포넌트 외부로 끌어내는 대신 Effect 내부로 이동시키는 것을 고려해볼 수 있음
			- ![[7) Removing Effect Dependencies#🔅 Effect 내로 객체 및 함수 이동]]
	- ### 3. 객체에서 원시값(primitive value) 읽기
		- props로 객체를 받고, 넘겨받을 객체를 부모가 렌더링 중에 생성하는 경우
			- ![[7) Removing Effect Dependencies#🔅 객체에서 원시값 읽기 - 수정 전]]
			- ![[7) Removing Effect Dependencies#🔅 객체에서 원시값 읽기 - 수정 후]]
	- ### 4. 함수에서 원시값 계산
		- props로 함수를 받고, 넘겨받을 함수를 부모가 렌더링 중에 생성하는 경우
			- ![[7) Removing Effect Dependencies#🔅 함수에서 원시값 계산 - 수정 전]]
			- ![[7) Removing Effect Dependencies#🔅 함수에서 원시값 계산 - 수정 후]]
			- 

## ------ Example List ------
#### 🔅 반응형 값을 비-반응형으로 만들기
```javascript
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'music'; // ⭐️ prop으로 받던 roomId를 컴포넌트 밖에서 선언하고 고정된 변수로 만듦

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
}
```
#### 🔅 Linter 억제로 인해 예상치 못한 동작이 발생하는 경우 (2)
```javascript
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  function onTick() {
	setCount(count + increment);
  }

  useEffect(() => {
    const id = setInterval(onTick, 1000);
    return () => clearInterval(id);
  }, []); // ⭐️ 본래는 onTick이 의존성으로 지정되어야 하지만, 의존성을 지정하지 않았으므로 첫 렌더링에서만 해당 Effect 코드가 실행되고 초기 렌더링의 onTick 함수가 계속 사용됨. 결과적으로 setCount(0 + 1)이 계속해서 들어가므로, counter에는 늘 1만 표시됨

  return (
    <>
      <h1>
        Counter: {count}
        <button onClick={() => setCount(0)}>Reset</button>
      </h1>
      <hr />
      <p>
        Every second, increment by:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}

```
#### 🔅 Effect가 서로 관련 없는 여러 작업을 하고 있는 경우 - 수정 전
```javascript
function ShippingForm({ country }) {
	const [cities, setCities] = useState(null)
	const [city, setCity] = useState(null)
	const [areas, setAreas] = useState(null)

	useEffect(() => {
		let ignore = false;
		fetch("/api/cities?country=${country}") // 선택한 country에 따라 도시 데이터를 가져옴
			.then(response => response.json())
			.then(json => {
				if (!ignore) {
					setCities(json)
				}
			})
		if (city) {
			fetch("/api/areas?city=${city}") // 선택한 city에 따라 지역 데이터를 가져옴
				.then(response => response.json)
				.then(json => {
					if (!ignore) {
						setAreas(json)
					}
				})
		}
	}
	return () => {
		ignore = true
	}
	}, [country, city]) // 의존성이 country와 city이므로, country나 city 중 어떤 것이 바뀌더라도 위의 Effect 코드 모두가 재실행
}
```
#### 🔅 Effect가 서로 관련 없는 여러 작업을 하고 있는 경우 - 수정 후
```javascript
function ShippingForm({ country }) {
	const [cities, setCities] = useState(null)
	useEffect(() => {
		let ignore = false
		fetch(`/api/cities?country=${country}`) // cities state variable을 이용해 city를 세팅하는 로직
			.then(response => response.json)
			.then(json => {
				if (!ignore) {
					setCities(json)
				}
			})
		return () => {
			ignore = true
		}
	}, [country])

	const [city, setCity] = useState(null)
	const [areas, setAreas] = useState(null)
	useEffect(() => {
		if (city) {
			let ignore = false
			fetch(`/api/areas?city=${city}`) // city state variable을 이용해 area를 세팅하는 로직
				.then(response => response.json)
				.then(json => {
					if (!ignore) {
						setArea(json)
					}
				})
			return () => {
				ignore = true
			}
		}
	}, [city])

}
```
#### 🔅 custom hook으로 고치면?
```javascript
import { useState, useEffect } from "react"

function useFetchData(url, dependency) {
	const [data, setData] = useState(null)

	useEffect(() => {
		let ignore = false
		fetch(url)
			.then(res => res.json())
			.then(json => {
				if (!ignore) {
					setData(json)
				}
			})
		return () => {
			ignore = true
		}
	}, [dependency])

	return data
}


function ShippingForm({ country }) {
	const cities = useFetchData(`/api/cities?country=${country}`, country)

	const [city, __] = useState(null)
	const areas = useFetchData(`/api/arear?city=${city}`, city)

	....
}
```
#### 🔅 다음 state를 계산하기 위해 어떤 state를 읽고 있는 경우 - 수정 전
```javascript
/* 새 메시지가 도착할 때마다 새로 생성된 배열로 'message' state를 업데이트하는 Effect
 * message state variable을 이용하여 기존 메시지 배열을 생성한 뒤 마지막에 새로 받아온 메시지 추가하는 로직
 */
 function chatRoom({ roomId }) {
	 const [messages, setMessages] = useState([])
	 useEffect(() => {
		 const connection = createConnection()
		 connection.connect()
		 connection.on("message", (receivedMessage) => {
			 setMessages([...messages, receivedMessage])
		 })
		 return () => connection.disconnect()
	 }, [roomId, messages])
 }
```
#### 🔅 다음 state를 계산하기 위해 어떤 state를 읽고 있는 경우 - 수정 후
```javascript
 function chatRoom({ roomId }) {
	 const [messages, setMessages] = useState([])
	 useEffect(() => {
		 const connection = createConnection()
		 connection.connect()
		 connection.on("message", (receivedMessage) => {
			 setMessages(msgs => [...msgs, receivedMessage]) // state가 아닌, update 로직을 담은 update 함수를 전달
		 })
		 return () => connection.disconnect()
	 }, [roomId])
 }
```
#### 🔅 Effect 안에 non-reactive 로직이 포함되어있을 경우: 이벤트 handler를 prop으로 받을 때 - 수정 전
```javascript
// 컴포넌트 사용시
<ChatRoom
	roomId={roomId}
	onRecieveMessage={receivedMessage => { ... }} // 렌더링 할 때마다 prop으로 매번 다른 익명함수를 전달
/>

function chatRoom({ roomId, onReceiveMessage }) {
	const [messages, setMessages] = useState([])
	useEffect(() => {
		const connection = createConnection()
		connection.connect()
		connection.on("message", (receivedMessage) => {
			onReceiveMessage(receivedMessage)
		})
		connection.connect()
		return () => {
			connection.disconnect()
		}
	}, [roomId, onReceiveMessage])
}
```
#### 🔅 Effect 안에 non-reactive 로직이 포함되어있을 경우: 이벤트 handler를 prop으로 받을 때 - 수정 후
```javascript
function chatRoom({ roomId, onReceiveMessage }) {
	const [messages, setMessages] = useState([])
	
	const onMessage = useEffectEvent(receivedMessage => { // useEffectEvent hook을 사용한 이벤트 핸들러 분리
		onReceiveMessage(receivedMessage)
	})
	
	useEffect(() => {
		const connection = createConnection()
		connection.connect()
		connection.on("message", (receivedMessage) => {
			onMessage(receivedMessage)
		})
		connection.connect()
		return () => {
			connection.disconnect()
		}
	}, [roomId])
}
```
#### 🔅 컴포넌트 본문에서 생성된 객체 또는 함수를 의존성으로 지정한 경우
```javascript
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = { // 👈🏻
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 👈🏻

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}
```
#### 🔅 객체와 함수를 컴포넌트 외부로 이동시키기
```javascript
const options = {  // 👈🏻
	serverUrl: 'https://localhost:1234',  
	roomId: 'music'  
};  

function ChatRoom() {  
	const [message, setMessage] = useState('');  

	useEffect(() => {  
		const connection = createConnection(options);  
		connection.connect();  
		return () => connection.disconnect();  
	}, []); // option이 컴포넌트 외부에 있어 변경되지 않는 값이므로 의존성에서 제외
// ...
```
#### 🔅 Effect 내로 객체 및 함수 이동
```javascript
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = { // 👈🏻 Effect 내에서 선언되었으므로 의존성 대상이 아님
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}
...
```
#### 🔅 객체에서 원시값 읽기 - 수정 전
```javascript
// 실제 사용
<ChatRoom
	roomId={roomId}
	options={{ //👈🏻 의존성으로 지정한 객체가 부모 컴포넌트가 렌더링중에 생성되므로 불필요한 Effect 실행을 촉발할 수 있음
		serverUrl: serverUrl,
		roomId: roomId
	}}

// 컴포넌트 선언
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);
  // ...

```
#### 🔅 객체에서 원시값 읽기 - 수정 후
```javascript
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');
  const { roomId, serverUrl } = options // 👈🏻 prop으로 받은 option의 값을 primitive value로 변수에 할당
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // 👈🏻 원시값을 의존성으로 넘겨주므로 useEffect의 불필요한 작동을 걱정하지 않아도 됨!
  // ...
```
#### 🔅 함수에서 원시값 계산 - 수정 전
```javascript
// 실제 사용
<ChatRoom
	roomId={roomId}
	getOptions={() => {
		return {
			serverUrl: serverUrl,
			roomId: roomId
		}
	}}
/>
```
#### 🔅 함수에서 원시값 계산 - 수정 후
```javascript
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');
  const { roomId, serverUrl } = getOptions() // 👈🏻 prop으로 받은 getOptions를 호출하여 그 결과를 각 변수에 원시값으로 할당
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // 👈🏻 원시값을 의존성으로 넘겨주므로 useEffect의 불필요한 작동을 걱정하지 않아도 됨!
  // ...
```