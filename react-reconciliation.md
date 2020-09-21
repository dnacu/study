# Reconciliation

## Motivation

`state`나 `props`가 갱신되면 `render()`함수는 새로운 `React Element Tree`를 반환한다. 이 때 React는 만들어진 트리에 맞게 가장 효과적으로 UI를 갱신하는 방법을 알아낼 필요가 있다.

하나의 트리를 다른 트리로 변환하기 위한 최소한의 연산 수를 구하는 알고리즘은 O(n^3)이다.

React에 이 알고리즘을 적용하면, 1000개 엘리먼트를 그리기 위해 10억번의 비교 연산을 수행하여야 한다. 대신 React는 두 가지 가정을 기반으로 O(n)복잡도의 휴리스틱 알고리즘을 구현했다.

1. 서로 다른 타입의 두 엘리먼트는 서로 다른 트리를 생성한다.
2. 개발자가 `key` prop을 통해 여러 렌더링 사이에서 어떤 자식 엘리먼트가 변경되지 않아야 할지 표시해 줄 수 있다.

# The Diffing Algorithm

React는 `root element`부터 시작하여 두 개의 트리를 비교한다. 이후 동작은 루트 엘리먼트의 타입에 따라 달라진다.

## Elements Of Different Types

만약 두 트리의 `root element type`이 다르다면, React는 이전 트리를 버리고 완전히 새로운 트리를 구축합니다.

트리를 버릴 때 이전 DOM 노드들은 모두 파괴된다. 컴포넌트 인스턴스는 `componentWillUnmount()`가 실행된다. 새로운 트리가 만들어질 때, 새로운 DOM 노드들이 DOM에 삽입되며, 그에 따라 새 컴포넌트 인스턴스는 `componentWillMount()`와 `componentDidMount()`가 이어서 실행된다. 이전 트리와 관련된 모든 state는 사라진다.

> 이전 컴포넌트 인스턴스가 사라지니까 당연히 state도 없어지리라 생각한다.

루트 엘리먼트 하위의 모든 컴포넌트도 unMount되고, state도 사라진다.

```html
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

이 예시에서, 이전 `Counter`는 사라지고, 새로 mount 된다.

## DOM Elements Of The Same Type

```html
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

타입이 같은 두 엘리먼트를 비교하면, React는 현재 DOM노드 상의 `className`만을 수정한다.

```html
<div style={{color: 'red', fontWeight: 'bold'}} /> <div style={{color: 'green',
fontWeight: 'bold'}} />
```

이러한 변경에서 React는 `fontWeight`는 수정하지 않고 `color`속성만을 수정한다.

DOM노드의 처리가 끝나면 React는 이어서 해당 노드들의 자식들을 재귀적으로 처리한다.

## Component Elements Of The Same Type

컴포넌트 갱신 시 인스턴스와 state가 그대로 유지된다. React는 새로운 엘리먼트의 내용을 반영하기 위해 현재 컴포넌트 인스턴스의 `props`를 갱신한다. 이 때 `componentWillReceiveProps()`와 `componentWillUpdate()`가 호출된다.

다음으로 `render()`메소드가 호출되고 `diffing algorithm`이 이전과 현재의 결과를 재귀적으로 처리한다.

# Recursing On Children

DOM노드의 자식들을 재귀적으로 처리할 때, React는 기본적으로 두 리스트를 순회하고 차이점이 있으면 변경을 생성한다.

```html
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

first부터 순차적으로 비교하여 third가 추가된다.

```html
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

이렇게 리스트의 맨 앞에 엘리먼트가 추가되는 경우 성능이 좋지 않다. 이 상황에서 React는 모든 자식을 변경할 것이다.

## Keys

이러한 문제를 해결하기 위해 React `key`속성을 지원한다. React는 key를 통해 기존 트리와 새 트리의 자식들이 일치하는지 확인한다.

```html
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

`key`를 사용하면 React가 2014 키를 가진 엘리먼트가 추가되었고, 2015, 2016 키를 가진 엘리먼트는 그저 이동만 하면 된다는 사실을 알 수 있다.

```html
<li key="{item.id}">{item.name}</li>
```

일반적으로 엘리먼트가 가지는 유니크한 식별자를 사용하면 되지만, 그럴 수 없다면 데이터 구조 내부에 `id`속성을 추가해주거나 데이터 일부를 해싱하여 key를 생성할 수 있다. **해당 `key`는 오로지 형제 사이에서만 유일하면 되고, 전역에서 유일할 필요는 없다.**

최후의 수단으로 배열의 인덱스를 `key`로 사용할 수 있다. 항목이 재배열되지 않는 경우에는 잘 동작하나, 재배열되는 경우 비효율적으로 동작할 것이다.

인덱스를 `key`로 사용하던 중 배열의 순서가 바뀌면 컴포넌트 `state`와 관련된 문제가 발생할 수 있다. 컴포넌트 인스턴스는 `key`를 기반으로 갱신되고 재사용 된다. 인덱스를 `key`로 사용한다면 항목의 순서가 바뀌었을 때 key또한 바뀔 것이고, 그 결과 컴포넌트의 `state`가 엉망이 되거나 의도하지 않은 방식으로 바뀔 수 있다.

예시: https://codepen.io/pen?&editable=true&editors=0010

> 정리하자면, 해당 idx값으로 다른 엘리먼트가 렌더링 될 수 있는 경우라면 인덱스를 사용하면 안된다. 예를 들어 배열 순서를 뒤바꿔서 렌더링하거나, 유저가 삭제/추가 및 순서를 바꿀 때도 그렇다.

### 참고

[배열의 인덱스를 key로 쓰면 안되는 이유](https://medium.com/sjk5766/react-%EB%B0%B0%EC%97%B4%EC%9D%98-index%EB%A5%BC-key%EB%A1%9C-%EC%93%B0%EB%A9%B4-%EC%95%88%EB%90%98%EB%8A%94-%EC%9D%B4%EC%9C%A0-3ce48b3a18fb)

# Tradeoffs

React는 휴리스틱에 의존하고 있기 때문에, 휴리스틱이 기반하고 있는 가정에 부합하지 않는 경우 성능이 나빠질 수 있습니다.

1. 알고리즘은 다른 컴포넌트 타입을 갖는 종속 트리들의 일치 여부를 확인하지 않습니다. 만약 매우 비슷한 결과물을 출력하는 두 컴포넌트를 교체하고 있다면, 그 둘을 같은 타입으로 만드는 것이 더 나을 수도 있습니다. 우리는 실제 사용 사례에서 이 가정이 문제가 되는 경우를 발견하지 못했습니다.
2. key는 반드시 변하지 않고, 예상 가능하며, 유일해야 합니다. 변하는 `key`(`Math.random()`으로 생성된 값 등)를 사용하면 많은 컴포넌트 인스턴스와 DOM 노드를 불필요하게 재생성하여 성능이 나빠지거나 자식 컴포넌트의 `state가` 유실될 수 있습니다.
