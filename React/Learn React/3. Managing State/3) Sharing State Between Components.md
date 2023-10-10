>[!Learning Goals]
>- How to share state between components by lifting it up
>- What are controlled and uncontrolled components

- Lifting state : 두 컴포넌트의 state를 항상 함께 변경하고자 할 때 state를 공통된 부모에게로 끌어올려 부모의 state로 만든 뒤 props를 통해 자식에게 전달하기!
	- 이 때 State를 끌어올릴 공통된 부모는 SSOT(A single source of truth)가 됨
- 자식 컴포넌트를 제어 컴포넌트로 만들 것인지, 비제어 컴포넌트로 만들 것인지에 대해 고민하기
	- 비제어 컴포넌트 : local state를 가져 자체적으로 상태를 제어하는 컴포넌트
	- 제어 컴포넌트 : 부모의 prop에 의해 구동이 제어되는 컴포넌트
	-  비제어 컴포넌트는 prop으로 넘겨줄 인자 개수가 줄어드는 등 configuration이 더 쉽다는 장점이 있으나, 비제어 컴포넌트를 함께 묶어 조정하거나 구동시키려는 경우 유연성이 떨어진다는 단점도 있기 때문에 ==컴포넌트 작성시 props에 의해 제어할 정보와 Local state에 의해 제어할 정보를 신중하게 고려하는 것이 좋다!==
## 예제 1) 자식 컴포넌트가 state를 갖고 독립적으로 동작하는 예시 (Panel = 비제어 컴포넌트)
```javascript
import { useState } from 'react';

function Panel({ title, children }) { // panel에서 isAcritve state를 관장하므로 Accordion이 panel의 active 여부를 제어할 수 없음
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Show
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About">
        With a population of about 2 million, Almaty is Kazakhstan's largest city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology">
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples". In fact, the region surrounding Almaty is thought to be the ancestral home of the apple, and the wild <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}
```
## 예제 2) 한 번에 하나의 패널만 열리도록 하기 - state lifting (Panel = 제어 컴포넌트)
- 자식 컴포넌트에서 state를 제거한 뒤
- 공통된 부모 컴포넌트에 하드코딩 데이터를 전달
- 부모에 state를 추가하고 이벤트 핸들러와 함께 내려주기
```javascript
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0); // 부모인 accordion에서 active한 Panel의 정보를 관리
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        With a population of about 2 million, Almaty is Kazakhstan's largest city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples". In fact, the region surrounding Almaty is thought to be the ancestral home of the apple, and the wild <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) { // 자식인 Panel에서는 title, children과 함께 isActive state와 이벤트 핸들러(onShow)를 부모에게 전달받음
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          Show
        </button>
      )}
    </section>
  );
}

```