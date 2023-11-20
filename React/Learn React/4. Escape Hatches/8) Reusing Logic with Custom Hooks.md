>[!Learning Goals]
>- What custom Hooks are, and how to write your own
>- How to reuse logic between components
>- How to name and structure your custom Hooks
>- When and why to extract custom Hooks

- Built-in Hook 외에 내가 필요한 목적에 맞게 커스텀 훅을 제작해 사용할 수 있음
## 1. Custom Hook?
- 예를 들면.....
	- ![[8) Reusing Logic with Custom Hooks#🔅 공통되는 로직을 가진 두 컴포넌트]]
	- ![[8) Reusing Logic with Custom Hooks#🔅 custom hook을 사용해 로직을 공유하기]]
- 🔥 컴포넌트의 UI와 관계 없이, 컴포넌트 간 로직을 공유할 수 있도록 해줌
	- state 자체가 아닌, stateful logic을 공유 (당연! hook이 호출 될 때마다 새로운 state variable이 생성됨)
	- 컴포넌트 사이에 state를 공유하고자 한다면 state를 공통 부모로 끌어올려 Prop으로 전달해주기
- React App >> components >> Hooks
	- 컴포넌트가 리액트 앱을 구성하는 블록이라면, 훅은 컴포넌트의 로직을 구성하는 블록
- naming convention : `use + (Something, in camel case)`
	- state, effect를 비롯한 React의 요소들이 들어있음을 알려주는 표지가 되도록!
		- Hook을 호출하지 않는 함수는 hook이 될 필요가 없음
		- React용으로 linter를 포매팅할 경우 hook의 naming convention을 따르지 않은 함수 안에서는 hook을 호출할 수 없도록 되어있음
	- `useState` / `useOnlineStatus`
- 컴포넌트 내부의 로직을 명령형이 아닌 선언형으로 변경하는 효과
	- 실제 작동은 Hook 내부의 로직으로 숨겨짐
	- 이름을 잘 지어야 이런 의도를 달성할 수 있다!
		- 코드를 작성하지 않은 사람이라도 이 hook이 어떤 역할을 하고, 무엇을 요소로 받고, 어떤 것을 반환하는지 파악할 수 있는 이름 짓기
			- `useData(url)` / `useImpressionLog(eventName, extraData)` / `useChatRoom(options)`
		- 외부 시스템과 동기화되는 hook의 경우 좀 더 전문적인 용어가 사용될 수 있지만 최소한 시스템에 익숙한 사람이 이해할 수 있는 이름으로!
			- `useMediaQuery(query)` / `useSocket(url)` / `useIntersectionObserver(ref, options)`
## 2. 기본적인 사용 방법
### 1) Passing reactive values between Hooks
- hook은 언제나 컴포넌트의 재렌더에 따라 (마치 컴포넌트 body의 일부인 것 처럼) 새로이 호출되기 때문에, 해당 컴포넌트의 최신 reactive value들을 받는다.
- ![[8) Reusing Logic with Custom Hooks#🔅 컴포넌트의 reactive values와 hook의 연결 예시]]
### 2) Passing event handlers to custom hooks
- custom hook 내부에서 핸들링하고 싶은 이벤트가 있다면, 이벤트 핸들러를 prop으로 받아와 핸들링할 수 있음
	- 이벤트 핸들링 로직을 분리하므로 이벤트를 좀 더 동적으로 핸들링할 수 있음
	- 이벤트 핸들러가 custom hook 내부 Effect의 의존성이 될 경우 불필요한 리렌더가 일어나므로, `useEffectEvent`훅을 이용해 문제 해결하기
	- ![[8) Reusing Logic with Custom Hooks#🔅 컴포넌트의 reactive values와 hook의 연결 예시]]
	- ![[8) Reusing Logic with Custom Hooks#🔅 custom hook에 event handler 전달하기]]
## 3. 언제 사용해야 할까?
- effect를 사용하고 있다면, 해당 effect를 custom hook으로 추출하는 것에 대해 고민해보기
	- effect = escape hatch, 따라서 적절하게 custom hook으로 만들면 코드가 좀 더 선언적이 되어 코드의 의도 및 데이터 flow를 명확하게 전달 가능
	- effect의 처리가 custom hook 안으로 숨겨지므로, effect가 컴포넌트 내에 직접 작성되었을 때에 비해 의존성 관련 실수가 줄어듦
	- ![[7) Removing Effect Dependencies#🔅 Effect가 서로 관련 없는 여러 작업을 하고 있는 경우 - 수정 후]]
	- ![[7) Removing Effect Dependencies#🔅 custom hook으로 고치면?]]
	- React의 궁극적인 목표는 필수적인 외부 연결에서 필요한 API를 built-in hook으로 제공하는 것이므로, 현재 escape hatch 케이스에 해당하는 로직 중 추후 built-in hook으로 대체될 수 있는 것들이 있음. 이런 경우 custom hook으로 분리해 사용하면 추후 API 가 제공되었을 때 더 편하게 마이그레이션 할 수 있음
		- `useSyncExternalStore`
- 단순히 useEffect의 wrapper로 기능하거나 useEffect를 이용해 lifecycle hook 역할을 하는 custom hook은 생성하지 않기
	- ![[8) Reusing Logic with Custom Hooks#🔅 잘못된 custom hook 사용 lifecycle hook]]
	- react는 함수형 컴포넌트와 hook을 통해 side effect를 효율적으로 관리할 수 있도록 최적화되고 있음
		- useEffect를 통해 lifecycle hook을 만드는 것은 이전 세대의 클래스 컴포넌트 훅(`componentDidMount`, `componentDidUpdate`, `componentWillUnmount` ... )들을 모방하는 것과 유사, 이미 함수형 컴포넌트와 hook에 맞추어진 최적화 수행을 방해할 우려
	- useEffect의 wrapper 용도로 custom hook을 만들게 되면 linter의 경고 대상에서 배제됨
## 4. 어디까지를 custom hook화 할 것인가?
- hook으로 고도화하는 범위는 프로그래머의 재량에 달려있다.
	- Effect를 사용하지 않고 일반 함수에서 작업을 처리하거나
	- Effect를 사용하되 custom hook으로 분리하지 않거나
	- custom hook 하나로 분리하거나
	- custom hook 내에서 또 다른 custom hook을 사용하는 식으로 다단계 고도화를 할 수도...
- Effect를 사용한다는 것은 결국 해당 로직을 '외부 시스템'화 한다는 것 → 사용하는 케이스를 잘 고려해 선택하기
## ------ Example List ------
#### 🔅 공통되는 로직을 가진 두 컴포넌트
```javascript
import { useState, useEffect } from 'react';
/* StatusBar : 네트워크의 on / offline 여부에 따라 현재 상태를 나타내주는 status bar 컴포넌트
 * SaveButton : 현 상태를 저장하고 로그를 찍음, 네트워크의 On / offline 여부에 따라 UI 및 활성화 상태 변경
 * state variable 및 useEffect 내부의 로직이 반복되고 있음:네트워크의 On/offline 여부를 확인하고 isOnline state를 조작
 */

export function StatusBar() {
 // 🔥 👇🏻
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => { 
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

export function SaveButton() {
   // 🔥 👇🏻
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => { 👈🏻 🔥
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}
```
#### 🔅 custom hook을 사용해 로직을 공유하기
```javascript
// 공통된 로직을 뽑아 만든 custom hook: useOnlineStatus
function useOnlineStatus () { // 👈🏻 🔥 자세한 로직은 hook 내부로 숨겨짐
  const [isOnline, setIsOnline] = useState(true)
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true)
    }
    function handleOffline() {
      setIsOnline(false)
    }
    window.addEventListener('online', handleOnline)
    window.addEventListener('offline', handleOffline)
    return () => {
      window.removeEventListener('online', handleOnline)
      window.removeEventListener('offline', handleOffline)
    };
  }, []);
  return isOnline
}

function StatusBar() {
  const isOnline = useOnlineStatus(); // 👈🏻 🔥 선언형으로 변화
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus(); // 👈🏻 🔥 선언형으로 변화

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}

```

#### 🔅 컴포넌트의 reactive values와 hook의 연결 예시
```javascript
import { useState, useEffect } from 'react'
import { createConnection } from './chat.js'
import { showNotification } from './notifications

/* custom hook 'useChatRoom'
 * serverUrl과 roomId를 받아 해당 요소 중 어떤 것이라도 변경되었으면 해당 정보로 다시 연결해주고 이전 연결을 끊음
 * 새로운 메시지를 수신했을 때 알림을 띄워줌
 */
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options)
    connection.connect()
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg) // 👈🏻 이벤트 핸들링
    })
    return () => connection.disconnect()
  }, [roomId, serverUrl])
}

// 실제로 useChatRoom Hook이 사용되고 있는 ChatRoom 컴포넌트
export function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234') // serverUrl은 state로 관리
  
  // 렌더시 동작할 연결 로직을 useChatRoom custom hook을 통해 사용, roomId prop과 serverUrl state를 hook에 연결
  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        Server URL:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```

#### 🔅 custom hook에 event handler 전달하기
```javascript
import { useEffect, useState } from 'react'
import { experimental_useEffectEvent as useEffectEvent } from 'react'
import { createConnection } from './chat.js'
import { showNotification } from './notifications

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) { // 👈🏻 🔥 이벤트 핸들러를 hook의 매개변수로 전달받기
  const onMessage = useEffectEvent(onReceiveMessage); // 이벤트 핸들러는 effect 내에서 onMessage의 이름으로 사용됨

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options)
    connection.connect()
    connection.on('message', (msg) => {
      onMessage(msg) // 👈🏻 🔥 새로운 메시지가 전달되면 onMessage로 이벤트를 핸들링
    });
    return () => connection.disconnect()
  }, [roomId, serverUrl]) // 👈🏻 🔥 onReceiveMessage 또는 onMessage를 EffectEvent 처리했기 때문에 의존성에서 제거됨
}

export function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234')

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) { // 👈🏻 🔥 실제 hook을 이용시. 이러한 분리를 통해 핸들링하는 이벤트의 로직을 동적으로 만들 수 있음
      showNotification('New message: ' + msg)
    }
  });

  return (
    <>
      <label>
        Server URL:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```
#### 🔅 잘못된 custom hook 사용 : lifecycle hook
```javascript
function useMount(fn) { // mount시 실행되는 useMount hook: 생명주기 hook
	useEffect(() => {
		fn()
	}, []) // 본래 linter가 사용된다면 fn의 의존성 누락에 대해 경고해야함
}
```
