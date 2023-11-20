- UI를 그리면서 data flow를 만드는 방법의 '리액트적' 접근
- mock-up
	- ![[Thinking in React#🔅 Mockup API format]]
	- ![|200](images/s_thinking-in-react_ui.png)
## 1. Break the UI into a component hierarchy
- 모든 컴포넌트와 하위 컴포넌트를 구분하고 이름붙이기
	- 다음의 측면을 고려하여 분할
		- 프로그래밍적 측면 : [단일 책임 원칙(single responsibility principle)]등의 기법을 사용하여 어떻게 함수나 객체를 생성할지 정하고, 생성된 함수 및 객체가 담당할 작업에 따라 컴포넌트를 분할
		- CSS : 무엇을 위해 클래스 선택자를 생성할 것인지(그래서 거기에 어떤 디자인요소를 연결할 것인지)
		- 디자인 : 디자인의 레이어를 따라 컴포넌트를 구성하기
- 예제
	- ![|400](/images/s_thinking-in-react_ui_outline.png)
	- FilterableProductTable : 전체 앱을 포함
		- SearchBar : 사용자 입력 수신
		- ProductTable : 사용자 입력에 따른 entry 목록 표시 및 필터링
			- ProductCategoryRow : 각 카테고리에 대한 제목 표시
			- ProductRow : 각 상품 행 표시
	- 예제에서는 ProductTable의 'Name'과 'Price'를 따로 컴포넌트로 지정하진 않았다. 이러한 컴포넌트의 분리는 전적으로 프로그래머의 재량

## 2. Build a static version in React
- 컴포넌트 계층 구조를 나누고 난 뒤에는 상호작용을 포함하지 않은 정적 UI 버전을 구현
	- '정적' UI 버전 - props만을 이용하여 구현 (one-way data flow를 가지도록)
	- 이 단계에서는 상호작용에 따라 변하는 데이터인 state를 적용하지 않음
- 간단한 구현에서는 top-down 방식의 컴포넌트 구현이, 대규모 프로젝트에서는 bottom-up 방식으로 컴포넌트를 구현해가는 것이 더 용이함
- 예제
	- ![[Thinking in React#🔅 Static UI version]]
## 3. Find the minimal but complete representation of UI State
- UI를 완전히 표현하기 위해 필요한 state를 찾기
	- 계산으로 처리할 수 있는 항목들을 제외한, UI를 표현하는데 최소한의 state를 찾기
- state : reactive value
	- 사용자가 기반 데이터 모델을 변경했을 때 이에 반응하여 변화하는 UI를 만들어 줄 목적
- State가 아닌 것을 식별하고, 남은 것을 state로 만드는 방법 사용
	- unchanging over time
	- passed by props
		- Props vs State
			- Props : 함수가 전달받는 인자와 유사, 부모로부터 자식 컴포넌트에게 전달됨
			- State : 컴포넌트에서 사용하는 메모리(컴포넌트가 추적하고, 상호작용에 따라 변화하는 값)
	- could be calculated based on existing state or props in the component
- 예제
	- ![|400](/images/s_thinking-in-react_ui_outline.png)
	- 앱의 데이터 목록
		- 제품의 원본 목록
		- 사용자가 입력한 검색어
		- 체크박스 값
		- 필터링된 제품 목록
	- state가 아닌 것을 식별하고, 나머지를 state로 지정
		- state가 아닌 것?
			- 제품 원본 목록 : props로 받아옴 (products)
			- 필터링된 제품 목록 : products + 사용자 검색어 + 체크박스 값을 통해 계산 가능
		- state
			- 사용자가 입력한 검색어
			- 체크박스 값
## 4. Identify where your state should live
- state를 식별했으므로 해당 state를 어느 component에서 관장할지를 결정해야 함
	1. 해당 state를 기반으로 렌더링되는 모든 Component 찾기
	2. 해당 state의 영향을 받는 컴포넌트들의 가장 가까운 공통 부모 컴포넌트 찾기
		- 일반적으로는 해당 부모 컴포넌트에 state를 두지만, 그것이 적절하지 않다면 해당 부모보다 더 상위에서 state를 관장하도록 할 수도 있다.
- 예제
	- ![|400](/images/s_thinking-in-react_ui_outline.png)
	- state를 기반으로 렌더링되는 모든 component 찾기
		- 사용자가 입력한 검색어
			- SearchBar : 검색어가 표시되어야 함
			- ProductTable : 검색어를 바탕으로 상품 목록이 필터링되어 표시되어야 함
		- 체크박스 값
			- SearchBar : 체크박스 체크 여부가 표시되어야 함
			- ProductTable : 체크박스 값을 바탕으로 상품 목록이 필터링되어 표시되어야 함
	- 가장 가까운 공통 부모 컴포넌트 찾기
		- SearchBar와 ProductTable 모두 level 1의 컴포넌트이므로, 가장 가까운 공통 부모 컴포넌트는 level 2인 Filterable Product Table
		- ![[Thinking in React#🔅 State 연결 후]]
## 5. Add inverse data flow
- 'inverse?' : 기본 데이터 모델이 사용자와의 상호작용 및 기타 원인으로 바뀌었을 때, 변화한 데이터가 state를 관장하는 곳까지 영향을 미쳐야 함
	- state를 관장하는 component에서 state setter 함수를 전달하고, 필요시 event handler를 추가하여 데이터를 변경
- 예제
	- ![[Thinking in React#🔅 state setter 전달 및 event handler 추가]]
	- 
## ------ Example List ------
#### 🔅 Mockup API format
```javascript
[  // json api return data format
	{ category: "Fruits", price: "$1", stocked: true, name: "Apple" },  
	{ category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },  
	{ category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },  
	{ category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },  
	{ category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },  
	{ category: "Vegetables", price: "$1", stocked: true, name: "Peas" }  
]
```

#### 🔅 Static UI version
```javascript
/* 카테고리에 대한 제목을 표시하는 ProductCategoryRow (level: 0) */
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

/* 각 상품 행을 표시하는 ProductRow (level: 0) */
function ProductRow({ product }) {
  // prop으로 받아온 product 객체에서 상품의 이름을 가져와 리턴
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

/* ProductTable: 사용자 입력에 따른 entry 목록 표시 및 필터링 (level: 1) */
function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => { // prop으로 받은 products 배열의 내용을 가지고 ProductCategoryRow를 그림
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

/* SearchBar : 사용자 입력 수신 (level 1) */
function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label>
        <input type="checkbox" />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

/* FilterableProductTable : 전체 앱 */
function FilterableProductTable({ products }) { // prop으로 상품 목록 배열인 products를 받고, ProductTable 컴포넌트로 넘겨줌
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} /> 
    </div>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```
#### 🔅 State 연결 후
```javascript
import { useState } from 'react'; // ⭐️

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('') // ⭐️ 검색창에 검색된 텍스트를 관리하는 state
  const [inStockOnly, setInStockOnly] = useState(false) // ⭐️ 체크박스 값을 관리하는 state

  return (
    <div>
      <SearchBar 
        filterText={filterText} // ⭐️ 검색어를 표시해주어야 하는 SearchBar 컴포넌트에 filterText state 연결
        inStockOnly={inStockOnly} /> // ⭐️ 체크박스 값을 표시해주어야 하는 SearchBar 컴포넌트에 inStockOnly state 연결
      <ProductTable 
        products={products}
        filterText={filterText} // ⭐️ 검색어의 값을 이용해 products를 필터링해주어야 하는 ProductTable 컴포넌트에 state 연결
        inStockOnly={inStockOnly} /> // ⭐️ 체크박스 값을 이용해 products를 필터링해주어야 하는 Products Table 컴포넌트에 state 연결
    </div>
  )
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  )
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) { // ⭐️ Producst를 필터링해주는 ProductTable에 filterText와 isStockOnly를 prop으로 내려줌
  const rows = []
  let lastCategory = null

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return
    }
    if (inStockOnly && !product.stocked) {
      return
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      )
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    )
    lastCategory = product.category;
  })

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  )
}

function SearchBar({ filterText, inStockOnly }) { //⭐️ 검색어와 체크박스 값을 표시해야 하는 SearchBar 컴포넌트에서 filterText와 inStockOnly prop을 받아와 사용
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} />
        {' '}
        Only show products in stock
      </label>
    </form>
  )
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />
}
```
#### 🔅 state setter 전달 및 event handler 추가
```javascript
import { useState } from 'react'

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('')
  const [inStockOnly, setInStockOnly] = useState(false)

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} 
        onFilterTextChange={setFilterText} 
        onInStockOnlyChange={setInStockOnly} /> // ⭐️ state setter인 setFilterText와 setInStockOnly 함수를 SearchBar 컴포넌트에 전달하여 이벤트가 일어날 수 있는 곳에서 사용할 수 있도록 처리
      <ProductTable 
        products={products} 
        filterText={filterText}
        inStockOnly={inStockOnly} /> 
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = []
  let lastCategory = null

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return
    }
    if (inStockOnly && !product.stocked) {
      return
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      )
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    )
    lastCategory = product.category
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly, onFilterTextChange, onInStockOnlyChange }) {
// ⭐️ 추가된 setter 함수들을 받아와 쓸 수 있도록 SearchBar 컴포넌트의 prop 확장해주기
  return (
    <form>
      <input 
        type="text" 
        value={filterText} placeholder="Search..." 
        { /* ⭐️ 이벤트가 실제로 일어났을 때 input에 쓰인 검색값을 바꿀 수 있도록 onFilterTextChange setter를 연결 */}
        onChange={(e) => onFilterTextChange(e.target.value)} /> 
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} 
          { /* ⭐️ 이벤트가 실제로 일어났을 때 checkbox 값이 바뀌도록 onInStockOnlyChange setter를 연결 */}
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        Only show products in stock
      </label>
    </form>
  )
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
]

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}

```