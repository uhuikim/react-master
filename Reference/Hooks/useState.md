# useState

## useState(initialState)

`initialState`

- 초기에 state를 설정한 값.
- 값은 모든 데이터 타입이 허용되지만, 함수에 대해서는 특별한 동작이 있다.
- 초기 렌더링 이후에는 무시가 된다.
[props를 state로 설정하면 안되는 이유](https://react-ko.dev/learn/choosing-the-state-structure#don-t-mirror-props-in-state)

`initialState로 함수가 넘어오는 경우`

함수를 InitialState로 전달하면 이를 초기화 함수로 취급한다.

1. 순수해야함
2. 인자를 받으면 안된다.
3. 반드시 어떤 값을 반환해야한다.

react는 컴포넌트를 초기화 할 때 초기화 함수를 호출하고, 그 반환값을 초기 state로 저장한다.

## 반환값

```jsx
const [age, setAge] = useState(28);
```

- state : 첫 번째 렌더링 중에는 전달한 InitialState와 일치한다.
- set 설정자 함수 : 다른 값으로 업데이트하고 리렌더링을 촉발한다.

## 주의사항

- useState는 훅이므로 컴포넌트의 최상위 레벨이나 직접 만든 훅에서만 호출 할 수 있다.
- 반복문이나 조건문안에서는 호출 할 수 없다.

## set 설정자 함수

state를 다른 값으로 업데이트하고 리렌더링을 촉발 할 수 있다.

```jsx
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor');
  setAge(a => a + 1);
  // ...
```

### 매개변수

모든 데이터 타입의 값과 함수를 넘길 수 있다. 

`업데이터 함수`

함수를 전달하면 업데이터 함수로 취급된다. 

- 대기중인 state를 유일한 인수로 사용
- 다음 state 반환

`업데이터 함수 동작`

- React는 업데이터 함수를 대기열에 넣고 컴포넌트를 리렌더링 한다.
- 렌더링 중에 React는 대기열에 있는 모든 업데이터를 이전 state에 적용하여 다음 state를 계산한다.

### 주의사항

- set 함수는 다음 렌더링에 대한 state 변수만 업데이트한다.set 함수를 호출한 후에도 state 변수에는 여전히 이전 값이 담겨져 있다.
    
    ```jsx
    function handleClick() {
      setName('Robin');
      console.log(name); // Still "Taylor"!
    }
    
    // 다음 state를 사용해야하는 경우, set 함수에 전달하기 전 변수에 저장하기
    const nextCount = count + 1;
    setCount(nextCount);
    
    console.log(count);     // 0
    console.log(nextCount); // 1
    ```
    
- 사용자가 제공한 새로운 값이 Object.is에 의해 현재 state와 동일하다고 판정되면, react는 컴포넌트와 그 자식들을 리렌더링하지 않는다.
    
    ```jsx
    obj.x = 10;  // 🚩 잘못된 방법: 기존 객체를 변이    
    setObj(obj); // 🚩 아무것도 하지 않습니다.
    
    // ✅ 올바른 방법: 새로운 객체 생성
    setObj({
      ...obj,
      x: 10
    });
    ```
    
- react는 state 업데이트를 일괄처리한다. 모든 이벤트 핸들러가 실행되고 set 함수를 호출한 후에 화면을 업데이트 한다.
- DOM에 접근하기 위해 React가 화면을 더 일찍 업데이트하도록 강제해야 하는 경우, flushSync를 사용할 수 있다.
- 렌더링 도중 set 함수를 호출하는 것은 현재 렌더링 중인 컴포넌트 내에서만 허용된다. 이 경우 React는 해당 출력을 버리고 즉시 새로운 state로 다시 렌더링을 시도한다.

## 이전 state를 기반으로 state 업데이트하기

```jsx
const [age, setAge] = useState(42);

function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

set 함수를 호출해도 이미 실행중인 코드에서 age state 변수가 업데이트 되지 않는다. 

`업데이터 함수`

```jsx
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

- 이 함수는 대기 중인 state를 가져와서 다음 state를 계산합니다.

### 항상 업데이터를 사용하는 것이 더 좋을까?

- 꼭 그래야하는 것은 아니다. 컴포넌트가 리렌더링을 하면 해당 state에는 값이 반영이 되어있을 것이라 다음 핸들러에서는 문제 없이 작동할 것이다.
- 의존성을 탈피하고 싶을 경우에도 사용 할 수 있다.
- 동일한 이벤트 내에서 여러 업데이트를 수행하는 경우에는 업데이터가 도움이 된다.
- state 변수 자체에 접근하는 것이 어려운 경우에도 유용하다.
- 코드의 일관성을 위해 이전 state가 사용 될 경우 업데이터를 사용하는 것이 합리적이다.

## 객체 및 배열 state 업데이트

- react에서 state는 읽기 전용으로 간주되므로 기존 객체를 변이하지 않고, 교체를 해야한다.

```jsx
// 🚩 state 안에 있는 객체를 다음과 같이 변이하지 마세요: 
form.firstName = 'Taylor';

// ✅ 새로운 객체로 state 교체합니다
setForm({
  ...form,
  firstName: 'Taylor'
});
```

## 초기 state 다시 생성하지 않기

```jsx
// 모든 렌더링에서 이 함수를 호출한다.
const [todos, setTodos] = useState(createInitialTodos());
// 초기화 함수로 전달해라
const [todos, setTodos] = useState(createInitialTodos);
```

`createInitialTodos()`의 결과는 초기 렌더링에만 사용되지만, 여전히 모든 렌더링에서 이 함수를 호출하게 된다.

초기화 함수를 전달해라!

함수를 useState에 전달하면 React는 알아서 초기화 중에만 함수를 호출한다.

## key로 state 재설정하기

컴포넌트에 다른 key를 전달하여 컴포넌트의 state를 재설정할 수 있다.

key가 변경되면 컴포넌트를 처음부터 다시 생성하므로 state가 초기화 된다.

## 이전 렌더링에서 얻은 정보 저장하기

희귀하게 렌더링에 대한 응답으로 state를 조정해야하는 경우가 있다. 

ex) count가 마지막 변경 이후 증가 또는 감소했는지 표시하고 싶은 경우

