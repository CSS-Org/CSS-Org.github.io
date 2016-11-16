---
title: 동시성 환경에서 SimpleDateFormat 사용시 주의가 필요하다.
date: 2016-11-16 11:22:37
tags: [ java, concurrency ]
author: seongminwoo
---

## 개요
웹은 기본적으로 동시성을 기반으로 하지만 의외로 우리 개발자들은 동시성에 취약한 코드를 작성하곤 한다.
책에도 종종 소개되고 본인도 경험해본, 심지어 최근 회사 레거시 프로젝트 코드에서도 발견한 SimpleDateFormat 이슈를 간단히 소개해 본다.

## 다음 코드는 thread-safe 할까?
출처 : '7가지 동시성 모델'
```java
class DateParser {
  private final DateFormat format = new SimpleDateFormat("yyyy-MM-dd");

  public Date parse(String s) throws ParseException {
    return format.parse(s);
  }
}
```

## 숨겨진 가변 상태의 위험성
언뜻 보기엔 유일한 멤버변수인 format이 final로 선언되어 있어 변경 불가능한 클래스로 보인다.
하지만 SimpleDateFormat 내부에서는 가변 상태를 가지고 있기 때문에 결국 DateParser 클래스는 thread-safe 하지 않다.
이 때 NumberFormatException 같은 명시적인 오류가 발생하면 차라리 버그를 인지하기 쉽다. 하지만 올바르지 못한 값을 반환하면서 오류가 발생하지 않는 경우 버그를 인지하는데 상당한 시간이 걸릴 수 있다.

## 그럼 어떻게 해야하나?
사실 SimpleDateFormat 객체를 필요할 때 마다 새로 생성해서 쓰면 쉽게 해결이 가능하다.
하지만 유틸성 작업에 매번 객체 생성이라니 뭔가 나이스하지 않다.
그렇다. 세상엔 이미 잘 만들어진 바퀴들이 존재한다. thread-safe하면서 빠르기까지한 아래 클래스들을 대신 사용하자.
 - Apache Commons 라이브러리 내 FastDateFormat 클래스.
 - Joda Time 라이브러리 내 DateTimeFormatter 클래스(java8에서는 기본 내장).

## 결론
불변성에 대한 언어적 지원(제약)이 약한 자바 언어로 웹 프로그래밍을 할 때는 이러한 동시성 이슈를 항시 고민하는 습관이 필요하다. 특히 위 경우처럼 외부 라이브러리를 가져다 멤버 변수로 사용할 때는 thread-safe 한지 반드시 확인후 사용해야 한다.
