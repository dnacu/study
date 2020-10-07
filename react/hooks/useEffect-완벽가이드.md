# useEffect 완벽 가이드

## TL;DR

### `useEffect`로 `componentDidMount`동작 흉내내기

`useEffect(fn, [])`으로 가능하다. 하지만 `componentDidMount`와 달리 초기의 `props`와 `state`를 잡아둔다. 만약 최신의 상태를 원한다면, `ref`에 담아둘 수 있다.

### 왜 가끔 이펙트 안에서 이전 `state`나 `props`값을 참조할까?

이펙트는 언제나 자신이 정의된 블록 안에서 렌더링이 일어날 떄마다 `props`와 `state`를 지켜보기 때문. 의존성 배열에 해당 값을 추가하거나 ref를 활용하여 관리할 수 있다.

## 모든 렌더링은 고유의 `props`와 `state`가 있다.

```js
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}

// 처음 랜더링 시
function Counter() {
  const count = 0; // useState() 로부터 리턴
  // ...
  <p>You clicked {count} times</p>;
  // ...
}

// 클릭하면 함수가 다시 호출된다
function Counter() {
  const count = 1; // useState() 로부터 리턴
  // ...
  <p>You clicked {count} times</p>;
  // ...
}

// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  const count = 2; // useState() 로부터 리턴
  // ...
  <p>You clicked {count} times</p>;
  // ...
}
```

`state`를 업데이트 할 때마다 리액트는 컴포넌트를 호출하고, 매 렌더 결과물은 고유의 `counter`상태값을 **살펴본다.** 그 값은 함수 안에 상수로 존재하는 값이다. 이 값은 시간이 지남에 따라 바뀌는 값이 아니라 컴포넌트가 다시 호출되고, 각각의 랜더링마다 격리된 고유의 `count`값을 **보는** 것이다.

특정 랜더링 시 그 내부에서 `props`와 `state`는 영원히 같은 상태로 유지된다.

```js
function sayHi(person) {
  const name = person.name;
  setTimeout(() => {
    alert("Hello, " + name);
  }, 3000);
}

let someone = { name: "Dan" };
sayHi(someone);

someone = { name: "Yuzhi" };
sayHi(someone);

someone = { name: "Dominic" };
sayHi(someone);
```

`() => { alert('Hello, ' + name); }` 이 콜백은 각 호출 시 마다 고유의 `name`값을 **본다.**

> 이와 비슷하게 생각하면 이해가 잘 된다.

## 모든 렌더링은 고유의 이펙트를 가진다.

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}

// 최초 랜더링 시
function Counter() {
  // ...
  useEffect(
    // 첫 번째 랜더링의 이펙트 함수
    () => {
      document.title = `You clicked ${0} times`;
    }
  );
  // ...
}

// 클릭하면 함수가 다시 호출된다
function Counter() {
  // ...
  useEffect(
    // 두 번째 랜더링의 이펙트 함수
    () => {
      document.title = `You clicked ${1} times`;
    }
  );
  // ...
}

// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  // ...
  useEffect(
    // 세 번째 랜더링의 이펙트 함수
    () => {
      document.title = `You clicked ${2} times`;
    }
  );
  // ..
}
```

이펙트에도 똑같은 개념이 적용된다. 이벤트 핸들러는 **그 렌더링에 속한 count 상태를 본다!** 이펙트 함수 자체가 매 렌더링마다 별도로 존재한다.

### 첫 번째 렌더링

- 리액트: state가 0 일 때의 UI를 보여줘.
- 컴포넌트
  - 여기 랜더링 결과물로 `<p>You clicked 0 times</p>` 가 있어.
  - 그리고 모든 처리가 끝나고 이 이펙트를 실행하는 것을 잊지 마: `() => { document.title - = 'You clicked 0 times' }`.
- 리액트: 좋아. UI를 업데이트 하겠어. 이봐 브라우저, 나 DOM에 뭘 좀 추가하려고 해.
- 브라우저: 좋아, 화면에 그려줄게.
- 리액트: 좋아 이제 컴포넌트 네가 준 이펙트를 실행할거야.
  - `() => { document.title = 'You clicked 0 times' }` 를 실행하는 중.

### 버튼 클릭 시

- 컴포넌트: 이봐 리액트, 내 상태를 1 로 변경해줘.
- 리액트: 상태가 1 일때의 UI를 줘.
- 컴포넌트
  - 여기 랜더링 결과물로 `<p>You clicked 1 times</p>` 가 있어.
  - 그리고 모든 처리가 끝나고 이 이펙트를 실행하는 것을 잊지 마: `() => { document.title - = 'You clicked 1 times' }`.
- 리액트: 좋아. UI를 업데이트 하겠어. 이봐 브라우저, 나 DOM에 뭘 좀 추가하려고 해.
- 브라우저: 좋아, 화면에 그려줄게.
- 리액트: 좋아 이제 컴포넌트 네가 준 이펙트를 실행할거야.
  - `() => { document.title = 'You clicked 1 times' }` 를 실행하는 중.

## 모든 렌더링은 고유의... 모든 것을 가지고 있다.

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      console.log(`You clicked ${count} times`);
    }, 3000);
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}

// 클래스 컴포넌트로 만들었을 때의 lifecycle method
componentDidUpdate() {
  setTimeout(() => {
     console.log(`You clicked ${this.state.count} times`);
  }, 3000);
}
```

