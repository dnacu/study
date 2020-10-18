# `this`

1. this는 실행 환경을 소유한 소유자 객체에 연결(바인딩)된다.
2. 전역 실행 환경에서 this는 전역 객체(브라우저인 경우 window)를 연결한다.
3. 객체 내의 this는 소유자 객체가 확인될 때 소유자 객체를 연결한다.
4. new 연산자로 실행된 함수 안에서 this는 새로운 생성 객체를 연결한다.
5. 화살표 함수 내에서 this는 화살표 함수 실행 맥락을 연결한다.
6. call, apply로 호출된 함수 안에서 this는 call, apply의 첫 번째 인자로 지정된 객체로 연결된다.
7. bind로 컨텍스트를 명시한 함수 내의 this는 명시한 컨텍스트로 고정된다.

## 예제

```js
// 객체 메소드
let counter = {
  count: 0,
  increment() {
    this.count++;
  },
};

setInterval(counter.increment, 100);

setInterval(function () {
  console.log(counter.count);
}, 1000);

// 함수 메소드
function Counter() {
  this.count = 0;

  this.increment = function () {
    this.count++;
  };

  setInterval(this.increment, 100);
}

let counter = new Counter();

setInterval(function () {
  console.log(counter.count);
}, 1000);

// call, apply로 this 조작
let counter = {
  count: 0,
  increment() {
    this.count++;
  },
};

let fakeCounter = {
  count: 100,
};

counter.increment.call(fakeCounter);

console.log(counter.count);
console.log(fakeCounter.count);
```

## 해결

```js
// 소유자를 명시함 (1번)
let counter = {
  count: 0,
  increment() {
    this.count++;
  },
};

setInterval(function () {
  counter.increment();
}, 100);

setInterval(function () {
  console.log(counter.count);
}, 1000);

// 바인딩
function Counter() {
  this.count = 0;

  this.increment = function () {
    this.count++;
  };

  setInterval(this.increment, 100);
}

let counter = new Counter();

setInterval(function () {
  console.log(counter.count);
}, 1000);

// 화살표함수를 이용한 바인딩 (5번)
function Counter() {
  this.count = 0;

  setInterval(() => this.count++, 100);
}

let counter = new Counter();

setInterval(function () {
  console.log(counter.count);
}, 1000);
```

## 출처

https://fastcampus-js-bootcamp.herokuapp.com/javascript/understanding-execution-context
