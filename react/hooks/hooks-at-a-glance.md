# Hooks at a Glance

## Rules of Hooks

- **Hook을 `top level`에서만 호출할 것.** 루프, 조건문, 중첩함수 안에서 호출하지 말자.
- **`React function components`에서만 Hook을 호출할 것.** 일반 js함수 내부에서 호출하지 마라.

[eslint-plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks)을 사용하면 자동으로 강제해준다.

> 컴포넌트 내의 훅들은 링크드 리스트처럼 연결되어 순차적으로 호출되는 구조를 가진다고 한다. 그렇기에 `top level`이 아닌 곳에서 호출하면

## Building Your Own Hooks

전통적으로, 우리는 `stateful logic`을 컴포넌트 사이에서 재사용하기 위하여 `higher-order components`나 `render props`를 사용하였었다. `Hook`은 컴포넌트 트리를 추가적으로 생성하지 않으면서 이를 할 수 있게 해준다.

```js
import React, { useState, useEffect } from "react";

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

커스텀 훅을 생성하고,

```js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return "Loading...";
  }
  return isOnline ? "Online" : "Offline";
}

function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? "green" : "black" }}>{props.friend.name}</li>
  );
}
```

여러 컴포넌트에서 재사용하자.

각 컴포넌트의 상태는 완전히 독립적이다. `Hook`은 `stateful logic`을 재사용하는 방법이며, 상태 자체가 아니다. 실제로 각각의 `Hook`에 대한 호출은 완전히 독립적인 상태이므로 하나의 컴포넌트에서 같은 훅을 두번 사용해도 상관없다.

`Custom Hooks`는 기능이라기 보다는 컨벤션에 가깝다. 함수 이름이 `use`로 시작하는 함수를 우리는 커스텀 훅이라고 칭한다.
