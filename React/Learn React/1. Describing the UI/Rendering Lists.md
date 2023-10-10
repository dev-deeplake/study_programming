> [!Learning Goals]
> - How to render components from an array using JavaScript’s `map()`
> - How to render only specific components using JavaScript’s `filter()`
> - When and why to use React keys

## 요약
- 데이터 모음을 통해 유사 컴포넌트를 여러 개 표시(구조는 동일하나 콘텐츠가 다를 경우)하고자 할 때는 JavaScript의 Array를 이용
	- key
		- 쓰임과 의의
			- 배열 내부 항목들에 이동, 삽입, 삭제 등 변화가 일어나게 되면 해당 key를 통해 매칭하게 됨
			- 데이터 변경의 올바른 추론 및 실제 DOM 트리를 올바르게 업데이트하는 데 필요
		- 지정 방식
			- 배열 항목에는 배열 내부 항목들 사이에서 고유하게 식별할 수 있는 '문자열' 또는 '숫자'를 지정해주어야 함
			- 여러 DOM 노드가 하나의 묶음인 경우
				- div 등의 명시적 요소로 해당 묶음을 둘러 거기에 key를 부여해주거나
				- [[#🔅 명시적 Fragment 구문과 key 전달 예시|`<>...</>` 같은 fragment 구문이 아닌, 명시적 `<Fragement>` 구문을 사용하여 key 부여]]
		- key의 규칙
			- 형제(sibling) 간에는 유일한 값이어야 함
				- database에서 데이터를 끌어올 때는 database key 또는 ID가 이미 유일하므로 해당 정보를 key로 사용하고, key를 직접 생성해야 하는 경우에는 카운터를 통해 유일한 id를 부여하거나 패키지([`crypto.randomUUID()`](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID) / [`uuid`](https://www.npmjs.com/package/uuid) 등) 사용
			- 다른 배열의 JSX 노드 간에는 꼭 유일하지 않아도 됨
			- 변경되지 않아야 함 ([[#🔅 key가 변경된다면 어떤 일이 생길까?]])
		- ⚠️ key는 prop이 아니므로, 컴포넌트에 ID가 필요한 경우 key 외 별도의 prop명을 통해 전달해야 함
	- [[#🔅 배열을 통한 데이터 렌더링 - map 예시|map()]]
	- [[#🔅 배열을 통한 데이터 렌더링 - filter 예시|filter()]]

### 🔅 배열을 통한 데이터 렌더링 - map 예시
```javascript
// 출력 데이터를 배열에 저장
const people = [  
	'Creola Katherine Johnson: mathematician',  
	'Mario José Molina-Pasquel Henríquez: chemist',  
	'Mohammad Abdus Salam: physicist',  
	'Percy Lavon Julian: chemist',  
	'Subrahmanyan Chandrasekhar: astrophysicist'  
];

// 해당 데이터를 map을 통해 JSX와 결합하고, 컴포넌트상에서 반환
export default function List() {
  const listItems = people.map(person =>
    <li>{person}</li>
  );
  return <ul>{listItems}</ul>;
}

```

### 🔅 배열을 통한 데이터 렌더링 - filter 예시
```javascript
// profession이 'chemist'인 사람만 표시하는 예제
const people = [{  
		id: 0,  
		name: 'Creola Katherine Johnson',  
		profession: 'mathematician',  
	}, {  
		id: 1,  
		name: 'Mario José Molina-Pasquel Henríquez',  
		profession: 'chemist',  
	}, {  
		id: 2,  
		name: 'Mohammad Abdus Salam',  
		profession: 'physicist',  
	}, {  
		name: 'Percy Lavon Julian',  
		profession: 'chemist',  
	}, {  
		name: 'Subrahmanyan Chandrasekhar',  
		profession: 'astrophysicist',  
	}];

export default function List() {
// 조건에 맞는 데이터만을 filter() 이용해 생성
  const chemists = people.filter(person =>
    person.profession === 'chemist'
  );
  // 해당하는 데이터를 JSX와 결합
  const listItems = chemists.map(person =>
    <li>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        known for {person.accomplishment}
      </p>
    </li>
  );
  // listItems 및 결과 return
  return <ul>{listItems}</ul>;
}
```

### 🔅 함수 표현식의 반환 방식
![[화살표 함수 (Arrow Function Expressions)#[반환 방식](https //developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions function_body)]]

### 🔅 명시적 Fragment 구문과 key 전달 예시
```javascript
import { Fragment } from 'react';

const listItems = people.map(person =>  
	<Fragment key={person.id}>  
		<h1>{person.name}</h1>  
		<p>{person.bio}</p>  
	</Fragment>  
);
```

### 🔅 key가 변경된다면 어떤 일이 생길까?
- key가 변경될 때마다 리액트는 해당 컴포넌트의 상태와 연관된 모든 것을 폐기하고 새로운 컴포넌트를 생성
	- 즉, key가 변경되면 해당 컴포넌트는 완전히 새로운 컴포넌트로 간주됨
- 렌더링 중에 key를 생성하게 되면 렌더링 할 때마다 다른 키가 생성될 수 있음
	- 배열에서 항목의 index를 key로 사용하거나, Math.random()을 이용해 key를 생성하는 것도 여기에 해당
	- 렌더링 시 key를 생성하게 되면 다음의 문제가 발생할 수 있음
		- 비효율적인 업데이트 : 새로운 key를 가진 것을 새 항목으로 간주하므로 불필요한 업데이트나 리렌더링을 수행
		- 상태 손실 : 렌더링된 컴포넌트가 내부 상태를 가지고 있었다면 key가 변경될 때마다 상태가 손실됨
		- interaction시 효과에 문제가 생길 수 있음 : 항목들의 변화를 key를 통해 감지하므로, drag-and-drop으로 항목의 순서를 변경하는 interaction이나 항목 간 전환, 애니메이션과 같은 동작들이 제대로 작동하지 않을 수 있음

