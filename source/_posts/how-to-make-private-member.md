---
title: ES6 Class에서 private member를 정의하는 방법
subtitle: 자바스크립트를 어떻게든 더 잘 써보려는 몸부림
date: 2016-11-27 17:34:00
tags: [es6, javascript, class]
author: gomugom(정재남)
email: power4ce@gmail.com
cover: "/img/home-bg.jpg"
---

es6의 class 문법에는 private data를 직접 지정할 수 있는 기능이 제공되지 않는다.
때문에 private data로 쓰고자 하는 변수는 우회적으로 관리하여야 하는데, 그 방법들을 소개한다.

<!-- more -->


---

## 1. naming convention `_`

변수에 접두어 `_`를 붙이면 private data로 간주하기로 하는 규칙을 정하는 방법.

실질적인 접근제한은 전혀 이뤄지지 않는다.


## 2. `constructor` 내부에서 할당

```js
class Count {
  constructor(_count) {
    const methods = {
      inc() { _count += 1; return _count; },
      dec() { _count -= 1; return _count; },
      getScore() { return _count; },
      get score() { return _count; },
      set score(v) { _count = v; }
    };
    Object.assign(this, methods);
  }
}
const test = new Count(0);
console.log(test.inc());        // 1
console.log(test.inc());        // 2
console.log(test.dec());        // 1
console.log(test.getScore());   // 1
```

이 방법은 constructor 내부에서만 접근 가능한 변수를 사용하는 모든 메소드를 constructor에서 정의하고,
이를 그대로 인스턴스에 반영하기 위해 `Object.assign` 메소드를 활용한다.
이로써 `_count` 변수는 값을 외부에 노출하지 않고 오직 내부에서만 접근이 가능해진다.

그러나 이는 메소드를 인스턴스에 직접 할당하는 것이므로,
메소드를 상속받아 사용하겠다는 Class의 본질적인 사용 의미를 무색케 만드는 셈이다.
또한 delete로 메소드를 삭제할 수도 있고, 메소드를 override가 아닌 완전한 대체를 할 수도 있다.

뿐만 아니라  `getter/setter`는 별도의 데이터(`this.score`)에 접근하는 등의 문제도 있다
(원인은 모르겠다. 아시는 분은 댓글 부탁드립니다).

```js
console.log(test.score);         // 0
test.score = 20;
console.log(test.score);         // 20
console.log(test.getScore());    // 1
```

한편 method를 모두 `this.constructor.prototype`에 할당한다면 _count 변수를 공통으로 사용하는 결과가 되므로,
각 인스턴스들의 독립성이 보장되지 않게 되어 마찬가지로 Class를 사용하는 의미가 없다.

```js
class Count {
  constructor(_count) {
    const methods = {
      inc() { _count += 1; return _count; },
      dec() { _count -= 1; return _count; },
      getScore() { return _count; },
      get score() { return _count; },
      set score(v) { _count = v; }
    };
    Object.assign(this.constructor.prototype, methods);
  }
}
const test1 = new Count(0);
const test2 = new Count(0);
console.log(test1.inc());        // 1
console.log(test1.inc());        // 2
console.log(test1.getScore());   // 2
console.log(test2.getScore());   // 2
```

이래저래 변수보호를 위해 잃는 것이 너무 많은 방법. 비추천.


## 3. `Symbol` 활용

즉시실행함수 혹은 블록 스코프 내에서 심볼을 통해 접근을 제한하는 방법이다.

```js
const Count = (() => {
  const count = Symbol('COUNT');
  class Count {
    constructor() {
      this[count] = 0;
    }
    inc() {
      return ++this[count];
    }
    dec() {
      return --this[count];
    }
    get score() { return this[count]; }
    set score(n) { this[count] = n; }
  }
  return Count;
})();
const test = new Count();
console.log(test.inc());   // 1
console.log(test.inc());   // 2
console.log(test.dec());   // 1
console.log(test.score);   // 1
test.score = 10;
console.log(test.score);   // 10
console.log(test.inc());   // 11
```

이 방법은 Symbol의 접근 루트가 제한적이라서 가능한 방법이지만, 접근 루트가 아예 없는 것은 아니다.

```js
const testSymbol = Object.getOwnPropertySymbols(test)[0];
test[testSymbol] = 20;
console.log(test.score);    // 20
console.log(test.inc());    // 21
```

```js
const testSymbol = Reflect.ownKeys(test)[0];
test[testSymbol] = 20;
console.log(test.score);    // 20
console.log(test.dec());    // 19
```

비록 완벽한 private member가 되진 않지만, 위와 같은 몇 가지 접근을 제외하고는 다른 모든 접근으로부터는 보호되므로
절대적인 보호가 필요한 경우가 아닌 한 적절하게 활용하기 좋은 방법이라 하겠다.


## 4. `WeakMap` 활용

weakMap의 key에는 오직 참조형 데이터만을 지정할 수 있으며, 이 키값을 정확히 알고 있을 때에만
해당 프로퍼티의 값을 받아올 수 있다는 점을 이용한 방법이다.

```js
const Count = (() => {
  const count = {COUNT: 'COUNT'};
  class Count {
    constructor() {
      this.map = new WeakMap([[count, 0]]);
    }
    inc() {
      return this.map.set(count, this.map.get(count) + 1);
    }
    dec() {
      return this.map.set(count, this.map.get(count) - 1);
    }
    get score() { return this.map.get(count); }
    set score(n) { this.map.set(count, n); }
  }
  return Count;
})();
const test = new Count();
console.log(test.inc());    // WeakMap {Object {COUNT: "COUNT"} => 1}
console.log(test.inc());    // WeakMap {Object {COUNT: "COUNT"} => 2}
console.log(test.dec());    // WeakMap {Object {COUNT: "COUNT"} => 1}
console.log(test.score);    // 1
test.score = 10;
console.log(test.score);    // 10
console.log(test.dec());    // WeakMap {Object {COUNT: "COUNT"} => 9}
```

WeakMap 활용법은 private member를 구현하는 가장 완벽한 방법이지만,
오직 WeakMap용 method만을 이용할 수 있다는 단점이 있다.


## 결론

ES6 Class 내부에서 private 변수를 할당할 명시적인 방법이 없어, 이를 우회적으로 구현하기 위한 다양한 방법을 살펴보았다.
무엇 하나 '이거다' 싶은 방법은 없지만, Symbol, WeakMap을 이용한 방법은 아쉬운 대로 써먹어볼 만 할 것 같다.

참고 : [Exploring ES6](http://exploringjs.com/es6/ch_classes.html#sec_private-data-for-classes)
