>[!Learning Goals]
>- What a reducer function is
>- How to refactor `useState` to `useReducer`
>- When to use a reducer
>- How to write one well

- 다음의 경우 reducer를 통해 모든 state update 로직 통합 및 해당 로직을 component 외부로 묶는 것 고려하기!
	1. state update가 많은 경우
	2. update logic이 여러 event handler로 퍼져있어 흐름을 읽기 어려운 경우
## 1. useState 대신 useReducer 사용하기
### Reducer ?
- state의 로직을 담당하는 함수
- js의 reduce() method에서 아이디어를 따 옴
	- 연산의 이전 상태와 새 요소를 가지고 값을 누적할 수 있으며, 여기에서 reduce로 넘겨주는 함수를 reducer라 함
		- 지금까지의 결과와 현재 item을 가지고 다음 결과로 넘겨줌
- 현재 state와 Action을 인자로 받아 다음 state를 출력하는 '순수' 함수
	- 입력값이 같으면 언제나 출력도 같아야 함
### 1) State setting을 action dispatch로 변경
- react가 어떤 일을 해야할지 setter로 지정하는 대신, setter를 동작시키기 위해 유저가 했을 행동을 action으로 만듦
	- ![[5) Extracting State Logic into a Reducer#🔅 기존의 state setting (in event handler)]]
	- ![[5) Extracting State Logic into a Reducer#🔅 변경 후의 action dispatch]]
### 2) Reducer 함수 작성
- 매개 변수 : 현재 state & action object
- 리턴값 : 다음 state
- ![[5) Extracting State Logic into a Reducer#🔅 state setting을 reducer로 변경 예시]]
### 3) 컴포넌트에서 Reducer 사용
- 순서
	1. useReducer hook import
	2. useState 대신 useReducer를 사용
- reducer에 해당하는 내용을 분리하여 컴포넌트 로직의 가독성을 더 높일 수 있음
- ![[5) Extracting State Logic into a Reducer#🔅 Component에서 Reducer 사용 예시]]
### 4) useState vs useReducer
- 상태 관리 복잡도와 App 규모에 따라 어떤 것을 사용할지 판단
- 구조 측면
	- useState: 초기 state만을 받고, state value와 state setter 리턴
	- useReducer: reducer 함수와 초기 state를 인자로 받고, state value와 dispatch function 리턴
		- dispatch function : 사용자의 action을 reducer에 내보내는 함수
- 코드 길이 측면
	- useState: 미리 작성해야 하는 코드가 적다
	- useReducer: reducer 함수와 action 전달 이벤트를 모두 작성해야 하므로 코드가 길어질 수 있음
- 가독성
	- useState: 간단한 구조일 때는 괜찮지만, 복잡하고 길어질 수록 로직 파악이 힘들어질 수 있다.
	- useReducer: 업데이트 로직과 이벤트 핸들러의 동작을 분리하여 어떤 일이 일어났는지 알기 편하다.
- debug
	- useState: 여러 곳에서 상태가 변경되는 경우(ex: state setter가 여러 곳에서 사용되고 있을 때) 어디에서 문제가 발생했는지 일일히 찾아야 하므로, 원인 파악이 어려울 수 있음
	- useReducer: reducer에 console.log를 추가하여 state 업데이트 중 어디에서 버그가 발생했는지 파악이 용이하지만, 살펴볼 코드의 양이 늘어날 수 있음
- Testing
	- useState: 간단한 상태 로직인 경우 testing 용이하지만, 특정 컴포넌트에 종속적이므로 테스팅 코드 재사용이 어려울 수 있음
	- useReducer: 컴포넌트에 종속적이지 않으므로, 복잡한 상태 로직의 분리 및 테스팅이 비교적 용이하다. 로직이 이미 분리되어있으므로 단위 테스트 작성 역시 편리하다.
## ------ example list ------
#### 🔅 기존의 state setting (in event handler)
```javascript
  // in App.js
  
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) { // 사용자가 버튼을 통해 해당 핸들러를 호출하는 경우 새 id, text와 done으로 이루어진 새 객체를 추가
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) { // 사용자가 버튼을 통해 해당 핸들러를 호출하는 경우 id가 동일한 요소를 찾아 task를 변경
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) { // 사용자가 버튼을 통해 해당 핸들러를 호출하는 경우 현재 task list에서 id가 동일한 요소만 제거하고 return
    setTasks(tasks.filter((t) => t.id !== taskId));
  }
  
```
#### 🔅 변경 후의 action dispatch
```javascript
// task를 추가 / 변경 / 삭제하는 action을 전달 → 사용자의 의도를 서술 (선언적...)

function handleAddTask(text) {  
	dispatch({  // dispatch 안에는 action object가 들어있음
		type: 'added',  // type: 행해질 일에 대한 정보를 포함하도록 작성
		id: nextId++,  
		text: text,  
	});  
}  

function handleChangeTask(task) { 
	dispatch({  // dispatch 안에는 action object가 들어있음
		type: 'changed',  // type: 행해질 일에 대한 정보를 포함하도록 작성
		task: task,  
	});  
}  

function handleDeleteTask(taskId) {  
	dispatch({  // dispatch 안에는 action object가 들어있음
		type: 'deleted',  // type: 행해질 일에 대한 정보를 포함하도록 작성
		id: taskId,  
	});
}
```
#### 🔅 state setting을 reducer로 변경 예시
```javascript
// switch문 사용 예시 (if-else도 괜찮음! 다만 가독성면에서 switch가 케이스에 대한 동작을 바로 인식할 수 있어 편함)
function tasksReducer(tasks, action) {  
	switch (action.type) {  
		case 'added': {  // 각 케이스의  braket은 scpoe를 만들어주기 위한 추가
			return [  
				...tasks,  
				{  
					id: action.id,  
					text: action.text,  
					done: false,  
				},  
			];  
		}  
		case 'changed': {  
			return tasks.map((t) => {  
				if (t.id === action.task.id) {  
					return action.task;  
				} else {  
					return t;  
				}  
			});  
		}  
		case 'deleted': {  
			return tasks.filter((t) => t.id !== action.id);  
		}  
		default: {  
			throw Error('Unknown action: ' + action.type);  
		} 
	} 
}
```
#### 🔅 Component에서 Reducer 사용 예시
```javascript
import { useReducer } from 'react'; // 1. useReducer 훅 import
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks); // 2. useState 대신 useReducer 사용: stateful value & 사용자의 action을 reducer에 내보내는 dispatch 함수 리턴, 변수에 할당하기

/* event handlers
** action(무슨 일이 일어났는지 지정한 것)을 전달
** reducer는 이벤트 핸들러가 전달한 action을 가지고 state를 어떻게 변경/응답할지 결정
**/
  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
```