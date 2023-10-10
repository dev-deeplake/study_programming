>[!Learning Goals]
>- How to add a state variable with the [`useState`](https://react-ko.dev/reference/react/useState) Hook
>- What pair of values the `useState` Hook returns
>- How to add more than one state variable
>- Why state is called local

- 리액트 컴포넌트에서 상호작용의 결과로 렌더링을 변경해야 할 때, 이전과 달라진 요소가 있음을 추적할 수 있어야 변경을 인지하고 반영 가능
- state는 이를 위해 사용하는 일종의 '컴포넌트 별 메모리'

## ▶️ Hooks?
![[hook]]


## 기본 hook - useState
### ==1. useState가 해결하는 문제==
- 컴포넌트에서 상호작용에 따라 렌더링을 변경하고 싶을 때, state가 없다면?
	- 컴포넌트 안에서 지역 변수를 정의하고, 렌더링 될 결과물에서 interaction이 일어나면 eventHandler를 통해 선언한 지역변수를 변경, 변경된 내용을 컴포넌트에 렌더링하려 시도해도 효과가 없다. 지역 변수는 컴포넌트가 렌더링 될 때 마다 초기화되기 때문이다 (컴포넌트 안에서 재선언됨).
	- 컴포넌트 바깥에서 사용할 수 있는 전역 변수를 정의하고, 그 변수를 컴포넌트들의 prop으로 전달하여 렌더링을 변경시킨다고 해도 리액트는 변경 사항을 알아차리지 못한다 - 전역 변수는 변경되었으나, 컴포넌트의 Prop으로 전달된 값은 변경되지 않는다.
- useState 훅은 다음을 제공하여 문제를 해결한다.
	1) 새로운 렌더링 및 재렌더링이 일어나더라도 데이터를 유지시켜줄 **state** 변수(variable)
	2) 데이터가 변경되었을 때 변수를 업데이트하고 렌더링을 촉발하는 state의 setter 함수

### 2. 사용 방법
1) `import { useState } from "React"`
2) `const [ /variable/, /setterFunction/ ] = useState(/initial value/)`
- useState는 언제나 `[variable, setterFunction]`의 형태를 반환하기 때문에 2)와 같이 사용하는 것은 배열의 구조분해 할당에 해당된다.

### 3. useState의 작동 방식
```javascript
import react, { useState } from "React"

const Counter = () => {
	const [number, setNumber] = useState(0)
	return (
		<div className="counter">{ number }</div>
		<button onClick={() => setNumber(number + 1)}>+1</button>
	)
}
```
위와 같은 컴포넌트가 렌더링될 때 useState는 다음과 같이 작동한다.
1) 첫 렌더링
	- useState를 이용해 number의 초기값을 0으로 전달하였으므로, useState가 number로 0, 그리고 number의 setter function인 setNumber를 배열에 담아 리턴한다.
		- `[0, setNumber]`
2) 유저 인터랙션 - 버튼 클릭
	- 유저가 +1 버튼을 클릭하면 이벤트 핸들러의 setNumber(number + 1)이 작동하여 number를 1로 업데이트한다.
	- 리액트는 number의 변경을 감지하고 다음과 같은 처리를 한다.
		- 새로운 number(state)로 1을 기억한다.
		- state 변경에 따른 새로운 렌더링을 일으킨다.
3) 두 번째 렌더링 (변경된 state로 인함)
	- 리액트는 useState(0)을 만나지만, 이미 유저가 변경한 state인 1을 기억하고 있으므로 다음을 리턴한다.
		- `[1, setNumber]`
-  ( ... 이후 반복 ... )

## 컴포넌트에 여러 state variable을 사용하기
- 한 컴포넌트에 여러 개별적인 state를 사용할 수 있다.
	- ex) index (number), isVisible (t/f), ....
- 하지만 state variable이 서로 연관되어있는 경우 합쳐서 관리하는 편이 나을 수 있다. (특히 2개 이상의 state가 항상 동시에 update 되는 경우)
	- ==객체 형태로 합쳐서 state를 관리하는 경우, 객체가 참조형이기 때문에 컴포넌트의 리렌더링은 state 값의 변경 유무보다는 세터 함수의 사용 여부에 따라 결정된다.==

## 🌟 **useState가 여러 번 사용될 때, React에서는 어떤 state를 반환해야 하는지 어떻게 알 수 있나?**
- 위의 예에서는 useState가 한 번만 사용되었지만, 만약 한 컴포넌트에서 여러 state를 사용하고 있다면?
	- React에서는 useState hook을 사용하면서 이 hook이 어떤 state variable을 참조해 돌려줘야 하는지에 대한 정보를 받고 있지 않은데...
