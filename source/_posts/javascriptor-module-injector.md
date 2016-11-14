---
title: JavaScript Module Injector 만들기
subtitle: 실용성은 없을것이다...개념이해 정도로..
date: 2016-11-04 17:00:00
tags: [javascript, di, module]
author: javarouka
email: javarouka@gmail.com
cover: "/img/about-bg.jpg"
---

> 과거 [개인 블로그 글](http://blog.javarouka.me/2014/09/javascript-module-injector.html)을 가져온 포스트입니다

## JavaScript에서 DI를...

최근 [AngularJS](https://www.angularjs.org/) 에 관심이 많아서 여러모로 살펴보는 중인데, 그 중에서도 재미있게 본 것은 [Dependecy Injection](http://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EC%84%B1_%EC%A3%BC%EC%9E%85) 을 JavaScript 레벨에서 지원해준다는 것이었다.

Java 등에서 쓰이는 Spring Framework에서는 ApplicationContext 에 빈을 등록해두면 특정 애노테이션을 확인하여 DI 해주는 방식으로 진행되지만 JavaScript에서는 Annotation 같은 것이 없고 (비슷하게 구현해볼 수는 있지만 낭비...) 다른 방법으로 구현해야 한다.

비결은 [Function.prototype.toString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/toString) 에 있었다.

## Function.toString

JavaScript의 함수는 toString을 할 경우 함수의 소스코드를 문자열로 반환한다.

```javascript
function imFunction(you, say, ho) {
    console.log(you, say, ho);
}

document.getElementById('result').innerHTML = imFunction.toString();
```

[jsFiddle](http://jsfiddle.net/javarouka/5wk4sofh/)

여기서 중요한 것은 함수의 인자 목록도 문자열에 포함되어 있다는 것이다.

이걸 활용하면, DI를 흉내내볼 수 있다.

## 구현시작...

먼저 정규식이 필요하다

함수의 toString 결과를 함수의 이름, 인자, 몸체.이 셋으로 나눠볼 정규식을 만들어보자.

(함수 몸체와 이름은 일단 쓸일이 없지만 후 확장을 위해 한번에 구해봤다..)

```javascript
var FN_PARSE = /^function\s*(\S+)[^\(]*\(\s*([^\)]*)\)\s*\{([\W\w]+)\}$/m
```

위 정규식으로 match 할 경우 [toString 결과, 함수 이름, 함수 인자, 함수 몸체] 의 배열을 얻을 수 있다.

주의할 점이, 자바스크립트는 함수 인자 목록에도 주석을 사용할 수 있기에 자칫 주석으로 인자 이름을 잘못 가져올 수 있다.

주석을 제거하는 정규표현식도 준비한다.

```javascript
var STRIP_COMMENT = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg
```

그렇다면 적당한 함수를 하나 준비해본다

```javascript
function hello(man, to, women) {
    console.log(man + to + women);
}
```

파싱해보자.

```javascript
var FN_PARSE = /^function\s*(\S+)[^\(]*\(\s*([^\)]*)\)\s*\{([\W\w]+)\}$/m,
    STRIP_COMMENT = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;

function hello(man, to, women) {
    console.log(man + to + women);
}

var parsed = hello.toString().match(FN_PARSE),
    fnName = parsed[1],
    fnArgs = parsed[2].replace(STRIP_COMMENT, '').split(',')
    fnBody = parsed[3];
```

잘 된다!

[jsFiddle](http://jsfiddle.net/javarouka/ca1g4jf3/)

## 모듈 레지스트리 및 인젝터 구현

그럼 남은일은 모듈을 등록할 레지스트리를 구현하는 일이다.

여기서는 간단히 이름 기반의 DI만 지원하는 것으로 하고, 키-값 객체로 관리하게 해보자.


일단 AMD 모듈이 아닌 일반적인 모듈로 구현해봤다.

```javascript
(function(ctx) {

    var FN_PARSE = /^function\s*(\S+)[^\(]*\(\s*([^\)]*)\)\s*\{([\W\w]+)\}$/m,
        STRIP_COMMENT = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg,
        M = {};

    ctx.Injector = {

        // 의존성 모듈을 새로 등록한다.
        register: function(name, mo) {
            M[name] = mo;
        },

        // 함수를 받아 의존성을 주입한 뒤 즉시 실행한다.
        execute: function(fn, ctx) {
            return this.di(fn, ctx || this)();
        },

        // 함수를 받아 의존성을 주입한다.
        di: function(fn, ctx){

            // 함수를 정규식으로 분해한다.
            var parsed = fn.toString().match(FN_PARSE),
                fnName = parsed[1],
                args = parsed[2].replace(STRIP_COMMENT, '').split(','),
                body = parsed[3],
                i = 0, j,
                injected = [];

            // 인자의 이름으로 레지스트리에서 찾아 순서대로 적재
            for(j = args.length; i < j; i++) {
                injected[i] = M[args[i].trim()] || undefined;
            }

            // 래핑 함수를 반환한다.
            return function() {
                return fn.apply(ctx || null, injected);
            }
        }
    };
})(this);
```

di 함수에서 대해 조금 설명하면, 함수를 먼저 분석기로 쪼개서 배열을 얻은 뒤, 인자 배열을 돌면서 등록된 모듈과 매치하는 배열을 생성한 뒤 wrap 하여 반환하는 방식이다.

어디 잘 돌아가나 테스트.

```javascript
var Coffee = {
    pour: function(some) {
        return "커피를 " + some + "에 따르고 ";
    }
}

var Milk = {
    pour: function(some) {
        return "우유를 " + some + "에 따르고 "
    }
}

Injector.register('coffee', Coffee);
Injector.register('milk', Milk);

function Cup(coffee, milk) {
    var me = "머그컵";
    return coffee.pour(me) + milk.pour(me) + " 섞어 마신다";
}

var drink = Injector.di(Cup);

var coffeeMilk = drink(),
    directDrink = Injector.execute(Cup);

console.log(coffeeMilk); // 커피를 머그컵에 따르고 우유를 머그컵에 따르고 섞어 마신다
console.log(coffeeMilk == directDrink); // true
```

맛있는 커피우유가 만들어진 것 같다.

[jsFiddle](http://jsfiddle.net/javarouka/dc28fxfg/)

## 생각해볼 것들.

현재 구현이 포스팅하며 날림한거라 미비하거나 주의할 점이 몇가지 있다
- uglify 등 minify 할 경우 인자 이름이 보존이 안된다. mangle 옵션 등으로 인자이름을 보전해야 올바른 동작이 가능하다.
- 래핑 함수를 반환하는 관계로 스코프가 꼬일 수 있다.