```jsx
export default function CountLabel({ count }) {
	// props로 받은 이전 count값을 저장한다.
	const [prevCount, setPrevCount] = useState(count);
	// count의 증가 또는 감소 여부 추적
	const [trend, setTrend] = useState(null);
	  
	if (prevCount !== count) {
	    setPrevCount(count);
	    setTrend(count > prevCount ? 'increasing' : 'decreasing');
	  }

  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}
```

렌더링 하는 동안 set함수를 호출하기

- prevCount !== count 와 같은 조건이 없을 경우 리렌더링을 반복하다 터질 것이다.
- 오직 현재 렌더링 중인 컴포넌트의 state만 업데이트 할 수 있다. 렌더링 중에 다른 컴포넌트의 set 함수를 호출하는 것은 에러다.
- 렌더링 도중 set 함수를 호출하면 React는 컴포넌트가 return 문으로 종료된 직 후, 자식을 렌더링하기 전에 해당 컴포넌트를 리렌더링 한다. 따라서, Effect에서 state를 업데이트 하는 것 보다는 낫다.

렌더링 하지 않는 애들을 state로 두는 것은 효율적이지 않다. 리팩토링을 해보자.

```jsx
export default function CountLabel({ count }) {
	const prev = useRef(count)
	
	useEffect(()=>{
		prev.current = count;
	},[count])

	const trend = count > prev.current ? 'increasing' : 'decreasing'
  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}

```

## state 값으로 함수를 설정하고 싶은 경우

```jsx
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}

```

함수를 값으로 전달하면 React는 someFunction을 초기화 함수로 여기고, someOtherFunction은 업데이터 함수라고 받아들이며, 따라서 이들을 호출해서 그 결과를 저장하려고 시도한다. 정말로 함수를 저장하길 원한다면, 두 경우 모두 함수 앞에 () =>를 넣어야 한다. 그러면 React는 전달한 함수를 값으로써 저장한다.

```jsx
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```