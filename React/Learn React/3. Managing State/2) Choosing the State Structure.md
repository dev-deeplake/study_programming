>[!Learning Goals]
>- When to use a single vs multiple state variables
>- What to avoid when organizing state
>- How to fix common issues with the state structure

- state를 잘 구조화해야 수정과 디버깅이 용이한 컴포넌트를 만들 수 있다!
- 양과 구조를 신경쓰기 : 얼마만큼의 state variable을 사용할지, 데이터 형태는 어떻게 할지

## 1. 연관된 state 그룹화하기
- 두 개 이상의 State variables가 항상 함께 변경되는 경우, 하나의 state variable로 통합하기
	- ex) 마우스 커서 좌표 업데이트 ::: `setPosition({ x: 0 , y: 0 })` (x와 y 좌표를 따로 설정하는 대신)
- state variable이 유동적으로 정해지는 경우에도 객체 또는 배열 데이터 형식을 사용해 state variable을 그룹화할 수 있음
	- ex) 사용자가 필드를 동적으로 추가 가능한 폼 컴포넌트 ::: `const [fields, setFields] = useState([])`과 같은 방식으로 입력한 필드명을 관리 가능
- 🔥 state variable을 object로 결정했을 경우, 특정한 하나의 Key-value만 업데이트하려고 할 때도 나머지 key-value값을 모두 복사해줘야 한다!
	- 즉, `const [position, setPosition] = useState({ x:0, y:0 })`의 구조인 경우
		- `setPosition({ x:100 })` → 불가능
		- `setPosition({ ...position, x:100 })` → 가능
		- 또는 x와 y 좌표를 다른 state variable로 분리해서 따로 관리할 수도 있다.
