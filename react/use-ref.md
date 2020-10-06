# 목표

`ProductList` 컴포넌트가 언마운트 되는 시점에 스크롤 위치 및 상품 리스트를 세션스토리지에 캐싱

## 첫 번째 코드

```js
const ProductList = () => {
  const [productList, setProductList] = useState([]);

  useEffect(() => {
    return () => {
      sessionStorage.setItem("cache", {
        productsList,
        scrollTop: window.scrollY,
      });
    };
  }, []);

  return <SomeComponent />;
};
```

`useEffect`의 callback함수를 이용하여 unmount시 `productsList`와 `scrollTop`값을 세션스토리지에 저장하고자 함

### 문제점

`productList` 값이 항상 `[]`이다.

> 정확히 모르겠으나 `useEffect`의 두 번째 인자인 의존성 배열이 빈 배열이여서 맨 처음 한번만 콜백이 등록(?)되면서 그 때 값들을 미리 계산해두는 것 같다.

## 두 번쨰 코드

```js
const ProductList = () => {
  const [productList, setProductList] = useState([]);

  useEffect(() => {
    return () => {
      sessionStorage.setItem("cache", {
        productsList,
        scrollTop: window.scrollY,
      });
    };
  }, [productsList]);

  return <SomeComponent />;
};
```

의존성 배열에 `productsList`를 넘겨주었다. `productsList`값이 현재 값이 원하는대로 잘 넘어갔다.

### 문제점

콜백함수가 `productsList`가 변경될 떄마다 호출된다.

> `useEffect`에 대해 잘못 이해하고 있나 싶다. 라이프사이클 메소드인 `componentWillUnmount`를 함수형 컴포넌트에서 구현할 때에는 `useEffect`에서 콜백을 반환하면 된다고 알고 있었다. 그래서 당연히 `productsList`가 변경될 때마다 새 콜백이 등록되고 호출은 unmount될 때에만 일어날 줄 알았는데 의존성 배열의 값이 변경될 때마다 호출되었다.

## 세 번째 코드

```js
const ProductList = () => {
  const [productList, setProductList] = useState([]);

  useEffect(() => {
    return () => {
      setProductsList((productsList) => {
        sessionStorage.setItem("cache", {
          productsList,
          scrollTop: window.scrollY,
        });

        return [];
      });
    };
  }, []);

  return <SomeComponent />;
};
```

setState callback을 활용하여 해결해 보고자 하였다. `productsList`에 현재 값이 담겼다.

### 문제점

원인을 알 수 없는 문제가 발생한다. unmount시 콜백함수는 실행되나, `sessionStorage.setItem()`함수가 간헐적으로 실행되지 않는다.

## 네 번째 코드 - 해결

```js
const ProductList = () => {
  const [productList, setProductList] = useState([]);
  const productsListRef = useRef([]);

  useEffect(() => {
    productsListRef.current = productsList;
  }, [productsList]);

  useEffect(() => {
    return () => {
      sessionStorage.setItem("cache", {
        productsList: productsListRef.current,
        scrollTop: window.scrollY,
      });
    };
  }, []);

  return <SomeComponent />;
};
```

`ref`를 사용하여 문제를 해결하였다. element를 저장할 때에만 ref를 사용하는 줄 알았지만, 그렇지 않았다.

The useRef hook creates a ref which has 2 main use cases, to access a child component outside of the usual props flow or to store a mutable value that exists for the lifetime of the component. In our case, we use the latter.

A ref is a generic container that can store a value using its current property similar to using instance variables in object oriented programming. It is different from state since it does not cause a rerender when it is updated.

Since the ref is mutable and exists for the lifetime of the component, we can use it to store the current value whenever it is updated and still access that value in the cleanup function of our useEffect via the valueRef.

출처: https://www.timveletta.com/blog/2020-07-14-accessing-react-state-in-your-component-cleanup-with-hooks/

`ref`는 변경 가능하지만 리렌더를 일으키지 않고, 컴포넌트의 수명 동안에 없어지지 않는다. 따라서 `ref`를 사용하여 `productsList`값을 저장하고, 그 값을 콜백에서 사용하였다.
