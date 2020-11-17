# `info` argument on resolver function

resolver함수는 각 object필드에 대한 데이터를 처리하는 함수이다.

`parent`, `args`, `context`, `info`의 총 4가지 인수를 받는데, 그 중 필드에 대한 정보 및 전체 스키마 정보를 가지고 있는 `info` 객체에 대해 살펴보자.

```js
type GraphQLResolveInfo = {
  fieldName: string,
  // 나머지 필드들의 selectionSet을 보여준다.
  // AST 트리의 일부(해당 필드부터 같은 레벨 필드순으로, 스칼라타입이 나올때까지 계속 타고들어가면서 보여줌)
  // 각 필드의 name, arguments, directives, selectionSet 등에 대한 정보가 포함된다.
  fieldNodes: Array<Field>,
  // 해당 필드의 Type
  returnType: GraphQLOutputType,
  // 해당 필드가 속한 필드(상위필드)의 type
  parentType: GraphQLCompositeType,
  // GraphQLSchema
  schema: GraphQLSchema,
  // 쿼리에 사용된 fragments들
  fragments: { [fragmentName: string]: FragmentDefinition },
  rootValue: any,
  // 루트부터 시작하는 전체 쿼리 AST (fieldNodes는 일부)
  operation: OperationDefinition,
  // graphql 실행 함수에 전달된 인수들
  variableValues: { [variableName: string]: any },
};
```

```js
type Query {
  author: User!
  feed: [Post!]!
}
```

`fieldName` 은 `author`, `returnType` 은 `User!`, `parentType` 은 `Query` 이다.

이 세 키값은 필드별로 다르다. 하지만 `schema`, `fragments`, `rootValue`, `operation`, `variableValues` 키에 대한 값은 resolver에 관계없이 항상 동일한 값을 가지는 Global 값이다.

## 참고

- https://www.prisma.io/blog/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a
