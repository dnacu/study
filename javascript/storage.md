# Storage

## Web Storage vs Cookie

- 쿠키는 매번 서버로 전송된다.
- 쿠키는 만료 시간이 존재한다.
- 쿠키는 용량이 작고(4KB) 갯수제한도 존재하는 반면, 웹 스토리지는 훨씬 크다.(브라우저마다 다르지만 평균 5MB 정도)

## LocalStorage vs SessionStorage

- LocalStorage는 영구적으로 저장이 가능하지만, SessionStorage는 브라우저 탭 종료시 초기화된다.

- 둘 다 도메인별로 생성된다. LocalStorage는 브라우저에 관계없이 공유되는 데 반해 SessionStorage는 브라우저를 하나 더 키거나, 새 탭을 켰을 때 각기 다른 저장소를 가진다.

- 크로스 브라우저를 지원하지 않는다. 말 그대로 `local`이다.
