# Managing the Instances

## traditional object-oriented UI programming

자식 컴포넌트 인스턴스의 생성/삭제가 사용자의 몫이다. 예를 들어 Form 컴포넌트가 Button 컴포넌트를 렌더링하고 싶다면, 인스턴스를 생성하고 새로운 정보를 사용하여 손수 최신 상태로 유지해야 한다.

```js
class Form extends TraditionalObjectOrientedView {
  render() {
    // Read some data passed to the view
    const { isSubmitted, buttonText } = this.attrs;

    if (!isSubmitted && !this.button) {
      // Form is not yet submitted. Create the button!
      this.button = new Button({
        children: buttonText,
        color: "blue",
      });
      this.el.appendChild(this.button.el);
    }

    if (this.button) {
      // The button is visible. Update its text!
      this.button.attrs.children = buttonText;
      this.button.render();
    }

    if (isSubmitted && this.button) {
      // Form was submitted. Destroy the button!
      this.el.removeChild(this.button.el);
      this.button.destroy();
    }

    if (isSubmitted && !this.message) {
      // Form was submitted. Show the success message!
      this.message = new Message({ text: "Success!" });
      this.el.appendChild(this.message.el);
    }
  }
}
```

각 컴포넌트 인스턴스는 해당 DOM 노드와 자식 컴포넌트 인스턴드에 대한 참조를 유지하고, 때에 따라 생성/업데이트/삭제해야 한다. 코드는 구성요소가 가질 수 있는 상태의 숫자에 따라 기하급수적으로 커지고, 부모 노드는 자식 컴포넌트 인스턴스에 직접적으로 접근할 수 있으므로 나중에 분리하기가 어렵다.

React는 어떻게 다를까?

# Elements Describe the Tree

**`An element is a plain object describing a component instance or DOM node and its desired properties`**

element는 컴포넌트 인스턴스나, 몇몇 프로퍼티들을 가진 DOM node를 나타내는 단순한 객체이다. 컴포넌트 타입(ex: Button), 프로퍼티(ex: 색상), 그리고 그 내부의 하위 엘리먼트들(ex: 글씨 등)에 대한 정보를 포함한다.

element는 실제 인스턴스가 아니다. 그보다는 React에게 내가 화면에 어떤 것을 보여주고 싶은지 말하는 방법이다. 이것은 `type`과 `props`를 가지는 `immutable description object`로 표현된다.

## DOM Elements

type이 문자열이면 해당 태그이름을 가진 DOM노드를 나타냄

```js
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

이렇게 렌더링된다.

```html
<button class="button button-blue">
  <b> OK! </b>
</button>
```

`prop`의 `children`속성을 통해 element들이 중첩될 수 있다.

중요한 것은 부모, 자식 element들이 실제 인스턴스가 아닌 단순한 설명일 뿐이라는 것이다. 당신이 element를 생성할 때 해당 element는 화면상의 아무것도 가리키지 않는다.

React element는 단지 객체이기 때문에 탐색하기 쉽고, 파싱 할 필요가 없으며, 실제 DOM elements보다 훨씬 가볍다

## Component Elements

`type`은 문자열도 될 수 있지만, React component(클래스형/함수형)일 때도 있다

```js
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

이것이 React의 핵심 아이디어이다.

**`An element describing a component is also an element, just like an element describing the DOM node. They can be nested and mixed with each other. `**

컴포넌트 타입 element도 또한 DOM node를 나타내는 element이다. 상호간 중첩되고 혼합될 수 있다.

이를 이용하여 특정 color 프로퍼티를 가지는 Button인 DangerButton을 정의할 수 있다.

```js
const DangerButton = ({ children }) => ({
  type: Button,
  props: {
    color: "red",
    children: children,
  },
});
```

mix and match!

```js
const DeleteAccount = () => ({
  type: 'div',
  props: {
    children: [{
      type: 'p',
      props: {
        children: 'Are you sure?'
      }
    }, {
      type: DangerButton,
      props: {
        children: 'Yep'
      }
    }, {
      type: Button,
      props: {
        color: 'blue',
        children: 'Cancel'
      }
   }]
});
```

```jsx
const DeleteAccount = () => (
  <div>
    <p>Are you sure?</p>
    <DangerButton>Yep</DangerButton>
    <Button color="blue">Cancel</Button>
  </div>
);
```

