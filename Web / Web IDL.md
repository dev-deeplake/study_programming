## 요약
- Web IDL (Interface Description Language)는 웹 API를 위한 인터페이스를 명세화하기 위한 언어
- 주로 <U>웹 표준에서 웹 브라우저가 제공해야 할 javascript 인터페이스를 기술</U>하는데 사용됨.

## 기본 구성 요소
1. interfaces : 객체의 형태와 그 객체가 제공해야 할 메소드 및 속성을 정의
2. attributes : 객체의 속성을 표현
3. methods : 객체의 메소드 표현
4. dictionaries : 키-값 쌍의 데이터 구조 정의

```webidl
// Interface 정의
interface Person {
	// attributes
	attribute DOMString name;
	attribute unsigned short age;

	// methods
	vodi greet();
}

// dictionary 정의
dictionary PersonInit {
	DOMString name;
	unsigned short age;
}
```
- 위의 Web IDL은 Person 인터페이스를 정의하고 있음.
	- Person에는 name과 age라는 두 속성(attributes)이 있음
	- greet라는 메소드 존재
	- PersonInit이라는 dictionary를 정의하여 name과 age를 초기화할 수 있음
- 실제 javascript 코드에서는 해당 [[DOM interface|인터페이스를 기반으로 생성된 객체를 사용하거나, 메소드를 호출하거나, 속성에 접근]]하게 됨