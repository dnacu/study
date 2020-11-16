# Schemas and Types

## Type system

GraphQL 쿼리 언어는 기본적으로 객체에서 필드를 선택한다.

```json
{
  hero {
    name
    appearsIn
  }
}
```

1. 특별한(?) "root" 객체로부터 시작한다.
2. 해당 객체에서 `hero` 필드를 선택한다.
3. 반환되는 `hero` 객체로부터 우리는 `name` 과 `appearsIn` 필드를 선택한다.

GraphQL 쿼리의 형태는 그 결과와 굉장히 닮아았기 때문에, 우리는 서버에 대하여 알지 못해도 해당 쿼리가 어떤 데이터를 반환할지 예측할 수 있다. 어떤 필드를 선택할 수 있는지, 그것들이 어떤 종류의 object를 반환하는지, 그의 sub-objects에는 어떤 필드들이 있는지에 대하여 알 수 있다면 더 좋을 것이다.

> GraphQL의 schema는 쿼리를 보고 반환되는 데이터를 예측할 수 있도록 도와준다.

GraphQL 서비스는 쿼리로부터 접근 가능한 데이터셋을 보여주는 타입에 대하여 정의한다. 이후에 쿼리가 들어오면, 서비스는 유효성을 검사한 뒤에 해당 스키마에 따라 실행한다.

## Type language

GraphQL은 어떤 언어로도 작성될 수 있다. 이는 GraphQL 스키마에 대하여 특정 프로그래밍 언어의 문법에 의존할 수 없음을 의미한다. 그리하여 **GraphQL schema langueage**가 등장했다. 이는 쿼리언어와 비슷하며, 언어에 상관없이 스키마에 대해 설명할 수 있도록 한다.

## Object type and fields

```js
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

- `Character` 는 _GraphQL Object Type_ 이다. 몇개의 필드를 가진 타입이라는 뜻이다.
- `name` 과 `appearsIn` 은 `Character` 타입의 필드들이다. 이는 두 필드가 `Character` 타입에 대한 요청에서만 나타날 수 있다는 것을 뜻한다.
- `String` 은 내장된 _Scalar Type_ 이다. 하위필드의 선택이 불가능하다.
- `String!` 은 _non-nullable_ 타입을 뜻한다.
- `[Episode!]!` 는 Episode객체의 배열을 뜻하며, 필드 자체가 non-null일 뿐만 아니라, 내열 내부 요소도 non-null이라는 뜻이다.

## Arguments

```js
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

다른 언어와 다르게 GraphQL에서는 모든 arguments에 대한 이름이 명시되어 전해진다. argument가 optional이라면, default값을 정의할 수 있다.

## The Query and Mutation types

두 가지 특별한 타입 `query` 와 `mutation` 이 있다. 이 두 타입은 일반적인 객체타입과 같으나, GraphQL 쿼리의 첫 부분에 정의 된다는 점에서 특별하다.

```json
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```

이는 GraphQL 서비스에 `hero` 와 `droid` 필드를 가진 `Query` 타입이 있어야 함을 뜻한다(?).

```js
type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
```

## Scalar types

이들은 쿼리에서 잎사귀(leaves)를 나타낸다.

```json
{
  hero {
    name
    appearsIn
  }
}
```

GraphQL은 기본적으로 `Int`, `Float`, `String`, `Boolean`, `ID` 스칼라 타입을 가진다.

> `ID` 스칼라 타입은 unique identifier를 나타낸다. 객체를 다시 가져오거나, 캐시의 키로 사용된다. 따라서 문자열로써 다뤄지며, 이는 인간이 읽기 편하게 만들어진 것이 아니다.

커스텀 스칼라 타입을 명시할 수 있다.

## Enumeration types

값들의 집합이다.

1. 이 타입의 모든 arguments가 허용된 값(enum 객체 내부에 정의된) 중 하나인지 유효성을 검사한다.
2. 어떤 필드가 항상 유한한 값들의 집합 중 하나가 될 것임을 타입 시스템을 통해 전달한다.

```js
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

## Lists and Non-Null

`!` 를 활용하여 non-null값에 대한 타입을 명시한다. null값이 전달되면 GraphQL 서버에서 유효성 에러를 반환한다.

```json
myField: [String!]

myField: null // valid
myField: [] // valid
myField: ['a', 'b'] // valid
myField: ['a', null, 'b'] // error
```

```json
myField: [String]!

myField: null // error
myField: [] // valid
myField: ['a', 'b'] // valid
myField: ['a', null, 'b'] // valid
```

## Interfaces

`interface` 는 구현에 반드시 포함되어야 하는 필드들의 집합이다.

```js
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```

```js
type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```

`implement` 키워드를 이용하여 인터페이스를 상속하고, 이외의 필드들을 추가할 수 있다.

`interface` 는 object 또는 object의 집합을 얻고 싶은데 몇몇의 타입들이 서로 다를 때 유용하다.

```json
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    primaryFunction
  }
}

// variables...
{
  "ep": "JEDI"
}
```

```json
{
  "errors": [
    {
      "message": "Cannot query field \"primaryFunction\" on type \"Character\". Did you mean to use an inline fragment on \"Droid\"?",
      "locations": [
        {
          "line": 4,
          "column": 5
        }
      ]
    }
  ]
}
```

`primaryFunction` 에서 에러가 발생한다. `hero` 필드는 `Character`타입 object를 반환하는데, `Character` 타입에는 `primaryFunction` 필드가 없기 때문이다.

```json
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```

`inline fragments` 를 활용하여 해당 문제를 고칠 수 있다. `Character` 타입의 하위 집합 중 하나인 `Droid` 타입일 때만 `primaryFunction` 필드가 추가된다.

## Union types

`union type` 은 `interface` 와 비슷하지만, 타입간의 공통 필드를 명시하지는 않는다.

```js
union SearchResult = Human | Droid | Starship
```

`SearchResult` 타입을 반환하면, `Human`, `Droid`, `Starship` 타입 중 하나를 받게 될 것이다. 중요한 것은 `union type` 의 구성 요소들은 반드시 객체 타입이라는 것이다. `interface` 나 다른 `union type` 으로는 만들 수 없다.

`union type` 이 포함되면 반드시 `inline fragment` 가 사용되어야 한다.

```json
{
  search(text: "an") {
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

### Scalars type in Union

`scalar type` 도 `union type` 에서 사용될 수 없다.

이렇게하면 에러

```js
type Post {
  content: String | Int
}
```

우회해서 구현할 수 있긴 하다. 하지만 여전히 `union type` 을 나타내는 신텍스에 스칼라타입을 추가할 수는 없다.

```js
type PostString {
  content: String
}

type PostInt {
  content: Int
}

union Post = PostString | PostInt
```

## Input types

복잡한 객체를 arguments로 전달 할 수 있다. 특히 변경될 객체에 대한 정보를 전달하는 `mutation` 의 경우에서는 굉장히 자주 쓰인다.

```js
input ReviewInput {
  stars: Int!
  commentary: String
}
```

```js
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}

// variables...
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

`input type` 의 필드는 `input type` 을 참조할 수 있지만, input과 output 타입을 하나의 스키마에 섞을 수 없다(?). `input type` 은 field에 arguments를 가질 수 없다.