위의 구조는 아래와 같이 is-a, has-a 관계를 모두 표현할 수 있기 때문에 컴포넌트들을 나누기 쉽게 한다.

- `Button`은 특정 property들을 가진 `DOM button`이다.
- `DangerButton`은 특정 속성을 가진 `Button`이다.
- `DeleteAccount`는 `div` 안에 `Button`과 `DangerButton`을 포함한다.

### Components Encapsulate Element Trees

```js
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

Button 컴포넌트에 대해 무엇을 렌더링 해야하는지?

```js
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

React는 페이지의 모든 컴포넌트들에 대한 DOM 태그 element를 알 때까지 이 작업을 반복한다.

위에서 언급한 고전적인 방법을 기억하면서 보자.

```jsx
const Form = ({ isSubmitted, buttonText }) => {
  if (isSubmitted) {
    // Form submitted! Return a message element.
    return {
      type: Message,
      props: {
        text: "Success!",
      },
    };
  }

  // Form is still visible! Return a button element.
  return {
    type: Button,
    props: {
      children: buttonText,
      color: "blue",
    },
  };
};
```

리액트 컴포넌트에서 props는 입력이고, element tree는 출력이다.

**`The returned element tree can contain both elements describing DOM nodes, and elements describing other components. This lets you compose independent parts of UI without relying on their internal DOM structure.`**

반환된 element tree에는 DOM nodes를 나타내는 elements나 다른 컴포넌트를 나타내는 elements가 포함될 수 있다. 이를 통해 내부적인 DOM 구조에 의존하지 않고 UI의 독립적인 부분을 구성할 수 있다.

우리는 React가 인스턴스를 생성하고, 업데이트하고, 삭제하도록 하면 된다. 우리는 컴포넌트에서 반환되는 element로 그것을 설명하며, 리액트는 인스턴스에 대한 관리를 담당한다.

> 사용자는 컴포넌트에서 렌더링하고 싶은 정보를 담은 element를 반환하기만 하고, React가 해당 elemenet를 받아 생성한 instance들을 관리해주기 때문에 우리가 직접 생성/수정/삭제 할 필요가 없다. 따라서 사용자는 UI구현에만 집중하면 되는 것이다. 라는 말로 이해했다.

## Components Can Be Classes or Functions

1. As a function of props
2. Using the React.createClass() factory
3. As an ES6 class descending from React.Component

## Top-Down Reconciliation

```js
ReactDOM.render(
  {
    type: Form,
    props: {
      isSubmitted: false,
      buttonText: "OK!",
    },
  },
  document.getElementById("root")
);
```

React는 `Form` 컴포넌트에게 주어진 `props`에 따라 어떤 `element tree`를 반환하는지 묻는다. 간단한 시작점으로부터 컴포넌트 트리를 점진적으로 세분화한다.

```js
// React: You told me this...
{
  type: Form,
  props: {
    isSubmitted: false,
    buttonText: 'OK!'
  }
}

// React: ...And Form told me this...
{
  type: Button,
  props: {
    children: 'OK!',
    color: 'blue'
  }
}

// React: ...and Button told me this! I guess I'm done.
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

이는 `ReactDOM.render()` 또는 `setState()` 호출 시 React가 수행하는 재조정(Reconciliation) 과정이다. 조정이 끝나면 리액트는 DOM tree의 결과를 알게 되고, `react-dom`이나 `react-native`같은 렌더러가 DOM nodes를 업데이트 하기 위해 최소한의 변경 사항을 적용한다.

이러한 점진적 다듬기(`gradual refine`) 프로세스는 또한 `React app`이 최적화하기 쉬운 이유 중 하나다. 컴포넌트 트리가 리액트가 효율적으로 방문하기에 너무 커지면, 다듬기를 건너뛰고 관련 props가 변하지 않은 경우에 트리의 특정 부분을 비교하도록 할 수 있다. props가 immutable하다면 변경되었는지에 대한 여부를 판단하는 것은 매우 빠르기 때문에 최소한의 노력으로 훌륭한 최적화를 적용시킬 수 있다.

> 인스턴스는 리액트가 관리해준다~~

### 출처

[React Components, Elements, and Instances](https://ko.reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)  
December 18, 2015 by Dan Abramov
