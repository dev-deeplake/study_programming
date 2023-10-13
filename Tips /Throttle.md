## 개념
이벤트를 일정한 주기마다 실행하도록 제한하는 기법

## 활용 케이스
주로 고밀도로 발생하는 이벤트가 있을 때 성능 최적화에 사용됨
- 스크롤
	- 사용자가 스크롤할 때마다 이벤트를 처리하지 않고 특정 시간마다 한 번만 이벤트를 처리하도록 제한
- 마우스 움직임

## 구현 코드 예시
```javascript
/* throttle 구현 */
const throttle = (callback, limit) => {
	let isThrottled
	return (...args) => {
		if (!isThrottled) {
			callback.apply(this, args)
			isThrottled = true
			// 지정 시간 경과 후 flag를 다시 false로 설정하기
			setTimeout(() => isThrottled = false, limit)
		}
	}
}
```