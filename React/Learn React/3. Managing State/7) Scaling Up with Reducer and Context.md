>[!Learning Goals]
> -  How to combine a reducer with context
> - How to replace repetitive prop passing with context
> - Common use cases for context
> - Common alternatives to context

- Reducer와 context를 함께 사용하여 복잡한 화면의 State를 관리해보자!
## 1. Combining a reducer with context
- 기본적인 reducer에서는 event handler를 통해 연산에 필요한 state와 dispatch를 제어
	- 다른 component들에서 해당 dispatch 로직을 사용하고자 할 때, 필요한 component들에 handler를 일일히 넘겨줘야 함!
	- 간단한 로직 / 트리 구조의 UI인 경우 괜찮지만, 복잡해질 수록 전달과 제어가 어려워짐
- state와 dispatch를, props가 아닌 context에 넣어 전달한다면 불필요한 prop drilling을 줄일 수 있음
### 1) context 생성
- state를 전달하는 context와, action을 dispatch하는 context를 개별적으로 작성
- 예시
	- TasksContext : 현재 task 리스트를 제공하는 컨텍스트
	- TasksDisptachContext : 컴포넌트에서 action을 dispatch 하는 함수 제공
	- (예제에서는 해당 context들의 재사용성을 고려해 별개 파일로 쪼개고 export / import함)
	- ![[7) Scaling Up with Reducer and Context#🔅 Create the context - Contexts.js]]
### 2) state와 dispatch를 context에 넣기
- 만든 context를 불러와 state와 action을 value로 지정하고 context 제공 (하위 tree 전체에 전달)
	- ![[5) Extracting State Logic into a Reducer#🔅 state setting을 reducer로 변경 예시]]
	- ![[7) Scaling Up with Reducer and Context#🔅 Put state and dispatch into context - App.js]]
### 3) context 제공 하위 트리 전역에서 context 사용하기
- 필요한 컴포넌트에서 useContext를 이용하여 전달된 context 사용 (이벤트 핸들러 등의 props 없이😎)
	- tasks list : `useContext(TasksContext)`
	- tasks list 업데이트를 위한 dispatch 함수 : `useContext(TasksDispatchContext)`
- ![[7) Scaling Up with Reducer and Context#🔅 Use context anywhere in the tree - AddTask.js]]
## 2.Provider : reducer와 context를 묶어 관리하기
- context와 reducer를 한 파일에 묶어 데이터를 제공하는 로직을 분리
	- 데이터 로직이 분리되어있으므로 Provider가 아닌 컴포넌트에선 UI 관리에 집중할 수 있음
- Provider 제작 step
	- useReducer와 context 제공 부분을 묶어 'Provider'로 만들기
		- ![[7) Scaling Up with Reducer and Context#🔅 Provider 제작 - Provider 예시]]
		- ![[7) Scaling Up with Reducer and Context#🔅 Provider 제작 - UI 파트 예시]]
	- useContext를 custom hook으로 만들어 내보내기
		- ![[7) Scaling Up with Reducer and Context#🔅 Provider 제작 - component에서 context를 이용하도록 하기 위한 custom hook 만들기]]
		- ![[7) Scaling Up with Reducer and Context#🔅 Provider 제작 - component에서 context 사용시 custom hook 이용 예시]]

## ------ example list ------
#### 🔅 Create the context - Contexts.js
```javascript
import { createContext } from "react"

export const TasksContext = createContext(null) // 실제 context 값은 taskApp 컴포넌트에서 전달하도록 할 예정
export const TasksDispatchContext = createContext(null) // 실제 context 값은 taskApp 컴포넌트에서 전달하도록 할 예정
```
#### 🔅 Put state and dispatch into context - App.js
```javascript
import { useReducer } from "react"
import { TasksContext, TasksDispatchContext } from "./Contexts.js"

const initialTasks = [
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];

export default function TaskApp() {
	const [tasks, dispatch] = useReducer(tasksReducer, initialTasks)
	...
	return (
		<TasksContext.Provider value={tasks}> // 사용할 state
			<TasksDispatchContext.Provider value={dispatch}> // dispach 함수: reducer
				...
			</TasksDispatchContext.Provider>
		</TasksContext.Provider>
	)
}
```
#### 🔅 Use context anywhere in the tree - AddTask.js
```javascript
const INIT_INPUT = ""
export default function AddTask() { // task add 기능이 있는 component
	const [text, setText] = useState(INIT_INPUT)
	const dispatch = useContext(TasksDispatchContext) // !!context 사용 예시
	...
	return (
		...
		<button onClick={() => { // add 버튼을 클릭했을 때의 동작
			setText(INIT_INPUT) // task 상세 string을 리셋
			dispatch({ // action object로 추가될 task의 정보를 서식대로 넣어주기
				type: "added",
				id: nextId++,
				text: text
			})
		}}>Add</button>
	)
}
```
#### 🔅 Provider 제작 - Provider 예시
```javascript
 /* TasksContext.js
 ** 기존 TaskApp 컴포넌트 안에 있던 reducer 및 provider 제공 로직을 가져오되,
 ** children을 받고 그 children에서 데이터를 사용할 수 있도록 한다.
 ** 따라서 결국 데이터를 제공하는 로직이 UI 로직과 완전히 분리된다.
 */
export function TasksProvider({ children }) {
	const [tasks, dispatch] = useReducer(taskReducer, initialTasks)
	return (
		<TasksContext.Provider value={tasks}> // 사용할 state
			<TasksDispatchContext.Provider value={dispatch}> // dispach 함수: reducer
				{ children } // context로 인해 이 children component들에서는 state와 dispatch를 제한 없이 사용 가능
			</TasksDispatchContext.Provider>
		</TasksContext.Provider>
	)
}
```
#### 🔅 Provider 제작 - UI 파트 예시
```javascript
/* App.js
** providef로 인해 TaskApp이 UI만을 담당하게 된 예시
*/
import AddTask from "./Addtask.js"
import TaskList from "./TaskList.js"
import { TasksProvider } from "./TasksContext.js"

export default function TaskApp() {
	return (
		<TasksProvider> // provider가 모든 데이터 로직을 담당 : TaskApp 내에서의 데이터 로직 사라짐
			<h1>Day off in Kyoto</h1>
			<AddTask /> // children인 AddTask와 TaskList 어디에서든 prop 없이 dispatch와 tasks를 사용
			<TaskList />
		</TasksProvider>
	)
}
```
#### 🔅 Provider 제작 - component에서 context를 이용하도록 하기 위한 custom hook 만들기
```javascript
/* TasksContext.js
** 데이터 로직을 관리하는 곳에서 context 사용을 위한 custom hook 만들기
*/
...
export function useTasks() {
	return useContext(TasksContext)
}

export function useTasksDispatch() {
	return useContext(TasksDispacthContext)
}
```
#### 🔅 Provider 제작 - component에서 context 사용시 custom hook 이용 예시
```javascript
/* TaskList.js : 입력된 Task list를 보여주고, task를 수정 및 삭제하는 기능을 탑재 */

import { useState } from 'react';
import { useTasks, useTasksDispatch } from './TasksContext.js';

export default function TaskList() {
  const tasks = useTasks(); // task list를 그려주기 위해 custom hook 이용
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useTasksDispatch(); // task의 수정 및 삭제 기능을 위해 custom hook 이용
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            dispatch({ // useTasksDispatch로 인해 dispatch를 자유롭게 사용 가능
              type: 'changed',
              task: {
                ...task,
                text: e.target.value
              }
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          dispatch({ // useTasksDispatch로 인해 dispatch를 자유롭게 사용 가능
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked
            }
          });
        }}
      />
      {taskContent}
      <button onClick={() => {
        dispatch({ // useTasksDispatch로 인해 dispatch를 자유롭게 사용 가능
          type: 'deleted',
          id: task.id
        });
      }}>
        Delete
      </button>
    </label>
  );
}
```
