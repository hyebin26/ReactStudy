# Redux Toolkit

## Reducers and Immutable Updates

리덕스에 가장 중요한 규칙은 우리의 리듀서는 절대 기존/현재의 state value 복제하는 것을 허락하지 않는다.

```jsx
state.value = 123 // X
```

리덕스에서 state를 안되는 이유는 여러가지가 있다.

- 버그를 발생시킨다. ex) UI가 가장 최근의 value 보여주는 것을 적절하게 업데이트 하지 않을 수 있다.
- 어떻게, 왜 상태가 업그레이드 되었는지 이해하는 것을 어렵게 만든다.
- test하기 어렵게 만든다.
- "time- travel debugging"사용하는 것을 못하게 한다.
- 리덕스에 기본 패턴 그리고 의도된 것에 반대된다.

## Immutable Updates with Immer

Immer은 불변 업데이트 로직의 처리과정을 간편하게 하는 라이브러리이다.

Immer는 원래의 state와 콜백함수 두개의 인자를 받는 함수 produce를 제공한다.   콜백 함수는 그 state에 "draft" 버전으로 주고. 이것은 그 draft value를 복제하는 코드를 작성하는 것은 안전하다. Immer는 draft value를 변경하려는 모든 시도를 추적 한 다음, 안전한 결과를 불변 등가물을 사용하여 복제한다.

## Redux Toolkit and Immer

Redux Toolkit은 immer가 내장되어 있다. 그래서, createReducer를 지나가는 reducer 함수 안에 있는 상태를 복제하는 것이 안전하다. 

createSlice은 createReducer를 사용하고, 그러므로 상태를 복제하는 것은 안전하다.

```jsx
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    todoAdded(state, action) {
      state.push(action.payload)
    },
  },
})
```

이것은 심지어 리듀서 함수가 createSlice/createReducer 외부에서 정의되었어도 적용된다. 예를 들어 너는 상태를 복제하는 재사용가능한 reducer 함수를 가질 수 있다.

```jsx
const addItemToArray = (state,action) =>{
	state.push(action.payload)
}

const todoSlice = createSlice({
	name : 'todos',
	initialState:[],
	reducer:{
		todoAdded : addItemToArray,
	}
})
```

이것은 작동된다. 왜냐하면 "mutating"로직은 이것이 실행될 때 immer produce 메쏘드에 적용되기 때문이다.

- "mutating"로직은 오직 immer안에 둘러싸여 있을 때만 작동한다는 것을 기억해라.

## Immer Usage Patterns

Redux Toolkit에서 Immer를 사용할 때 알아야 할 몇 가지 유용한 패턴과 주의해야 할 사항이 있다.

### Mutating and Returning State

Immer는 중복된 필드에 할당하거나 value를 복제하는 함수를 호출하거나 이미 존재하는 초안 상태 값을 복제하는 시도를 추적하면서 작동한다.  그것은 state는 Immer가 변화의 시도를 보기위해 반드시 JS객체 또는 배열이여야 한다는 것을 의미한다.( slice state를 문자열 또는 boolean 같은 원시타입으로 가질 수 있다. 하지만 원시타입은 절대로 복제될 수 없다. 너가 할 수 있는 것은 새로운 value를 리턴하는 것이다.)

어떤 리듀서의 경우에도, Immer는 존재하는 state를 복재하거나 너 스스로 새로운 value를 구성하고 이것을 리턴하는 것을 기대한다. 그러나 같은 함수에서는 아니다. 예를 들면,
```jsx
const todoSlice = createSlice({
	name: "todos",
	initialState:[],
	reducers:{
		todoAdded(state,action){
      //기존 상태를 "Mutate"하고 반환 값이 필요하지 않는다.
		  state.push(action.payload) 
    },
    todoDeleted(state,action.payload){
      // 불변으로 새로운 배열을 만들고 이것을 리턴한다.
      return state.filter(todo => todo.id !== action.payload)
    }
  }
})
```

그러나, 변경 불가능한 업데이트를 사용하여 작업의 일부를 수행 한 다음 "mutation"를 통해 결과를 저장할 수 있다. 이에 대한 예는 중첩 배열을 filter하는 것이다.

```jsx
const todosSlice = createSlice({
  name: 'todos',
  initialState: {todos: [], status: 'idle'}
  reducers: {
    todoDeleted(state, action.payload) {
      //새로운 배열을 불변으로 만든다.
      const newTodos = state.todos.filter(todo => todo.id !== action.payload)
      // 기존의 존재하는 상태를 "Mutate" 하고 새로운 배열을 저장한다.
      state.todos = newTodos
    }
  }
})
```

화살표 함수안에서 mutating 상태를 변경하면이 규칙이 위반되고 오류가 발생합니다.이것은 상태와 함수의 호출은 값을 리턴할 수 있고 Immer는 복제의 시도와 새로운 리턴된 값을 둘다 보지만 결과로 사용되는 값을 모르기 때문이다. 약간의 잠재적인 해결책은 값을 리턴하는 것을 가지는 것을 스킵하기 위해 void키워드를 사용하거나 중괄호를 사용하여 화살표 함수에 본문을 제공하고 리턴 값은 제공하지 않는다.

