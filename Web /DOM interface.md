## 요약
- [[Web / DOM (Document Object Model)]] 내의 개별 요소들과 상호작용하기 위한 프로그래밍 인터페이스
	- 웹 페이지의 각 요소(태그)를 프로그래밍적으로 조작하고 상호작용하기 위한 구조와 메소드들을 정의
	- 여러 프로퍼티, 메소드 및 이벤트를 포함하는 '클래스'와 유사한 개념
	- 각 HTML 요소에는 해당 요소의 특징과 동작을 반영하는 DOM interface가 존재
## 예시
- a 태그가 있을 때, 이 a 태그와 프로그래밍적으로 상호작용하기 위한 모든 메소드와 프로퍼티는 HTMLAnchorElement라는 DOM interface에 정의되어 있음.
	- 이 HTMLAnchorElement DOM interface는 [[Web / Web IDL]] 언어로 정의됨
		- **WHATWG Living HTML Standard 문서의 HTMLAnchorElement 정의**
		 ![[images/CleanShot 2023-09-15 at 21.54.55@2x.png]]
			 - `interface HTMLAnchorElement : HTMLElement`
				 - HTMLAnchorElement가 HTMLElement에서 파생된 인터페이스라는 것을 나타냄. 즉 a 요소는 일반적인 HTML 요소의 모든 기본 특성을 상속받는다.
			 - `HTMLAnchorElement includes HTMLHyperlinkElementUtils`
				 - HTMLAnchorElement는 HTMLHyperLinkElementUtils라는 인터페이스에 정의된 속성 및 메소드를 포함함.
					 - 'mixin' 패턴 : 다른 인터페이스에 정의된 속성 및 메서드를 포함하기
		 ![[images/CleanShot 2023-09-15 at 22.07.14@2x.png]]
	- 'href' 속성이 위와 같이 DOM interface에 정의되어 있기 때문에 javascript에서 다음과 같이 링크 요소의 href 속성에 접근할 수 있게 된다!
```javascript
const link = document.getElementById("myLink");
const newHref = "https://www.naver.com"
const hrefValue = link.href // 기존에 어떠한 url이 들어있었다고 가정
link.href = newHref
```



