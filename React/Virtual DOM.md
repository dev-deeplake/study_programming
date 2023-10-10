## 가상 DOM (Virtual DOM)

가상 DOM은 실제 DOM을 추상화한 JavaScript 객체이다. React에서는 컴포넌트의 상태가 변할 때마다 실제 DOM을 직접 조작하지 않고, 이 가상 DOM을 이용하여 변화를 처리한다.

## 가상 DOM의 작동 원리

1. **렌더링**
   - 컴포넌트의 상태가 변경될 때마다 새로운 가상 DOM 트리가 생성된다.
2. **비교(Diffing)**
   - 이전 가상 DOM 트리와 새로운 가상 DOM 트리를 비교하여 차이점을 찾는다.
3. **업데이트(Reconciliation)**
   - 찾아낸 차이점을 실제 DOM에 반영한다.

## 가상 DOM과 실제 DOM의 차이

1. **효율성**
   - 실제 DOM의 변화는 비용이 많이 들 수 있으나 가상 DOM을 사용하면 실제 DOM과의 불필요한 인터랙션을 최소화하여 성능을 향상시킬 수 있다.
2. **추상화 수준**
   - 가상 DOM은 JavaScript 객체이므로, 실제 DOM보다 더 높은 수준의 추상화를 제공한다.
3. **속도**
   - 가상 DOM은 메모리 상에서 동작하기 때문에 실제 DOM 조작보다 훨씬 빠르게 처리할 수 있다.
4. **직접 조작의 최소화**
   - 리액트는 개발자가 실제 DOM을 직접 조작하지 않도록 하여 버그 발생 가능성을 줄이고 코드의 품질을 유지한다.

## 예시

실제 DOM은 브라우저에서 HTML 문서의 구조를 표현하는 인터페이스이다. 아래와 같이 HTML 문서가 있다고 하면:

```html
<div id="root">
    <span>텍스트</span>
</div>
```

이 HTML 문서의 DOM 트리는 아래와 같이 된다.

```
html
└── body
    └── div#root
        └── span: "텍스트"
```

리액트의 가상 DOM에서는 이 구조를 JavaScript 객체로 표현한다.

```javascript
const virtualDOM = {
    type: 'div',
    props: { id: 'root' },
    children: [
        {
            type: 'span',
            props: {},
            children: ['텍스트']
        }
    ]
};
```

이 가상 DOM 객체는 컴포넌트의 상태가 변할 때마다 업데이트되며, 변경사항이 실제 DOM에 최적화되어 반영된다.