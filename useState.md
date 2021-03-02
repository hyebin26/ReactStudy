### useState

리액트 Hook 중 하나로, 함수형 컴포넌트에서 상태를 관리하는 함수이다. 

```jsx
import { useState } from "react";

const Message = () => {
  const [message, setMessage] = useState("");
  return (
    <div>
      <input
        type="text"
        placeholder="Enter a message"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <p>
        <strong>{message}</strong>
      </p>
    </div>
  );
};
```

기본적인 문법은 배열에 구조로 되어있고 첫 상태를 useState다음 괄호에 설정할 수 있다.  그리고 useState는 배열을 return 한다. 이유는 객체에 비해 사용하기 쉽고 유연하기 떄문이다.

---

 2.  useState를 이용해 object state 관리하기

useState를 이용해 객체를 관리 할 때 꼭 생각해야 되는 것이 있다.

만약 사용자가 업데이트를 해도 같은 상태를 사용할 때 리액트는 리렌더링 시키지 않는다.

```jsx
const Message = () => {
  const [messageObj, setMessageObj] = useState({ message: "" });
  const handleChange = (e) => {
    messageObj.message = e.target.value;
    setMessageObj(messageObj); // 작동하지 않는다.
  };
  return (
    <div>
      <input
        type="text"
        placeholder="Enter a message"
        value={messageObj.message}
        onChange={handleChange}
      />
      <p>
        <strong>{messageObj.message}</strong>
      </p>
    </div>
  );
};
```

위에 예제는 객체를 관리할 때 자주하는 실수이다. 위에 예제는 새로운 객체를 만드는 것이 아니라 이미 존재하는 객체의 상태를 복재했다. 그러므로 리액트는 같은 객체로 받아들여 re-render 하지 않는다.

```jsx
const handleChange = (e) => {
    const newMessageObj = { message: e.target.value };
    setMessageObj(newMessageObj);
  };
```

위에 예제처럼 새로운 객체를 만드는 것이 효과적이다.

```jsx
const Message = () => {
  const [messageObj, setMessageObj] = useState({ message: "",id: 1 });
  const handleChange = (e) => {
    const newMessageObj = { message: e.target.value };
    setMessageObj(newMessageObj); 
  };
  return (
    <div>
      <input
        type="text"
        placeholder="Enter a message"
        value={messageObj.message}
        onChange={handleChange}
      />
      <p>
        <strong>{messageObj.id}:{messageObj.message}</strong>
      </p>
    </div>
  );
};
```

위에 예제는 useState에 id를 추가했다. 만약 id값은 그대로지만 message값을 유동적으로 바꾸고 싶다면 위에 예제처럼 하면 처음으로 지정한 id값만 출력되고 다음부터는 출력되지 않는다.

```jsx
const handleChange = (e) => {
    const val = e.target.value;
    setMessageObj((prevState) => ({ ...prevState, message: val }));
  };
```

...prevState 부분은 객체의 모든 properties를 가져오고 message: val 는 메세지 프로퍼티에 val을 덮어쓰는 것이다.

---

 3. 중복객체를 업데이트 하는 방법

```jsx
{
  'row1' : {
    'key1' : 'value1',
    'key2' : 'value2'
  },
  'row2' : {
    'key3' : 'value3',
    'key4' : 'value4'
  }
}

[
  ['value1','value2'],
  ['value3','value4']
]
```

문제는 다수의 배열과 중복 객체를 사용할 때 Object.assign 과 spread 문법은 깊은 카피 대신 얇은 카피를 한다는 것이다. 

> From the spread syntax documentation:
Spread syntax effectively goes one level deep while copying an array. Therefore, it may be unsuitable for copying multidimensional arrays, as the following example shows. (The same is true with Object.assign() and spread syntax.)

```jsx
const [messageObj, setMessageObj] = useState({
	author:'Hyebin',
	message:{
		id:1,
		text:""
	}
});
```

위에 예제에서 text 의 내용만 My message 로 바꾸고 싶다면 어떻게 해야 할까.

