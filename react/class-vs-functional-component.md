# Functional Components vs Class Components in React

## Rendering JSX

**Syntax의 차이**

첫번쨰로, 눈에 보이는 확연한 차이점은 `syntax`이다. `functional component`는 단순히 `JSX`를 반환하는 함수인데 반해, `class component`는 `render`메소드를 가지고 있는 `React.Component`를 확장하는 클래스이다.

class component

```js
import React, { Component } from "react";

class ClassComponent extends Component {
  render() {
    return <h1>Hello, world</h1>;
  }
}
```

functional component

```js
import React from "react";

const FunctionalComponent = () => {
  return <h1>Hello, world</h1>;
};
```

## Passing props

`functional component`는 함수의 인자로 props를 전달하지만, `class component`에서는 `this`에 담아준다.

class component

```js
class ClassComponent extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

functional component

```js
const FunctionalComponent = (props) => {
  return <h1>Hello, {props.name}</h1>;
};
```

## Handling state

React 16.8에서 `Hook`이 발표됨에 따라 `functional component`에서도 상태관리가 가능하게 되었다.

from [React official documentation](https://reactjs.org/docs/react-component.html#constructor)
_"The constructor for a React component is called before it is mounted. When implementing the constructor for a React.Component subclass, you should call super(props) before any other statement. Otherwise, this.props will be undefined in the constructor, which can lead to bugs."_

React 컴포넌트의 생성자는 해당 컴포넌트가 mount되기 전에 호출된다. `React.Component`를 상속받는 클래스 구현 시, `constructor`에서 다른 문보다 먼저 `super(props)`를 호출하여야 한다. 그렇지 않으면 `this.props`가 생성자에서 정의되지 않아 의도치 않은 버그가 발생할 수 있다.

## Lifecycle Method

기존 `class component`의 생명주기 함수들을 대체하는 `hook`들이 존재한다.

### useEffect hook (componentDidMount, componentWillUnmount)

`componentDidMount` 메소드는 첫 번째 렌더링이 완료된 직후에 호출된다. `componentWillMount` 메소드는 렌더링 전에 호출되는 함수였으나 `deprecated`되었다.

`class component`

```js
class ClassComponent extends React.Component {
  componentDidMount() {
    console.log("Hello");
  }

  render() {
    return <h1>Hello, World</h1>;
  }
}
```

`functional component`

```js
const FunctionalComponent = () => {
  React.useEffect(() => {
    console.log("Hello");
  }, []);
  return <h1>Hello, World</h1>;
};
```

`useEffect hook`의 두 번째 인자로 전달된 `state`들이 변할 떄 함수가 호출되며, 빈 배열로 넘기면 첫 한번만 호출된다.

`callback`함수를 반환하면 component가 unmount되기 전에 해당 콜백을 실행하고, umount된다.

## 동작방식의 차이

> 함수형 컴포넌트는 `render` 될 때의 값들을 유지한다. 특정 `render`에 종속된 값을 가진다.

그렇지 않은게 `ref`이다. 클래스 컴포넌트의 this처럼 변하지 않으며 `render`들끼리 공유할 수 있다(?).
