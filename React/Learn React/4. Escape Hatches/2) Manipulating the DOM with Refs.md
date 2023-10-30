>[!Learning Goals]
>- How to access a DOM node managed by React with the ref attribute
>- How the ref JSX attribute relates to the useRef Hook
>- How to access another component's DOM node
>- In which cases it's safe to modify the DOM managed by React

- 일반적으로 React에서 가상 DOM과 실질 DOM이 일치되도록 렌더링하기 때문에 유저가 실질 DOM을 조작할 일이 많지는 않다.
- 하지만 일부 다음과 같은 경우 DOM 엘리먼트에 대한 접근이 필요할 수 있다. 이 때 DOM 노드에 대한 ref를 통해 접근한다.
	- focusing a node
	- scroll to a node
	- measure a node's size or position ...

## 1. 기본 - Node에 대한 ref 가져오기
- 일반적으로 다음과 같은 step을 거쳐 node에 대한 Ref를 가져온다.
	1. useRef hook import
		- `import { useRef } from "react"`
	2. 컴포넌트 내부에서 ref 선언
		- `const myRef = useRef(null)`
	3. DOM 노드를 가져오고 싶은 JSX 태그에 ref 속성으로 선언한 ref를 넘겨주기
		- `<div ref={myRef}>`
- 동작 방식
	- useRef는 `{ current: initialValue }`를 반환하고, 유저가 넘겨준 초기값을 initialValue로 지정한다. 
	- React는 ref가 걸린 엘리먼트에 대응하는 실제 DOM 노드를 생성하면서 myRef.current에 실제 DOM 노드에 대한 참조를 저장한다. (예시에선 `<div>`) 
	- 이렇게 저장된 노드는, 이벤트 핸들러에서 myRef.current를 통해 접근할 수 있으며 이 노드에 built-in browser API를 사용할 수 있다.
