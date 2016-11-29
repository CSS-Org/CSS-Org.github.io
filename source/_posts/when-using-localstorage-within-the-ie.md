---
layout: post
title: "ie에서 localstorage 사용할 때 유의점"
subtitle: localstorage 삽질 이야기
description: "ie에서 localstorage 사용기"
author: cacacoo(권형신)
date: 2016-11-29
tags: [ ie, localstorage ]
comments: true
share: true
image: 'images/learncomputer-programing.jpg'
thumbnail: 'images/learncomputer-programing.jpg'
categories: ['Tech']
---


### 갑자기 왜

로컬 스토리지를 사용해서, 페이지의 Form 값을 임시로 저장하는 기능을 개발했다.

Chrome에서 만들어 테스트하고 배포를 하려던 참에,

혹시나해서 IE(10,Edge 모드)에서 테스트하려고 보니까 왠일인지 동작하지 않았다.


### 두 창에서 Local Storage key값의 동기화 문제

Form은 팝업이였고, Form 값이 변경되면 로컬스토리지에 저장된 후

다른 창에 떠있는 개인 페이지에서 임시저장된 Form 팝업을 링크하거나 삭제하는 식의 기능이였다.


문제는 서로 다른 두 창에 대해서, 로컬스토리지의 key값이 동기화 되지 않는데서 시작했다.

첫 번째 창에서 Storage에 저장하면, 두번째 창에서는 Storage를 읽고

변경된 사항에 대해서 표현되어야 했으나 도무지 갱신된 key값을 다른 창에서 찾을 수가 없었다.


예를 들어서, 두 창을 열어둔 후에, 첫번째 창에서 다음과 같이 localStorage에 Item을 추가해보자.

```js
var ls = window.localStorage;

ls.setItem('party', 'yeah!');
```

헌데 두번째 창에서 localStorage에 저장된 key값을 확인해보면, 첫번째 창과 key값이 동일하지 않다.

```js
var ls2 = window.localStorage;

Object.keys(ls2);
```

### 어떻게 해야할까

사용자 PC마다 localStorage를 얼마나 사용하고 있는지 알 수 없었기에,

키값으로 미리 데이터를 필터링해서 사용하지 않으면

어떤 성능 상 이슈가 발생할 지 알 수 없는 상황이였다.


localStorage를 이것 저것 사용해보다가 한가지 대안이라고 하기에도 민망한 방법을 발견했는데

** 창이 다르더라도 getItem(key)를 통해서 바로 호출하면, 데이터를 가져올 수 있었다 **

ls2 내부를 아무리 찾아봐도 없던 'party' 데이터가 key값으로 바로 호출하면 보인다.

```js
ls2.getItem('party');
```

이렇게해서, IE인 경우에, 서로 다른 창에서 localStorage를 사용할 경우, 동기화된 Key값을 얻기 위해

```js
	if(browserDetector.detectWindow(window).browser === 'ie') {
		keys = function () {
			let idx = 0;
			let result = [];
			while (l.key(idx)) {
				result.push(l.key(idx));
				idx++;
			}
			return result;
		};
		return keys();
	}
```
localStorage에 존재하지 않는 index를 요청할 때, null을 리턴하는 것을 이용해서

다음과 같이 key값을 빼서 사용하게 되었다.

### 결론

개발 환경과 사용자 환경에 대한 테스트의 중요성을 다시 한번 느낀다.

잘되겠지 싶은 것도 반드시 사용자 환경에서 다시 한번 테스트 해봐야 겠다.

