# Validation

타입 시스템을 통해 GraphQL 쿼리가 유효한지에 대해 미리 알 수 있다. 이는 유효하지 않은 쿼리가 생성되었을 때 개발자에게 런타임 시점 체크에 의존하지 않고 효과적으로 알려줄 수 있게 한다.

```json
{
  hero {
    ...NameAndAppearances
    friends {
      ...NameAndAppearances
      friends {
        ...NameAndAppearances
      }
    }
  }
}
​
fragment NameAndAppearances on Character {
  name
  appearsIn
}
```

위 쿼리는 유효하다.

```json
{
  hero {
    ...NameAndAppearancesAndFriends
  }
}
​
fragment NameAndAppearancesAndFriends on Character {
  name
  appearsIn
  friends {
    ...NameAndAppearancesAndFriends
  }
}
```

하지만 이 쿼리는 유효하지 않은데, 이는 무한루프를 초래하기 때문이다.

```json
// INVALID: favoriteSpaceship does not exist on Character
{
  hero {
    favoriteSpaceship
  }
}
```

`hero` 필드가 반환하는 `Character` 타입에는 `favoriteSpaceship` 필드가 존재하지 않기 때문에 에러가 발생한다.

```json
// INVALID: hero is not a scalar, so fields are needed
{
  hero
}
```

`scalar type` 혹은 `enum type` 이외의 것을 리턴하는 필드에는 반드시 받고싶은 데이터에대한 정의(필드)가 필요하다.

```json
// INVALID: name is a scalar, so fields are not permitted
{
  hero {
    name {
      firstCharacterOfName
    }
  }
}
```

같은 맥락에서, `scalar type` 필드에 추가적인 필드를 요청할 수 없다.

```json
# INVALID: primaryFunction does not exist on Character
{
  hero {
    name
    primaryFunction
  }
}
```

`primaryFunction` 가 `Charater` 타입의 필드가 아니기 때문에 에러가 발생한다.

```json
{
  hero {
    name
    ...DroidFields
  }
}
​
fragment DroidFields on Droid {
  primaryFunction
}
```

`fragments` 를 활용하여 `Droid` 타입일 때에만 `primaryFunction` 필드가 추가되도록 함으로써 이를 해결할 수 있다.

```json
{
  hero {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```

`inline fragment` 를 활용하면 `fragment` 에 일일히 이름을 붙이지 않고 사용할 수 있기 떄문에 코드가 짧아진다.

> `named fragment` 는 해당 `fragment` 가 여러 곳에서 재사용될 때 유용하다.
