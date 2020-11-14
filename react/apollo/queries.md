# Queries

`useQuery` 훅을 이용하여 서버로부터 데이터를 fetching하는 작업이다. `ApolloProvider` 컴포넌트로 리액트 앱을 감싸는 것이 선행되어야 한다.

## Executing a query

`useQuery` 훅은 query를 연산해주는 주요 API 중 하나이며, `loading`, `error`, `data`를 반환한다.

```js
const GET_DOGS = gql`
  query GetDogs {
    dogs {
      id
      breed
    }
  }
`;

function Dogs({ onDogSelected }) {
  const { loading, error, data } = useQuery(GET_DOGS);

  if (loading) return "Loading...";
  if (error) return `Error! ${error.message}`;

  return (
    <select name="dog" onChange={onDogSelected}>
      {data.dogs.map((dog) => (
        <option key={dog.id} value={dog.breed}>
          {dog.breed}
        </option>
      ))}
    </select>
  );
}
```

`loading` 값은 현재 쿼리가 실행중인지를 나타낸다. `loading` 값이 `false` 이고, `error` 값이 없다면, 우리가 실행한 쿼리가 정상적으로 동작을 마쳤음을 나타낸다.

`useQuery` 의 두 번째 인자로 옵션객체를 전달할 수 있다. `variables` 는 그중 하나이며, query에 필요한 변수들을 전달할 수 있다.

## Caching query results

쿼리를 통해 데이터를 fetching한 직후, 자동으로 해당 결과값을 캐싱한다. 쿼리실행시에 캐싱된 데이터가 있다면 서버까지 가지 않고 캐싱된 데이터를 반환한다.

## Updating cahced query results

캐싱된 쿼리결과는 사용하기 편하지만, 서버와의 동기화적인 문제점이 있다. 아폴로에서는 이를 해결하기 위해 두 가지 전략을 제시하는데, 이는 `polling` 과 `refetching` 이다.

### Polling

`Polling`은 **realtime에 가까운** 동기화를 보여준다. 주기적으로 서버로부터 데이터를 받아옴으로써, 캐싱데이터를 최신으로 유지한다. `useQuery`의 옵션객체에 `pollInterval` 를 추가함으로써 사용할 수 있다.

```js
const { loading, error, data } = useQuery(GET_DOG_PHOTO, {
  variables: { breed },
  pollInterval: 500,
});
```

`pollInterval` 값을 0으로 설정하면, 폴링이 이루어지지 않는다.

`useQuery` 로부터 반환되는 `startPolling` 과 `stopPolling`을 활용하여 폴링을 동적으로 관리할 수 있다.

### Refetching

`Refetching` 은 원하는 시점에 서버로부터 데이터를 다시 페칭한다는 점에서 `polling` 과의 차이점을 보인다.

```js
const { loading, error, data, refetch } = useQuery(GET_DOG_PHOTO, {
  variables: { breed },
});
```

`Refetching` 은 데이터를 최신으로 유지하는 좋은 방법이지만, `loading` 상태값에 대한 복잡성을 야기한다.

## Inspecting loading states

`loading` 상태값은 첫 로딩 시에는 유용하나, `polling` 및 `refetching` 이 일어날 때 indicator를 표현하고 싶은 경우에는 유용하지 못하다.

`useQuery` 는 이를 위해 `networkStatus` 객체를 반환한다. `notifyOnNetworkStatusChange` 옵션을 true로 설정해 줌으로써, 우리는 `polling` 및 `refetching` 의 로딩 상태가 변화를 추적할 수 있다.

[NetworkStatus](https://github.com/apollographql/apollo-client/blob/main/src/core/networkStatus.ts) 를 통해 상태값에 대한 정보를 볼 수 있다.

```js
import { NetworkStatus } from "@apollo/client";

function DogPhoto({ breed }) {
  const { loading, error, data, refetch, networkStatus } = useQuery(
    GET_DOG_PHOTO,
    {
      variables: { breed },
      notifyOnNetworkStatusChange: true,
    }
  );

  if (networkStatus === NetworkStatus.refetch) return "Refetching!";
  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <div>
      <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
      <button onClick={() => refetch()}>Refetch!</button>
    </div>
  );
}
```

## Inspecing error states

`errorPolicy` 옵션을 활용하여 쿼리에서 발생하는 에러에 대한 핸들링을 customize 할 수 있다. 기본값은 `none` 이며, 아폴로가 에러를 관리한다. 에러가 발생하면, 아폴로가 쿼리에 대한 응답을 폐기하고, `error` 상태값이 true로 변경된다.

만약 `errorPolicy`를 `all` 로 설정하면 `useQuery` 는 응답 데이터를 폐기하지 않아 부분적인 렌더링이 가능하게 해준다(?)

## Executing queries manually

리액트가 컴포넌트를 마운트/렌더하면서 `useQuery` 훅을 호출한다. 만약 특정 이벤트가 발생했을 때 데이터를 가져오고 싶다면 어떻게 해야할까?

`useLazyQuery` 를 사용하면 위 문제가 완벽하게 해결된다. `useQuery` 와 같은 역할을 하지만, 호출 시점에 즉각적으로 쿼리에 대한 연산을 수행하지 않는다. 반환하는 tuple형 객체의 첫 번째 인자를 활용하여 원하는 시점에 쿼리연산을 진행할 수 있다.

```js
const [getDog, { loading, data }] = useLazyQuery(GET_DOG_PHOTO);
```

## Setting a fetch policy

기본적으로 `useQuery` 훅은 아폴로 클라이언트의 캐시를 확인한다. 만약 사용 가능한 데이터가 있다면, GraphQL 서버로 쿼리를 전송하지 않고 캐싱된 데이터를 반환한다. `cache-first`이 default fetch policy이다.

```js
const { loading, error, data } = useQuery(GET_DOGS, {
  fetchPolicy: "network-only",
});
```

| name                | description                                                                                                                                                   |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cache-first`       | 기본 옵션. 캐싱된 데이터가 있다면 그것을 반환한다.                                                                                                            |
| `cache-only`        | 캐시에 데이터가 없다면 에러를 던진다.                                                                                                                         |
| `cache-and-network` | 캐시와 서버 둘 다에 대하여 쿼리를 수행한다. 서버측 쿼리 결과가 캐시를 수정하면 쿼리가 업데이트된다.                                                           |
| `network-only`      | 캐시를 체크하지 않고 서버에 쿼리를 요청한다. 쿼리 결과는 캐시에 저장된다.                                                                                     |
| `no-cache`          | 캐시를 체크하지 않고 서버에 요청하며, 쿼리결과도 저장하지 않는다.                                                                                             |
| `standby`           | `cache-first` 와 비슷하다. 쿼리의 field가 수정되어도 동으로 업데이트되지 않는다. `refetch`나 `updateQueries` 메소드를 통해 직접 쿼리를 업데이트해주어야 한다. |

## 참고

- https://www.apollographql.com/docs/react/data/queries/