```jsx
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    // ❌ ERROR
    brokenReducer: (state, action) => state.push(action.payload),
    // ✅ void키워드를 사용해서 안전하다.
    fixedReducer1: (state, action) => void state.push(action.payload),
    // ✅ 중괄호는 이것을 함수 본문으로 만들고 리턴하지 않는다.
    fixedReducer2: (state, action) => {
      state.push(action.payload)
    },
  },
})
```

반면에 중첩된 불변 업데이트 로직을 작성하는 것은 어렵지만, 개별 필드를 할당하는 것보다 한 번에 여러 필드를 업데이트하는  객체 spead opration을 수행하는 것이 더 간단한 경우가 있다.

```jsx
function objectCaseReducer1(state,action){
	const {a,b,c,d} = action.payload;
  return {
    ...state,
    a,
    b,
    c,
    d,
  }
}

function objectCaseReducer2(state, action) {
  const { a, b, c, d } = action.payload
  // 이것은 작동하나 계속해서 state.x 를 반복하고 있다.
  state.a = a
  state.b = b
  state.c = c
  state.d = d
}
```

## Resetting and Replacing State

가끔 전체의 상태를 재배치하는 것을 원할 수 있다. 너가 새로운 데이터를 로드하거나 너가 상태를 처음의 상태로 돌아가는 것을 원할 수 있다.

### Caution

> 대부분의 실수는 state = someValue로 직접적으로 할당하는 것이다. 이것은 작동하지 않는다. 이것은 오직 local state가 다른 참조를 가르킨다. 이는 메모리의 기존 상태 객체 / 배열을 변경하거나 완전히 새로운 값을 반환하지 않으므로 Immer는 실제 변경을 수행하지 않는다.

대신 기존 상태를 바꾸려면 직접적으로 새 값을  리턴해야 한다.

```jsx
const initialState =[];
const todoSlice = createSlice({
	name:"todos",
	initialState,
	reducers: {
    brokenTodosLoadedReducer(state, action) {
      // ❌ ERROR: 복제하지 않거나 새로운 것을 리턴하지 않았다.
      state = action.payload
    },
    fixedTodosLoadedReducer(state, action) {
      // ✅ CORRECT: 이전의 상태를 바꾸기 위해 새로운 값을 리턴했다.
      return action.payload
    },
    correctResetTodosReducer(state, action) {
      // ✅ CORRECT: 이전의 상태를 바꾸기 위해 새로운 값을 리턴했다.
      return initialState
    },
  },
})
```

## Updating Nested Data

Immer는 중첩된 데이터 업데이트를 크게 단순화한다. 중첩된 객체들과 배열도 프록시로 래핑되고 초안화되며 중첩된 값을 자체 변수로 가져온 다음 변경하는 것이 안전하다.

그러나 이것은 여전히 객체와 배열에만 적용된다. 원시값을 우리의 변수로 가져 와서 업데이트하려고하면 Immer는 둘러쌀것이 없고 업데이트를 추적 할 수 없다.

```jsx
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    brokenTodoToggled(state, action) {
      const todo = state.find((todo) => todo.id === action.payload)
      if (todo) {
        // ❌ ERROR: Immer는 원시값의 업데이트를 추적할 수 없다.
        let { completed } = todo
        completed = !completed
      }
    },
    fixedTodoToggled(state, action) {
      const todo = state.find((todo) => todo.id === action.payload)
      if (todo) {
        // 이 객체는 여전히 프록씨로 둘러싸여 있고, 우리는 이것을 복제할 수 있다.
        todo.completed = !todo.completed
      }
    },
  },
})
```

여기에 문제가 있다. Immer는 상태에 새로 삽입 된 개체를 래핑하지 않습니다. 대부분의 경우 이것은 중요하지 않지만 값을 삽입하고 다음 추가 업데이트를 수행하려는 경우가있을 수 있습니다.

이와 관련하여 RTK의 createEntityAdapter 업데이트 함수는 독립형 리듀서 또는 "mutating"업데이트 함수로 사용될 수 있다. 이러한 함수는 주어진 상태가 초안으로 래핑되었는지 여부를 확인하여 "변형"할지 아니면 새 값을 반환할지 여부를 결정한다. 만약 리듀서 내에서 이러한 함수를 직접 호출하는 경우 초안 값 또는 일반 값을 전달하는지 확실히 알아야 한다.

마지막으로, Immer가 자동으로 중첩된 객체 또는 배열을 생성하지 않는다는 점에 유의할 가치가 있다. 너는 직접 생성해야한다. 예를 들어, 중첩된 배열이 포함 된 조회 테이블이 있고 해당 배열 중 하나에 항목을 삽입하려고한다고 가정해 보겠다. 해당 배열의 존재를 확인하지 않고 무조건 삽입을 시도하면 배열이 존재하지 않을 때 논리가 충돌한다. 대신 배열이 먼저 존재하는지 확인해야한다.

```jsx
const itemsSlice = createSlice({
  name: 'items',
  initialState: { a: [], b: [] },
  reducers: {
    brokenNestedItemAdded(state, action) {
      const { id, item } = action.payload
      // ❌ ERROR: 만약 `id`에 대한 배열이 없으면 충돌한다.
      state[id].push(item)
    },
    fixedNestedItemAdded(state, action) {
      const { id, item } = action.payload
      // ✅ CORRECT: 중첩된 배열이 항상 먼저 존재하도록 해야한다. 
      if (!state[id]) {
        state[id] = []
      }
      state[id].push(item)
    },
  },
})
```
