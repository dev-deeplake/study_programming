>[!Learning Goals]
>- What “prop drilling” is
>- How to replace repetitive prop passing with context
>- Common use cases for context
>- Common alternatives to context

- 일반적으로는 부모에서 자식에게 props를 넘겨 데이터를 전달하지만, 다음의 경우 이러한 방식이 장황하고 불편해질 수 있음
	- 중간에 여러 컴포넌트를 거쳐야하거나
	- 앱의 다양한 컴포넌트에서 동일한 정보를 필요로 하는 경우
- Context를 사용하면 props를 명시적으로 전달하지 않으면서, 깊이와 상관 없이 데이터 전달 가능
## 1. The Problem with Passing Props
- Prop Drilling
	- 상위 컴포넌트에서 하위 컴포넌트로 연속적으로 Data를 전달하는 과정
	-  ![|900](images/react_managing-state_passing-data-with-context_01.png)
	- 왜 문제가 될까?
		- 코드의 복잡성 증가 (+지저분...)
			- 공통된 조상이 멀리 있을 경우, 그만큼 '중간 통로' 역할을 하는 컴포넌트가 많아지므로 가독성이 떨어지고 유지보수가 어려워짐
		- 재사용성 감소
			- props를 전달하는 로직이 포함되기 때문에 다른 곳에서 컴포넌트를 재사용하기 어려워질 수 있음
		- 불필요한 리렌더링
			- 중간 component들이 실제로 해당 props를 사용하지 않음에도 불구하고 props 변경으로 인한 리렌더링이 일어날 수 있음
	- context는 prop을 전달하지 않고 데이터가 필요한 컴포넌트에서 해당 데이터를 사용할 수 있게 해준다!
		- ![|900](images/react_managing-state_passing-data-with-context_02.png)
