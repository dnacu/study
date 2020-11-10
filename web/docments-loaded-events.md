# Docments loaded events

## document.DOMContentLoaded

브라우저가 HTML을 전부 읽어들인 이후, DOM트리를 완성하는 즉시 발생한다. 이미지파일이나 스타일시트 등의 여타 리소스들의 로딩과는 관련이 없다.

이 이벤트는 원하는 DOM노드를 찾아 이벤트 핸들러를 등록하는 등의 인터페이스 초기화 작업에 사용될 수 있다.

`document` 객체에서 발생하기 때문에, `addEventListenr` 를 사용한다.

```js
document.addEventListenr("DOMContentLoaded", eventHandler);
```

### with scripts

기본적으로 브라우저는 HTML문서를 처리하는 도중에 script태그를 만나면 DOM트리 구성을 멈추고 script를 실행한 뒤에야 다시 HTML문서를 처리한다. 따라서 HTML문서 중간에 script 태그가 있다면, 해당 script가 처리되고, HTML문서 처리가 완료되면 `DOMContentLoaded` 이벤트가 발생함에 유의하자.

- `async` 속성을 가진 script는 `DOMContentLoaded` 이벤트 발생을 막지 않는다.
- `document.createElement('script')` 처럼 동적으로 생성되어 웹페이지에 추가되는 script는 `DOMContentLoaded` 이벤트 발생을 막지 않는다.

### browser form autofill

브라우저의 폼 자동완성(아이디, 비밀번호 등)은 `DOMContentLoaded` 이벤트에서 일어난다. (사용자가 자동완성을 허용했다면)

따라서 매우 큰 script때문에 `DOMContentLoaded` 이벤트의 발생이 늦춰진다면, 자동완성이 지연되어 유저가 불편함을 느낄 수 있기 떄문에 조심하자.

## window.onload

`window` 객체의 `unload` 이벤트는 스타일시트, 이미지 등의 리소스들이 모두 로드되었울 때 발생한다. 그렇기에 `DOMContentLoaded` 에서는 확인하지 못할 수 있는 이미지 크기 등의 것들은 이 이벤트가 발생된 이후에 살행되어야 한다.

## window.onunload

`window` 객체의 `unload` 이벤트는 사용자가 페이지를 떠날 때(문서를 완전히 닫을 때) 실행된다. unload 이벤트에선 팝업창을 닫는 것과 같은 딜레이가 없는 작업을 수행할 수 있다(?)

이 이벤트는 사용자가 웹사이트에서 어떤 행동을 하는지에 대한 분석 정보를 모을 때 사용할 수 있다. 결과응답이 중요하지 않은 비동기 작업을 백그라운드에서 실행할 수 있도록 해주는 `navigator.sendBeacon(url, data)` 메소드를 사용하면, 사용자가 페이지를 떠날 때 분석정보를 백그라운드에서 서버로 송신할 수 있다.

> 링크 클릭 시에 분석정보를 보내고 해당 링크로 이동하는 방식은 좋지 않다.

```js
let analyticsData = { /* 분석 정보가 담긴 객체 */ };

window.addEventListener("unload", function() {
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData));
};
```

- 요청은 `POST Method` 로 전송된다.
- 문자열 형태의 객체를 전달한다.
- 데이터는 64kb제한이 있다.
- 서버응답을 받을 수 있는 방법이 없는 경우에 주로 사용되기 때문에 응답은 대게 빈 상태이다.

## window.onbeforeunload

사용자가 현재 페이지를 떠나 다른 페이지로 이동하려 할 때나 창을 닫으려고 할 때 beforeunload 핸들러에서 추가적인 확인을 요청할 수 있다.

```js
window.onbeforeunload = function () {
  return false;
};
```

beforeunload 이벤트를 취소하려 하면 브라우저는 사용자에게 종료 여부에 대한 컨펌창을 띄워준다.

> 문자열을 리턴하면 컨펌창에서 보였었지만, 모던 브라우저에서는 그렇지 않다.

## 참고

- https://ko.javascript.info/onload-ondomcontentloaded
