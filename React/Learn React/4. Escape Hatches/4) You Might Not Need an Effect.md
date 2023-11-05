>[!Learning Goals]
>- Why and how to remove unnecessary Effects from your components
>- How to cache expensive computations without Effects
>- How to reset and adjust component state without Effects
>- How to share logic between event handlers
>- Which logic should be moved to event handlers
>- How to notify parent components about changes

- Effect = "escape hatch"
	- 외부 시스템이 엮여있지 않은 경우 effect가 필요치 않다.
	- effect의 무분별한 사용은 알아보기 어렵고, 속도가 느리고, 에러 가능성이 높은 코드를 만들어내므로 주의!
- effect를 사용하지 않는 것이 더 효율적인 case의 모음
## 1. Effect가 필요치 않은 경우
### 1) 렌더링을 위한 데이터 변형 
- 리액트의 렌더링 및 effect 작동 순서
	- State update → 컴포넌트 함수 호출 → 표시될 내용 계산 → 실제 DOM에 커밋 → 화면 update → effect 실행
- 만일 effect가 실행되면서 state를 업데이트하면 다시 처음으로 돌아가 똑같은 과정이 반복되므로 불필요한 렌더링이 일어남
-  모든 데이터 변형을 컴포넌트의 최상위 레벨에서 실행하면 prop 및 state가 변경될 때마다 코드가 자동으로 실행됨
### 2) 유저 이벤트 핸들링
- Effect로 유저 이벤트를 핸들링할 경우 사용자의 정확한 동작을 알기 힘듦. 따라서 유저 이벤트는 이벤트 핸들러에서 작성해주기
	- ![[4) You Might Not Need an Effect#🔅 useEffect를 사용해 이벤트를 핸들링하는 예시]]
		- 이벤트 핸들러를 이용한다면 사용자가 버튼을 클릭하는 순간 로직이 실행되지만, useEffect를 사용하면 버튼 클릭 → 상태 변화 → 로직 실행의 과정을 거치게 됨
			- 코드가 복잡해지면 이 로직이 왜, 언제 실행되는지 알기 어려울 수 있음
		- 특히 사용자와의 직접적인 상호작용에 의해 발생하는 동작의 경우 이벤트 핸들러를 사용할 것
## 2. Effect를 쓰기 전에 생각할 것들
### 1) 현재 존재하는 props나 state에서 계산할 수 있는 것들을 effect로 처리하고 있지는 않은가?
- 굳이 effect를 이용해서 state를 한 번 더 변경하는 과정을 거치지 말고 렌더링 중에 바로 계산하기
	- ![[4) You Might Not Need an Effect#🔅 effect + state update refactoring]]
### 2) useEffect에서 비용이 많이 드는 계산을 핸들링하고 있지 않은가?
- 고비용 계산을 알아내기
	- `console.time() / console.timeEnd()` 를 이용해 측정 가능
		- 기록 시간이 1ms 이상일 경우 캐싱 이용을 고려해봐도 좋음
	- CPU 쓰로틀링을 이용한 성능 테스트
		- chrome의 performance 탭 내에 쓰로틀링 옵션이 존재
			- 프로세서 속도가 감소되면 평소에 빠르게 처리되어 눈에 띄지 않던 계산이 UI의 반응 시간에 영향을 줌
			- 저성능 기기 사용시의 케이스를 시뮬레이션 가능
		- 개발 중 성능 측정은 정확하지 않으므로(ex. Strict Mode) 되도록 프로덕션과 유사한 환경에서 테스트하기
- 고비용 계산의 useEffect 처리 및 리팩토링 예시
	- ![[4) You Might Not Need an Effect#🔅 Expensive calculation + useEffect refactoring]]
		- useMemo를 사용한 캐싱 이용
			- useMemo로 감싸인 함수는 렌더링 중에 실행 → 실행되는 함수는 순수해야 함
### 3) Prop의 변경으로 컴포넌트의 state를 전체 재설정해야 하는 경우가 아닌가?
- key는 컴포넌트를 고유하게 만들어줌 : 변경시 DOM 재생성 및 state 재설정
	- 따라서 prop이 변경되었을 때 모든 state를 재생성하고 싶다면 Key를 활용해보기 
	- ![[4) You Might Not Need an Effect#🔅 reset all state when a prop changed - refactoring]]
### 4) 렌더링 중 연산으로 대체 가능한 것을 Effect + state로 관리하고 있지 않은가?
- prop이 변경될 때 state의 전체가 아닌 일부를 조정 및 재설정하고 있는 경우
	- 해당 조정 과정에서 useEffect에 state 설정 로직이 들어가있다면 불필요한 렌더링이 한 번 더 일어나고 있다는 것이므로 되도록 지양
	- state로 설정된 값을 렌더링 중 연산으로 대체 가능한지 생각해보고 가능하면 해당 방향으로 변경
	- ![[4) You Might Not Need an Effect#🔅 adjusting some state when a prop changes - refactoring]]
### 5) Effect 내에서 처리하고 있는 로직이 사용자에게 component가 표시되었기 때문에 실행되어야 하는 로직으로만 구성되어있는가?
- 🔥 사용자에게 component가 표시되었기 때문에 실행되어야하는 경우에만 Effect에서 처리하기
- ex 1) **이벤트 핸들러 간 동일한 로직을 사용할 때**
	- 이벤트 핸들러 간 동일한 로직을 사용하는 경우 해당 로직이 중복처럼 여겨져 useEffect 안에 넣어 한번에 처리하고 싶을 수 있다. 그러나 이것은 함정 -ㅅ-
		- 이벤트는 유저와의 interaction에 연관되어있다. 그러나 useEffect는 새로고침시에도 동작한다. 즉 이벤트를 useEffect 안에서 처리할 경우 예상치 못한 결과가 초래될 수 있다!
	- ![[4) You Might Not Need an Effect#🔅 sharing logic between event handlers (1) - refactoring]]
- ex 2) **POST 요청을 보내는 경우**
	- 컴포넌트가 보여짐으로 인해서 POST 요청이 필요한 게 아닌, 유저 인터랙션에 의해 POST가 이루어지고 있는 경우 Effect가 아닌 event handler에서 POST 처리
	- ![[4) You Might Not Need an Effect#🔅 sharing logic between event handlers (2) - refactoring]]





### 6) Effect 내에서 state를 바탕으로 다른 state를 연쇄적으로 변경하고 있지 않은가?
- Effect 내에서 연산 체인(Chains of computations)이 일어나고 있는 경우 setState 과정에서 매번 불필요한 리렌더링이 일어나게 된다.
- 코드의 경직도가 높다(rigid) : 체인 중에 수행해야 할 단계가 추가될 경우 전체 체인을 손봐야 할 수 있고, 로직 재사용이 어려움
- 이런 경우 렌더링 중에 연산이 가능한 것은 연산으로 빼고, state 조정은 event handler 안에서 수행한다.
- ![[4) You Might Not Need an Effect#🔅 chains of computations - refactoring]]
	- 이벤트 핸들러 내부에서도 state는 snapshot처럼 동작하므로, 연산에 다음 값을 사용해야 하는 경우 수동으로 다음 값으로 정의해 사용하기
- 이벤트 핸들러 내부에서 다음 state를 계산할 수 없는 경우에는 Effect에서 관리하는 것이 적절할 수 있으므로 상황에 따른 선택 필요
	- ex) 과거 드롭다운 Option 선택값에 따라 다음 드롭다운 option이 달라지는 경우

### 7) App 초기화를 Effect 안에서 처리하고 있지 않은가?
- App 초기화의 경우 컴포넌트의 초기화와 상관 없이 App의 전체 동작에서 단 한 번만 일어나야 하지만, Effect 안에서 처리되면 개발 과정에서는 언제나 두번씩 실행됨
- Effect가 아닌 최상위 레벨에서 관련 로직 처리하기
	- 최상위 변수를 통해 App이 이미 실행되었는지 추적하거나
		- ![[4) You Might Not Need an Effect#🔅 app 초기화 - 최상위 variable 이용]]
	- App이 렌더되기 전에 한 번만 실행할 수도 있음
		- ![[4) You Might Not Need an Effect#🔅 app 초기화 - app 렌더링 전 1회만 실행]]
	- 최상위 레벨의 코드는 언제나 한 번은 실행되므로 과도하게 사용할 경우 속도 저하를 초래할 수 있으므로 신중하게 사용할 것

### 8) State 변경을 부모 컴포넌트에게 알리는 로직을 Effect에서 실행하고 있지 않은가?
- State 변경 사실을 부모 컴포넌트에게 알리기 위해 prop으로 onChange setter를 받아 제어할 수 있는 경우,
	- onChange 세팅을 useEffect에서 하게 되면 역시 불필요한 렌더링이 일어나게 됨
	- 예시의 'toggle'은 인터랙션에 해당하므로 이벤트 핸들러에서 컴포넌트의 state 업데이트가 이루어지는 것이 바람직
		- ![[4) You Might Not Need an Effect#🔅 부모에게서 setter를 내려받아 부모에게 state 변경을 알리기]]
	- state를 제거하고 부모 컴포넌트로부터 isOn을 prop으로 받는 해결책도 존재 (state 끌어올리기)
		- ![[4) You Might Not Need an Effect#🔅 State 끌어올리기로 해결]]
### 9) Effect를 통해 부모 컴포넌트에게 데이터를 전달하려고 시도하고 있지 않은가?
![[4) You Might Not Need an Effect#🔅 React의 data flow를 벗어난 경우 (자식에서 부모에게 데이터 전달)]]
- React의 데이터 흐름은 단방향 : 부모 → 자식
	- 이와 같은 flow를 벗어나는 경우 데이터가 잘못되었을 때 그 출처를 찾기 어려워짐
	- 따라서 자식에서 부모 컴포넌트에 데이터를 전달하지 말고, 부모 컴포넌트가 데이터를 자식에게 전달하는 방식으로 변경할 것
### 10) 외부 저장소를 구독하고 있지 않은가?
- useEffect를 사용해도 되지만, React에서 해당 목적을 위해 제작된 useSyncExternalStore 훅이 있으므로 해당 훅을 사용
- ![[4) You Might Not Need an Effect#🔅 useSyncExternalStore 훅의 사용 예시]]
### 11) 데이터 fetch를 useEffect에서 처리하고 있다면 클린업 함수를 꼭 추가하고, 로직을 커스텀 훅으로 추출하는 것을 고려하기
- 데이터 fetch가 사용자의 인터랙션과 연결되어있지 않고 컴포넌트 표시와 연관이 있는 경우 useEffect에서 Fetch를 수행하는 것이 일반적
- 다만 [[조건 경합 (Race condition)]]을 고려하고 제대로 fetch 결과를 보여주기 위해서는 클린업 함수가 꼭 필요
	- ex) hello를 키보드로 쳐 해당 결과를 fetching한다 했을 때 h, he, hel, hell, hello에 대한 페칭이 모두 수행될 테지만 응답의 순서는 보장할 수 없음
	- 조건 경합 외에도 응답 캐싱, 서버에서 fetch해오는 방법, 네트워크 워터폴을 피하기 위한 방법 등도 고려 필요
		- 최신 프레임워크의 데이터 페칭 mechanism을 통해 해결할 수도 있고
		- Effect에서 Fetching을 좀 더 우아하게 하고 싶다면 커스텀 훅으로 만드는 것도 좋음
		- ![[4) You Might Not Need an Effect#🔅 fetching 로직의 커스텀 훅 제작 예시]]
## ------ example list ------
#### 🔅 useEffect를 사용해 이벤트를 핸들링하는 예시
```javascript
import { useState, useEffect } from 'react';

function BuyButton() {
  const [isClicked, setIsClicked] = useState(false);

  useEffect(() => {
    if (isClicked) {
      const sendPurchaseRequest = async () => {
        try {
          const response = await fetch('/api/buy', { method: 'POST' });
          if (response.ok) {
            alert('Purchase is successful!');
          }
        } catch (error) {
          console.error('Purchase failed', error);
        }
      };
      
      sendPurchaseRequest();
      setIsClicked(false); // 요청을 보낸 후 클릭 상태를 재설정
    }
  }, [isClicked]); // 1. isClicked에 따라 useEffect가 동작. 사용자가 'Buy' 버튼을 클릭할 경우 isClicked가 true로 변하고, 이것이 useEffect를 트리거하여 하위 코드가 작동하도록 함.

  const handleBuyClick = () => {
    setIsClicked(true);
  };

  return (
    <button onClick={handleBuyClick}>Buy</button>
  );
}

```

#### 🔅 effect + state update refactoring
```javascript

// 리팩토링 전
function Form() {  
	const [firstName, setFirstName] = useState('Taylor');  
	const [lastName, setLastName] = useState('Swift');  
	
	const [fullName, setFullName] = useState('');  
	useEffect(() => {  
		setFullName(firstName + ' ' + lastName); // firstName과 lastName으로 계산할 수 있는 fullName을 useEffect 안에서 계산 : 불필요한 렌더링이 한 단계 더 일어남
	}, [firstName, lastName]);  
	// ...  

}


// 리팩토링 후
function Form() {  
	const [firstName, setFirstName] = useState('Taylor');  
	const [lastName, setLastName] = useState('Swift');  
	
	const fullName = firstName + " " + lastName; // effect + state 조합을 사용하지 않고 바로 계산

}
```
#### 🔅 Expensive calculation + useEffect refactoring
```javascript
// getFilteredTodos가 고비용 연산인 경우

// 1. 초기 상태
function TodoList({ todos, filter }) {  
	const [newTodo, setNewTodo] = useState('');  
	const [visibleTodos, setVisibleTodos] = useState([]);  
	useEffect(() => {  
		setVisibleTodos(getFilteredTodos(todos, filter));  
	}, [todos, filter]);  // state setting이 useEffect 안에서 처리: 렌더링이 한번 더 일어나는 결과
	// ...
}


// 2. 1차 리팩토링
function TodoList({ todos, filter }) {  
	const [newTodo, setNewTodo] = useState('');  
	const visibleTodos = getFilteredTodos(todos, filter); // newTodo에 의해 getFilteredTodos의 연산이 영향을 받게 됨 → 비싼 연산을 불필요하게 호출하게 될 가능성
	// ...
}


// 3. 2차 리팩토링 : memoization 이용 - todos와 filter가 변경되지 않으면 useMemo에 이미 캐싱된 값을 이용
import { useMemo, useState } from 'react';  

function TodoList({ todos, filter }) {  
	const [newTodo, setNewTodo] = useState('');  
	const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);  
	// ...  

}

```
#### 🔅 reset all state when a prop changed - refactoring
```javascript
// 1. 리팩토링 전
export default function ProfilePage({ userId }) {  
	const [comment, setComment] = useState('');
	
	useEffect(() => {  // useEffect는 DOM 커밋 및 렌더가 끝난 이후에 실행됨 ... 변경된 props로 인해 ProfilePage의 렌더링이 먼저 일어난 뒤 useEffect에 의해 setter가 동작하여 컴포넌트가 재렌더링되는 문제...
		setComment('');  
	}, [userId]);  
	
	// ...  
}


// 2. 리팩토링 후
export default function ProfilePage({ userId }) {  
	return (  
		<Profile  
			userId={userId}  
			key={userId}  // key를 통해, userId가 변경된 이후 만들어진 Profile 컴포넌트는 개념적으로 다른 컴포넌트임을 명시하여 state 자동 재설정
		/>  
	);  
}  

function Profile({ userId }) {  // key가 변하면 이 컴포넌트 및 자식 컴포넌트의 state가 자동으로 재설덩됨
	const [comment, setComment] = useState('');  
	// ...  
}
```
#### 🔅 adjusting some state when a prop changes - refactoring
```javascript
// 1. 리팩토링 전
// List가 items 목록을 prop으로 받아오고, selection state 변수에 선택된 항목을 저장
// items prop의 배열이 달라질 때마다 selection을 null로 재설정
function List({ items }) {  
	const [isReverse, setIsReverse] = useState(false);  
	const [selection, setSelection] = useState(null);  
	
	useEffect(() => {  
		setSelection(null);  // Effect 안에서의 state 설정 : 불필요한 렌더링 촉발
	}, [items]);  
	
	// ...  

}


// 2. 1차 리팩토링 : 렌더링 중에 state 직접 조정 & 이전 렌더링 정보 저장
function List({ items }) {  
	const [isReverse, setIsReverse] = useState(false);  
	const [selection, setSelection] = useState(null);  
	const [prevItems, setPrevItems] = useState(items);  // 이전 items의 정보를 prevItems state에 저장
	
	if (items !== prevItems) {  // Effect 없이, 받은 items와 이전 item를 비교해서 다를 때만 이전 Item을 현재 item으로 바꾸고 selection을 null로 바꾸기 → 불필요한 렌더 줄어듦
		setPrevItems(items);  
		setSelection(null);  
	}  
	
	// ...  

}


// 3. 2차 리팩토링 : 렌더링 중 state를 연산할 수 있다면 연산으로 변경
function List({ items }) {  
	const [isReverse, setIsReverse] = useState(false);  
	const [selectedId, setSelectedId] = useState(null);  
		
	const selection = items.find(item => item.id === selectedId) ?? null;
	// .find() 배열에서 주어진 대상과 일치하는 가장 첫 요소를 반환, 해당 요소가 없으면 undefined 리턴
	// items 배열이 변경될 때마다 'selectedId'에 해당하는 아이템을 배열에서 찾아내고, 없으면 null을 사용
	// selection을 상태로 관리하는 대신 연산으로 변경, 재렌더링을 피할 뿐 아니라 다른 state를 바탕으로 다시 state를 조정치 않기 때문에
	// 예측 가능성이 높아짐 
	
	// ...  

}

```
#### 🔅 sharing logic between event handlers (1) - refactoring
```javascript
// 제품 구매를 두 개의 버튼에서 수행할 수 있는 경우 (구매 & 결제)
// 사용자가 제품을 장바구니에 넣을 때마다 알림을 표시

// 1. 리팩토링 전
function ProductPage({ product, addToCart }) {  
	useEffect(() => {  
		if (product.isInCart) {  
			showNotification(`Added ${product.name} to the shopping cart!`);
			// handleBuyClick과 handleCheckoutClick 모두에 showNotification을 넣으려니 중복인 것 같아서 이렇게 처리했겠지만...
			// 이렇게 되면 제품을 장바구니에 넣을 때 뿐 아니라 카트에 제품이 추가된 상태에서 페이지를 새로고침했을 경우 알림이 계속 뜨게 됨.
		}  
	}, [product]);  
	
	function handleBuyClick() {  
		addToCart(product);  
	}  
	
	function handleCheckoutClick() {  
		addToCart(product);  
		navigateTo('/checkout');  
	}  
	// ...  
}


// 2.리팩토링 후
function ProductPage({ product, addToCart }) {  
	function buyProduct() {  // 물건을 구매했을 때 실행되어야 하는 로직을 묶고
		addToCart(product);  
		showNotification(`Added ${product.name} to the shopping cart!`);  
	}  
	
	function handleBuyClick() {  
		buyProduct();  // 각각의 핸들러에 연결해주는 게 버그를 줄이는 방법! 중복이라고 불편해하지 않기
	}  
	
	function handleCheckoutClick() {  
		buyProduct();  // 각각의 핸들러에 연결해주는 게 버그를 줄이는 방법! 중복이라고 불편해하지 않기
		navigateTo('/checkout');  
	}  
	// ... 
}
```
#### 🔅 sharing logic between event handlers (2) - refactoring
```javascript
// 컴포넌트 마운트시에는 분석 이벤트를 보내고,
// 양식 작성 후 제출 버튼을 클릭했을 때 /api/register 로 POST 요청하기

// 1. 리팩토링 전
function Form() {  
	const [firstName, setFirstName] = useState('');  
	const [lastName, setLastName] = useState('');  
	useEffect(() => {  
		post('/analytics/event', { eventName: 'visit_form' });
		// analytics 이벤트는 컴포넌트가 보여졌을 때 전송되어야 하므로 useEffect 내에서 처리되는 것이 옳음
	}, []);  
	
	const [jsonToSubmit, setJsonToSubmit] = useState(null);  
	useEffect(() => {  
		if (jsonToSubmit !== null) {  
			post('/api/register', jsonToSubmit);
		}
		// json POST 요청은 '제출' 버튼을 클릭했을 때 실행되어야 하는 것이므로
		// Effect가 아닌 event handler가 담당해야하는 로직
		// 굳이 jsonToSubmit이라는 상태를 두어 처리할 필요 없음.
	}, [jsonToSubmit]);  
	
	function handleSubmit(e) {  
		e.preventDefault();  
		setJsonToSubmit({ firstName, lastName });  
	}  
	// ... 
}


// 2. 리팩토링 후
function Form() {  
	const [firstName, setFirstName] = useState('');  
	const [lastName, setLastName] = useState('');  
	useEffect(() => {  
		post('/analytics/event', { eventName: 'visit_form' });
	}, []);  
		
	function handleSubmit(e) {  
		e.preventDefault();  
		post('/api/register', { firstName, lastName });  // 상태를 제거하고, submit이 클릭되었을 때 단순하게 post하는 방향으로!
	}  
	// ... 
}
```
#### 🔅 chains of computations - refactoring
```javascript
// gold 카드를 한 라운드 안에 3번 놓아야 다음 라운드로 진행, 게임은 5라운드까지

// 1. 리팩토링 전 : chain of computation
function Game() {  
	const [card, setCard] = useState(null);  
	const [goldCardCount, setGoldCardCount] = useState(0);  
	const [round, setRound] = useState(1);  
	const [isGameOver, setIsGameOver] = useState(false);  
	
	useEffect(() => {  
		if (card !== null && card.gold) {  
			setGoldCardCount(c => c + 1);  
		}  
	}, [card]);
	// 새 카드가 놓일 때마다 gold 카드의 개수를 추적하기 위한 로직
	// card 상태가 변경될 때마다 실행
	// 새로운 카드가 null이 아니며 gold 카드인 경우 goldCardCount를 1 증가
	
	useEffect(() => {  
		if (goldCardCount > 3) {  
			setRound(r => r + 1)  
			setGoldCardCount(0);  
		}  
	}, [goldCardCount]);
	// gold 카드의 개수가 특정 값을 초과하면 다음 라운드로 진행하기 위한 로직
	// goldCardCount가 변경될 때마다 실행 (= 위의 useEffect와 chained!)
	// gold 카드가 3개 초과이면 round를 1 증가시키고 goldCardCount를 0으로 되돌림
	
	useEffect(() => {  
		if (round > 5) {  
			setIsGameOver(true);  
		}  
	}, [round]);
	// 라운드의 진행에 따라 게임 종료 여부를 결정하는 로직
	// round state가 변경될 때마다 실행 (= 위의 useEffect와 chained!)
	// 라운드가 5를 초과하면 isGameOver를 true로 설정
	
	useEffect(() => {  
		alert('Good game!');  
	}, [isGameOver]);  
	// 게임이 종료되었을 때 사용자에게 피드백을 주기 위한 로직
	// isGameOver가 true로 변경될 때 실행 (= 위의 useEffect와 chained!)
	// isGameOver가 true이면 "Good game!" alert를 띄움

	// 다음 카드를 냈을 때의 이벤트를 핸들링
	function handlePlaceCard(nextCard) {  
		if (isGameOver) {  // isGameOver가 true이면 게임이 끝났다는 alert
			throw Error('Game already ended.');  
		} else {  // isGameOver가 false이면 카드를 다음 카드로 설정, 처음의 useEffect 로직이 실행됨
			setCard(nextCard);  
		}  
	} 	
	// ...
}



// 2. 리팩토링 후 : 렌더링 중 가능한 것을 계산 및 이벤트 핸들러에서 state 설정
function Game() {  
	const [card, setCard] = useState(null);  
	const [goldCardCount, setGoldCardCount] = useState(0);  
	const [round, setRound] = useState(1);  

	// isGameOver를 state가 아닌 연산으로 대체
	const isGameOver = round > 5;

	function handlePlaceCard(nextCard) {  
		if (isGameOver) {  // isGameOver가 true이면 게임이 끝났다는 alert
			throw Error('Game already ended.');  
		}
		
		// 카드를 냄: 이벤트에 해당, 이벤트에서 state를 계산
		setCard(nextCard);  
		if (nextCard.gold) {  // 낸 카드가 gold이고
			if (goldCardCount <= 3) {  // 낸 gold 카드의 수가 3 이하일 경우
				setGoldCardCount(goldCardCount + 1);  // goldCardCount를 1 증가
			} else {  // 낸 gold 카드의 수가 3 초과인 경우
				setGoldCardCount(0);  // goldCardCount를 0으로 되돌리고
				setRound(round + 1);  // round를 1 증가
			if (round === 5) {  // round가 5이면
				alert('Good game!');  // 피드백 alert을 띄움
			}  
		}
	}
}
// ...


```
#### 🔅 app 초기화 - 최상위 variable 이용
```javascript
let didInit = false;  

function App() {  
	useEffect(() => {  
		if (!didInit) {  
			didInit = true;
			loadDataFromLocalStorage();  
			checkAuthToken();  
		}  
	}, []);  
	// ...  
}
```
#### 🔅 app 초기화 - app 렌더링 전 1회만 실행
```javascript
if (typeof window !== 'undefined') { // window 객체가 존재하면? (우리가 브라우저 환경에 있으면)
	checkAuthToken();  
	loadDataFromLocalStorage();  
}  

function App() {  
	// ...  
}
```
#### 🔅 부모에게서 setter를 내려받아 부모에게 state 변경을 알리기
```javascript
function Toggle({ onChange }) {  
	const [isOn, setIsOn] = useState(false);  
	function updateToggle(nextIsOn) { // 이벤트가 발생하면 아래의 모든 업데이트를 한꺼번에 수행하고
		setIsOn(nextIsOn);  
		onChange(nextIsOn);  
	}  
	  
	function handleClick() {  // 해당 수행을 핸들러에 연결
		updateToggle(!isOn);  
	}  
	
	function handleDragEnd(e) {  // 해당 수행을 핸들러에 연결
		if (isCloserToRightEdge(e)) {  
			updateToggle(true);  
		} else {  
			updateToggle(false);  
		}  
	}  
	// ...  
}
```
#### 🔅 State 끌어올리기로 해결
```javascript
function Toggle({ isOn, onChange }) {  // isOn을 부모에게서 받아와 처리, 부모의 state로부터 동작을 제어
	function handleClick() {  
		onChange(!isOn);  
	}  
	function handleDragEnd(e) {  
		if (isCloserToRightEdge(e)) {  
			onChange(true);  
		} else {  
			onChange(false);  
		}  
	}  
	// ...  
}
```
#### 🔅 React의 data flow를 벗어난 경우 (자식에서 부모에게 데이터 전달)
```javascript
function Parent() {  
	const [data, setData] = useState(null);  
	// ...  
	return <Child onFetched={setData} />;  
}  
	
function Child({ onFetched }) {  
	const data = useSomeAPI();    
	
	useEffect(() => {  // 잘못된 사용! 자식이 부모에게 데이터를 전달하고 있음.
		if (data) {  
			onFetched(data);  
		}  
	}, [onFetched, data]);  
	// ...  

}
```
#### 🔅 useSyncExternalStore 훅의 사용 예시
```javascript
function subscribe(callback) {  
	window.addEventListener('online', callback);  
	window.addEventListener('offline', callback);  
	return () => {  
		window.removeEventListener('online', callback);  
		window.removeEventListener('offline', callback);  
	};  
}  

function useOnlineStatus() {  
	return useSyncExternalStore(  
		subscribe, // 외부 소스를 구독하는 함수, 리스너를 등록하는 방법이 구현되어있어야 하고, 변경 사항을 구독 해제하는 함수를 반환해야 함. 또한 변경 사항이 발생할 때마다 리액트 컴포넌트를 다시 렌더링해야 함
		() => navigator.onLine, // 외부 클라이언트의 현재값을 가져오기 (getSnapshot) - 리액트는 이 값을 컴포넌트의 상태로 사용
		() => true // 서버측 외부 소스의 현재 상태를 가져오기 (getServerSnapshot) - (선택적) getSnapshot과 동일한 역할을 하지만 서버에서 호출됨, 서버에서 데이터를 사전 로드할 필요가 있을 때 유용
	);  
}  

function ChatIndicator() {  
const isOnline = useOnlineStatus();  
// ...  
}
```
#### 🔅 fetching 로직의 커스텀 훅 제작 예시
```javascript
function SearchResults({ query }) {  
	const [page, setPage] = useState(1);  
	const params = new URLSearchParams({ query, page });  
	const results = useData(`/api/search?${params}`);  
	
	function handleNextPageClick() {  
		setPage(page + 1);  
	}  
	// ...  
}  

function useData(url) {  // 오류 처리 등의 다른 로직을 추가할 때도 이런 식으로 hook을 만들어 빼는 것이 훨씬 편리
	const [data, setData] = useState(null);  
	useEffect(() => {  
		let ignore = false;  
		fetch(url)  
			.then(response => response.json())  
			.then(json => {  
				if (!ignore) {  
					setData(json);  
				}  
		});  
		return () => {  
			ignore = true;  
		};  
	}, [url]);  
	return data;  
}
```