- 예제
	- ![[2) Manipulating the DOM with Refs#🔅 텍스트 Input에 초점 맞추기]]
## 2. 하나 이상의 ref 조작하기
### 1) ref가 될 대상을 정확히 알고 있는 경우
- 1의 일반적 케이스와 마찬가지이지만 여러 개의 ref를 사용한다.
	- 대응하는 JSX 태그에 각각 ref 속성을 걸어주고, 필요한 시점에 handler를 통해 원하는 행동을 연결한다.
### 2) ref callback : ref를 걸 대상이 확정되지 않은 경우
- useRef 역시 hook이기 때문에 언제나 컴포넌트의 최상단에서 호출되어야 한다.
	- ref 걸 대상의 목록이 리스트로 관리될 경우 다음과 같은 코드를 작성할 수 있지만, 유효하지 않다. 컴포넌트의 최상단이 아닌 곳에서 hook이 사용되었기 때문.
		- ![[2) Manipulating the DOM with Refs#🔅 useRef의 잘못된 용례]]
- ref를 걸 대상이 여럿이면서 확정되지 않은 경우 부모 엘리먼트에 대한 단일 ref를 가져온 뒤 querySelectAll과 같은 DOM 조작 메소드를 사용하여 하위 노드를 찾을 수 있지만, DOM 구조가 변경되는 경우 매번 코드를 수정해야 한다. 이에 따라 일반적으로는 ==ref 속성에 ref callback 함수를 전달== 하는 방식을 사용한다.
- #### ref callback의 동작
	- 컴포넌트 마운트 시 : DOM 요소가 화면에 처음으로 그려질 때, React는 DOM 요소의 ref를 인자로 전달하여 ref callback을 호출
	- 컴포넌트 언마운트 시 : DOM 요소가 화면에서 사라질 때, React는 null을 인자로 전달하여 ref callback을 재호출
	- 컴포넌트 업데이트 시 : DOM 요소의 타입이 변경되면 (ex. `<button>` → `<div>` ) React는 먼저 null을 인자로 전달하여 ref callback을 호출한 뒤, 새로운 요소의 참조(`<div>`)를 다시 ref callback의 인자로 전달하여 호출
- ![[2) Manipulating the DOM with Refs#🔅 ref callback을 이용한 다수의 비확정 ref 핸들링 예시]]
## 3. forwardRef : 브라우저 엘리먼트가 아닌 custom component의 DOM 노드에 접근하기
- React에서는 기본적으로 한 Component에서 다른 component의 DOM 노드에 ref로 접근하는 것을 허용하지 X
	- ref = escape hatch, 곧 '외부 요소와 연결' 하기 위한 수단이지만 다른 컴포넌트는 '외부 요소'가 아님
	- 따라서 다음 예시와 같이 custom component에 ref를 할당하더라도 React는 component의 브라우저 엘리먼트에 ref를 연결하지 않고 null을 반환함
	- ![[2) Manipulating the DOM with Refs#🔅 Custom component에 ref 연결 input focusing - 잘못된 예시]]
- 다른 component의 DOM node에 접근하려면 forwardRef를 사용
	- ref의 대상이 될 custom component를 만들 때 forwardRef를 사용하여 만들어주기
	- forwardRef에서 props와 ref 인수를 JSX 태그로 넘기고, ref를 어떤 태그와 연결시켜줄지 정확하게 명시한다.
	- ![[2) Manipulating the DOM with Refs#🔅 forwardRef의 사용 예시 input focusing]]
- ### 🔥 ref로 인한 '노출(exposure)'에 유의하기
	- ![[노출 (exposure)]]
	- 위 input focusing 예시의 경우 다음과 같이 노출되어 있음
		- `MyInput` 컴포넌트는 `forwardRef`를 사용하여 부모 컴포넌트에서 전달된 `ref`를 내부의 `input` 엘리먼트에 직접 연결
		- 그 결과, 부모 컴포넌트인 `Form`에서 `inputRef.current`를 통해 내부 `input` 엘리먼트에 직접 접근 가능
		- 이로 인해 `handleClick` 함수에서 `inputRef.current.focus()`를 호출 가능
		- 즉, `MyInput` 컴포넌트는 내부의 `input` 엘리먼트를 "노출"하여 부모 컴포넌트가 이를 직접 조작할 수 있게 해주고 있음
		- 이렇게 ==노출되어있을 경우 ref에 접근하여 다른 작업도 가능함 → 위험 요소!==
			- ex) CSS 변경 : `inputRef.current.style.backgroundColor = 'yellow'`
	- 일반적으로 저수준 컴포넌트에서는 노출을 허용하여 유연성을 높이지만, 고수준 컴포넌트에서는 노출을 피함
		- ![[저수준 컴포넌트와 고수준 컴포넌트]]
	- ==useImperativeHandle : 노출하고 싶은 기능만 골라 노출하기==
		- useImperativeHandle ?
			- 부모 컴포넌트가 자식 컴포넌트의 인스턴스에 접근하고, 그 인스턴스의 특정 메서드나 값에 접근할 수 있게 해주는 Hook
			- 자식 컴포넌트의 내부 구현을 부모 컴포넌트로부터 숨기면서, 선택적으로 일부 기능만 노출하는데 사용 (캡슐화 유지!)
		- 사용법
			- 3 parameters
				- `ref`: 부모 컴포넌트로부터 받은 `ref` 객체
				- `create`: 함수, 이 함수 내에서 노출하고 싶은 값이나 메서드를 반환
				- `deps`: 선택적 파라미터. `create` 함수 안에서 사용되는 외부 값의 리스트로 이 값들이 변경될 때마다 `create` 함수가 다시 실행
			- ref를 걸 component에서 useImperativeHandle을 사용하여 노출할 기능을 명시적으로 표시
			- 이렇게 처리할 시 ref.current에 노출할 기능만 보이게 됨
			- ![[2) Manipulating the DOM with Refs#🔅 useImperativeHandle의 사용 예시 input focusing 변형]]
				- useImperativeHandle은 inputRef.current에 { focus: \[Function] } 형태의 사용자 정의 객체를 할당
				- 따라서 inputRef.current는 DOM 노드가 아니게 됨..!
## 4. React가 ref를 업데이트하는 시점
- React의 모든 업데이트는 다음의 두 단계로 나뉨
	- 렌더링(Rendering) - React는 무엇이 화면에 표시되어야 하는지 알아내기 위해 컴포넌트를 불러옴
	- 커밋(Commit) - React가 DOM에 변경 사항을 적용
- ref의 경우 렌더링과 관련 없는 정보, 따라서 커밋 시점에 ref.current가 설정됨
	- DOM이 업데이트 되기 전에는 ref.current = null
	- DOM이 업데이트된 직후 해당 DOM 노드로 ref.current가 설정됨
	- cf) state
		- state 업데이트 작업은 큐에 등록됨 = DOM에 즉시 (동기적으로) 업데이트 되지 않음
		- flushSync를 이용하면, flushSync 코드로 감싼 코드가 실행된 직후 React가 DOM을 업데이트
			- ![[2) Manipulating the DOM with Refs#🔅 flushSync 사용 예시 마지막 목록으로 스크롤하기]]
- 일반적으로는 이벤트 핸들러에서 ref에 접근하지만, 그 작업을 수행할 특정 이벤트가 없다면 Effect (useEffect)를 이용 가능


## 5. ref 사용시 지켜야 할 것들
- ref = escape hatch임을 항상 기억하기 - 'React 바깥으로 나가야' 할 때만 사용
	- focusing, scrolling, calling browser APIs
- Ref를 사용해 DOM을 조작하고자 할 때는 언제나 주의할 것
	- React가 업데이트해야 하는 DOM의 노드를 변경할 시 (요소 수정, 자식 추가 및 제거 등) ui 렌더링에 문제가 생기거나 충돌이 일어날 수 있음

## ------ example list ------
#### 🔅 텍스트 Input에 초점 맞추기
```javascript
import { useRef } from 'react'; // 1. hook import

export default function Form() {
  const inputRef = useRef(null); // 2. 컴포넌트 내부에서 ref 선언

  function handleClick() { // 4. 이벤트 핸들러 안에서 선언한 ref 사용하기
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} /> // 3. DOM 노드를 가져오고 싶은 JSX 태그에 ref 속성으로 선언된 inputRef 넘기기
      <button onClick={handleClick}> // 5. 이벤트 핸들러 연결해주기
        Focus the input
      </button>
    </>
  );
}
```
#### 🔅 useRef의 잘못된 용례
```javascript
<ul>
	{ items.map(item => {
		const ref = useRef(null)
		return <li ref={ ref } />
	})}
</ul>
```
#### 🔅 ref callback을 이용한 다수의 비확정 ref 핸들링 예시
```javascript
import { useRef } from 'react';

export default function CatFriends() {
  const itemsRef = useRef(null);

  function scrollToId(itemId) {
    const map = getMap();
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function getMap() { // 생성한 Ref에 map 자료 구조를 할당해주어 key-value쌍으로 관리될 수 있게 해줌
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>
          Tom
        </button>
        <button onClick={() => scrollToId(5)}>
          Maru
        </button>
        <button onClick={() => scrollToId(9)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          {catList.map(cat => ( 
            <li // map에 <li> 태그에 대한 참조를 value로 추가
              key={cat.id}
              /*
              ** ref callback에 의해 <li>가 처음으로 DOM에 마운트될 때는
              ** node로 대상 태그인 <li>가 넘어가게 되며,
		      ** 언마운트 될 시 null이 넘어가게 되는 구조
		      ** 현재 ref callback은 무명 함수
              */
              ref={(node) => {
                const map = getMap(); 
                if (node) { // map에 저장된 <li> 노드를 cat.id를 key로 하여 저장 또는 삭제
                  map.set(cat.id, node);
                } else {
                  map.delete(cat.id);
                }
              }}
            >
              <img
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: 'https://placekitten.com/250/200?image=' + i
  });
}
```
#### 🔅 Custom component에 ref 연결 : input focusing - 잘못된 예시
```javascript
import { useRef } from 'react';

function MyInput(props) { // 1. custom component인 MyInput을 선언, MyInput은 내부에 기본 태그인 <input>을 가짐
  return <input {...props} />; 
}

export default function MyForm() {
  const inputRef = useRef(null); // 2. useRef 훅을 이용해 inputRef에 { current: initialValue } 할당

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} /> // 3. 작성한 inputRef가 MyInput을 ref로 가지도록 설정 → 그러나 의도대로 동작하지 않고 null 반환
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
```
#### 🔅 forwardRef의 사용 예시 : input focusing
```javascript
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {// MyInput 컴포넌트를 만들 때 forwardRef를 이용하여 아래 태그로 ref를 넘겨줄 것임을 표시 (=어떤 태그와 ref를 바인딩할 것인지 결정)
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```
#### 🔅 useImperativeHandle의 사용 예시 : input focusing 변형
```javascript
import {
  forwardRef, 
  useRef, 
  useImperativeHandle
} from 'react';

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // inputRef.current only exposes focus method
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```
#### 🔅 flushSync 사용 예시 : 마지막 목록으로 스크롤하기
```javascript
import { useState, useRef } from 'react';
import { flushSync } from 'react-dom';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    flushSync(() => { // flushSync를 사용하지 않으면, state 업데이트 내용이 코드와 동기적으로 DOM에 반영되지 않기 때문에 스크롤이 언제나 마지막-1 위치로 감
      setText('');
      setTodos([ ...todos, newTodo]);      
    }); // flushSync로 감싼 코드블록이 실행된 직후 DOM이 업데이트됨
    listRef.current.lastChild.scrollIntoView({ // 따라서 이 라인의 scroll은 정상적으로 LastChild로 향하게 됨
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```