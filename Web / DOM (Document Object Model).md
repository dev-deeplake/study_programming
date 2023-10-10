## 요약
- 웹 문서의 구조와 내용을 프로그래밍적으로 조작하고 접근할 수 있도록 하는 프로그래밍 인터페이스
- 웹 문서의 구조를 트리 형태로 표현
- 각 노드가 문서의 일부분이 됨

## DOM의 핵심 개념
1. 노드 (Node)
	- 모든 요소, 속성 및 텍스트는 DOM에서 노드로 표현
		- ex) Element, Text, Attribute... (전부 노드!)
2. 트리 구조
	- 웹 페이지의 계층적 구조를 나타내는 트리
	- 트리의 최상위 노드는 Document 노드
3. 선택자 (Selector)
	- DOM 요소를 선택하기 위한 표현식
	- CSS 선택자와 같은 방식으로 작동
	- javascript에서는 querySelector이나 querySelectorAll 등의 메소드를 통해 요소 선택 가능

## DOM 조작 기본
1. 요소 선택
    - `getElementById`: ID를 기반으로 요소 선택
    - `getElementsByClassName`: 클래스 이름을 기반으로 여러 요소 선택
    - `getElementsByTagName`: 태그 이름을 기반으로 여러 요소 선택
    - `querySelector`: CSS 선택자를 사용해 단일 요소 선택
    - `querySelectorAll`: CSS 선택자를 사용해 여러 요소 선택
2. 요소 조작
    - `createElement`: 새 요소 생성
    - `appendChild`: 요소를 다른 요소의 자식으로 추가
    - `removeChild`: 요소의 자식 제거
    - `replaceChild`: 요소의 특정 자식을 다른 요소로 교체
    - `innerText` / `innerHTML`: 요소의 텍스트나 내부 HTML 조작
3. 속성과 스타일 조작
    - `getAttribute`: 요소의 속성 값 가져오기
    - `setAttribute`: 요소의 속성 값 설정
    - `removeAttribute`: 요소의 속성 제거
    - `style`: 요소의 인라인 스타일 조작

## DOM 이벤트
1. 이벤트 리스너
    - 웹 페이지에서 발생하는 다양한 이벤트 (클릭, 키보드 입력, 마우스 움직임 등)에 반응하기 위해 사용
    - `addEventListener`: 요소에 이벤트 리스너 추가
    - `removeEventListener`: 요소에서 이벤트 리스너 제거
2. 이벤트 버블링 및 캡처링
    - 이벤트가 DOM 트리를 통해 전파되는 방식
    - 기본적으로 이벤트는 버블링 단계에서 발생하며, 상위 요소로 전파
3. 이벤트 객체
    - 이벤트 핸들러 내에서 이벤트에 대한 정보를 제공하는 객체.
    - 예를 들어, 마우스 이벤트의 경우, 마우스의 위치, 클릭된 버튼 등의 정보를 가져올 수 있음