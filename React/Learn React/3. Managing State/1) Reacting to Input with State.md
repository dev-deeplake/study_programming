>[!Learning Goals]
>- How declarative UI programming differs from imperative UI programming
>- How to enumerate the different visual states your component can be in
>- How to trigger the changes between the different visual states from code

- Keys
	- 리액트는 UI를 선언적(declarative)으로 조작한다. → 어떻게? 🌟
	- UI의 시각적 상태가 여러 가지일 때는 배열 및 map을 통해 한 번에 표시해주는 것이 편리할 수 있다 - "living styleguides" / "storybooks"

## 1. 선언형(declarative) 프로그래밍과 명령형(imperative) 프로그래밍
- ![[선언형 프로그래밍 (Declarative Programming) vs 명령형 프로그래밍 (Imperative Programming)]]
- ==Ex) 사용자가 답변을 작성하여 제출하는 form==
	- form에 뭔가를 입력했을 때 submit 버튼이 활성화된다.
	- submit 버튼을 누르면 form과 버튼이 비활성화되고 spinner가 노출된다.
	- 네트워크 요청이 성공했을 경우 form이 숨겨지고 성공 메시지("Thank you")가 노출된다.
	- 네트워크 요청이 실패했을 경우 에러 메시지가 나타나고 form이 다시 노출된다.
	- ![[1) Reacting to Input with State#🔅 명령형 UI 프로그래밍 예시 코드]]
		- 코드에서 어떤 일이 일어나는지 비교적 자세히 적혀있고 이해하기 쉽지만,
		- 새로운 요소를 추가허가나 로직을 변경해야 할 때 모든 단계를 하나하나 확인해야 하는 번거로움이 있고 (어느 곳에 변경 요소의 동작이 연결되어 있을지 예상하기 어려움)
		- 프로그램이 복잡해지고 커질 수록 위 단점이 기하급수적으로 심화됨
	- React에서 채택하고 있는 것 처럼 UI 조작에서 선언형 프로그래밍 방식을 사용할 경우 코드의 길이는 길어질 수 있지만 ==새로운 상태의 UI를 추가할 때 기존 상태가 영향을 받지 않는다는 장점==이 있음

## 2. UI를 선언적으로 만드는 과정

>[!선언적 UI 제작 단계]
>1. 컴포넌트의 시각적 '상태' 구분하기
>2. state change를 트리거하는 요인을 파악 (필요시 다이어그램 등으로 visualize)
>3. 컴포넌트의 상태를 나타내주기 위해 필요한 state 변수를 작성
>4. 필수적인 state 변수만 남기기
>5. event handler를 연결하여 state 설정을 완료하기
### 1. 컴포넌트의 시각적 '상태' 구분하기
- UI를 구성하는 요소가 뭉치는 경우의 수 쌍을 찾기
- 예시를 다음과 같이 재구성 가능
	- empty : textarea에 어떤 것도 입력되어있지 않고 submit button이 비활성화된 상태
	- typing : textarea에 어떤 내용이 채워지거나 채워지는 도중이며 submit button이 활성화 된 상태
	- submitting : 활성화된 submit button을 클릭해 form을 제출중인 상태, form이 사라지고 spinner(waiting message)가 나타남
	- success : form 제출에 성공한 상태. success message가 나타나고 form은 숨겨져있음
	- error : form 제출이 실패한 상태. error message가 나타나고 form이 다시 노출됨 (버튼 활성화)
- ![[1) Reacting to Input with State#1. 시각적 상태 구분 표 (0 활성화, X 비활성화, - 숨김)]]
- 위 케이스들의 mockup을 만들되, state에 의해 활성화 / 비활성화 / 숨김을 조절
- 컴포넌트에 시각적 상태가 많은 경우 → living styleguides / storybooks 형태로 한꺼번에 나타내주면 편리
	- ![[1) Reacting to Input with State#🔅 컴포넌트에 시각적 상태가 많은 경우, 한번에 나타내주기]]
	- ![|400](images/storybooks_example.png)

### 2. state change를 트리거하는 요인을 파악
- 사용자의 입력 또는 컴퓨터에서 일어나는 입력(네트워크 응답 도착, 시간 초과, 이미지 로딩 등)으로 인해 상태가 바뀌지만, 이것을 state variable로 설정해주어야만 UI가 변경된다.
- 예시의 경우 다음에 따라 상태가 전환됨
	- textarea가 비워져있는지에 따라 empty ↔︎ typing
	- button이 click되었을 때 submitting
	- resolved 되었으면 success
	- rejected 되었으면 error
	- ![[1) Reacting to Input with State#2. state variable 구분 표]]
- 다음과 같이 시각화해보면 로직 파악 및 버그 예방에 효율적!
	- ![|800](images/form_state_visualize.png)
### 3. useState를 이용해 컴포넌트의 상태 표현에 필요한 state variable 작성
- state variable의 수는 적을 수록 좋음
	- 필수적인 state부터 작성
	- 컴포넌트의 시각적 상태 중 어떤 상태를 표시할지 나타내는 state의 경우 줄일 수 있는지 생각해 볼 것. 단 가장 효율적인 방법을 모르겠다면 우선은 모든 경우의 수를 나타내는 데 충분한 state variable 설정하고 차차 줄여가기!
- ![[1) Reacting to Input with State#3. state variable 설정 - 초기 단계]]
### 4. 필수적인 State 변수만 남기기
- state가 역설을 일으키거나 서로 충돌되지 않는지
	- ex) isTyping과 isSubmitting은 동시에 True일 수 없음 (typing이 끝난 뒤 submit하게 됨)
- 다른 state가 동일한 정보를 나타내는지
	- ex) isEmpty와 isTyping은 동시에 True일 수 없음 → isTyping === !isEmpty의 관계임에 더해 이를 동기화시켜야만 앱이 제대로 작동함 (서로 분리되어있을 경우 버그 발생 확률 ↑)
- 다른 state 변수를 반전했을 때 동일한 정보를 얻을 수 있는지
	-  ex) isError의 경우 !error를 통해 확인할 수 있음
- ![[1) Reacting to Input with State#4. state variable 설정 - 리팩토링]]
- reducer를 사용해서 state를 다루면 좀 더 정확하게 state 관리가 가능
	- 현재는 status가 success일 때 null이 아닌 'error'는 의미가 없는 값으로 남아있음
	- Reduce를 사용하면 status가 success일 때의 error state를 다시 'null'로 설정해주는 등 세밀한 관리 가능
### 5. event handler를 연결하여 state 설정을 완료하기
- ![[1) Reacting to Input with State#5. 이벤트 핸들러를 연결한 최종 form 예시]]

------ example list ------
#### 🔅 명령형 UI 프로그래밍 예시 코드
##### 1. 프로그래밍한 내용에서 기본 동작에 해당하는 요소를 함수화
```javascript
// 요소 나타내기
const show = (element) => {
	element.style.display = ""
}

// 요소 숨기기
const hide = (element) => {
	element.style.display = "none"
}

// 요소 활성화
const enable = (element) => {
	element.disabled = false
}

// 요소 비활성화
const disable = (element) => {
	element.disabled = true
}
```
##### 2. 폼 제출시 답안 Check
```javascript
// 1500ms 후에 setTimeOut 내부의 로직이 처리됨, 답이 istanbul이면 fulfilled, 그렇지 않으면 Reject로 error 던짐
const submitForm = (answer) => {
	return new Promise(resolve, reject) => {
		setTimeOut(() => {
			if (answer.toLowerCase() == "istanbul") {
				resolve()
			} else {
				reject(new Error("this is the error message"))
			}
		}, 1500)
	}
}
```
##### 3. 이벤트 핸들링 : 동작의 조건과 동작을 조합
- 'form에 뭔가를 입력했을 때' (조건) + (submit 버튼을) '활성화' (동작)
```javascript
// 대상 요소는 이미 DOM에서 ID 또는 클래스로 읽어와 변수에 할당했다고 가정
const handleTextareaChange = () => {
	if (textarea.value.length == 0) {
		disable(button)
	} else {
		enable(button)
	}
}
```
- 이벤트 전체 핸들링 (폼 동작)
```javascript
const handleFormSubmit = async (e) => { // submitForm의 비동기적 작업 완료를 기다림
  e.preventDefault(); // form submit시 새로고침 막가

  // submit 버튼을 누르고 난 뒤의 초기 상태 : textarea 및 button 비활성화, spinner(loadingMessage) 노출
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);

  // 그러는 동안 다음을 시도
  try {
	// 폼 제출시 답안을 check하여 응답(resolve/reject) 기다림 (Await)
    await submitForm(textarea.value);
    // 정답일 경우의 로직 : successMessage 노출, form 숨김
    show(successMessage);
    hide(form);
  } catch (err) {
    // 오답일 경우 추가해줘야 할 로직 : promise에서 던진 errorMessage 노출
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
	// 답 여부에 상관 없이 폼을 초기 상태로 돌림 : spinner 숨김, textarea 및 버튼 재활성화
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
};

```
#### 🔅 선언적 UI 프로그래밍 예시
##### 1. 시각적 상태 구분 표 (0: 활성화, X: 비활성화, -: 숨김)
| target 및 case | empty | typing | submitting  | success     | error     |
| -------------- | ----- | ------ | ----------- | ----------- | --------- |
| textarea       | O     | O      | -           | -           | O         |
| button         | X     | O      | -           | -           | O         |
| 기타           | -     | -      | waiting msg | success msg | error msg |
##### 2. state variable 구분 표
| target 및 case   | empty     | typing    | submitting | success  | error     |
| ---------------- | --------- | --------- | ---------- | -------- | --------- |
| textarea-filled         | ==false== | ==true== | -       | -     | false      |
| button-clicked   | false     | false     | ==true==   | -     | -      |
| network response | -     | -     | -      | ==true== | ==false== |
##### 3. state variable 설정 - 초기 단계
```javascript
// 필수 state: textarea에 들어간 text, error시 반환될 text 담는 state
const [answer, setAnswer] = useState("")
const [error, setError] = useState(null)

// UI의 시각적 상태를 표현하는 state
const [isEmpty, setIsEmpty] = useState(true); // 이 부분이 filled로 되어있으면 모든 state의 기본값을 false로 통일할 수 있을듯
const [isTyping, setIsTyping] = useState(false);  
const [isSubmitting, setIsSubmitting] = useState(false);  
const [isSuccess, setIsSuccess] = useState(false);  
const [isError, setIsError] = useState(false);
```
##### 4. state variable 설정 - 리팩토링
```javascript
// 필수 state: textarea에 들어간 text, error시 반환될 text 담는 state
const [answer, setAnswer] = useState("")
const [error, setError] = useState(null)

// UI의 시각적 상태를 표현하는 state
const [status, setStatus] = useState("typing") // typing, submitting, success 3가지로 줄이기
```
##### 5. 이벤트 핸들러를 연결한 최종 form 예시
```javascript
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing'); // 기본 Status : typing

  if (status === 'success') { // success일 때의 msg
    return <h1>That's right!</h1>
  }
  
/* handleSubmit 함수의 경우 선언적 + 명령적 프로그래밍으로 구성되어있다.
** set 관련 Method와 submitForm 함수, preventDefault 메소드를 이용한 것은 선언적이지만,
** try / catch / finally 구조는 '처음 시도는 이렇게, 에러 발생시는 저렇게, 마지막으로는 이것을 수행하라'라는 의미이기 때문에 명령형에 가까움
**/
  async function handleSubmit(e) { // submit 버튼 클릭시 form 제어 핸들러
    e.preventDefault();
    setStatus('submitting'); // 버튼 클릭시 status를 submitting으로 전환
    try {
      await submitForm(answer); // 제출한 Form에 대한 응답이 오길 기다림
      setStatus('success'); // 성공시 status를 success로 변경
    } catch (err) { // promise가 rejected되었을 시
      setStatus('typing'); // 다시 form의 상태를 typing으로 전환 (form이 활성화되게)
      setError(err); // 그리고 error message를 받아 상태 변수에 세팅
    }
  }

  function handleTextareaChange(e) { // textarea 기입시 event handler
    setAnswer(e.target.value); // Textarea에 쓰인 string을 answer 상태 변수에 저장
  }

  /* UI 구성
  ** h2, p와 Form으로 구성
  ** Textarea, button의 disabled 옵션을 state로 따로 지정하지 않고 status가 submitting일 때만 비활성화하는 것으로 단순화
  **/
  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer} // 선언적 : 현재 상태의 'answer'값을 textarea에 표시할 것(how가 아닌 what을 지시)
          onChange={handleTextareaChange}
          disabled={status === 'submitting'} // 선언적 : status 값에 따라 textarea를 비활성화
        />
        /** cf) 위의 Textarea를 명령형 프로그래밍 방식으로 조작한다면 다음과 같은 로직이 된다.
        * if (textarea.value.length === 0) {
        *     button.disabled = true;
        * } else {
        *     button.disabled = false;
        * }
        **/
        <br />
        <button disabled={
          answer.length === 0 || // 선언적 : answer의 길이에 따라 button을 비활성화
          status === 'submitting' // 선언적 : status 값에 따라 버튼을 비활성화
        }>
          Submit
        </button>
        {error !== null && // 선언적 : error 값이 null이 아닐 때만 에러 메시지를 표시
          <p className="Error">
            {error.message} // 선언적 : 현재 상태의 'error' 메시지를 표시함 (어떤 메시지인지는 정해주지 않음..!)
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) { // 비동기 작업을 수행하기 위한 명령형 로직이 포함됨
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('Good guess but a wrong answer. Try again!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```
#### 🔅 컴포넌트에 시각적 상태가 많은 경우, 한번에 나타내주기
```javascript
// 이미 Form에 해당하는 컴포넌트는 작성되어있다고 가정
import Form from './Form.js';

let statuses = [
  'empty',
  'typing',
  'submitting',
  'success',
  'error',
];

export default function App() {
  return (
    <>
      {statuses.map(status => (
        <section key={status}>
          <h4>Form ({status}):</h4>
          <Form status={status} />
        </section>
      ))}
    </>
  );
}
```