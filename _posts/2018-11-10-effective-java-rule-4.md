---
layout: article
title: Effective Java - 4. 객체 생성을 막을 때는 private 생성자를 사용
date: 2018-11-10 03:00:00 # 작성 시간 2018-11-08 00:00:00
category: java
tags: java effective-java
key: 20181108d
---

# Rule 4. 객체 생성을 막을 때는 private 생성자를 사용

<!--more-->

> 본 글은 [이펙티브 자바 2nd](https://book.naver.com/bookdb/book_detail.nhn?bid=8064518)를
읽고 개인적으로 학습한 내용을 복습하기 위해 작성된 글로 내용상 오류가 있을 수 있습니다.
오류가 있다면 지적 부탁드리겠습니다.

유틸 클래스(정적 메서드, 필드만 모은 클래스)는 객체를 만들 목적의 클래스가 아니기 때문에
생성자가 필요하지 않다. 하지만 생성자를 생략하면 컴파일러가 기본 생성자를 만들어버리기
때문에 `private` 생성자를 추가해야한다.

```java
public class UtilityClass {

  // 기본 생성자가 자동 생성되지 못하도록 객체 생성 방지
  private UtilityClass() {
      throw new AssertionError();
  }

  //...

}
```

- 생성자가 `private`이므로 클래스 외부에서는 사용할 수 없다.
- `AssertionError`는 반드시 필요한 것은 아니지만, 클래스 안에서 실수로 생성자를 호출하면
알 수 있다.
- 모든 생성자는 상위 클래스의 생성자를 명시적이든 묵시적이든 호출할 수 있어야 하는데 호출
가능한 생성자가 상위 클래스에 없기 때문에 하위 클래스를 만들 수 없다.
