> [!Learning Goals]
> - How to return different JSX depending on a condition
> 	- In React, you control branching logic with JavaScript.
> - How to conditionally include or exclude a piece of JSX
> 	- You can return a JSX expression conditionally with an `if` statement.
> 	- You can conditionally save some JSX to a variable and then include it inside other JSX by using the curly braces
> - Common conditional syntax shortcuts you’ll encounter in React codebases
> 	- In JSX, `{cond ? <A /> : <B />}` means _“if `cond`, render `<A />`, otherwise `<B />`”_.
> 	- In JSX, `{cond && <A />}` means _“if `cond`, render `<A />`, otherwise nothing”_.
> 	- The shortcuts are common, but you don’t have to use them if you prefer plain `if`.

## 요약
- React에서의 제어 흐름은 JavaScript로 처리된다.
	- if / else를 사용해 문으로 표현
		- [[#🔅 변수에 조건부로 JSX 할당하기 예시|변수와 함께 사용하여 유연한 표현이 가능]]
	- 식 표현
		- [[JavaScript / 삼항 연산자 (Conditional (ternary) operator)]] '? :' 
			- 삼항 연산자는 간단한 조건에서의 렌더링 표현에 적합하지만, nested 마크업이 너무 많아질 경우 컴포넌트가 지저분해질 수 있으므로 적당히 사용하기
				- [[#🔅 '**자식 컴포넌트를 추출하여 정리하기'의 예시**]]
		- [[JavaScript / 논리 and 연산자 (Logical AND operator)]] '&&'
			- 🌟 논리 and 연산자를 사용할 시, 좌항에 숫자를 사용하지 말기!
				- 0을 false로 간주하여 렌더링하지 않아야 하지만 0을 출력
				- !!를 통해 boolean으로 변경하거나 > 0을 통해 조건으로 숫자값을 제어해주기
	- [[#⚠ 각 제어 예제가 완전히 동일할까? 라는 질문이 왜 나온 것인지 이해하기]]
- ### [[Tips / DRY (Don't Repeat Yourself)]]
- 아무것도 렌더링하고 싶지 않을 때는 null을 사용해도 되지만, null이 리턴된다는 것을 예상하지 못하는 경우도 있기 때문에 일반적인 방법은 아니다.
	- 한 컴포넌트의 렌더링 여부에 조건을 걸고 싶다면, 해당 컴포넌트 자체에서 제어하는 것이 아니라 <u>렌더링 되지 않는 경우를 다른 사람이 충분히 예측할 수 있도록 부모 컴포넌트의 JSX 내에서 조건적으로 포함하거나 제외</u>한다.
	- [[#🔅 컴포넌트 자체 렌더링 제어 vs 부모 컴포넌트 JSX 내 조건 렌더링 비교]]

### 🔅 '**자식 컴포넌트를 추출하여 정리하기'의 예시**
```javascript
// React 문서 예제 원본
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? (
        <del>
          {name + ' ✔'}
        </del>
      ) : (
        name
      )}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```
```javascript
// 예제의 'Item' 컴포넌트에서 이름을 표시하는 부분을 다른 자식 컴포넌트로 분리하기
function ItemName({ name, isPacked }) {
  if (isPacked) {
    return <del>{name + ' ✔'}</del>;
  } else {
    return <span>{name}</span>;
  }
}

function Item({ name, isPacked }) {
  return (
    <li className="item">
      <ItemName name={name} isPacked={isPacked} />
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

### 🔅 컴포넌트 자체 렌더링 제어 vs 부모 컴포넌트 JSX 내 조건 렌더링 비교
```javascript
// React Document에서 제시된 null 사용 예시 (컴포넌트 자체에서 렌더링 여부 제어 - by 'isPacked' prop)
function Item({ name, isPacked }) {
  if (isPacked) {
    return null;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}


// 부모 컴포넌트 JSX에서 조건적 렌더링 예시
function Item({ name }) { // isPacked가 사라짐
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        {false ? <Item name="Space suit" /> : null}
        //{ false && <Item name="Space suit" /> }
        {false ? <Item name="Helmet with a golden leaf" /> : null}
        //{ false && <Item name="Helmet with a golden leaf" /> }
        {false ? <Item name="Photo of Tam" /> : null}
        //{ false && <Item name="Photo of Tam" /> }
      </ul>
    </section>
  );
}

```

### ⚠ 각 제어 예제가 완전히 동일할까? 라는 질문이 왜 나온 것인지 이해하기
```javascript
// if - else를 통한 제어
if (isPacked) {
	return <li className="item">{name} ✔</li>;  
}  
return <li className="item">{name}</li>;
}

// 삼항 연산자를 통한 제어
...
return (  
	<li className="item">  
		{isPacked ? name + ' ✔' : name} 
	</li>  
)
```
 - if-else 방식에서는 isPacked에 따라 두 다른 li 요소 중 하나를 반환함. 즉, 두 개의 다른 JSX 요소가 정의되고, 조건에 따라 해당하는 요소가 하나만 반환됨.
 - 삼항 연산자 방식에서는 항상 같은 li 요소가 반환되지만 내부의 내용이 isPacked에 따라 달라진다는 차이.
 - 객체 지향 프로그래밍의 관점에서, if-else 방식은 다른 조건에 따라 다른 인스턴스가 생성되는 것처럼 여겨질 수 있음
	 - JSX : React.createElement 함수를 통해 만들어진 '객체'
 - 객체 지향 프로그래밍에서 객체의 인스턴스는 메모리 상에서 공간을 차지하는 실제 객체를 의미함
	 - JSX 요소는 DOM 노드가 아님
		 - UI의 특정 부분을 묘사하는 가벼운 blueprint 또는 설명
		 - React는 JSX로부터 (가상 DOM에) UI 트리를 만들고, 추후 해당 트리와 일치하도록 브라우저 DOM 엘리먼트를 업데이트
		 - 따라서 해당 제어 로직이 쓰였다 하더라도, 쓰인 시점에서는 '설명'에 불과하므로 실제 인스턴스가 리턴되는 것이 아니고, 이미 결정된 제어 결과에 따라 가상 DOM이 업데이트 된 후 실제 DOM이 업데이트 될 것이기 때문에 어떤 경우에도 인스턴스가 2개 생성되지 않음.
- 따라서 '두 예제가 (인스턴스 생성 측면에서) 완전히 동등하다'는 결론!
- React Component의 경우에도 동일한 렌더링 흐름(선언 및 사용 → 가상 DOM 적재 → 브라우저 DOM 업데이트)을 가지기 때문에 인스턴스는 한 번만 사용되지 않을까...

### 🔅 변수에 조건부로 JSX 할당하기 예시
```javascript
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✔";
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```