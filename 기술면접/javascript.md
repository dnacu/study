# JS에서 모듈 내의 private한 속성을 만드는 방법

## Closure

내부 변수로 선언해두면 외부에서 접근할 수 없다. `getter`, `setter`역할을 하는 함수를 정의하여 해당 함수 내에서만 접근하게 만들면 된다.

```js
function SomeConstructor() {
  const privateProp = 'dont touch this';
  this.publicProp = 'you can touch this';

  this.doSomethingWithPrivateProp = () => { ... }
}
```

## Symbol

외부에서는 `privateMethodName`과 `privatePropName`의 값을 알 수 없기 때문에 접근이 불가능하다. 모든 Symbol은 유니크하기 때문에 새로운 심볼을 생성하여 접근할 수도 없다.

```js
const privateMethodName = Symbol();
const privatePropName = Symbol();

class SomeClass {
  [privatePropName] = "dont touch this";
  publicProp = "you can touch this";

  [privateMethodName]() {
    console.log("private method");
  }

  publicMethod() {
    this[privateMethodName](this[privatePropName]);
  }
}
```

## Hash prefix

ES2019의 hash prefix 문법. **모든 Private 필드는 소속된 클래스에 고유한 스코프를 갖는다.**

```js
class Human {
  #age = 10;
}

const person = new Human();

console.log(person.#age); // Error TS18013: Property '#age' is not accessible outside class 'Human' because it has a private identifier.
```

```js
class Human {
  #age = 10;

  getAge() {
    return this.#age;
  }
}

class Person extends Human {
  #age = 20;

  getFakeAge() {
    return this.#age;
  }
}

const p = new Person();
console.log(p.getAge()); // 10
console.log(p.getFakeAge()); // 20
```

## 참고

- [TOAST 포스팅](https://ui.toast.com/weekly-pick/ko_20200312/)

---

# 실행 컨텍스트, 클로저, 호이스팅

실행 컨텍스트란 코드의 실행 환경이다.

- 처음 코드를 실행하는 순간 **전역 컨텍스트**가 생성되고, 종료될 때까지 유지된다. 함수가 실행될 때마다 **함수 컨텍스트**가 생성된다.
- 컨텍스트 생성 시 **변수객체(arguments, variables), scope chain, this**가 생성된다.
- 컨텍스트 생성 후에 함수가 실행된다. 사용되는 변수들은 해당 스코프 안에서 찾고, 없다면 스코프 체인을 따라 올라가며 찾는다.
- 함수 실행이 마무리되면 해당 컨텍스트는 사라진다.

## 실행 컨텍스트의 생성 과정

### 1. Creation Phase

**Lexical Environment**(let/const/함수선언)와 **Variable Environment**(var)를 생성한다.

**Lexical Environment**

- 변수와 함수 선언(함수 표현식 x)을 저장한다.
- 외부 lexical 환경으로의 참조값을 저장한다. (scope chain을 저장한다.)
- `this`의 값을 결정한다.

### 2. Execution Phase

실제 코드를 실행하며, 저장해두었던 선언부에 값을 대입하는 과정을 거친다.

## 호이스팅

이렇듯 JS는 함수를 실행시키기 이전 `lexical environment` 생성 과정에서 선언에 대한 정보를 먼저 기록하는 작업을 거치기 때문에 **호이스팅**(선언부가 최상단으로 끌어올려지는 현상)이 일어난다.

## 클로저

반환된 내부함수가 자신이 선언되었을 때의 환경(lexical scope)를 기억한다.

**렉시컬 스코핑(lexical scoping)**이란 scope는 함수가 호출될 때가 아닌 **선언될 때** 생성된다.

> 외부함수의 실행 컨텍스트는 반환되나, 외부함수의 변수를 필요로 하는 반환된 내부함수가 하나라도 존재한다면 GC(garbage collector)에 의해 회수되지 않고 남는다. c언어에서 동적할당 해주고 free 안해준거랑 비슷한 것 같다. 클로저 변수를 null로 변경함으로써 GC가 회수해 갈 수 있도록 하는 환경을 만들어 주면 메모리 낭비를 막을 수 있다.

## 참고

- [imacoolgirlyo님의 글](https://velog.io/@imacoolgirlyo/JS-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-Hoisting-The-Execution-Context-%ED%98%B8%EC%9D%B4%EC%8A%A4%ED%8C%85-%EC%8B%A4%ED%96%89-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-6bjsmmlmgy)
- [zerocho님의 글](https://www.zerocho.com/category/JavaScript/post/5741d96d094da4986bc950a0)
