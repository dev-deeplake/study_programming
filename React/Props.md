## 컨셉 및 특징
- 리액트 컴포넌트 사이에 데이터를 주고 받는 수단
- 자바스크립트의 어떤 데이터 타입이든 props로 사용될 수 있다.
- 위계적으로는 부모 컴포넌트에서 자식 컴포넌트로, 구성 요소적으로는 JSX 태그로 정보를 전달해주는 역할을 한다.
- props는 존재하는 컴포넌트에서 독립적으로 작용하기 때문에 부모 컴포넌트와 자식 컴포넌트 간의 의존도(결합도)를 줄인다.
	- 부모 컴포넌트에서 구조가 변경되더라도 자식 컴포넌트로 넘겨주는 props에 변화가 없으면 자식 컴포넌트는 영향을 받지 않는다.
	- 결국에는 컴포넌트의 위계보다 각자의 로직 및 표현이 중요해지므로 컴포넌트의 재사용성이 강화된다.
- Props는 불변하며, 다른 값을 렌더링해야 하는 경우 부모 컴포넌트에 다른 props를 요청하여 새 값을 받는다.
	- 유저에게 input으로 받은 값 또는 일반적으로 외부에서 변수로 사용해야 하는 유형의 값은 'state'를 통해 관리한다.
## How to pass props
- HTML 태그의 경우 HTML 표준에 따라 정해진 속성을 가지고, React에서는 prop으로 해당 속성을 넘길 수 있다.
	- HTML 태그에 임의의 props를 정의해 넘겨준다면 브라우저는 해당 속성을 알지 못하기 때문에 제대로 사용할 수 없다.
	- 참고 : HTML Standard - [[DOM interface]]
	- ![[images/CleanShot 2023-09-15 at 21.20.54@2x.png]]
- Custom Component의 경우 props가 미리 정의되어있지 않고, 어떤 것이든 props로 넘길 수 있다.
	- Parent Component : passing props to the child component
	- Child Component : reading props in itself
```javascript
export default function ExampleParent() {
	return (
	// 자식인 ExampleChildren component에게 다음과 같이 prop을 넘긴다.
		<ExampleChildren
			passingObj={{
				passingType: "object",
				passingValue: {
					some, values, toPass
				}
			}}
			passingStr="a str passing for the child comp"
			passingNum={200}
		/>
	)
}

function ExampleChildren({ passingObj, passingStr, passingNum }) {
 // 자식 component 안에서는, 위에서 명시한 passingObj, passingStr, passingNum을 변수처럼 자유롭게 사용할 수 있다.
}
```
- 만약 부모 컴포넌트가 받은 props가 부모 컴포넌트 자체에서 사용되지 않고 자식에게 그대로 넘어가는 경우, props를 모두 명시해서 써주기보다는 전개(spread)구문을 사용하는 것이 효율적이다.
```javascript
function ExampleComponent({ name, age, height }) {
	return (
		<ExampleChild
			name={name}
			age={age}
			height={height}
		/>
	)
}

function ExampleComponent(props) { //typeof(props) : object
	return (
		<ExampleChild {...props} />
	)
}
```
- JSX 태그 사이에 콘텐츠를 넣을 경우 해당 내용은 children prop으로 간주된다.
### 의문점 ✅
- "React component functions accept a single argument, <u>a props object</u>"
	- [[JavaScript / 구조분해 (destructuring)|리액트 컴포넌트도 결국 함수이고, argument로 단 하나, 'props'라는 '객체'를 받는 것이라면, 왜 key: value 형태가 아닌 key = value 형태로 초기화를 하는 것일까?]]
```javascript
// 1. 기본값이 지정되지 않은 경우
function ExampleComponent({ param01, param02 }) {
 ...
}

// 2. 기본값이 지정된 경우
function AnotherComponent({ num = 1, tf = false}) {
 ...
}
```
- 일반적으로 리액트 컴포넌트 함수에서 패러미터를 받는 방식은 1과 같다. 객체의 특성을 생각해볼 때 해당 방식과 동일하게 표기될 수 있으려면 key와 value가 동일해야 한다.
- 또한 2와 같이 초기화를 하려 할 때에도, 객체의 경우 num:1 , tf: false의 형태를 취하는 것이 올바르다.
- 만약 위 두 가지 경우를 빗겨가면서 1과 같이 표기할 수 있으려면 { param01, param02 }는 구조분해할당 표현의 일부여야 한다.
- 구조분해할당은 '표현식'이다(["The **destructuring assignment** syntax is a JavaScript <u>expression</u>..."](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)). 따라서 일반적으로는 할당되는 값이 있는 형태로 다음과 같이 사용한다.
```javascript
const [a, b] = [10, 20]
const {c, d} = {true} // 이 경우 d는 undefined

let {e, f} // Uncaught SyntaxError: Missing initializer in destructuring declaration
```
- 그렇지만 우리의 경우는 함수가 '선언'되는 순간의 표현이다. 함수의 매개변수는 함수가 호출될 때 값을 받기 때문에, 실질적으로 함수가 호출되기 전까지 구조 분해 문법을 사용했더라도 선언만으로는 실제 값이 분해되지 않는다. 즉, 함수의 매개변수에서의 구조 분해는 함수 호출 시점에서 값이 전달되기 때문에 함수 선언 시에는 ExampleComponent({ param01, param02 })와 같은 형태로 작성해도 에러가 발생하지 않는다.