- **==React에서는 hook이 호출된 순서를 기억하고, 그에 따라 올바른 state를 돌려준다.==**
	- ![[hook#hook의 특성]]
- React를 사용하지 않고 구성한 useState 및 useState를 사용하는 컴포넌트의 작동 예시
	- ![[3) State - a component's memory#🔅 변수 componentHooks와 currentHookIndex]]
	- ![[3) State - a component's memory#🔅 Hook useState]]
	- ![[3) State - a component's memory#🔅 Component Gallery]]
	- ![[3) State - a component's memory#🔅 function updateDOM]]

## 고립되어있으며 독립적인 'state'
- ![[State#특징]]
***
--- example lists ---
#### 🔅useState의 동작 원리에 대한 공식문서의 풀이 코드 예시
```javascript
let componentHooks = [];
let currentHookIndex = 0;

// How useState works inside React (simplified).
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // This is not the first render,
    // so the state pair already exists.
    // Return it and prepare for next Hook call.
    currentHookIndex++;
    return pair;
  }

  // This is the first time we're rendering,
  // so create a state pair and store it.
  pair = [initialState, setState];

  function setState(nextState) {
    // When the user requests a state change,
    // put the new value into the pair.
    pair[0] = nextState;
    updateDOM();
  }

  // Store the pair for future renders
  // and prepare for the next Hook call.
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}

function Gallery() {
  // Each useState() call will get the next pair.
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  // This example doesn't use React, so
  // return an output object instead of JSX.
  return {
    onNextClick: handleNextClick,
    onMoreClick: handleMoreClick,
    header: `${sculpture.name} by ${sculpture.artist}`,
    counter: `${index + 1} of ${sculptureList.length}`,
    more: `${showMore ? 'Hide' : 'Show'} details`,
    description: showMore ? sculpture.description : null,
    imageSrc: sculpture.url,
    imageAlt: sculpture.alt
  };
}

function updateDOM() {
  // Reset the current Hook index
  // before rendering the component.
  currentHookIndex = 0;
  let output = Gallery();

  // Update the DOM to match the output.
  // This is the part React does for you.
  nextButton.onclick = output.onNextClick;
  header.textContent = output.header;
  moreButton.onclick = output.onMoreClick;
  moreButton.textContent = output.more;
  image.src = output.imageSrc;
  image.alt = output.imageAlt;
  if (output.description !== null) {
    description.textContent = output.description;
    description.style.display = '';
  } else {
    description.style.display = 'none';
  }
}

// ... (기타 관련 코드 생략)

// Make UI match the initial state.
updateDOM();
```
##### 🔅 변수 : componentHooks와 currentHookIndex
```javascript
let componentHooks = [] // hook이 호출된 결과인 pair ([state, setter])를 저장할 배열
let currentHookIndex = 0 // 현재의 hook index : hook이 호출되어 pair가 저장된/될 위치를 나타내는 일종의 커서 인덱스
```
##### 🔅 Hook : useState
```javascript
function useState(initialState) { // useState는 함수의 인자로 state의 초기값을 받는다.
  let pair = componentHooks[currentHookIndex]; // componentHooks 배열에서 currentHookIndex 위치를 탐색하고, 해당 값을 pair 변수에 할당한다.

// 1. pair(componentHooks 배열의 currentHookIndex 위치)에 값이 무언가 저장되어있을 때 - 이미 과거에 컴포넌트에서 이 hook이 호출된 적이 있을 때 = 첫번째 렌더링이 아닐 때
  if (pair) {
    currentHookIndex++; // 이미 저장된 값이 있으므로 현재의 hook index를 1 증가시켜 다음 커서로 이동시켜준다.
    return pair; // 그리고 해당 hook index에 저장되어있는 [state, setter] 배열을 hook 호출의 결과로 리턴시켜준다. 사용자는 이 결과를 구조분해할당으로 받아 사용하게 될 것임.
	// 🌟 결국 돌려줄 [state, setter] 데이터를 결정하는 것은 currentHookIndex이다! → 리액트는 hook의 호출 순서에 따라 state를 기억하므로, 반복문, 조건문, 중첩 함수 등 이 순서가 어그러질 수 있는 사용 환경에서 Hook을 사용해서는 안된다.
  }

// 2. pair(componentHooks 배열의 currentHookIndex 위치)에 값이 저장되어있지 않아 undefined일 때 - 과거에 컴포넌트에서 이 hook이 호출된 적이 없을 때 = 첫번째 렌더링일 때
  pair = [initialState, setState]; // pair 변수에 useState의 인자로 받은 initialState와 state의 setter 함수인 setState를 배열로 저장한다.

	// state의 setter함수를 정의 : setState는 해당 함수를 정의했을 때의 pair 값을 기억하여, 호출되었을 때 사용하게 된다 (클로져!)
  function setState(nextState) { // state의 setter인 setState는 함수의 인자로 변경될 (다음) state 값을 받는다.
    pair[0] = nextState; // 저장된 pair 배열의 첫 번째에 있는 state를, 받아온 nextState 값으로 교체해준다.
    updateDOM(); // 그런 뒤 DOM을 업데이트해준다. 이 과정에서 현재 hook index는 0으로 초기화된다.

  }

  componentHooks[currentHookIndex] = pair; // componentHooks의 현재 커서 위치에 [initialState, setState]를 저장한다.
  currentHookIndex++; // 다음 hook의 호출 결과를 저장하기 위해 커서를 1 증가시켜준다.
  return pair; // 마지막으로 [initialState, setState]를 리턴하여 유저가 이용할 수 있게 해준다.
}
```
##### 🔅 Component : Gallery
```javascript
function Gallery() {
  const [index, setIndex] = useState(0); // 컴포넌트의 첫 렌더링에서는 useState가 initialState로 0을 가지고 호출되어 다음과 같은 실행 결과를 가진다.
/**
* currentHookIndex: 1
* componentHooks: [[0, setState]]
* const [index, setIndex] = [0, setState]
* */
  const [showMore, setShowMore] = useState(false); // 컴포넌트의 첫 렌더링에서는 useState가 initialState로 false를 가지고 호출되어 다음과 같은 실행 결과를 가진다.
/**
* currentHookIndex: 2
* componentHooks: [[0, setState], [false, setState]]
* const [showMore, setShowMore] = [false, setState]
**/

  function handleNextClick() { // nextClick 핸들러에서는 index state의 setter인 setIndex를 이용해, 핸들러가 연결된 요소가 클릭될 때마다 현재의 state index를 1씩 증가시켜준다.
    setIndex(index + 1);
  }

  function handleMoreClick() { // moreClick 핸들러에서는 showMore state의 setter인 setShowMore을 이용해, 핸들러가 연결된 요소가 클릭될 때마다 현재의 showMore의 boolean 값을 반전시켜준다.
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index]; // index state 변수를 이용해 sculptureList 안에 저장된 특정 데이터값을 읽고 sculpture 변수에 할당한다.

// 본 예시에서는 리액트를 사용하지 않으므로 jsx 대신 object를 리턴
  return {
    onNextClick: handleNextClick,
    onMoreClick: handleMoreClick,
    header: `${sculpture.name} by ${sculpture.artist}`,
    counter: `${index + 1} of ${sculptureList.length}`,
    more: `${showMore ? 'Hide' : 'Show'} details`,
    description: showMore ? sculpture.description : null,
    imageSrc: sculpture.url,
    imageAlt: sculpture.alt
  };
}
```
##### 🔅 function : updateDOM
```javascript
function updateDOM() {
  currentHookIndex = 0; // 🌟 DOM을 업데이트할 때 현재의 hook index를 0으로 초기화시켜준다! → 컴포넌트 재렌더링시 Index 0부터 다시 시작
  let output = Gallery(); // Gallery의 리턴 object를 output에 저장

  // output의 내용을 실제 DOM에 업데이트
  nextButton.onclick = output.onNextClick;
  header.textContent = output.header;
  moreButton.onclick = output.onMoreClick;
  moreButton.textContent = output.more;
  image.src = output.imageSrc;
  image.alt = output.imageAlt;
  if (output.description !== null) {
    description.textContent = output.description;
    description.style.display = '';
  } else {
    description.style.display = 'none';
  }
}
```
##### 🔅 함수 및 컴포넌트 정의부를 제외한 초기 렌더링 관련 코드
```javascript
let componentHooks = [];
let currentHookIndex = 0;

let nextButton = document.getElementById('nextButton');
let header = document.getElementById('header');
let moreButton = document.getElementById('moreButton');
let description = document.getElementById('description');
let image = document.getElementById('image');
let sculptureList = [{
	  name: 'Homenaje a la Neurocirugía',
	  artist: 'Marta Colvin Andrade',
	  description: 'Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.',
	  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
	  alt: 'A bronze statue of two crossed hands delicately holding a human brain in their fingertips.'  
	}, {
	  name: 'Floralis Genérica',
	  artist: 'Eduardo Catalano',
	  description: 'This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.',
	  url: 'https://i.imgur.com/ZF6s192m.jpg',
	  alt: 'A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.'
	}, {
	  name: 'Eternal Presence',
	  artist: 'John Woodrow Wilson',
	  description: 'Wilson was known for his preoccupation with equality, social justice, as well as the essential and spiritual qualities of humankind. This massive (7ft. or 2,13m) bronze represents what he described as "a symbolic Black presence infused with a sense of universal humanity."',
	  url: 'https://i.imgur.com/aTtVpES.jpg',
	  alt: 'The sculpture depicting a human head seems ever-present and solemn. It radiates calm and serenity.'
	}, {
	  name: 'Moai',
	  artist: 'Unknown Artist',
	  description: 'Located on the Easter Island, there are 1,000 moai, or extant monumental statues, created by the early Rapa Nui people, which some believe represented deified ancestors.',
	  url: 'https://i.imgur.com/RCwLEoQm.jpg',
	  alt: 'Three monumental stone busts with the heads that are disproportionately large with somber faces.'
	}, {
	  name: 'Blue Nana',
	  artist: 'Niki de Saint Phalle',
	  description: 'The Nanas are triumphant creatures, symbols of femininity and maternity. Initially, Saint Phalle used fabric and found objects for the Nanas, and later on introduced polyester to achieve a more vibrant effect.',
	  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
	  alt: 'A large mosaic sculpture of a whimsical dancing female figure in a colorful costume emanating joy.'
	}, {
	  name: 'Ultimate Form',
	  artist: 'Barbara Hepworth',
	  description: 'This abstract bronze sculpture is a part of The Family of Man series located at Yorkshire Sculpture Park. Hepworth chose not to create literal representations of the world but developed abstract forms inspired by people and landscapes.',
	  url: 'https://i.imgur.com/2heNQDcm.jpg',
	  alt: 'A tall sculpture made of three elements stacked on each other reminding of a human figure.'
	}, {
	  name: 'Cavaliere',
	  artist: 'Lamidi Olonade Fakeye',
	  description: "Descended from four generations of woodcarvers, Fakeye's work blended traditional and contemporary Yoruba themes.",
	  url: 'https://i.imgur.com/wIdGuZwm.png',
	  alt: 'An intricate wood sculpture of a warrior with a focused face on a horse adorned with patterns.'
	}, {
	  name: 'Big Bellies',
	  artist: 'Alina Szapocznikow',
	  description: "Szapocznikow is known for her sculptures of the fragmented body as a metaphor for the fragility and impermanence of youth and beauty. This sculpture depicts two very realistic large bellies stacked on top of each other, each around five feet (1,5m) tall.",
	  url: 'https://i.imgur.com/AlHTAdDm.jpg',
	  alt: 'The sculpture reminds a cascade of folds, quite different from bellies in classical sculptures.'
	}, {
	  name: 'Terracotta Army',
	  artist: 'Unknown Artist',
	  description: 'The Terracotta Army is a collection of terracotta sculptures depicting the armies of Qin Shi Huang, the first Emperor of China. The army consisted of more than 8,000 soldiers, 130 chariots with 520 horses, and 150 cavalry horses.',
	  url: 'https://i.imgur.com/HMFmH6m.jpg',
	  alt: '12 terracotta sculptures of solemn warriors, each with a unique facial expression and armor.'
	}, {
	  name: 'Lunar Landscape',
	  artist: 'Louise Nevelson',
	  description: 'Nevelson was known for scavenging objects from New York City debris, which she would later assemble into monumental constructions. In this one, she used disparate parts like a bedpost, juggling pin, and seat fragment, nailing and gluing them into boxes that reflect the influence of Cubism’s geometric abstraction of space and form.',
	  url: 'https://i.imgur.com/rN7hY6om.jpg',
	  alt: 'A black matte sculpture where the individual elements are initially indistinguishable.'
	}, {
	  name: 'Aureole',
	  artist: 'Ranjani Shettar',
	  description: 'Shettar merges the traditional and the modern, the natural and the industrial. Her art focuses on the relationship between man and nature. Her work was described as compelling both abstractly and figuratively, gravity defying, and a "fine synthesis of unlikely materials."',
	  url: 'https://i.imgur.com/okTpbHhm.jpg',
	  alt: 'A pale wire-like sculpture mounted on concrete wall and descending on the floor. It appears light.'
	}, {
	  name: 'Hippos',
	  artist: 'Taipei Zoo',
	  description: 'The Taipei Zoo commissioned a Hippo Square featuring submerged hippos at play.',
	  url: 'https://i.imgur.com/6o5Vuyu.jpg',
	  alt: 'A group of bronze hippo sculptures emerging from the sett sidewalk as if they were swimming.'
	
}];

updateDOM() // initial rendering

```

