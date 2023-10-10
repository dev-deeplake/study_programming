## 정의와 특성
==특정 요소에서 발생한 이벤트가 그 요소의 상위 요소로 전파되는 특성==
- 최상위 요소까지 계속 전파됨
	- Ex) `div > div 1 > div 2 > button`의 계층 구조를 가진 경우
		- button에서 일어난 이벤트가 div 2 > div 1 > div 순으로 전파된다
- 🌟 ==같은 이벤트끼리만 전파가 일어남==
	- Ex) `div (onClick) > div 1 (onKeyDown) > div 2 (onClick) > button (onClick)`의 계층 구조를 가진  경우
		- button에서 일어난 이벤트는 div 2 > div 순으로 전파된다(div 1에서 막히지 않고 ==건너 뛴다==).