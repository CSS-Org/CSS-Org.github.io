---
title: java8 써본 후기
date: 2016-10-24 23:31:18
tags: [ java8, lambda, design pattern ]
subtitle: 여러 삽질기... static typing 에 무릎꿇다.
author: javarouka
---

## 시작은 참 쉬웠다.

[Spring Data JPA](http://projects.spring.io/spring-data-jpa/) 의 `Specification` 을 쓰다보면 자주 보는 다음과 같은 코드가 있다.

```java
public class ContactInquiry {

  public static Specification<ContactInquiry> isCompleted() {
    return new Specification<ContactInquiry> {
      public Predicate toPredicate(Root<T> root, CriteriaQuery query, CriteriaBuilder cb) {
        return cb.equal(root.get(ContactInquiry_.status), InquiryStatus.COMPLETE);
      }
    };
  }

}
```

이 장황함이란...

```java
public class ContactInquiry {

  // Oh!
  public static Specification<ContactInquiry> isCompleted() {
    return (root, query, cb) -> cb.equal(root.get(ContactInquiry_.status), InquiryStatus.COMPLETE);
  }

}
```

뭔가 해낸 듯 하다.

잘 살펴보면 위 코드와 아래 코드를 잘 살펴보면 컴파일러가 예측 가능한 타입추정 범위 내에서 코드를 줄였다는걸 알 수 있다.

`isCompleted` 메서드는 [Specification](http://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/domain/Specifications.html) 을 반환해야 하고 `Specification` 은 하나의 메서드를 가지는데, 그 메서드에서는 [Predicate](http://docs.oracle.com/javaee/7/api/javax/persistence/criteria/Predicate.html) 를 반환하면 된다.

이런 정보들은 컴파일러가 메서드 시그니처의 타입 분석 레벨에서 이미 알 수 있는 것들이라 생략할 수 있고, 결국 남는 것은 인자 3개와 각자의 이름, 그리고 구현 내용인 메서드의 바디만 남는다.

보통 Sugaring 이라고 하는 단축 문법이다.

아무튼 이러한 변화만으로 감격에 겨운 나머지 좀더 여러 부분에 적용해보려고 배고픈 하이에나에 빙의하여 프로젝트를 뒤지기 시작했다

## 다른 함수형 언어와 좀 다르다.

blah blah
