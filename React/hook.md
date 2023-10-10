- ==React 내에서 'use'로 시작하는 함수==
- ==React가 렌더링 중일때만 (only ... while React is rendering) 사용할 수 있는 특수 함수==
- Hook을 통해 React의 여타 feature들과 연결할 수 있다.

## hook의 특성
- 🌟 반드시 컴포넌트의 최상위 레벨 또는 custom hook에서만 호출할 수 있다.
- 🌟 ==반복문, 조건문 또는 중첩 함수 안에서 호출할 수 없다.==
	- 관련 문서: [React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)
		- ==리액트는 hook의 호출 순서에 따라 state를 기억한다. 따라서 반복문, 조건문, 중첩 함수 등 이 순서가 어그러질 수 있는 사용 환경에서 Hook을 사용해서는 안된다.==

## hook 사용시의 naming convention
- `const [something, setSomething]`
	- 구조분해할당을 이용해 위와 같이 적용하는 것이 Convention. 벗어날 수 있지만, 프로젝트 전반에서 가독성을 위해 위와 같이 사용한다.