둘이 결과가 다르다. 함수형 컴포넌트에서는 순차적으로 출력되나(**클로저**), 클래스 컴포넌트에서는 그렇지 못하다. 리액트가 `this.state`가 최신 상태를 가리키도록 변경하기 때문이다.

```js
componentDidUpdate() {
  const count = this.state.count
  setTimeout(() => {
     console.log(`You clicked ${count} times`);
  }, 3000);
}
```

클로저를 이용하여 클래스 컴포넌트에서 원하는대로 동작하도록 변경할 수 있다.

## 흐름을 거슬러 올라가기

컴포넌트의 랜더링 안에 있는 모든 함수는 (이벤트 핸들러, 이펙트, 타임아웃이나 그 안에서 호출되는 API 등) 랜더(render)가 호출될 때 정의된 props와 state 값을 잡아둔다.

```js
function Example(props) {
  useEffect(() => {
    setTimeout(() => {
      console.log(props.counter);
    }, 1000);
  });
  // ...
}

function Example(props) {
  const counter = props.counter;
  useEffect(() => {
    setTimeout(() => {
      console.log(counter);
    }, 1000);
  });
  // ...
}
```

클래스 컴포넌트에서의 예제와 다르게 `props`나 `state`를 컴포넌트 안에서 일찍 읽어들였는지에 대한 여부는 상관이 없다. 어차피 바뀌지 않으므로! **하나의 렌더링 스코프 안에서 `props`와 `state`는 변하지 않는 값으로 남아있다.**

종종 이펙트 안에 정의해둔 콜백에서 최신의 값을 이용하고 싶을 때가 있을 텐데, 가장 쉬운 방법은 `ref`를 이용하는 것이다.

클린업(useEffect에서 반환하는 콜백함수)에서도 똑같이 작용한다.

리액트는 페인팅 이후에 이펙트를 다루기 때문에 이팩트때문에 렌더링 속도가 느려짐을 방지한다.

## 리액트에게 이펙트를 비교하는 법 가르치기

리액트는 매 리렌더링마다 DOM 전체를 새로 그리는 것이 아니라, 실제로 바뀐 부분의 DOM 만을 업데이트 한다. 이펙트도 마찬가지로 매 렌더링 마다 실행되는 것이 아닌, 특정 것들(deps)이 변경되었을 때에만 실행되도록 할 수 있다.

### 의존성으로 거짓말 하지 말자.

매 초마다 숫자가 올라가는 카운터를 작성하는 예를 보자. 직관적으로 우리는 "인터벌을 한 번만 설정하고, 한 번만 제거하자!" 라는 생각으로 이런 코드를 작성한다.

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);

    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```

결과는 **숫자가 한 번만 증가**한다. 의존성 배열이 `[]` 이기 떄문에 count값이 항상 0이기 때문이다.

### 의존성을 솔직하게 적는 두 가지 방법

첫 번째 방법. 이펙트에서 사용되는 모든 값을 의존성 배열 안에 포함되도록 수정하는 것.

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);

  return () => clearInterval(id);
}, [count]);
```

count값이 1보다 커지지 않는 문제는 해결되겠으나, count값이 변경될 때마다 인터벌이 해제되고 다시 설정 될 것이다. 이는 우리가 원하는 동작이 아니다.

두 번째 방법은 이펙트의 코드를 변경하여 자주 바뀌는 값을 요구하지 않도록 만드는 방법이다.

