---
title: JS 숫자 변환 이모저모
date: 2016-10-28 11:25:41
tags: [javascript, parseInt, bitwise]
author: javarouka
email: javarouka@gmail.com
cover: "/img/about-bg.jpg"
---

## js 의 숫자 변환

자바스크립트 숫자 변환은 [parseInt](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/parseInt), [parseFloat](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/parseFloat), [Number](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Number) 등이 있다.

`parseInt` 와 `parseFloat` 는 이름 그대로의 기능을 한다

```javascript
var num = "32.4" 
parseInt(num); // 32
parseFloat(num); // 32.4
```

`Number` 도 일견 `parseFloat` 와 비슷하다. 히지만 세부 동작은 조금씩 다르다. 자세한건 아래에서 계속 이야기해보자.

```javascript
var num = "32.4" 
Number(num); // 32
Number(num); // 32.4
```

### parseInt

`parseInt` 는 첫번째 인자가 숫자가 아니면 [ECMAScript 표준의 ToString 연산](http://www.ecma-international.org/ecma-262/6.0/#sec-tostring) 을 수행한다. (영문 및 장문 주의)
그 다음에 ToString 결과로 반환된 값으로 숫자 변환을 시작한다.

여기서 `parseInt` 와 `parseFloat` 가 약간 다른데, `parseInt` 는 추가적인 두번째 인자로 `radix` 를 지정해줄 수 있다.

다음 예제를 보자

```javascript
var num = "1010";
parseInt(num, 2); // 10

var num = "FF";
parseInt(num, 16); // 255
```

명세에서는 radix 를 주지 않을 경우 10진수 해석을 명시하고 있지만, 구형 브라우저나 (가령 IE7 이하) 구현에 따라 바뀔 수 있으므로, parseInt 사용시에는 두번째 인자에 반드시 radix 를 주는게 좋다.

- [radix가 없는 8진 해석](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/parseInt#radix가_없는_8진_해석)

또한 인자 첫번째의 공백이나 첫번째가 아닌 자릿수에 문자열이 섞일 경우 무시하며 `0x` 나 `0X` 로 시작하는 문자열은 radix 를 16으로 가정한다.

```javascript
parseInt("1A", 10); // 1
parseInt("  3 2", 10); // 3
parseInt("0xFF"); // 255
```

### parseFloat

문자열 인수를 인자로 받아 부동 소수점 수를 반환한다. 인자가 문자열이 아니면 [ToString](http://www.ecma-international.org/ecma-262/6.0/#sec-tostring)) 연산을 수행한다.

`parseFloat` 는 주어진 인자를 `parseInt` 와는 다르게 변환 대상이 될 인자 하나만 받는다. 이 함수 역시 parseInt 와 같게 ToString 을 수행 후 변환을 시도한다. 변환 실패 시 [NaN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NaN) 을 반환하는 것도 같다.

```javascript
parseFloat("1.23") // 1.23;
```

[Infinity](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Infinity) 를 해석할 수 없어 [NaN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/NaN) 을 반환하는 `parseInt` 와 다르게 `parseFloat` 는 `NaN` 을 반환하지 않는다

```javascript
parseInt(Infinity) // NaN
parseFloat(Infinity) // Infinity
```

### Number

`Number` 는 통상 `parseFloat` 와 비슷하다고 생각할 수 있다.

하지만 세부적으로 약간 다른데, [ToNumber](http://www.ecma-international.org/ecma-262/6.0/#sec-tonumber) 라는 작업으로 진행된다. (영어 및 표준문서 압박 주의)

정수형 변환일 불가능한 문자열이 있을 경우 `NaN` 이 반환된다

```javascript
parseFloat("1a") // 1 
Number("1a") // NaN 
```

16진수 형식의 문자열일 경우 `parseFloat` 와는 다르게 해석이 가능하다

```javascript
parseFloat("0x10") // 0 
Number("0x10") // 16
```

어떤 코드를 보다보면 값에 + 연산자를 붙여 처리하는 표현식을 본 일이 있을것이다. (없다고??)

이 표현은 `Number(val)` 과 거의 동일하다고 보면 정확하다

```javascript
+"2" === Number("2") // true
+[] === Number([]) // true
+new Date === Number(new Date()) // true
```

### bitwise

비트연산이 왜 나오는지 뜬금없어 할 수도 있다.

하지만 비트 연산으로도 숫자 변환이 가능하다. 그리고 명세상 성능이 타 변환보다 빠를 수 있다. (물론 구현에 따라 다를 수 있다.)

```javascript
// 777.77 을 정수형으로 변환 시도
// 결과는 전부 10진 정수 [777]

parseInt(777.77, 10)
Math.floor(777.77)
~~777.77
777.77 | 0
777.77 >> 0
777.77 >>> 0
777.77 << 0

```

비트연산 시 명세에서는 피연산자를 [ToUint32](http://www.ecma-international.org/ecma-262/6.0/#sec-touint32) 작업을 선행하여 강제 변환하게 되어 있기에 이런게 가능하다.

이런 문법은 일종의 언어 스펙을 이용한 트릭이라고 할 수 있다. 이것에 대해서 잘 알지 못하면 알수없는 암호문이 되어 버리니 팀원과의 소통은 필수다.

비트연산을 썼을때의 장점은

- 일반적으로 타 방법 (`parseXXX`, `Number`) 보다 빠르다
- 뭔가 뛰어난 프로그래머가 된 느낌을 받을 수 있다.

주의할 점은 Uint32 형이기에 원래의 값에 변형이 올 수 있으니 주의해야 한다. (음수라거나 int 형을 초과하는 경우)

##### 참고: 각 연산법에 대한 성능비교

[Number() vs parseInt() vs plus vs bitwise](https://jsperf.com/number-vs-parseint-vs-plus/3)