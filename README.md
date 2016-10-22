# CSS Org

## Installation

```
$ git clone https://github.com/CSS-Org/CSS-Org.github.io.git
$ cd CSS-Org.github.io
$ npm i
$ hexo server
```

## Documentation
https://hexo.io/ko/

## Cinfiguration

{root}/_config.yml 파일을 열어 다음 프로퍼티를 수정한다.

> yml 파일이란?
[YAML](https://ko.wikipedia.org/wiki/YAML). 일단은 '사람이 쉽게 읽을 수 있는' 데이터 직렬화 양식 이라고 한다...

```sh
theme: [{root}/themes/테마디렉토리 에서 테마디렉토리명]
```

가령 theme/clean-blog 테마를 적용한다면

```sh
theme: clean-blog
```

가 된다.

현재 다운로드 되어 있는 테마는 다음과 같다.

clean-blog 의 경우 약간 커스터마이징을 해서 clean-blog-custom 으로 리네임해둔 상태다.

- [clean-blog](https://github.com/klugjo/hexo-theme-clean-blog) [미리보기](http://www.codeblocq.com/assets/projects/hexo-theme-clean-blog/)
- [hueman](https://github.com/ppoffice/hexo-theme-hueman) [미리보기](https://ppoffice.github.io/hexo-theme-hueman/)

더 많은 테마는 다음 링크에 있다.
https://hexo.io/themes/
