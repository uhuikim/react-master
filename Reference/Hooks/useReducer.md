# useReducer

## **useReducer(reducer, initialArg, init?)**

```jsx
function reducer(state, action) {
}

const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
}
```

### 매개변수

- reducer : state가 업데이트되는 방식을 지정하는 함수. 순수 함수여야 하며, state와 액션을 인자로 받아야 하고, 다음 state를 반환해야 한다. state와 액션은 어떤 유형이든 가능
- initialArg : 초기 state가 계산되는 값. 모든 유형의 값일 수 있다. init 인자에 따라 초기 state를 계산하는 방법이 달라진다.
- init : 초기 state 계산 방법을 지정하는 초기화 함수.이것을 지정하지 않으면 초기 state는 initialArg로 설정된다. 그렇지 않으면 초기 state는 init(initialArg)를 호출한 결과로 설정 된다.
    
    ```jsx
    function createInitialState(username) {
    //
    }
    
    function TodoList({ username }) {
      const [state, dispatch] = useReducer(reducer, username, createInitialState);
    }
    ```
    

### 반환값

- state : 첫 번째 렌더링 중에는 init(initialArg) 또는 (init이 없는 경우) initialArg로 설정된다.
- dispatch : state를 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있는 함수

### 주의사항

- hook 임으로 컴포넌트의 최상위 레벨 또는 custom hook에서만 호출 가능. 반복문이나 조건문 내부에서는 호출 불가능

## dispatch

`useReducer` 가 반환하는 `dispatch` 함수를 사용하면 state를 다른 값으로 업데이트하고 다시 렌더링을 촉발할 수 있다. 화면에 표시되는 내용을 업데이트하려면 사용자가 수행한 작업을 나타내는 객체, 즉,액션을 사용하여 dispatch를 호출한다.

```jsx
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
 }
```

### 매개변수

- action : 사용자가 수행한 작업, 관용적으로 액션은 type 속성이 있는 객체 사용, 선택적으로 추가 정보 포함

### 반환값

없음

### 주의사항

- dispatch함수는 다음 렌더링에 대한 state 변수만 업데이트한다.  만약 dispatch함수를 호출한 후 state 변수를 읽으면, 호출 전 화면에 있던 이전 값이 계속 표시됨. 다음 state 값을 추측해야 하는 경우 reducer를 직접 호출하여 수동으로 계산할 수 있다.
    
    ```jsx
    function handleClick() {
      console.log(state.age);  // 42
    
      dispatch({ type: 'incremented_age' }); // Request a re-render with 43
      console.log(state.age);  // Still 42!
    
      setTimeout(() => {
        console.log(state.age); // Also 42!
      }, 5000);
    }
    
    // 다음 state 값 reducer를 직접 호출
    const action = { type: 'incremented_age' };
    dispatch(action);
    
    const nextState = reducer(state, action);
    console.log(state);     // { age: 42 }
    console.log(nextState); // { age: 43 }
    ```
    
- 새 값이 Object.is로 비교했을 때 현재 state와 동일하다면, React는 컴포넌트와 그 자식들을 다시 렌더링하는 것을 건너뛴다.
    
    ```jsx
    // 기존 state 객체를 변경하고 반환하면 React는 업데이트를 무시한다.
    function reducer(state, action) {
      switch (action.type) {
        case 'incremented_age': {
          // 🚩 Wrong: mutating existing object
          state.age++;
          return state;
        }
        // ...
      }
    }
    
    function reducer(state, action) {
      switch (action.type) {
        case 'incremented_age': {
          // ✅ Correct: creating a new object
          return {
            ...state,
            age: state.age + 1
          };
        }
     
      }
    }
    ```
    
- react는 state 업데이트를 일괄 처리한다.

## reducer

Reducer는 다음 state를 계산하고 반환한다.

```jsx
function reducer(state, action) {
	switch (action.type) {
    case 'incremented_age': {
      return {
        ...state, // Don't forget this!
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}

export default function Counter() {
	const [state, dispatch] = useReducer(reducer, { age: 42 });
	//
}
```

- useReducer 는 useState와 매우 유사하지만 이벤트 핸들러의 state 업데이트 로직을 컴포넌트 외부의 단일 함수로 옮길 수 있다.
- 관례상 switch 문으로 작성한다.
- 각 action은 하나의 사용자 상호작용을 설명해야한다.
- 새 state를 반환할때 기존 필드를 모두 복사하는지 확인하기
- switch에서 오류를 발생시키면 액션 유형이 case문 중 어느것과도 일치 하지 않을때 이유를 찾을 수 있다.

## 초기 state 재생성 방지하기

```jsx
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
```

- `createInitialState(username)`의 결과는 초기 렌더링에만 사용되지만, 이후의 모든 렌더링에서도 여전히 이 함수를 호출하게 된다.
- useReducer 세번째 인수에 초기화 함수 전달하기. 초기화 함수가 초기 state를 계산하는 데 아무런 정보가 필요하지 않은 경우, useReducer의 두 번째 인수로 null 을 전달할 수 있다.