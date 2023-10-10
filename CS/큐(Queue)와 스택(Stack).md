- 큐와 스택은 모두 데이터 구조의 종류이다.
## 1. 큐
### 특징
- 선입선출 (First In, First Out - FIFO)
- 두 쪽이 모두 뚫린 파이프같은 구조, 왼쪽에서 들어가 오른쪽으로 나옴
- 순차적 데이터 처리에 사용
### 구현 예시
```javascript
class Queue {
  constructor() {
    this.items = [];
  }
  
  // 요소를 큐의 끝에 추가
  enqueue(element) {
    this.items.push(element);
  }
  
  // 큐의 시작부터 요소를 제거하고, 제거한 요소를 반환
  dequeue() {
    if(this.isEmpty())
      return "Queue is empty";
    return this.items.shift();
  }
  
  // 큐의 첫 번째 요소를 확인
  front() {
    if(this.isEmpty())
      return "Queue is empty";
    return this.items[0];
  }
  
  // 큐가 비어 있는지 확인
  isEmpty() {
    return this.items.length == 0;
  }
  
  // 큐의 모든 요소를 문자열로 출력
  printQueue() {
    var str = "";
    for(var i = 0; i < this.items.length; i++)
      str += this.items[i] +" ";
    return str;
  }
}

// 예제 사용
var queue = new Queue();

console.log(queue.dequeue()); // Queue is empty

queue.enqueue(10);
queue.enqueue(20);
queue.enqueue(30);

console.log(queue.printQueue()); // 10 20 30
console.log(queue.dequeue()); // 10
console.log(queue.front()); // 20

```
## 2. 스택
### 특징
- 후입선출 (Last in, First out - LIFO)
- 아래쪽이 막힌 양동이 모양의 구조, 위로 들어가서 위로 나옴
### 구현 예시
```javascript
class Stack {
  constructor() {
    this.items = [];
  }
  
  // 요소를 스택의 상단에 추가
  push(element) {
    this.items.push(element);
  }
  
  // 스택의 상단에서 요소를 제거하고, 제거한 요소를 반환
  pop() {
    if (this.isEmpty())
      return "Stack is empty";
    return this.items.pop();
  }
  
  // 스택의 상단 요소를 확인
  peek() {
    if (this.isEmpty())
      return "Stack is empty";
    return this.items[this.items.length - 1];
  }
  
  // 스택이 비어 있는지 확인
  isEmpty() {
    return this.items.length == 0;
  }
  
  // 스택의 모든 요소를 문자열로 출력
  printStack() {
    let str = "";
    for (let i = 0; i < this.items.length; i++)
      str += this.items[i] + " ";
    return str;
  }
}

// 예제 사용
let stack = new Stack();

console.log(stack.pop()); // Stack is empty

stack.push(10);
stack.push(20);
stack.push(30);

console.log(stack.printStack()); // 10 20 30
console.log(stack.pop()); // 30
console.log(stack.peek()); // 20

```