## 2. State의 모순 피하기
- ![[2) Choosing the State Structure#🔅 state 모순 피하기 - 리팩토링 전]]
	- 위 예시에서 isSending과 isSent는 동시에 true가 될 수 없어야 하지만, 둘 모두를 함께 호출해 동시에 변경하는 것을 잊어버린다면 두 상태가 동시에 true가 되는 모순이 생길 소지가 있음
	- 따라서 각 state를 분리하기보다 status라는 통합된 state variable에서 상태를 관리하는 것이 버그 요소를 줄이는 방법
- ![[2) Choosing the State Structure#🔅 state 모순 피하기 - 리팩토링 후]]

## 3. 불필요한 state 피하기
- 렌더링 중에, 컴포넌트의 Prop이나 기존재하는 state variable에서 계산해낼 수 있는 데이터는 state로 설정하지 말기
	- ex) 성과 이름을 입력받고 full name을 보여줘야 하는 컴포넌트 : full name은 입력받은 데이터를 통해 연산하여 표출할 수 있으므로 full name을 state variable로 만들지 않아도 된다.
- props를 state에 그대로 미러링하지 않기
	- ![[2) Choosing the State Structure#🔅 불필요한 state 피하기 - state 미러링]]
	- 위와 같이 부모에게서 받아온 prop을 state로 그대로 미러링하는 경우, 부모가 다른 prop을 전달해주어도 자동으로 state variable(color)이 업데이트되지 않는다! (state는 컴포넌트의 첫 렌더링 중에만 초기화되므로)
	- 따라서 부모에게 받은 prop을 컴포넌트 내에서 그대로 사용 혹은 상수에 할당하여 사용하거나,
	- 부모에게서 받은 해당 Prop의 모든 업데이트를 의도적으로 무시하려는 경우 미러링을 사용할 수 있다. 그러나 이 경우 prop 이름을 `initial~`이나 `default~`로 설정하여 해당 Prop의 역할을 명확하게 기술해준다.
	- ![[2) Choosing the State Structure#🔅 불필요한 state 피하기 - state 미러링 convention]]

## 4. state의 중복을 피하기
- 한 state variable이 다른 state variable 내용 중 일부를 포함할 경우 중복을 없애는 방향으로 수정한다.
	- why? ) 한쪽이 다른 쪽을 내포하는 관계이기 떄문에, 언제나 서로의 변경사항이 연동되어야 하지만 그렇지 않은 경우가 생길 수 있으므로
	- ![[2) Choosing the State Structure#🔅 State의 중복을 피하기 - 리팩토링 전]]
	- ![[2) Choosing the State Structure#🔅 State의 중복을 피하기 - 리팩토링 후]]

## 5. nested state 피하기
- 중첩된 state를 업데이트하려면 변경된 부분부터 위쪽 객체의 복사본을 만들어야 하므로 번거롭고 코드가 장황해질 수 있음
- 업데이트가 어려울 만큼 깊은 state는 flat(=normalize, 정규화)하게 만드는 것을 고려
- 중첩된 state의 일부를 하위 컴포넌트로 이동하여 state 중첩을 줄이는 것도 가능
	- 해당 상태 값이 임시적이거나 / 컴포넌트에서 공유되거나 외부 소스에서 참조 및 저장될 필요가 없을 때 자식 컴포넌트에 상태를 제한하여 사용하면 좋음
- ex)
	- ![|400](images/nested_state_example.png)
	- ![[2) Choosing the State Structure#🔅 nested state 피하기 - 리팩토링 전]]
	- ![[2) Choosing the State Structure#🔅 nested state 피하기 - 리팩토링 후]]
	- childPlaces 배열을 childIds 배열로 바꾸어 정규화
------ example list ------
#### 🔅 state 모순 피하기 - 리팩토링 전
```javascript
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }
...
}
```
#### 🔅 state 모순 피하기 - 리팩토링 후
```javascript
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing'); // typing (초기 상태), sending, sent의 3가지 상태가 될 수 있음

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending'); // 상태 변경점
    await sendMessage(text);
    setStatus('sent'); // 상태 변경점
  }
 ...
}

```
#### 🔅 불필요한 state 피하기 - state 미러링
```javascript
function Message({ messageColor }) {  
	const [color, setColor] = useState(messageColor);
	...
}
```
#### 🔅 불필요한 state 피하기 - state 미러링 convention
```javascript
function Message({ initialColor }) {  
// The `color` state variable holds the *first* value of `initialColor`.  
// Further changes to the `initialColor` prop are ignored.  
	const [color, setColor] = useState(initialColor);
	...
}
```
#### 🔅 State의 중복을 피하기 - 리팩토링 전
```javascript
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(items[0]); //items의 0번째 인덱스를 사용 : 중복
  ...
}
```
#### 🔅 State의 중복을 피하기 - 리팩토링 후
```javascript
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0); // items의 id 정보만을 읽어 중복을 없앰.
  const selectedItem = items.find(item => item.id === selectedId); // id를 통해 어떤 item이 선택되었는지 연산
  // 이러한 변경을 통해 선택된 아이템의 이름을 변경해도 해당 아이템의 이름이 언제나 동기화되어 나타남
...
}
```
#### 🔅 nested state 피하기 - 리팩토링 전
```javascript
export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Earth',
    childPlaces: [{
      id: 2,
      title: 'Africa',
      childPlaces: [{
        id: 3,
        title: 'Botswana',
        childPlaces: []
      }, {
        id: 4,
        title: 'Egypt',
        childPlaces: []
      }, {
        id: 5,
        title: 'Kenya',
        childPlaces: []
      }, {
        id: 6,
        title: 'Madagascar',
        childPlaces: []
      }, {
        id: 7,
        title: 'Morocco',
        childPlaces: []
      }, {
        id: 8,
        title: 'Nigeria',
        childPlaces: []
      }, {
        id: 9,
        title: 'South Africa',
        childPlaces: []
      }]
    }, { ... }
```
#### 🔅 nested state 피하기 - 리팩토링 후
```javascript
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 43, 47],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 27, 35]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6, 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egypt',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenya',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  }, 
  7: {
    id: 7,
    title: 'Morocco',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'South Africa',
    childIds: []
  },
  ...
```
#### 🔅 메모리 사용량 개선
```javascript
// 개선 전
function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // Create a new version of the parent place
    // that doesn't include this child ID.
    const nextParent = {
      ...parent,
      childIds: parent.childIds
        .filter(id => id !== childId) // childId를 parent의 childIds 목록에서 제거
    };
    // Update the root state object...
    setPlan({
      ...plan,
      // ...so that it has the updated parent.
      [parentId]: nextParent // 부모의 childIds 목록을 업데이트하지만 실제 childId 객체는 plan에서 삭제되지 않음.
    });
}


// 개선 후
function handleComplete(parentId, childId) {
    updatePlan(draft => {
      // Remove from the parent place's child IDs.
      const parent = draft[parentId];
      parent.childIds = parent.childIds
        .filter(id => id !== childId);

      // Forget this place and all its subtree.
      deleteAllChildren(childId); // 해당 위치와 그 위치의 모든 자식들을 삭제하는 함수 호출
      function deleteAllChildren(id) {
        const place = draft[id];
        place.childIds.forEach(deleteAllChildren); // 자식의 자식들까지 재귀적으로 호출하여 삭제
        delete draft[id]; // 선택한 위치를 draft에서 삭제
      }
    });
}


```