## 2. Context의 특성
- context는 사용하고 싶은 곳에서만 useContext를 이용해 사용할 수 있다 (=중간 component들을 뛰어넘을 수 있다)
- 'adapting to their surroundings' : 위치에 따라 자동으로 context 값을 증가하여 적용하거나 다른 context를 적용할 수 있다.
- 각 context는 다른 context와 완전히 분리되어있기 때문에, 다른 context를 사용하려면 children을 다른 context provider로 감싸주어야 함
## 3. Context 활용
- 현재 색상 테마, 로그인한 사용자 정보를 전달하는 등의 경우
- Step
	- context 생성
		- 'createContext' import
		- 'createContext'를 사용해 기본값과 함께 전달
			- createContext는 context 객체를 반환 - Provider와 Consumer 컴포넌트로 구성
		- ![[6) Passing Data Deeply with Context#🔅 context 생성 예시]]
	- context 사용
		- \[option] 'useContext' import + context를 분리해 선언했다면 context를 사용할 component에서 import
		- useContext hook을 사용해 context 생성시 지정한 값을 읽어오기
			- hook이므로 기존의 constraint는 동일: 컴포넌트의 최상단에서만 호출 가능
		- ![[6) Passing Data Deeply with Context#🔅 context 사용 예시]]
	- context 제공
		- 정보를 내려보낼 곳에서 context를 제공
			- component는 상위 UI tree상에서 가장 가까운 context를 사용하므로, 이를 고려하여 정보를 내려보낼 곳을 정함
			- ![[6) Passing Data Deeply with Context#🔅 Section 컴포넌트 하위의 컴포넌트들에게 context를 제공하는 경우]]
		- 자식 component(useContext를 호출하여 LevelContext를 사용하겠다고 한)에서는 필요한 경우 가장 가까운 LevelContext 값을 요청하여 props를 대신함
## 4. 동일한 component에서의 context 사용 및 제공
- 예시의 'Section' 컴포넌트에서 명시적으로 level을 지정해 넘겨줄 수도 있지만, 상위 'Section' 컴포넌트의 context를 읽어와 자동적으로 level을 계산하는 등 동일 component에서의 context를 활용할 수 있다.
- ![[6) Passing Data Deeply with Context#🔅 level 수동 지정 예시]]
- ![[6) Passing Data Deeply with Context#🔅 동일 component의 context를 사용하여 level 지정]]
## 5. Context 사용시의 주의사항
- 남용되지 않도록 하기!
	- 우선은 props 전달로 시작 - 특히 명시적인 data 전달이 중요한 경우.
	- ==컴포넌트를 충분히 분리하기==
		- "Extract components and pass JSX as children to them. If you pass some data through many layers of intermediate components that don’t use that data (and only pass it further down), this often means that you forgot to extract some components along the way."
		- ![[6) Passing Data Deeply with Context#🔅 컴포넌트의 분리 예시 - Before]]
		- ![[6) Passing Data Deeply with Context#🔅 컴포넌트의 분리 예시 - After]]
## 6. 어디에 쓸 수 있을까?
- 테마 지정 (ex. 다크 모드)
- 현재 계정 정보 전달
- [[라우팅 (Routing)]]
	- ![[라우팅 (Routing)#3. 클라이언트 사이드 라우팅(Client-side Routing)]]
	- "다른 부분은 그대로 유지하면서 해당 섹션만 변경" → path 값에 따라 다른 컴포넌트로 switching
- State 관리
	- 특히 멀리 떨어진 다수의 컴포넌트에서 state를 사용하는 경우
## ------ example list ------
#### 🔅 context 생성 예시
```javascript
import { createContext } from "react"
export const LevelContext = createContext(1) // Provider와 Consumer 컴포넌트를 갖는 Context 객체
```
#### 🔅 context 사용 예시
```javascript
import { useContext } from "react" // hook을 사용하기 위한 import
import { LevelContext } from "./LevelContext.js" // 생성한 LevelContext가 분리되어있는 경우 위와 같이 import

export default function Heading({ children }) {  
	const level = useContext(LevelContext) // useContext를 이용해 만들어놓은 context를 가져오기
	switch(level) {
		case 1 : 
			// ...
	}
}
```
#### 🔅 Section 컴포넌트 하위의 컴포넌트들에게 context를 제공하는 경우
```javascript
import { LevelContext } from "./LevelContext.js"

export default function Section({ level, children }) {
	return (
		<section className="section">
			<LevelContext.Provider value={level}> // 어디에선가 LevelContext를 요청하면 이 value를 전달함 (LevelContext context 객체 중 Provider를 이용)
				{children}
			</LevelContext.Provider>
		</section>
	)
}
```
#### 🔅 level 수동 지정 예시
```javascript
export default function Page() {
	return (
		<Section level={1}>
			...
			<Section level={2}>
				...
				<Sectipn level={3}>
					...
	)
}
```
#### 🔅 동일 component의 context를 사용하여 level 지정
```javascript
import { useContext } from 'react';  
import { LevelContext } from './LevelContext.js';  

export default function Section({ children }) {  
	const level = useContext(LevelContext)
	return (  
		<section className="section">
			<LevelContext.Provider value={level + 1}>  
				{children}  
			</LevelContext.Provider>  
		</section>  
	);  
}
```
#### 🔅 컴포넌트의 분리 예시 - Before
```javascript
// Layout 컴포넌트는 posts를 직접 사용하지 않지만, posts를 props로 전달 받아서
function Layout({ posts }) {
  return (
    <div className="layout">
	    ...
      {/* posts를 실제로 사용하는 Posts 컴포넌트에 전달 */}
      <Posts posts={posts} />
    </div>
  );
}

// Posts 컴포넌트는 posts를 직접 사용
function Posts({ posts }) {
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

// 사용 예:
const myPosts = [
	{ id: 1, title: 'Post 1' },
	{ id: 2, title: 'Post 2' }
];

<Layout posts={myPosts} />

```
#### 🔅 컴포넌트의 분리 예시 - After
```javascript
// Layout 컴포넌트는 더 이상 posts를 직접 받지 않고, children prop을 받아 사용
function Layout({ children }) {
  return (
    <div className="layout">
      {children}
    </div>
  );
}

// Posts 컴포넌트는 이전과 동일
function Posts({ posts }) {
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

// 사용 예:
const myPosts = [
	{ id: 1, title: 'Post 1' },
	{ id: 2, title: 'Post 2' }
];

<Layout>
  <Posts posts={myPosts} />
</Layout>

```