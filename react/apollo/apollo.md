# Apollo Client

`Apollo Client` 는 `local state` 와 graphql을 활용한 `remote state` 를 관리해주는 라이브러리이다.

## 장점

- 기존의 API를 활용했을 때는 받는 데이터를 Typescript의 interface 등으로 명세해두는 작업을 했었는데, 이는 API 명세가 바뀔 때마다 해당 인터페이스를 직접 수정해야 하는 동기화 이슈가 있었다. apollo에서는 query를 사용해서 graphql 서버로부터 데이터를 받아오는데, 작성 시 받아올 데이터에 대한 명세가 클라이언트 측에 정의되어 있다. query 내부의 field를 수정하면 수신 데이터의 형태도 따라 바뀌기에 둘의 동기화에 대한 걱정거리가 사라진다고 생각한다.
