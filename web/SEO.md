# 검색

## 간단한 검색 동작방식

1. 검색 서비스가 주기적으로 웹 콘텐츠를 수집 (크롤링)
2. 수집한 웹 사이트 중 유저가 검색한 검색어와 가장 일치하는 웹사이트를 순서대로 노출

## 웹 콘텐츠 수집

구글의 웹 크롤러 `Goolebot`이 주기적으로 웹사이트들을 탐색하여 검색 가능한 컨텐츠를 추가함.

`Robots.txt` 파일을 이용하여 특정 `PATH`가 `Goolebot`으로부터 수집되지 못하도록 할 수 있다.

```
User-agent: *
Disallow: /admin/
Disallow: /internal/
```

외부인이 admin 또는 사내 페이지의 경로를 알 수 있기 때문에 도메인을 분리하거나, 로그인 등의 인증을 통하여 막는 것이 권장되는 방법이다. `Goolebot`은 인증이 필요한 부분에 대해서 검색결과로 노출시키지 않는다.

> Googlebot이 인증과정까지 거칠 수 없기 때문에 노출시킬 수 없지 않을까란 생각을 해본다.

## Goolebot의 동작방식

1. HTML 다운로드 및 해석
2. CSS, JS파일이 있다면 모두 포함하여 렌더
3. 최종 결과값을 기준으로 수집

대부분 JS에 대해서 무시한다고 생각하지만 전혀 그렇지 않다. Puppeteer를 이용하여 페이지를 렌더하고 결과물을 수집한다.

검색 결과 순위에 페이지 렌더링 속도가 조건으로 포함되어있기 때문에 SSR이 CSR보다 SEO에 도움이 될 수 있다.

> Next.js의 SSR, code splitting 기능을 지원받아 프로젝트를 구성한다면 초기 렌더링 속도의 상승을 꾀할 수 있기 때문에 SEO에 유리하다.

**Puppeteer란?**  
Headless Chrome 혹은 Chromium를 제어하도록 도와주는 라이브러리

**Headless Browser란?**  
CLI에서 동작하는 브라우저. 실제로 브라우저 창을 띄우지 않고 백그라운드에서 가상으로 진행되며 특정 페이지에 접속하고 렌더링한다.

# SEO - Search Engine Optimization

검색 결과에서 상위에 노출될 수 있도록 웹 콘텐츠를 최적화하는 작업.

1. 초기 렌더링 속도를 높힐 것
2. 의미있는 HTML태그를 사용할 것

- 콘텐츠 제목에 따른 적절한 title태그 사용
- 콘텐츠에 따른 적절한 description메타태그 사용

```html
<meta name="description" content="콘텐츠의 설명" />
```

3. 반응형 웹 디자인으로 모바일과 데스크탑을 적절히 지원할 것
4. 유저의 방문 비율

### 참고 페이지 링크

- https://medium.com/@euncho/google-%EA%B2%80%EC%83%89%EC%9D%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94%EA%B0%80-fbc0e421a97a
- https://drive.google.com/file/d/1cArGPApYvbrAVP01X7-Q1BVsRqHyKVDE/view
- [google search console](https://support.google.com/webmasters/answer/7451184)
