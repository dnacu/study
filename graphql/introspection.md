# Introspection (스키마 확인)

종종 GraphQL schema가 어떤 쿼리를 지원하는지에 대한 정보를 요청하는 것이 유용할 때가 있다. `introspection` 이 이를 가능케 한다.

타입 시스템을 사용하기 때문에 우리는 어떤 타입이 유효한지 알지만, 그렇지 않은 경우에는 쿼리의 루트 타입에서 항상 사용할 수 있는 `__schema` 필드를 이용하여 GraphQL에 요청할 수 있다.

```js
{
  __schema {
    types {
      name
    }
  }
}

// reponses
{
  "data": {
    "__schema": {
      "types": [
        {
          "name": "Query"
        },
        {
          "name": "Episode"
        },
        {
          "name": "Character"
        },
        {
          "name": "ID"
        },
        {
          "name": "String"
        },
        {
          "name": "Int"
        },
        {
          "name": "FriendsConnection"
        },
        {
          "name": "FriendsEdge"
        },
        {
          "name": "PageInfo"
        },
        {
          "name": "Boolean"
        },
        {
          "name": "Review"
        },
        {
          "name": "SearchResult"
        },
        {
          "name": "Human"
        },
        {
          "name": "LengthUnit"
        },
        {
          "name": "Float"
        },
        {
          "name": "Starship"
        },
        {
          "name": "Droid"
        },
        {
          "name": "Mutation"
        },
        {
          "name": "ReviewInput"
        },
        {
          "name": "__Schema"
        },
        {
          "name": "__Type"
        },
        {
          "name": "__TypeKind"
        },
        {
          "name": "__Field"
        },
        {
          "name": "__InputValue"
        },
        {
          "name": "__EnumValue"
        },
        {
          "name": "__Directive"
        },
        {
          "name": "__DirectiveLocation"
        }
      ]
    }
  }
}
```

정리하면,

- **Query, Character, Human, Episode, Droid** : 타입 시스템에서 정의한 것
- **String, Boolean, Int, ID etc...** : 타입 시스템에서 제공하는 built-in scalars
- ****Schema, **Type, **TypeKind, **Field, **InputValue, **EnumValue, \_\_Directive** : 모두 앞에 두개의 밑줄(\_\_)이 붙어있다는 특징이 있다. 이것은 `introspection` 시스템의 일부임을 나타낸다.

이제, 어떤 쿼리를 사용할 수 있는지 알아보자. 타입 시스템을 설계할 때, 시작점에서 사용되는 모든 쿼리들에 대해 특정했다.

```js
{
  __schema {
    queryType {
      name
    }
  }
}

// response
{
  "data": {
    "__schema": {
      "queryType": {
        "name": "Query"
      }
    }
  }
}
```

타입 시스템 섹션에서 설명했던 것과 동일하게, `Query` 타입이 우리의 시작 지점이다. 이름은 컨벤션일 뿐이기에 다를 수 있지만, 쿼리의 시작지점이 반환되는것은 동일하다. 하지만, 일반적으로 `Query` 라는 이름을 사용하는 것이 관습이다.

## 예시

```js
{
  __type(name: "Droid") {
    name
    fields {
      name
      type {
        name
        kind
        ofType {
          name
          kind
        }
      }
    }
  }
}
```

```js
{
  "data": {
    "__type": {
      "name": "Droid",
      "fields": [
        {
          "name": "id",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "ID",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "name",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "String",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "friends",
          "type": {
            "name": null,
            "kind": "LIST",
            "ofType": {
              "name": "Character",
              "kind": "INTERFACE"
            }
          }
        },
        {
          "name": "friendsConnection",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "FriendsConnection",
              "kind": "OBJECT"
            }
          }
        },
        {
          "name": "appearsIn",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": null,
              "kind": "LIST"
            }
          }
        },
        {
          "name": "primaryFunction",
          "type": {
            "name": "String",
            "kind": "SCALAR",
            "ofType": null
          }
        }
      ]
    }
  }
}
```

`kind` 키에 대한 값이 `NON-NULL` 인 것들은 `wrapper` 타입이다. `name` 프로퍼티를 통해 값을 즉시 알 수 없으며, `type` 필드 안에서 `ofType` 을 쿼리하면 해당 타입에 대해 알 수 있다.
