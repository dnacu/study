# Execution

validation 체크 이후에 쿼리가 GraphQL 서버에 의해 실행되고, 요청한 쿼리에 따라 알맞은 결과값을 반환한다. (거의 JSON 형식으로)

GraphQL은 타입 시스템 없이 쿼리를 실행시킬 수 없다.

```js
type Query {
  human(id: ID!): Human
}

type Human {
  name: String
  appearsIn: [Episode]
  starships: [Starship]
}

enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}

type Starship {
  name: String
}
```

아래 예시를 통해 쿼리가 어떻게 실행되는지 알아보자.

```js
{
  human(id: 1002) {
    name
    appearsIn
    starships {
      name
    }
  }
}
```

각 필드의 각 타입은 GraphQL 서버 개발자가 만든 `resolver` 함수에 의해 지원된다. 필드가 실행되면, 그에 대응하는 resolver함수가 호출되어 다음 값을 생성한다.

필드가 스칼라 값을 생성하면 실행이 완료된다. 그러나 필드가 object라면, 쿼리는 해당 object에 적용되는 또 다른 필드를 포함하게 될 것이다. 이는 필드가 스칼라 값일 때까지 계속된다.

> **GraphQL 쿼리는 항상 스칼라 값에서 종료된다.**

## Root fields & resolvers

모든 GraphQL 서버에서의 최상단은 GraphQL API의 가능한 모든 entry point이다. 이는 _RootType_ 혹은 *QueryType*이라고 부른다.

이 예제에서, QueryType은 aurgments로 `id` 값을 받는 `human` 필드를 제공한다. 이 필드에 대한 resolver함수는 DB에 접근하여 구성을 변경하고, `Human` 객체를 반환한다.

```js
Query: {
  human(obj, args, context, info) {
    return context.db.loadHumanByID(args.id).then(
      userData => new Human(userData)
    )
  }
}
```

이 예제는 JS로 쓰였지만, GraphQL 서버는 여러 다른 언어에서 생성될 수 있다. resolver 함수는 4가지 arguments를 받는다.

- `parent` : 이전 resolver의 결과. (상위 필드)
- `args` : GraphQL 쿼리 내부에서 필드에 제공되는 arguments이다.
- `context` : 모든 resolver에게 제공되는 값이며, 중요한 문맥적 정보를 가지고 있다. 최신에 로그인한 유저에 대한 기록이나, 디비 접근 기록 등. (custom object)
- `info` : 각 필드마다 현재 쿼리에 관련된 특정한 정보를 가지며, 또한 스키마 상세에 대한 정보도 함께 가진다.

## Asynchronous resolvers

```js
human(obj, args, context, info) {
  return context.db.loadHumanByID(args.id).then(
    userData => new Human(userData)
  )
}
```

`context` 는 db에 접근하기 위한 인스턴스 등을 담는 데에 쓰인다. 이 예제에서는 arugment로 전달된 id값을 가지고 데이터베이스에서 유저에 대한 정보를 가져온다. db에서 데이터를 가져오는 것은 비동기 작업이며, 이는 `Promise` 객체를 반환한다. js에서 `Promise` 는 비동기로 값을 반환한다. db로부터 원하는 값이 반환된 이후, 새로운 `Human` 인스턴스를 반환한다.

resolver 함수는 `Promise`와 같은 비동기 작업이 끝나는 것을 알아차릴 필요가 있지만, GraphQL 쿼리는 이를 알 필요가 없다. 단순히 `human` 필드를 기대하며, 요청된 하위 필드들을 반환하는데에만 관심이 있다.

## Trivial resolvers

이제 `Human` 객체가 사용 가능해졌다. GraphQL은 요청받은 관련 필드에 대한 실행을 계속한다.

```js
Human: {
  name(obj, args, context, info) {
    return obj.name
  }
}
```

GraphQL 서버는 타입 시스템에 의해 동작하며, 이는 다음에 어떤 작업을 할 지 정의하는데에 사용된다. 심지어 `human` 필드에 대한 반환 전에 GraphQL은 다음에 `Human` 타입의 어떤 필드에 대해 resolve할지 알고 있다. 타입 시스템으로부터 `human` 필드에 대한 반환이 `Human` 객체일 것이라는 것에 대해 들었을 때부터 말이다.

`name` 필드를 resolve하는 것은 매우 직관적이다. name함수가 호출되고, `obj` 는 이전 필드에서 반환된 `Human` 인스턴스이다. 이 경우 우리는 `Human` 객체가 읽어서 바로 리턴할 수 있는 `name` property를 가지고 있다는 것을 안다.

사실, 많은 GraphQL 라이브러리들은 이 작은 resolver를 생략할 수 있도록 할 것이며, 만약 필드에 대한 resolver가 생략되면, 필드 이름과 같은 `name` property가 있을 것이라고 가정한다.

## Scalar coercion

`name` 필드가 resolve되는 동안, `appearsIn` 과 `startship` 필드도 동시에 함께 resolve될 수 있다. `appearsIn` 필드는 `trivial resolver` 를 가진다.

```js
Human: {
  appearsIn(obj) {
    return obj.appearsIn // returns [ 4, 5, 6 ]
  }
}
```

타입 시스템이 `appearsIn` 필드가 `Enum` 값들이 리턴되기를 요구한다. 하지만, 이 함수는 숫자를 반환한다! 하지만 실제로 결과를 보면 적절한 enum값들이 반환된 것을 볼 수 있다. 무슨 일이 일어난 걸까?

이것은 `scalar coercion` 의 예시이다. 타입 시스템은 어떤것을 기대해야 할 지 알고 있고, resolver함수에 의해 리턴된 값들을 API 명세에 따라 변환한다. 이 예제에서, GraphQL 타입 시스템에서는 Enum값들로 나타나지만, 아마 Enum은 서버에서 내부적으로 4, 5, 6으로 정의되어있을 것이다.

## List resolvers

우리는 `appearsIn` 필드를 통해 리스트를 반환할 때 어떤일이 일어나는지 살짝 보았다. 이것은 enum값들의 리스트를 반환하며, 그것은 타입시스템이 기대하는 바와 같이 각각의 아이템들이 적합한 enum값으로 강제변환된다. `starships` 필드가 resolve될 때 어떤 일이 일어날까?

```js
Human: {
  starships(obj, args, context, info) {
    return obj.starshipIDs.map(
      id => context.db.loadStarshipByID(id).then(
        shipData => new Starship(shipData)
      )
    )
  }
}
```

이 필드에 대한 resolver는 그냥 Promise가 아니라, Promise list를 반환한다. `Human` 객체는 그들이 조종하는 `Starships` 의 `id` 에 대한 리스트(`starshipIDs`)를 가지고 있는데, 우리는 실제 `Starship` 객체를 가져오기 위해 모든 `id` 값들을 로드해야 한다.

GraphQL은 모든 Promise가 완료될 때까지 기다린다. 모든 Promise가 완료되어 객체 리스트가 되면 각 객체에 대해 `name` 필드를 로드할 것이다.

## Producing the result

각 필드가 resolve될 때, 결과값은 필드이름을 key값으로 하고, resolve의 반환값을 value로 하는 key-value map으로 반환된다. 이 방식은 쿼리의 최하단부터 루트 쿼리 타입까지 반복된다. 최종적으로 요청받은 쿼리와 닮은 구조의 결과 객체를 만들어 클라이언트에게 보냅니다.
