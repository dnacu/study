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

