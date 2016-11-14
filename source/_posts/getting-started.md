---
title: 블로그 글 쓰기
tags: [ document, start, writing, manual ]
subtitle: 시작하기 전에 알이둘 지식들
author: javarouka
---

## Installation

```sh
$ git clone https://github.com/CSS-Org/CSS-Org.github.io.git
$ cd CSS-Org.github.io
$ git checkout -b working origin/working
$ npm i
$ npm run server
```

## Deploy

```sh
$ npm run deploy
```

## Documentation
https://hexo.io/ko/

## Configuration

{root}/_config.yml 파일을 열어 다음 프로퍼티를 수정한다.

```sh
# @file {root}/_config.yml

theme: [{root}/themes/테마디렉토리 에서 테마디렉토리명]
```

> yml 파일이란?
[위키백과:YAML](https://ko.wikipedia.org/wiki/YAML). 일단은 '사람이 쉽게 읽을 수 있는' 데이터 직렬화 양식 이라고 한다...
JSON 과 유사하지만 bracket 대신 줄바꿈과 들여쓰기로 표현한다고 보면 얼추 정확하다.

가령 theme/clean-blog 테마를 적용한다면

```sh
# @file {root}/_config.yml

theme: clean-blog
```

가 된다.

현재 다운로드 되어 있는 테마는 다음과 같다.

clean-blog 의 경우 약간 커스터마이징을 해서 clean-blog-custom 으로 리네임해둔 상태다.

- [clean-blog](https://github.com/klugjo/hexo-theme-clean-blog) [미리보기](http://www.codeblocq.com/assets/projects/hexo-theme-clean-blog/)
- [hueman](https://github.com/ppoffice/hexo-theme-hueman) [미리보기](https://ppoffice.github.io/hexo-theme-hueman/)

더 많은 테마는 다음 링크에 있다.
https://hexo.io/themes/

## Writing

```sh
$ hexo new [제목]
```

실행 후에는 {root}/source/_post/ 아래에 글제목.md 파일이 생성된다.

문법은 [마크다운](http://daringfireball.net/projects/markdown/) 문법이고, wiki 등에서 쓰는 문법과 비슷하다.