### 이펙트가 자급자족 하도록 만들기

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount((c) => c + 1);
  }, 1000);

  return () => clearInterval(id);
}, []);
```

setState 콜백을 활용하여 최신 값을 받아오기. `count` 상태의 의존성을 없앴다.

하지만 `setCount(c => c + 1)`도 그리 좋은 방법은 아니다. 서로에게 의존하는 두 상태값이 있거나 prop기반으로 다음 상태를 계산할 필요가 있을 때에는 도움이 되질 않는다. 하지만 강력한 자매 패턴인 `useReducer`가 있다.

## 액션을 업데이트로부터 분리하기

```js
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: "tick" }); // setCount(c => c + step) 대신에
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```

(리액트가 `dispatch`, `setState`, `useRef` 컨테이너 값이 항상 고정되어 있다는 것을 보장하니까 의존성 배열에서 뺄 수도 있습니다. 하지만 명시한다고 해서 나쁠 것은 없습니다.)

## 함수를 이펙트 안으로 옮기기

```js
function SearchResults() {
  const [data, setData] = useState({ hits: [] });

  async function fetchData() {
    const result = await axios(
      'https://hn.algolia.com/api/v1/search?query=react',
    );
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []); // 이거 괜찮은가?
  // ...
```

이 코드는 일단 동작한다. 하지만 로컬 함수를 의존성에서 제외하다 보면 컴포넌트가 커졌을 때 모든 의존성에 대해 핸들링하고 있는지 보장하기가 힘들어진다.

```js
function SearchResults() {
  const [query, setQuery] = useState("react");

  // 이 함수가 길다고 상상해 봅시다
  function getFetchUrl() {
    return "https://hn.algolia.com/api/v1/search?query=" + query;
  }

  // 이 함수가 길다고 상상해 봅시다
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```

컴포넌트, 함수가 길어졌을 떄 `deps`업데이트를 깜빡했다면 `props`와 `state`의 변화에 동기화 하는 데 실패할 것이다.

어떤 함수를 이펙트 내부에서만 사용한다면, 그 함수를 직접 이펙트 안으로 옮기자. 그러면 외부에 정의된 함수를 볼 필요가 없기 때문에 일기 편하다.

> 사실 `eslint-plugin-react-hooks` 플러그인의 `exhaustive-deps` 규칙을 사용하면 lint가 이를 체크해준다.

### 하지만 저는 이 함수를 이펙트 안에 넣을 수 없어요.

하지만 여러 곳에서 재사용되는 함수여서 이펙트 내부로 옮기기 쉽지 않을 수 있다.

```js
function SearchResults() {
  function getFetchUrl(query) {
    return "https://hn.algolia.com/api/v1/search?query=" + query;
  }

  useEffect(() => {
    const url = getFetchUrl("react");
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // 🔴 빠진 dep: getFetchUrl

  useEffect(() => {
    const url = getFetchUrl("redux");
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // 🔴 빠진 dep: getFetchUrl

  // ...
}
```

`getFetchUrl`을 각각의 이펙트 안으로 옮기게 되면 로직을 공유할 수 없어 달갑지 못하다.

```js
function SearchResults() {
  // 🔴 매번 랜더링마다 모든 이펙트를 다시 실행한다
  function getFetchUrl(query) {
    return "https://hn.algolia.com/api/v1/search?query=" + query;
  }
  useEffect(() => {
    const url = getFetchUrl("react");
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // 🚧 Deps는 맞지만 너무 자주 바뀐다

  useEffect(() => {
    const url = getFetchUrl("redux");
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // 🚧 Deps는 맞지만 너무 자주 바뀐다

  // ...
}
```

이렇게 하자니 `getFetchUrl`은 매 렌더링마다 바뀌어 이펙트가 너무 자주 실행된다.

첫 번째 해결 방법은 외부로 끌어올리는 방법이다.

```js
// ✅ 데이터 흐름에 영향을 받지 않는다
function getFetchUrl(query) {
  return "https://hn.algolia.com/api/v1/search?query=" + query;
}

function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl("react");
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // ✅ Deps는 OK

  useEffect(() => {
    const url = getFetchUrl("redux");
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // ✅ Deps는 OK

  // ...
}
```

이는 함수가 컴포넌트 스코프 안의 어떠한 것도 사용하지 않을 때만 가능한 방법이다.

두 번째 해결 방법은 `useCallback` 훅으로 감싸는 것이다. 이는 의존성 체크에 레이어를 하나 추가하는 방법이다. 함수의 의존성을 피하기보다는 함수 자체가 필요할 때에만 바뀔 수 있도록 만드는 것이다.

```js
function SearchResults() {
  const [query, setQuery] = useState("react");

  // ✅ query가 바뀔 때까지 항등성을 유지한다
  const getFetchUrl = useCallback(() => {
    return "https://hn.algolia.com/api/v1/search?query=" + query;
  }, [query]); // ✅ 콜백 deps는 OK
  useEffect(() => {
    const url = getFetchUrl();
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK

  // ...
}
```

부모 컴포넌트로부터 함수 `props`를 전달 할 때도 사용할 수 있다.

```js
function Parent() {
  const [query, setQuery] = useState("react");

  // ✅ query가 바뀔 때까지 항등성을 유지한다
  const fetchData = useCallback(() => {
    const url = "https://hn.algolia.com/api/v1/search?query=" + query;
    // ... 데이터를 불러와서 리턴한다 ...
  }, [query]); // ✅ 콜백 deps는 OK
  return <Child fetchData={fetchData} />;
}

function Child({ fetchData }) {
  let [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ✅ 이펙트 deps는 OK

  // ...
}
```

`fetchData`는 오로지 부모의 `query`상태가 바뀔 때만 변경되기 때문에 자식 컴포넌트는 꼭 필요할 때에만 데이터를 페칭할 것이라는것을 알고 있다.

## 함수도 데이터 흐름의 일부인가?

위의 예제 패턴은 클래스 컴포넌트에서 사용하면 제대로 동작하지 않는다.

```js
class Parent extends Component {
  state = {
    query: "react",
  };

  fetchData = () => {
    const url =
      "https://hn.algolia.com/api/v1/search?query=" + this.state.query;
    // ... 데이터를 불러와서 무언가를 한다 ...
  };

  render() {
    return <Child fetchData={this.fetchData} />;
  }
}
```

```js
class Child extends Component {
  state = {
    data: null,
  };

  componentDidMount() {
    this.props.fetchData();
  }

  componentDidUpdate(prevProps) {
    // 🔴 이 조건문은 절대 참이 될 수 없다
    if (this.props.fetchData !== prevProps.fetchData) {
      this.props.fetchData();
    }
  }

  render() {
    // ...
  }
}
```

`fetchData`는 클래스 메소드이다. 클래스 메소드는 state가 변경되었다고 달리지지 않기 때문에 `this.props.fetchData`와 `prevProps.fetchData`는 항상 같아 데이터를 페칭하지 않는다.

```js
componentDidUpdate(prevProps) {
  this.props.fetchData();
}
```

조건문을 제거하면 매 렌더링 시에 데이터를 가져올 것이다...

```js
render() {
  return <Child fetchData={this.fetchData.bind(this, this.state.query)} />;
}
```

그렇다면 `query`를 바인딩해놓자...? 이는 반대로 조건문이 항상 틀리기 때문에 데이터를 매번 페칭한다.

이를 해결하는 방법은 `query` 자체를 직접 자식 컴포넌트에게 넘겨 비교하는 수밖에 없다. 자식 컴포넌트는 `query`를 직접적으로 사용하지 않음에도 `query`가 바뀔 때마다 데이터를 다시 불러와야 한다...

부모 컴포넌트로부터 내려온 `this.props.fetchData`가 어떤 상태에 의존하는지, 그저 상태가 바뀌기만 한 것인지 알 수가 없다.

`useCallback`을 사용하면 함수도 데이터 흐름에 포함된다. 함수의 입력값이 변하면 함수 자체가 바뀌고, 그렇지 않다면 같은 함수로 남아있는다. 따라서 우리는 `props.fetchData`의 변화를 하위 컴포넌트에게 직접적으로 전달할 수 있다.

`useMemo`또한 같은 방식의 해결책을 제공한다.

```js
function ColorPicker() {
  // color가 진짜로 바뀌지 않는 한
  // Child의 얕은 props 비교를 깨트리지 않는다
  const [color, setColor] = useState("pink");
  const style = useMemo(() => ({ color }), [color]);
  return <Child style={style} />;
}
```

### 출처

[overreacted](https://overreacted.io/ko/a-complete-guide-to-useeffect/)
