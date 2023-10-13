## 개념
어떤 이벤트가 연속해서 발생할 경우, 일정 시간동안 추가적인 이벤트가 발생하지 않을 때 까지 그 이벤트의 처리를 지연시키는 기법

## 활용 케이스
- 검색창
	- 사용자의 입력을 감지하고, 연속적인 입력 중에는 요청을 보내지 않도록 한 뒤 사용자가 입력을 멈춘 후 마지막 입력값을 바탕으로 검색 요청을 보내는 케이스
- 화면 리사이즈시 추가 실행 로직
	- 리사이즈시 함수가 재실행되어야 할 때, 모바일이 아닌 데스크톱 환경에서의 리사이즈는 사용자가 창 크기를 줄이는 동안에도 계속적으로 일어나므로 이런 경우 debounce를 사용

## 구현 코드 예시
```javascript
/* debounce 구현 */
const debounce = (callback, waitTime) => {
	let timeout
	return (...args) => {
		clearTimeout(timeout)
		timeout = setTimeout(() => callback.apply(this, args), waitTime)
	}
}
```