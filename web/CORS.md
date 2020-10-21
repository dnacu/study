# CORS

CORS(Cross-Origin Resource Sharing)

## 출처(Origin)란?

`https://www.google.com/users?sort=asc&page=1#foo` 라는 하나의 URL은 여러 구성요소로 이루어져 있다.

`https://` 는 `Protocol`, `www.google.com`은 `Host`, `/users`는 `Path`, `?sort=asc&page=1`는 `Query String`, `#foo`는 `Fragment`.

이 때 출처(Origin)란 `Protocol`과 `Host`, `Port Number(:443(https), :80(http))` 까지 모두 합친 것을 의미한다.

> `window.location.origin` 프로퍼티를 활용하여 현재 웹 페이지의 출처를 알아낼 수 있다.

## 동일 출처 정책(SOP: Same-Origin Policy)

일반적으로 다른 출처에서 정보를 읽는 것은 금지되어 있다. 예외 상황으로 다른 출처로부터 정보를 읽어와야 한다면, `CORS(Cross-Origin Resource Sharing)`정책을 따라야 한다.

> 출처 비교 로직은 브라우저에 구현되어있는 스펙이다. 때문에 서버 간 통신에서는 적용되지 않는다.

## 교차 출처 리소스 공유(CORS: Cross-Origin Resource Sharing)

### 동작 과정

1. HTTP 요청 시 브라우저가 요청 헤더의 `Origin`에 출처를 담아 요청한다.
2. 서버는 응답 헤더의 `Access-Control-Allow-Origin`에 \**해당 리소스를 접근하는 것이 허용된 출처*에 대한 정보를 담아 응답한다.
3. 브라우저는 자신이 보냈던 `Origin` 헤더값과 서버 응답의 `Access-Control-Allow-Origin` 헤더값을 비교하여 유요한 응답인지(CORS 정책에 위반되지 않는지)를 판단한다.

세 가지의 시나리오에 따라 CORS의 동작 방식이 달라진다.

### Preflight Request

브라우저가 본 요청을 보내기 전에 예비 요청(`Preflight Request`)을 보낸다. `HTTP OPTIONS`메소드가 사용되며, 이 요청의 역할은 본 요청 전에 브라우저가 해당 요청이 안전한지(CORS정책을 위반하지는 않는지) 확인하는 작업이다.

![image](https://user-images.githubusercontent.com/36905916/96689993-19a6ae00-13be-11eb-87b4-be9cfdb95e09.png)

### Simple Request

브라우저가 모든 상황에서 위처럼 요청을 두 번씩 보내는 것은 아니다.

`Simple Request` 의 경우에는, `Preflight Request` 와 전반적인 로직이 비슷하게 흘러가나, 예비 요청의 존재 유무가 다르다. 특정 조건을 만족하는 경우에만 예비 요청을 생략할 수 있다.

1. 요청의 메소드는 `GET, HEAD, POST` 중 하나여야 한다.
2. `Accept, Accept-Language, Content-Language, Content-Type, DPR, Downlink, Save-Data, Viewport-Width, Width`를 제외한 헤더를 사용하면 안된다.
3. 만약 `Content-Type`를 사용하는 경우에는 `application/x-www-form-urlencoded, multipart/form-data, text/plain`만 허용된다.

> 위 세 조건을 모두 만족해야 하는데, 쉽지않다. 1번은 쉽다. 2번은 매우 기본적인 헤더값만 있어 `Authorization`헤더를 추가하는 등의 작업조차 허용되지 않는다. 3번의 경우 `text/xml, application/json`을 많이 사용하기 때문에 세 조건을 만족하긴 어렵다.

### Credentialed Request

인증된 요청을 사용하는 방법이다. 교차 출처간 통신에서 보안을 강화하고자 할 때 사용하는 방법이다.

기본적으로 `XMLHttpRequest` 객체나 `Fetch API` 는 별도의 옵션 없이 브라우저 쿠키정보나 인증관련 헤더를 요청에 담지 않는다. 이 때 인증관련 정보를 담을 수 있게 해주는 옵션이 `credentials` 옵션이다.

| 옵션                | 값 설명                                        |
| ------------------- | ---------------------------------------------- |
| same-origin(기본값) | 같은 출처 간 요청에만 인증 정보를 담을 수 있다 |
| include             | 모든 요청에 인증 정보를 담을 수 있다           |
| omit                | 모든 요청에 인증 정보를 담지 않는다            |

`include`옵션 사용 시 추가적인 제약이 따른다.

1. `Access-Control-Allow-Origin`에는 \*를 사용할 수 없으며, 명시적인 URL을 작성하여야 한다.
2. 응답 헤더에는 반드시 `Allow-Control-Allow-Credentials: true`가 존재해야한다.

## CORS 문제 해결하기

CORS정책 위반으로 인한 문제를 해결하는 방법을 알아보자.

### `Access-Control-Allow-Origin` 세팅

서버에서 `Access-Control-Allow-Origin` 헤더값을 알맞게 세팅해주는게 가장 기본적이다.

와일드카드인 `*` 으로 세팅하게 되면 보안적 문제가 생길 수 있으니, 가급적이면 출처를 명시해주는 것이 좋다.

### Webpack Dev Server로 리버스 프록싱

```js
module.exports = {
  devServer: {
    proxy: {
      "/api": {
        target: "https://api.evan.com",
        changeOrigin: true,
        pathRewrite: { "^/api": "" },
      },
    },
  },
};
```