```jsx
// Wrong
setMessage(prevState => ({
  ...prevState,
  text: 'My message'
}));

// Wrong
setMessage(prevState => ({
  ...prevState.message,
  text: 'My message'
}));

// Wrong
setMessage(prevState => ({
  ...prevState,
  message: {
    text: 'My message'
  }
}));
```

알맞게 업데이트를 위해서는 기존의 객체를 전부 복사하고 새로운 객체를 다시 만들어야 한다. 

```jsx
// Correct
setMessage(prevState => ({
  ...prevState,           // 객체 전부를 복사
  message: {              // 업데이트를 할 객체를 다시 만들기
    ...prevState.message, // message객체를 복사
    text: 'My message'    // 업데이트할 내용 덮어쓰기
  }
}));
```

---

3. 다수의 변수 상태 혹은 다수의 변수가 포함된 객체 관리 

```jsx
const [id, setId] = useState(-1);
const [message, setMessage] = useState('');
const [author, setAuthor] = useState('');
=> 
const [messageObj,setMessageObj] = useState({
	id:1,
	message: '',
	author: ''
})
```

state가 한눈에 보기 쉽지만 중복 객체를 깊게 복사하다보면 리액트는 변하지 않은 것들도 re-render할 수도 있으므로 조심해야 한다.

그러므로 함께 변하는 state만을 묶어서 사용하는 것이 좋다.

@[https://reactjs.org/docs/hooks-faq.html#should-i-use-one-or-many-state-variables](https://reactjs.org/docs/hooks-faq.html#should-i-use-one-or-many-state-variables)

---

 4. useState 규칙 

- top level 에 호출해야 한다.
- 리액트 함수안에서만 호출해야 한다.

```jsx
class App extends React.Component {
  render() {
    const [message, setMessage] = useState( '' );

    return (
      <p>
        <strong>{message}</strong>
      </p>
    );
  }
}
```

```jsx
function getState() {
  const messageState = useState( '' );
  return messageState;
}
const [message, setMessage] = getState();
const Message = () => {
 /* ... */
}
```

위에 자바스크립트 함수, render에 선언한 useState는 에러가 나온다.

첫번째 규칙의 의미는 반복문, 가정문 또는 중복 함수 등등에 useState를 선언할 수 없다. 왜냐하면 React는  useState 함수를 호출하여 특정 상태 변수에 대한 올바른 값을 가져오는 순서에 의존하고 있기 떄문이다.

React Hook 일의 순서

1. React initializes the list of Hooks and the variable that keeps track of the current Hook
2. React calls your component for the first time
3. React finds a call to `useState`, creates a new Hook object (with the initial state), changes the current Hook variable to point to this object, adds the object to the Hooks list, and return the array with the initial state and the function to update it
4. React finds another call to `useState` and repeats the actions of the previous step, storing a new Hook object and changing the current Hook variable
5. The component state changes
6. React sends the state update operation (performed by the function returned by `useState`) to a queue to be processed
7. React determines it needs to re-render the component
8. React resets the current Hook variable and calls your component
9. React finds a call to `useState`, but this time, since there’s already a Hook at the first position of the list of Hooks, it just changes the current Hook variable and returns the array with the current state and the function to update it
10. React finds another call to `useState` and since a Hook exists in the second position, once again, it just changes the current Hook variable and returns the array with the current state and the function to update it

---

요약

- useState 은 함수에서 상태 관리를 할 수 있게 해주는 훅이다. 첫 번째 상태를 정할 수 있고 유동적으로 value를 관리 할 수 있다.
- 이전에 value를 사용하려면 이전에 상태를 받아와야 한다. ex) setMessage(prev⇒ prev + currentVal);
- 이전에 value와 같은 값을 사용하면 re-render되지 않는다.

참고:
<a href="https://blog.logrocket.com/a-guide-to-usestate-in-react-ecb9952e406c/">https://blog.logrocket.com/a-guide-to-usestate-in-react-ecb9952e406c/<a/>
