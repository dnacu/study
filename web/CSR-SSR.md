# CSR / SSR

## 동작과정

### SSR

1. 서버에서 렌더링될 준비를 마친 HTML파일을 내려준다.
2. 브라우저는 페이지를 렌더링한다. **페이지는 우리눈에 보인다**
3. 브라우저가 JS를 다운로드하고 실행시킨다.
4. **페이지는 이제 인터렉션이 가능해진다.**

### CSR

1. 서버에서 HTML파일을 내려준다. (깡통)
2. 브라우저가 JS를 다운로드한다.
3. 다운로드한 JS파일을 실행시킨다.
4. **페이지는 우리눈에 보이고, 인터렉션이 가능해진다.**

> 가장 큰 차이는 SSR의 경우 렌더링 준비를 마친 온전한 HTML파일이 오지만, CSR은 JS링크가 포함된 빈 문서를 가져온다는 것이다. 그렇기에 SSR에서는 JS로딩 이전에도 페이지 렌더를 시작할 수 있지만, CSR은 그렇지 못하다. 렌더링에 대한 정보가 JS파일 내부에 있기 때문이다.

> 초기 렌더링 속도는 SSR이 더 빠르지만, 페이지 이동 시에도 같은 시간이 걸린다. 하지만 CSR은 초기에 받아왔고, 필요한 데이터만을 요청하여 보여주면 되므로 더 빠르다.

## SSR: Server Side Rendering
