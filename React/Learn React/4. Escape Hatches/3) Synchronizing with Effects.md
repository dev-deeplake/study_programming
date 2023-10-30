>[!Learning Goals]
>- What Effects are
>- How Effects are different from events
>- How to declare an Effect in your component
>- How to skip re-running an Effect unnecessarily
>- Why Effects run twice in development and how to fix them

- Effect는 렌더링 이후에 코드를 실행할 수 있도록 만들어줌
	- 컴포넌트를 React 외부의 시스템과 동기화하는 데 사용 가능

## 1. Effect ?
- 리액트 컴포넌트 내 2가지 로직
	- Rendering Code (Describing the UI 파트)
		- 컴포넌트의 최상위 레벨에 존재
		- Props & state 읽어오기, 변경하기, JSX 리턴
		- 순수성을 지켜야 함 (input이 동일할 시 output도 동일해야 함)
	- Event Handlers (Adding interactivity 파트)
		- 컴포넌트 내부에 선언된 중첩 함수
		- 연산 외 사용자의 action으로 인해 일어난 side effect 포함되어 있음
- Effect : event handler가 포괄하지 못하는, 이벤트가 없지만 발생해야 하는 작업을 커버
	- 특정 이벤트가 아닌 렌더링 자체로 인해 발생하는 side effect
	- '외부 시스템과의 동기화' 목적 중요!
	- Effect가 연결된 외부 system 없이 state를 기반으로 일부 state를 조장하는 역할만 하고 있을 경우, Effect를 사용하지 않아도 될 수 있다!
## 2. Effect 사용하기
- 다음의 3단계를 거쳐 작성
	1. Effect 선언
	2. Effect 의존성 명시 (필요할 때만 렌더링되도록 하기 위함)
	3. (필요한 경우) clean up 추가 : 수행중이던 작업의 중지, 취소, 정리법을 명시
### 1) Effect 선언
1. useEffect hook import
2. 컴포넌트의 최상위 레벨에서 호출 및 Effect 내부 코드 추가 (렌더링으로 인해 일어난 side effect는 모두 Effect로!)
3. 외부 시스템과의 동기화
### 2) 의존성 지정
- 렌더링 후 Effect 코드가 매번 실행되는 것을 막으려면 의존성을 추가해야 함
	- ex) 타이핑으로 인한 재렌더링 막기
