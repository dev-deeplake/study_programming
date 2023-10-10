>[!Learning Goals]
>- How to correctly update an object in React state
>- How to update a nested object without mutating it
>- What immutability is, and how not to break it
>- How to make object copying less repetitive with Immer
>----------------------------------------
>- How to add, remove, or change items in an array in React state
>- How to update an object inside of an array
>- How to make array copying less repetitive with Immer

- React의 state 값으로 객체나 배열을 사용하고 싶을 경우, 언제나 불변성을 지켜주어야 한다
	- = 직접 변이하지 말고 복사본을 만들어 사용!
	- 왜? ) 원시 데이터 타입과는 달리 참조형 데이터들은 변이가 되었을 때에도 가리키는 메모리 주소가 변경되지 않는다.
		- 예측 불가능한 작동 방지 : 참조형 데이터를 변이하면 원본이 수정되기 때문에, 그 데이터를 참조하던 다른 코드들이 있을 때 예상하지 못한 동작을 일으킬 수 있음
		- 데이터 변경 감지 : 참조 주소가 그대로이기 때문에 데이터의 변경이 감지되지 않는다.
		- 리액트의 최적화 : 리액트는 이전 프로퍼티 및 state가 다음 state와 동일한 경우 작업을 건너뛰는 방식으로 최적화하기 때문에, 데이터의 변경 여부를 확실히 감지할 수 있도록 불변성을 지켜주는 것이 좋다.
		- 디버깅 : 과거의 기록이 state 변이에 의해 지워지지 않기 때문에 console.log를 사용해서도 state 변경을 명확하게 확인할 수 있음
		- 리액트의 방향성 : state가 snapshot처럼 취급되는 것에 기반에 새로운 react 기능들이 개발될 것이므로, 이후를 생각해서라도 불변성을 지키기
	- react의 state 변경은 언제나 setter 함수를 통해 이루어져야 하고 + 사용하는 state가 참조형인 경우 언제나 불변성을 지켜주어야 state를 제대로 사용할 수 있다.

## State로 사용할 새로운 참조형 데이터 만들기
### 1. Spread Syntax 이용하기
- 모든 속성을 개별적으로 복사하지 않아도 되어 편리하다!
- 내부에 객체 및 배열이 중첩된 경우 중첩된 요소까지는 복사가 불가능하여 추가 작업을 해주어야 함.
	- 업데이트하려는 지점부터 최상위 수준까지 복사본을 만들어줘야 함!
- 객체의 경우 원하는 property의 값을 복사할 수 있고, 배열의 경우 원하는 요소를 추가할 수 있다.
	- ![[6 & 7) Updating Objects & Arrays in State#🔅 Spread syntax 사용한 객체와 배열의 복사 예시]]
### 2. Immer 사용하기
- 외부 라이브러리: 특수한 유형의 객체를 사용해 사용자가 수행하는 작업을 draft에 기록하기 때문에 불변성 걱정 없이 사용할 수 있음.
### TIP: 객체에서 프로퍼티 동적으로 지정하기
- 객체 내에서 대괄호를 사용하여 프로퍼티 이름을 동적으로 지정할 수 있다.
- 프로퍼티로 정해질 요소를 대괄호로 감싸서 사용 : `[e.target.name]: e.target.value`
### TIP: 배열 복사-변경에 사용할 수 있는 메소드 정리
1. 요소 추가 : spread operator (...)
2. 요소 제거 : filter
3. 로직에 따른 변경 : map
4. 요소 교체 : map (index와 함께 사용)
5. 삽입 : spread operator & slice
6. 배열의 반전 및 정렬 : spread operator로 배열 전체 복사 후 reverse 또는 sort 적용


------ example list ------
#### 🔅 Spread syntax 사용한 객체와 배열의 복사 예시
```javascript
const INITIAL_PERSON = {
	name: "Eric",
	age: 24,
	email: "example01@gmail.com"
}
const INITIAL_HOBBIES = ["soccer", "listening music", "reading books"]

const Profile = () => {
	const [person, setPerson] = useState(INITIAL_PERSON)
	const [hobby, setHobby] = useState(INITIAL_HOBBIES)
	...
}

...

const changedPerson = {...INITIAL_PERSON, name: "Karl"} // Spread syntax로 이름만 변경된 새 객체 생성
const changedHobby = [...INITIAL_HOBBY, "running"] // running이 추가된 새 배열 생성

setPerson(changedPerson)
setHobby(changedHobby)
```

#### 🔅 중첩된 객체를 복사하기
```javascript
const [person, setPerson] = useState({  
	name: 'Niki de Saint Phalle',  
	artwork: {  
		title: 'Blue Nana',  
		city: 'Hamburg',  
		image: 'https://i.imgur.com/Sd1AgUOm.jpg', 
	}
});
```

```javascript
// 방법 1
const nextArtwork = { ...person.artwork, city: 'New Delhi' };  
const nextPerson = { ...person, artwork: nextArtwork };  
setPerson(nextPerson);

// 방법 2
setPerson({  
	...person, 
	artwork: {
		...person.artwork,
		city: 'New Delhi' 
	}  

});
```