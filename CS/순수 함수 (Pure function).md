[Wikipedia: pure function](https://en.wikipedia.org/wiki/Pure_function)
## 컨셉 및 특징
- Minds its own business : 호출 이전 존재했던 객체나 변수를 변경하지 않음
- SISO (Same Inputs, same output) : 동일한 입력이 주어질 시 항상 동일한 결과를 반환해야 함
- 수학 공식이 순수 함수의 예가 될 수 있음
```javascript
// 다음 함수는 언제나 투입된 input의 2배수를 반환
function double(number) {
	return 2 * number
}
```

