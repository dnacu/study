# Queries and Mutations

## Fields

필드를 작성함으로써 쿼리를 통해 원하는 데이터를 얻을 수 있다. 클라이언트 측에서 필드를 조합한 object(기댓값)에 따라 서버로부터 결과 데이터를 얻을 수 있다는 것과, 서버는 클라이언트에서 어떤 필드들에 대해 요청했는지를 정확히 알 수 있다는 점이 GraphQL의 핵심이다.

> 쿼리는 **interactive** 하다. 쿼리에 포함된 필드 조합을 변경함으로써 다른 결과를 얻을 수 있다.

필드들은 또한 object의 refer가 될 수 있다. 이는 sub-selection이 가능하다는 뜻이다. GraphQL은 필드들을 순회하며 한번에 여러 개의 관련 데이터를 가져올 수 있다.

> 기존 REST API에서는 이를 얻기 위해 학교 리스트 -> 특정 학교의 과 리스트 -> 특정 과의 특정 분반 -> 해당 분반의 학생. 이런 식으로 다수 요청에 의한 처리가 불가피한 경우가 있는데, GraphQL에서는 sub-selection과 field별 argument 전송 가능한 특징 때문에 이를 한번에 가져오는 것이 가능하다.

## Arguments

쿼리에 필요한 인자들을 정의할 수 있다. 이는 특정 data를 fetching해올 떄 굉장히 유용하게 사용될 수 있다.

기존 REST API에서는 arguments를 하나의 object를 통해 통째로 보냈었다. 하지만 GraphQL 에서는 모든 field 및 nested object에 각각 argument를 전달할 수 있다. 이 특징이 바로 multiple API fetches를 GraphQL 쿼리가 완벽하게 대체할 수 있는 이유이다. scalar field에도 argument를 전달 할 수 있다.

> 이러한 이점은 데이터 전송과정이 클라이언트에서 여러번 분리되어 일어나는 것이 아니라, 한 번에 일어날 수 있도록 해준다.

arguments들은 GraphQL에서 제공되는 간단한 타입 시스템에 따라 정의할 수 있다.

## Aliases

alias를 활용하여 내용이 같지만, arguments만 다른 여러 쿼리들에 대하여 구분해낼 수 있디.

```json
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

arguments가 다른 `hero` 필드가 두 개 있다. 각각 필드에 별칭(alias)을 붙임으로써 conflict를 해결할 수 있다.

## Fragments

GraphQL은 reusable unit인 fragment를 제공한다. fragment는 field들의 조합이며, 이를 정의함으로써 여러 필드에서 재활용 할 수 있다.

```json
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

### Using variables inside fragments

```json
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}
```

fragment에 argument를 설정할 수 있다.

## Operation name

`query`, `mutation`, `subscription` and `describes` 등의 operation name type들이 있다.

multi-operation 수행시에, 각 작업이 어떤 종류의 것인지를 명백히 드러낸다.

> GraphQL은 single-operation에서도 operation name을 명시할 것을 권장하는데, 이는 이것이 서버사이드 loggin에 굉장히 유용하기 때문이다. 함수이름, 변수이름을 잘 지어야 디버깅에 편하듯이 말이다.

## Variables

대부분의 어플리케이션에서 aurgments는 동적인 경우가 많다. 그렇기에 argument를 동적으로 쿼리스트링에 전달하는 방법을 variables를 통해 지원한다.

```json
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

> GrpahQL은 쿼리의 `string interpolation`(문자열을 조합하는) 을 지양한다.

### Variable definitions

- 변수의 형태는 scalars, enums, objects 가 될 수 있다.
- `$` prefix를 활용하여 사용한다.
- `($episode: Episode)` 는 타입언어처럼 동작한다. (`Episode` 가 타입)
- variables는 optional or required하다. `!` 를 통해 이를 나타낼 수 있다. (GraphQL 스키마 페이지 참고)

### Default variables

```json
($episode: Episode = JEDI)
```

의 형태로 variables에 대하여 default value를 전달 할 수 있다.

## Directives

variables를 통해 arguments를 동적으로 결정할 수 있지만, varibles를 통해 쿼리의 구조 자체를 동적으로 결정하는 방법이 없다. 그게 이거다.

```json
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}

// varibales...
{
  "episode": "JEDI",
  "withFriends": false
}
```

- `@include(if: Boolean)` 지시자를 통해서 `true` 일 때만 해당 정보를 포함하도록 할 수 있다.
- `@skip(if: Boolean)` 지시자를 통해서 `true`일 때 해당 정보를 스킵할 수 있다.

## Mutaions

GraphQL을 data fetching에 초점을 맞추어 설명한다. 하지만 완벽한 데이터 플랫폼이 되려면 서버사이드의 데이터를 수정할 수 있어야 한다.

REST에서 데이터의 수정에 대한 side-effects가 포함된 api method를 GET 메소드로 보내지 않는 것처럼, GraphQL에서는 mutation operation name을 통해 서버사이드 데이터의 수정이 일어남을 명시한다.

mutation 필드는 해당 필드정보에 맞는 수정된 object를 반환한다. 이를 업데이트 이후에 새로운 데이터를 받아오는 등에 유용하게 활용할 수 있다.

```json
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

```json
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```

### Multiple fields in mutation

하나의 쿼리에 여러 필드에 대한 변경을 포함할 수 있다. 쿼리와 뮤테이션의 차이는 이름 이외에도 차이가 있다.

**쿼리는 병렬적으로 수행되나, 뮤테이션은 순차적으로 실행된다.**
이는 우리가 여러 뮤테이션을 하나의 요청에 담아 보냈을 때, 첫 뮤테이션이 종료된 이후에 다음 뮤테이션이 실행됨을 보장해 준다는 것이다.

> 이를 통해 서버사이드 데이터 수정에 있어 발생할 수 있는 race condition등의 문제가 야기되지 않도록 한다.

## Inline Fragments
