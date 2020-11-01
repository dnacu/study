# Hooks API reference

## useState

```js
const [state, setState] = useState(initialState);
```

**리액트는 `setState`가 리렌더 시에 변경되지 않을 것을 보장한다.** 그렇기에 이를 dependency list에 넣을 필요가 없다.

### Functional updates

새로운 `state`의 계산에 이전 상태가 필요하다면, `setState`에 함수를 전달해라.

```js
setState((previousState) => {
  // use previousState logic...

  return nextState;
});
```

### Lazy initial state

```js
const [state, setState] = useState(someExpensiveComputation(props));

const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

함수실행(값)을 전달하면 매 리렌더마다 계산되지만, 함수를 전달하면 최초 렌더링 시에만 동작하고 그 이후로 무시된다.

### Bailing out of a state update

리액트는 state 변경을 감지하는 로직으로 `Object.is`(얕은비교, 주솟값 비교)를 사용한다. 동일한 값으로의 변경 시 자식 렌더링이나 함수실행 등을 회피한다.

## useEffect

기본적으로 모든 렌더링이 완료된 후에 실행된다.

### Cleaning up an effect

```js
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // Clean up the subscription
    subscription.unsubscribe();
  };
});
```

메모리 누수 방지를 위해 UI에서 컴포넌트를 제거하기 전에 수행된다.

### Timing of Effects

`componentDidMount와` `componentDidUpdate와는` 다르게, `useEffect로` 전달된 함수는 지연 이벤트 동안에 layout과 paint를 완료한 후 발생한다.

### Conditionally firing an effect

```js
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    subscription.unsubscribe();
  };
}, [props.source]);
```

두 번째 인자로 전달된 값들이 변경될 때만 실행할 수 있도록 한다.

> effect 함수 안에서 참조되는 모든 값은 의존성 값의 배열에 드러나야 한다.

## useContext

```js
const value = useContext(MyContext);
```

context 객체(React.createContext에서 반환된 값)을 받아 그 context의 현재 값을 반환한다. context의 현재 값은 트리 안에서 이 Hook을 호출하는 컴포넌트에 가장 가까이에 있는 `<MyContext.Provider>`의 value prop에 의해 결정된다.

상위 컴포넌트가 `React.memo`나 `shouldComponentUpdate`메소드를 사용하여 업데이트되지 않는다고 하더라도, 이 훅이 사용된 컴포넌트부터 리렌더링된다.

context의 값을 읽고, 변경을 구독하는 일만 할 수 있다. 값을 전달하기 위해서는 여전히 `<MyContext.Provider>`가 필요하다.

## useReducer

`(state, action) => newState` 형태의 `reducer`를 받아 `state`와 `dispatch`를 반환한다. **리액트는 `dispatch`가 리렌더 시에 변경되지 않을 것을 보장한다.**

복잡한 상태관리 로직(복잡한 객체 또는 다음 상태가 이전 상태에 의존적일 때)을 사용할 때 `useState`보다 바람직하다.

상태 로직을 변경하는 `deep update callback`함수(계산 + setState 등...) 전달 대신에 `dispatch`를 전달함으로써 성능의 최적화를 꾀할 수 있다.

> useState, useReducer로부터 반환받은 setState, dispatch 함수는 컴포넌트가 리렌더링 되어도 항상 같음을 보장한다. 이는 이 함수들을 하위 컴포넌트의 prop으로 전달했을 때, 컴포넌트가 업데이트되더라도 해당 props값이 변하지 않음을 의미하는 것이다.

### 번외: `someFunc={() => someFunc()}` 와 `someFunc={someFunc}`의 차이

**someFunc={() => someFunc()}**

인라인함수로 전달한 props는 컴포넌트가 리렌더링될 때마다 새로운 함수가 생성되기 때문에 하위컴포넌트에서는 props가 변경된 것처럼 보인다.

**someFunc={someFunc}**

어떤함수냐에 따라 다르다.

1. 컴포넌트 내에서 선언된 함수(메소드)일 때

클래스형 컴포넌트에서는 해당 메소드가 변하지 않기 때문에(새로 생성되지 않기 때문에) props의 변경이 없다.

함수형 컴포넌트에서는 리렌더 시 함수가 재실행됨에 따라 새로운 함수가 생성되고, props의 변화를 초래한다.
하지만 `useCallback` 훅으로 감싸져있다면, 미리 설정한 dependency 배열에 따라 달라진다.

> dependency 배열에 있는 값이 변경되었다면 새로운 함수가 생성되지만, 그렇지 않다면 같은 함수이므로 props의 변화를 일으키지 않을 것이다.

2. 함수형 컴포넌트에서 `useState` 훅을 통해 반환받은 set함수이거나, `useReducer`함수를 통해 반환받은 `dispatch`함수일 때
   언급한 위 두 함수는 리렌더 시에 변경되지 않는다. 그러므로 props의 변화를 일으키지 않는다.

### Specifying the initial state

```js
const [state, dispatch] = useReducer(reducer, { count: initialCount });
```

초기값을 두 번째 인자로 전달하는 방법.

### Lazy initialization

```js
function init(initialCount) {
  return { count: initialCount };
}

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    case "reset":
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({ type: "reset", payload: initialCount })}
      >
        Reset
      </button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
}
```

init함수를 세 번째 인자로 전달함으로써 지연생성할 수 있다.(?)
reducer 외부에서 초기 state를 계산하는 로직을 추출할 수 있도록 한다. 또한, 어떤 행동에 대한 대응으로 나중에 state를 재설정하는 데에도 유용하다.

### Bailing out of a dispatch

state 변경을 감지하는 로직으로 `Object.is`(얕은비교, 주솟값 비교)를 사용한다. 동일한 값으로의 변경 시 자식 렌더링이나 함수실행 등을 회피한다.
