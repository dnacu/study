# Mutations

`useQuery` 를 통해 데이터를 fetching 할 수 있었다면, `useMutation` 을 통해 GraphQL 서버에 있는 데이터를 변경할 수 있다.

## Executing a mutation

컴포넌트가 렌더될 때, `useMutation` mutate function과 state를 담은 tuple 객체를 반환한다.

```js
const ADD_TODO = gql`
  mutation AddTodo($type: String!) {
    addTodo(type: $type) {
      id
      type
    }
  }
`;

function AddTodo() {
  let input;
  const [addTodo, { data }] = useMutation(ADD_TODO);

  return null;
}
```

## Updating the cache after a mutation

mutation을 수행하면, 백엔드 데이터가 수정된다. 이 수정사항을 아폴로 캐시에도 적용할 수 있다.

### updating a single existing entity

single entity 에 대한 mutation이 일어났을 때, 아폴로는 자동으로 캐시에서도 해당 엔티티에 대한 수정을 진행한다. 그러기 위해서, mutation은 변경된 엔티티의 `id` 값을 반환해야만 한다. 아폴로 mutation은 이를 자동으로 수행한다.

> 아폴로 클라이언트 캐시의 엔티티들은 `id` 값을 기반으로 관리된다. 따라서 mutation 수행 시 `id` 값을 포함시켜 자동으로 캐시가 수정되게 하거나, 그럴 수 없는 경우에는 캐시 관리 메소드를 활용하여 직접 로직을 구현해주어야 한다.

### Making all other cache updates

```js
const [addTodo] = useMutation(ADD_TODO, {
  update(cache, { data: { addTodo } }) {
    cache.modify({
      fields: {
        todos(existingTodos = []) {
          const newTodoRef = cache.writeFragment({
            data: addTodo,
            fragment: gql`
              fragment NewTodo on Todo {
                id
                type
              }
            `,
          });
          return [...existingTodos, newTodoRef];
        },
      },
    });
  },
});
```

여러 엔티티에 대한 수정은 자동 캐시 업데이트를 지원하지 않기 때문에 관련 로직을 직접 구현해 주어야 한다. `useMutation` 의 두 번째 인자 객체에 update 함수를 전달함으로써 캐시 업데이트 로직에 대한 코드를 실행시킬 수 있다.

`update` 함수에는 `cache` 객체가 전달되며, 이 객체의 여러 메소드들(`readQuery`, `writeQuery`, `readFragment`, `writeFragment`, `modify`)을 통해 캐시를 수정할 수 있다.

mutation을 통해 새로운 todo 데이터가 캐시에 추가되었다고 해보자. 이전 todo list 에 대한 캐싱값이 다른 쿼리로부터 참조되고 있고, 자동으로 업데이트되지 않는다. 이는 쿼리가 새로운 엘리먼트가 추가되었는지 등에 대한 이벤트를 알람받지 못한다는 의미이고, 화면에 새 todo가 보이지 않을 것이다. `cache.modify` 는 새로운 오브젝트를 생성하는 것이 아니라, 기존 객체를 수정한다. 각 쿼리들은 데이터가 바뀌었음을 인지할 수 있으며, 화면은 알맞게 업데이트 될 것이다.

## Tracking loading and error states

`loading`, `error` 상태값이나, `onError`, `onCompleted` 콜백을 등록함으로써 mutation의 상태에 따라 다른 작업을 수행할 수 있다.
