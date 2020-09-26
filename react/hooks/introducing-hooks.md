# Introducing Hooks

React 16.8의 새로운 addition.

Hooks are:

- **Completely opt-in.**
- **100% backwards-compatible.** 이전 버젼과 완벽하게 호환된다.
- **Available now.**

**리액트에서 class를 삭제할 계획은 없다.**  
**`Hook`은 당신이 기존에 알고 있던 `React concepts`를 대체하지 않는다.**

## Motivation

`Hook`은 5년간 수많은 component를 작성하고 유지보수하면서 겪었던 **연결되지 않은 것처럼 보이는 다양한 문제들**을 해결한다.

### It's hard to reuse stateful logic between components

`stateful logic`을 컴포넌트간에 재사용하도록 하는 것이 어렵다.

React는 재사용가능한 동작을 컴포넌트에 "붙이는" 동작을 제공하지 않는다. 그래서 사람들은 그동안 [render props](https://reactjs.org/docs/render-props.html)나 [higher-order components](https://reactjs.org/docs/higher-order-components.html)와 같은 방법을 사용하여 이를 해결하려 했다. 그러나 이러한 패턴을 사용하여 컴포넌트를 재구성하는 것은 번거로우며, 코드를 읽기 힘들게 한다. `React DevTools`를 통해 React application을 살펴보면, 각종 `consumers`, `providers`, `higher-order-components`, `render-props`등등의 레이어로 감싸져 있는 형태의 "wrapper hell"을 만날 수 있다. 이런 이유로 깊이가 너무 깊어지고, React는 더 나은 방법을 찾고자 했다.

`Hook`을 사용하면 `stateful logic`을 컴포넌트로부터 추출할 수 있으며, 그것들을 독립적으로 테스트하고 재사용 할 수 있다. **Hook은 stateful logic을 컴포넌트 계층구조의 변화 없이도 재사용 가능케 만들어 준다.**

> 기존 class형 컴포넌트에서 `stateful logic`을 재사용하고자 할 떄, `consumers`, `providers`, `higher-order-components`, `render-props`등의 방법을 사용했어야 했는데, 이는 wrapper hell이라고 불릴만큼 컴포넌트의 깊이가 깊어지게 하여 가독성을 떨어뜨린다. 또한 해당 컴포넌트 파일 외부의 다른 상위 컴포넌트 파일에서 `provider`를 작성하는 등의 하나를 얻기위해 여러 파일을 건드리는 작업을 병행하기 때문에 코드를 이해하고 유지보수 하는 데에 굉장히 좋지 않았다.

### Complex components become hard to understand

복잡한 컴포넌트는 이해하기 힘들다.

우리는 간단하게 시작했으나 점차 커져서 유지보수하기 어려운 거대한 `stateful logic`을 가진 컴포넌트들을 만나게 된다. 각각의 `lifecycle method`들은 종종 관련없는 논리들이 혼합되어 있다. 예를 들어, `componentDidMount`과 `componentDidUpdate`에서 데이터를 가져오는 작업을 수행한다. 또한 `componentDidMount`에는 각종 이벤트리스너를 다는 작업이 포함되어 있으며, `componentWillUnount`에서 cleanup된다. 관련된 코드들은 분할되며, 관련없는 코드들은 하나의 메소드에 포함되는 형태가 되어버립니다. 그것들은 모순적이며, 버그가 발생하기 쉽습니다.

이것을 해결하기 위해, `lifecycle method`로 코드를 분할하는 것을 강제하지 않고, **관련된 조각들(subscribe 또는 data fetching등의 로직)을 컴포넌트 내부 하나의 작은 함수안에 둘 수 있도록 하였다.**

> 기존 클래스형 컴포넌트를 사용하면서 lifecycle method를 쓸 때, 함수가 분리되어 있기 때문에 강제적으로 관련된 코드들이 나뉘어 들어가게 되었고, 이는 코드의 가독성을 떨어뜨릴 뿐만 아니라 버그의 발생을 쉽게 한다. `Hook`에서는 관련있는 코드들을 한데 모으도록 하여 이러한 문제점들을 해결해 보고자 한 것 같다.

### Classes confuse both people and machines

클래스는 기계와 사람에게 혼란스럽다.

클래스는 사용자로 하여금 코드 재사용, 코드 구성 등을 더 어렵게 할 뿐만 아니라 자바스크립트의 클래스를 이해하고 학습해야 한다는 불편함이 있다.

또한 핫 리로딩이 불안정하고, 코드의 축소가 힘들다. 최적화가 잘 안된다 등등...

### 출처

[React docs - Introducing Hooks](https://reactjs.org/docs/hooks-overview.html)
