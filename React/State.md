## 특징
- 각 컴포넌트 인스턴스마다 고유하게 생성된다.
- 인스턴스마다 고유하게 생성된 state는 다른 인스턴스의 state에 영향을 미치지 않는다.
	- hook은 컴포넌트 최상단에서 사용되어야 한다는 위치적 제약이 있고, 호출된 순서도 중요하다. state는 이러한 위치적 제약이 없지만 컴포넌트 인스턴스라는 자신의 구역에서 '지역적'이다(but it's 'local' to the specific place on the screen. You rendered two \<Gallery /> components, so their state is stored separately).
- 부모 컴포넌트와 자식 컴포넌트가 가지고 있는 state는 완전히 독립적이다.
	- 컴포넌트 내에서 state가 선언되므로, 해당 컴포넌트에서 prop으로 자식 컴포넌트에게 내려주지 않는 이상은 어떤 state를 가지고 있는지 알 수 없다. 이처럼 state는 폐쇄/은닉되어 있으므로 부모 컴포넌트라 해도 수정하거나 영향을 미칠 수 없다.