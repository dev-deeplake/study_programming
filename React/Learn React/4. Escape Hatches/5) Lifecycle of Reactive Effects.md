>[!Learning Goals]
>- How an Effect's lifecycle is different from a component's lifecycle
>- How to think about each individual Effect in isolation
>- When your Effect needs to re-synchronize, and why
>- How your Effect's dependencies are determined
>- What it means for a value to be reactive
>- What an empty dependency array means
>- How React verifies your dependencies are correct with a linter
>- What to do when you disagree with the linter

- Effect의 lifecycle은 component와 다르다.
	- component: mount, update, unmount
	- Effect: start sync, stop sync
- Effect의 의존성을 제대로 설정했는지를 linter가 파악하고 알려준다.

## 1. The Lifecycle of an Effect
![|800](images/lifecycle_comparison_component_vs_effect.svg)
- component의 lifecycle과는 달리 Effect의 경우 lifecycle이 start sync / stop sync의 2가지 뿐이며, 이는 useEffect의 시작 부분에서 sync를 시작하는 로직이, return부에서 cleanup function이 sync 중단 로직을 제공함으로써 명시된다.
- Effect의 lifecycle은 component의 lifecycle과 무관하므로, component의 관점과 엮어 생각하지 말고 동기화의 시작 및 중지에만 집중하면 된다.
### 1) Effect의 re-sync
```javascript
const serverUrl = 'https://localhost:1234';  

function ChatRoom({ roomId }) {  
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  // start sync
		connection.connect();  // start sync
		return () => {  
			connection.disconnect();  // stop sync
		};  
	}, [roomId]);  
	// ...  

}
```
- useEffect는 roomId에 따라 서로 다른 채팅 서버에 연결
	- 사용자가 roomId를 "general"에서 "travel"로 변경하는 경우, Effect는 먼저 cleanup을 이용해 general 채팅 서버와의 연결을 끊는다.
	- 새로 변경된 travel 채팅 서버로 연결을 시도한다.
- component lifecycle 측면에서 보았을 때 Effect는 다음과 같이 동작
	- ChatRoom이 "general" roomId로 마운트됨 -- Effect가 "general" 채팅 서버에 연결
	- ChatRoom이 "travel" roomId로 변경됨 -- Effect는 "general" 채팅 서버 연결을 끊고 "travel" 채팅 서버에 연결
	- ChatRoom이 "music" roomId로 변경됨 -- Effect는 "travel" 채팅 서버 연결을 끊고 "music" 채팅 서버에 연결
	- ChatRoom이 마운트 해제됨 -- Effect가 "music" 채팅 서버와의 연결을 끊음
- 그러나 Effect의 측면에서 보면 매우 단순한 로직으로 움직이고 있음 → 시작 & 중지 cycle로만 생각하기!
	- Effect가 "general" 채팅 서버에 연결됨 (끊어지기 전까지)
	- Effect가 "travel" 채팅 서버에 연결됨 (끊어지기 전까지)
	- Effect가 "music" 채팅 서버에 연결됨 (끊어지기 전까지)
### 2) React가 Effect의 re-sync 가능 여부를 파악하는 법
- 개발 모드 : re-sync를 강제 (Effect 시작 → Effect 중지 → Effect 시작)하여 Effect의 시작 및 중지가 잘 되는지를 확인
- re-sync되는 경우: Effect가 사용하는 데이터에 변경이 있는 경우 (의존성 배열 및 내부 조직에서의 변화)
### 3) React가 Effect의 re-sync 필요성을 인식하는 법
- 🔥 의존성 배열
	- React에서는 component 재렌더링시 의존성 배열을 체크, 하나라도 이전과 다르면 Effect를 re-sync한다.
