[MDN : Arrow function expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

## [반환 방식](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#function_body)
### 암시적 반환 (Implicit return)
- 화살표 함수 내에 중괄호 `{}` 없이 바로 표현식을 쓸 때, 그 표현식의 결과가 자동으로 반환
- `const double = x => x * 2;`
	- 여기서 `x * 2`의 결과가 자동으로 반환
- 간단한, 한 줄짜리 로직을 작성할 때 유용
### 명시적 반환 (Explicit return)
```javascript
const double = x => {
  const result = x * 2;
  return result;
}
```
- 화살표 함수 내에서 중괄호 `{}`를 사용하여 여러 줄의 코드를 작성할 때는 `return` 키워드를 사용하여 값을 반환
- 중괄호 `{}`를 사용하면 여러 줄의 코드를 작성할 수 있으므로, 좀 더 복잡한 로직을 포함하는 함수를 작성 가능
- ⚠️ return을 생략하게 되면 아무것도 반환하지 않게 되므로 주의!