1. 빈 배열을 추가 : 초기 렌더링 시에만 실행
2. 배열 내에 의존성을 넣어 추가 : 해당 의존성 요소 중 바뀐 것이 없는 경우 재렌더링 X
- ![[3) Synchronizing with Effects#🔅 의존성 배열 동작 비교]]
- 의존성 배열에는 변하지 않는 (안정적인) 값은 추가하지 않아도 된다! → 변치 않는 값은 결국 렌더링에 영향을 미치지 않기 때문
- 우리가 추가할 의존성을 선택하는 것이 아니라, Effect 내부의 코드를 기반으로 React가 예상하는 요소와 일치하는 의존성을 사용해야 함 (그렇지 않으면 error!)
- ![[3) Synchronizing with Effects#🔅 Effect 선언과 의존성 지정 - Video player 예시]]
### 3) cleanup 추가
- 필요한 경우 실행한 내용을 종료해주기 위한 클린업 함수 리턴
	- ![[3) Synchronizing with Effects#🔅 cleanup 함수의 사용 필요성 예시 - connecting to a server]]
		- 위 예에서 컴포넌트가 화면에서 사라져 unmount 되었더라도 자동으로 disconnect가 되지 않고, 해당 컴포넌트가 다시 마운트 되었을 때 새로운 연결이 또 시작됨
			- 이러한 것들이 중첩 및 중복되다 보면 앱의 속도와 성능이 현저히 저하될 수 있음
			- react는 이러한 케이스를 쉽게 발견할 수 있도록 모든 컴포넌트를 최초 마운트 직후에 다시 한 번 마운트함 (개발 모드에서만!)
				- 위 케이스의 경우 disconnect가 불러와지고 있지 않아 '✅ Connecting...'만 2번 출력됨
		- 연결이 쌓이지 않고 제대로 동작하게 하려면 ChatRook 컴포넌트 내 useEffect 코드에 다음과 같은 clean up을 추가해주어야 함
			- ![[3) Synchronizing with Effects#🔅 추가 cleanup]]
		- 컴포넌트에서 cleanup을 필요한 곳에 잘 구현했을 경우, react의 테스트 과정에서 행위-cleanup이 쌍으로 한 번 일어나고, 그 다음 컴포넌트 재렌더링으로 인한 행위가 다시 실행된다.
			- 예시처럼 코드를 고치면  '✅ Connecting...' → '❌ Disconnected.' → '✅ Connecting...' 순으로 로그가 찍힘
			- 마운트 로그 등을 보고 Effect에서 닫혀야 할 행동들이 cleanup 되지 않은 것을 파악할 수 있음
		- Strict mode를 해제하면 개발 모드에서도 위와 같은 재마운트를 없앨 수 있지만 디버깅으로 인해 유지하는 것을 추천

## 3. cleanup function - 개발환경에서 두 번씩 실행되는 Effect 처리 예시
- React의 의도적인 재마운트에도 불구하고 Effect가 잘 동작하게 하려면 일반적으로 cleanup 함수를 잘 구현하면 된다!
	- 재마운트가 되더라도 상용 버전에서 유저들이 사용하게 될 것과 동작의 차이가 없어야 함
- ### 1) React가 아닌 UI 컴포넌트 제어시
	- React 라이브러리나 프레임워크를 사용하지 않고 만들어진 UI 컴포넌트들을 제어할 때 : 해당 컴포넌트의 특성에 따라 제어에 차이
		- ex1) 중복 호출에도 문제가 없는 경우 - 외부 지도 컴포넌트 (Google Maps API 같은...) 추가 예제
			- API에서 확대 / 축소 비율을 결정하는 setZoomLevel 메소드가 제공됨
			- 메소드에 들어갈 값은 React의 zoomLevel state 변수를 통해 제어 가능
			- ![[3) Synchronizing with Effects#🔅 Cleanup pattern Controlling non-React widgets (1)]]
			- 이 경우 따로 cleanup을 추가하지 않아도 됨 : zoomLevel이 react에서 관리되는 state이므로, 해당 값이 변하지 않는다면 useEffect를 통한 재렌더 및 호출 쌓임이 일어나지 않을 것
		- ex2) 컴포넌트 자체적으로 중복 호출을 허용하지 않는 경우 (Browser built-in [`<dialog>`](https://developer.mozilla.org/ko/docs/Web/HTML/Element/dialog))
			- `<dialog>` : represents a modal / non-modal dialog box or other interactive component
				- dialog에서 창을 띄우는 showModal()의 경우 두 번 호출시 에러를 던지므로 cleanup이 필요
					- ![[3) Synchronizing with Effects#🔅 Cleanup pattern Controlling non-React widgets (2)]]
- ### 2) 이벤트 리스닝이 있는 경우
	- Effect 안에서 특정 이벤트를 리스닝하는 경우 Effect의 종료 전에 이벤트 리스닝을 cleanup 해주기
		- ![[3) Synchronizing with Effects#🔅 Cleanup pattern Subscribing to events]]
		- 스크롤 이벤트가 발생할 때마다 콘솔에 현재의 스크롤 위치를 출력하는 로직
		- cleanup이 없다면 컴포넌트가 언마운트 되더라도 event listener가 계속 남아있으며, 새롭게 컴포넌트가 마운트 될 시 새로운 event listener가 계속 추가되는 문제 발생
- ### 3) 애니메이션을 트리거한 경우
	- Effect가 무언가 animate를 했다면 해당 animation을 초기화해주는 cleanup을 반드시 추가해줘야 함
		- 성능 저하 문제 외에도 컴포넌트가 제거되었다가 다시 마운트될 때, 이전 언마운트 된 상태의 animation (초기화하지 않은!)을 기반으로 시작하는 문제가 생길 수 있음
		- ![[3) Synchronizing with Effects#🔅 Cleanup pattern Triggering animations]]
- ### 4) 데이터 fetching
	- data fetching은 '취소'할 수 없는 행위
		- 따라서 Effect에서 fetching이 일어나고 있다면, cleanup으로 fetch를 중단하거나 결과를 무시하는 로직을 추가해야 함
			- 사용자가 빠르게 여러 페이지를 전환한다고 할 때, cleanup이 존재하지 않으면 컴포넌트가 언마운트되었을 때도 이전 페이지의 데이터 가져오기 요청이 시도 및 완료될 수 있고, 이 때문에 setState 등으로 상태를 업데이트하려 할 때 에러가 발생할 수 있음
		- ![[3) Synchronizing with Effects#🔅 Cleanup pattern Fetching data]]
			- useEffect가 실행될 때마다 고유한 ignore 변수를 가짐
				- cleanup 함수로 `ignore = true`를 설정하여 userId가 변경되었을 때 앞에 전송되었던 데이터 fetching 요청을 무시하도록 설정
					- Alice에 대한 useEffect가 실행, 해당 scope에서 ignore 변수를 false로 설정
					- Alice 데이터를 가져오기 위한 비동기 요청 시작
					- userId가 Bob으로 변경
						- Alice에 대한 ignore 변수가 true로 변경되고 Alice userId를 통해 요청된 건이 무시되도록 cleanup 작동
					- Bob에 대한 useEffect가 실행, 해당 scope에서 ignore 변수가 false로 설정됨
					- Bob 데이터를 가져오기 위한 비동기 요청 시작
			- 설정이 유효한 경우
				- Alice에 대한 Data fetching이 끝나기 전에 유저가 userId를 Bob으로 전환한 경우
					- 도착한 Bob 데이터는 정상적으로 처리
					- 이후 Alice 데이터가 도착한다 해도 이미 Ignore 변수가 true로 설정되어 데이터를 무시하도록 조치되어 UI에 표시되는 등의 잠재적 문제를 방지
		- 개발 환경에서 보여지는 중복된 request를 없애고 싶다면, 받아온 응답을 캐싱하여 컴포넌트 간에 이동시 동일한 request가 발생했을 때 request를 하는 대신 해당 캐싱 데이터를 사용하는 방법을 고려할 수 있음
	- Effect 내부에서 데이터를 fetching했을 때의 단점
		- useEffect 내부의 코드는 SSR 동안에는 실행되지 않으므로, 서버에서 보내준 초기 HTML이 데이터를 포함하지 않는다. 그 결과 JS가 클라이언트측에서 다운로드되고 실행될 때 까지 로딩 상태가 지속된다.
		- Network waterfall
			- useEffect 내에서 데이터를 Fetching하게 되면 부모 컴포넌트에서 일부 데이터 fetch → 하위 컴포넌트 렌더링 → 하위 컴포넌트 데이터 Fetch ... 처럼 작동하게 될 수 있음
			- 모든 데이터를 동시에 가져오는 것이 훨씬 효율적
		- 사전 로딩이나 캐싱 부재
			- 사전 로딩이나 캐싱 없이 컴포넌트의 마운트 및 마운트 해제에 따라 데이터 fetching이 이루어짐에 따라 불필요한 네트워크 요청이 발생하거나 앱이 느려질 수 있음
		- 비효율적
			- 두 개 이상의 연산이 서로의 실행 순서에 따라 다른 결과를 낼 수 있는 [[조건 경합 (Race condition)]]이나 기타 오류 처리를 위해 많은 보일러플레이트와 코드 작성을 필요로 함
	- 해결책?
		- framework 이용시 Built-in된 fetching 매커니즘을 사용하기
			- [리액트의 최신 프레임워크](https://react-ko.dev/learn/start-a-new-react-project#production-grade-react-frameworks)에서 통합된 data fetching 매커니즘을 지원함
		- 클라이언트측 캐싱을 제공하는 솔루션을 사용하거나 직접 구축하기
			- React Query, useSWR, React Router (6.4+)
- ### 5) 분석 이벤트 전송시
	-  production 모드에서는 분석 이벤트 전송이 중복되어 일어나지 않기 때문에 코드를 그대로 놔두는 것이 좋지만, 다음과 같은 대안을 채택할 수 있음
		- 스테이징 환경에 앱 배포
			- 스테이징 환경 : 실제 프로덕션 환경과 유사하지만 실제 사용자가 사용하는 것이 아닌, 개발 및 테스트를 위한 환경
			- 이 환경에서 앱을 production 모드로 실행하여 실제 서비스가 어떻게 동작할지 확인 및 분석 이벤트 디버깅이 가능
		- Strict Mode 선택 해제
		- 예제의 경우 추가적으로 선택할 수 있는 해결책?
			- ![[3) Synchronizing with Effects#🔅 Cleanup pattern Sending Analytics]]
				- 라우트 변경 이벤트 핸들러에서 분석 보내기
					- Router의 이벤트 핸들러에서 직접 분석 이벤트를 보내면 언제 route가 변경되었는지 더 정확하게 알 수 있음
				- [Intersection Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) 사용하기
					- 웹 페이지의 특정 요소가 뷰포트에 들어오거나 나갈 때 알려주는 기능을 제공
					- 사용자가 페이지에서 어떤 요소를, 얼마나 오래 보고 있는지 등을 파악 가능
- ### 6) Effect로 처리하지 않는 경우 
	- 애플리케이션 전체가 실행될 때 한 번 실행되어야 하는 로직
		- 다음과 같이 컴포넌트 외부로 로직을 추출
			- ![[3) Synchronizing with Effects#🔅 Not an Effect (1) Initializing the application]]
	- Cleanup 함수가 있음에도 Effect가 여러번 실행되었을 때 결과가 크게 달라지는 경우
		- 제품 구매 (2번 실행되면...중복 구매...) : 사용자가 다른 페이지로 이동했다가 다시 제품 구매 request가 일어나는 페이지로 돌아오면 중복 구매가 일어날 가능성이 있음!
			- 구매는 렌더링이 아니기 때문에 사용자가 구매 버튼을 누를 때만 실행되도록 버튼 이벤트 핸들러로 이동시키기

## ------ example list ------
#### 🔅 의존성 배열 동작 비교
```javascript
useEffect(() => {  // 의존성 배열이 없는 기본 형태 : 렌더시마다 실행
	...
});  


useEffect(() => {  // 의존성 배열이 빈 배열인 형태 : 컴포넌트의 초기 렌더링시에만 실행
	...
}, []);  

  
useEffect(() => {  // 의존성 배열에 내용이 있는 경우 : 초기 렌더링시 + a 또는 b가 직전 렌더와 달라졌을 때 실행
	...
}, [a, b]);
```
#### 🔅 Effect 선언과 의존성 지정 - Video player 예시
```javascript
import { useState, useRef, useEffect } from 'react'; // 1. hook import

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  // 2. 컴포넌트 최상단에서 useEffect 호출 및 내부 로직 코드 추가
  useEffect(() => { // 이 side effect를 useEffect로 감싸지 않으면 렌더링 중에 DOM 노드를 조작하는 형태가 되고, 이 코드가 호출된 시점이 JSX 리턴 전이므로 play()나 pause()를 호출할 DOM 노드가 존재하지 않아 에러가 난다.
  // useEffect로 감싼 코드는 렌더링 및 커밋이 모두 끝난 이후 실행되므로, 외부 요소 (여기에서는 브라우저 API)와 연결해서 작업을 처리해도 안정적이다.
    if (isPlaying) { // 3. 외부 시스템과의 동기화 코드 (side effect)
      console.log('Calling video.play()');
      ref.current.play();
    } else {
      console.log('Calling video.pause()');
      ref.current.pause();
    }
  }, [isPlaying]); // 4. 의존성 추가 - 의존성을 isPlaying state로 설정해주어 해당 state 값이 바뀔 때마다 useEffect 안의 코드가 실행된다.

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```
#### 🔅 cleanup 함수의 사용 필요성 예시 - connecting to a server
```javascript
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []);
  return <h1>Welcome to the chat!</h1>;
}

function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting...');
    },
    disconnect() {
      console.log('❌ Disconnected.');
    }
  };
}
```
#### 🔅 추가 cleanup
```javascript
...
export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect(); // 추가된 cleanup
  }, []);
  return <h1>Welcome to the chat!</h1>;
}
```
#### 🔅 Cleanup pattern : Controlling non-React widgets (1)
```javascript
...
useEffect(() => {  
	const map = mapRef.current;  
	map.setZoomLevel(zoomLevel);  
}, [zoomLevel]);
```
#### 🔅 Cleanup pattern : Controlling non-React widgets (2)
```javascript
...
useEffect(() => {  
	const dialog = dialogRef.current;  
	dialog.showModal();  
	return () => dialog.close();  
}, []);
```
#### 🔅 Cleanup pattern : Subscribing to events
```javascript
...
useEffect(() => {  
	function handleScroll(e) {  
		console.log(window.scrollX, window.scrollY);  
	} 

	window.addEventListener('scroll', handleScroll);  
	return () => window.removeEventListener('scroll', handleScroll);  
}, []);
```
#### 🔅 Cleanup pattern : Triggering animations
```javascript
useEffect(() => {  
	const node = ref.current;  
	node.style.opacity = 1; // 투명도를 1로 바꾸는 애니메이션  
	return () => { // cleanup
		node.style.opacity = 0; // 투명도 0 (초기값)으로 애니메이션 재설정
	};  
}, []);
```
#### 🔅 Cleanup pattern : Fetching data
```javascript
useEffect(() => {  
	let ignore = false;  
	
	async function startFetching() {  
		const json = await fetchTodos(userId);  
		if (!ignore) {  
			setTodos(json);  
		}  
	}  
	
	startFetching();  
	
	return () => {  
		ignore = true;  
	};  
}, [userId]);
```
#### 🔅 Cleanup pattern : Sending Analytics
```javascript
useEffect(() => {  
	logVisit(url); // url이 변경될 때마다 어떤 url을 방문했는지 log를 post
}, [url]);
```

#### 🔅 Not an Effect (1) : Initializing the application
```javascript
if (typeof window !== 'undefined') { // 실행 환경이 브라우저이면 아래를 실행  
	checkAuthToken();  
	loadDataFromLocalStorage();  
}  

function App() {  
	// ...  
}
```
#### 🔅 Putting it all together
```javascript
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 Schedule "' + text + '" log');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancel "' + text + '" log');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        What to log:{' '}
        <input
          value={text}
          onChange={e => setText(e.target.value)}
        />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Unmount' : 'Mount'} the component
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
```