### 4) 동일한 대상을 동기화하는 경우에만 동일한 Effect로 묶기
- 각 Effect는 개별적이고 독립된 동기화 process를 나타내야 한다. 즉, 한 Effect 안에 작성된 코드가 서로 다른 것을 동기화하는 경우 분리해야 한다.
	- ![[5) Lifecycle of Reactive Effects#🔅 분석 이벤트 전송과 connection effect의 분리 - 리팩토링 전]]
		- logVisit과 connecting 로직은 서로 어떠한 관련도 없다.
			- 이렇게 한 useEffect에 묶을 경우, effect가 re-sync시 동일한 방에 대해 logVisit이 다시 한 번 남게 된다.
		- log를 남기는 것은 연결과는 분리된 과정이므로, 개별 Effect로 작성해야 함
	- ![[5) Lifecycle of Reactive Effects#🔅 분석 이벤트 전송과 connection effect 분리 - 리팩토링 후]]
	- 연관 있는 로직을 별도의 effect로 분리할 경우 코드가 깔끔해보일 수는 있지만 유지보수가 힘들어진다...
## 2. 반응형 값(reactive values)에 반응하는 Effect
- 변하지 않을 값은 의존성으로 설정하지 않아도 된다.
	- ![[5) Lifecycle of Reactive Effects#🔅 serverUrl이 static value인 경우의 코드]]
- 컴포넌트 내부에서 선언된 props, state 등의 값은 렌더링 중 연산 가능성 및 변동 가능성이 모두 존재한다. 이 경우에는 의존성 설정이 필요하다.
	- ![[5) Lifecycle of Reactive Effects#🔅 serverUrl 역시 reactive value인 경우의 코드]]
- useEffect 안에 명시해준 effect 시작 및 종료 방법은 어떤 의존성 배열이 들어오더라도 동일하게 작동한다.
	- 위의 예시에서 의존성 배열이 비었다면, 초기 렌더링 시 effect가 실행되고 난 후 effect 실행에 참조할 데이터가 없어 component가 언마운트 될 때까지 연결이 끊기지 않을 것
- 변이 가능한 값(mutable values)과 전역 값(global values)의 경우 - 의존성 대상에서 제외
	- 위 값들은 리액트에서 렌더링 시 추적하는 값이 아니기 때문에 변경이 된다 하더라도 react의 재렌더링을 트리거하지 않는다.
	- 더욱이 렌더링 중 변경 가능한 값을 읽는 것은 예측하지 못한 동작을 야기할 수 있다. (순수성 파괴)
	- ex) location.pathname, ref.current
- 의존성 대상은 우리가 선택하는 것이 아니라, Effect에서 읽은 모든 반응형 값을 포함해야 한다.
	- React는 linter를 통해 모든 반응형 값이 의존성 지정이 되었는지 알려준다.
	- 만일 어떤 값을 의존성에서 제외하고 싶다면 렌더링 중에 계산되지 않는 static value로 만들어주기
- 만일 linter가 강제하는 의존성 추가로 인해 문제가 발생한다면 다음을 확인해볼 것
	1. Effect가 서로 독립적인 sync 프로세스를 나타내고 있는지
	2. Effect가 무언가를 동기화하고 있는지 확인하고, 그렇지 않다면 제거를 고려하기
	3. props와 state에 반응하지 않고 마지막 값을 읽어오고자 한다면 Effect와 Effect Event로 둘을 분리하여 'Effect에 반응되지 않아야하는 이벤트'를 표시해줄 것
	4. 객체와 함수를 의존성으로 사용하고 있지 않은지 - 참조 기반의 값이므로 메모리 주소가 늘 달라짐

## ------ Example List ------
#### 🔅 분석 이벤트 전송과 connection effect의 분리 - 리팩토링 전
```javascript
function ChatRoom({ roomId }) {  
	useEffect(() => {  
		logVisit(roomId);  // 추적용 코드 logVisit
		const connection = createConnection(serverUrl, roomId);  // 서버 연결 로직 (sync start)
		connection.connect();  
	
		return () => {  
		connection.disconnect();  // 서버 연결 중지 로직 (sync stop)
		};  
	
	}, [roomId]);  
	
	// ...  

}
```
#### 🔅 분석 이벤트 전송과 connection effect 분리 - 리팩토링 후
```javascript
function ChatRoom({ roomId }) {  

	useEffect(() => {  
		logVisit(roomId);  
	}, [roomId]);  
	
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  
	// ...  
	}, [roomId]);  
	
	// ...  
}
```
#### 🔅 serverUrl이 static value인 경우의 코드
```javascript
const serverUrl = 'https://localhost:1234';    

function ChatRoom({ roomId }) {  
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  
		connection.connect();  
	
		return () => {  
			connection.disconnect();  
		};  
	
	}, [roomId]);  // 의존성 배열에는 roomId만 들어있다 (static value인 serverUrl은 제외됨!)
	
	// ...  

}
```
#### 🔅 serverUrl 역시 reactive value인 경우의 코드
```javascript
function ChatRoom({ roomId }) {  // prop은 받은 값에 따라 변경됨
	const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl 역시 state로 관리 : 변경가능성 있음
	
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId); // Effect에서 serverUrl과 roomId를 모두 사용함
		connection.connect();  
		
		return () => {  
			connection.disconnect();  
		};  
		
	}, [roomId, serverUrl]); // 의존성 배열에 roomId, serverUrl이 모두 포함
	
}
```
