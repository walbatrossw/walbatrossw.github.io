---
layout: article
title: Spring-MVC 게시판 예제 05 - 예외처리
category: spring-mvc
tags: spring-mvc exception
key: 20180228b
cover: /assets/cover/spring.png
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(34, 139, 87 , .4), rgba(139, 34, 139, .4))'
    src: /assets/cover/spring-article.png
---

<!--more-->

> 본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위해 기록한 내용이기 때문에 오류가 있다면 지적 부탁드리겠습니다.
> 포스팅의 예제는 STS 또는 Eclipse를 사용하지 않고 IntelliJ를 통해 구현하고 있습니다. 그래서 기존의 STS에서 생성된 Spring 프로젝트의 스프링관련 설정 파일명과 프로젝트 구조가 약간 다를 수 있습니다. [IntelliJ를 통한 Spring MVC 프로젝트 생성](https://walbatrossw.github.io/spring/mvc/2017/11/22/intellij-springmvc-create.html) 포스팅을 참고해주시면 감사하겠습니다.
> 현재 프로젝트의 전체코드는 [Github 저장소](https://github.com/walbatrossw/spring-mvc-ex)에서 확인하실 수 있습니다.

---

Spring-MVC 기본 개념 및 테스트 예제 관련 포스팅 링크

1. [IntelliJ에서 Spring MVC Project 생성하기](https://walbatrossw.github.io/spring/mvc/2017/11/22/intellij-springmvc-create.html)
2. [Spring MVC - MySQL 연결테스트](https://walbatrossw.github.io/spring/mvc/2017/11/22/mysql-junit-test.html)
3. [Spring MVC - MyBatis 설정 및 테스트](https://walbatrossw.github.io/spring/mvc/2017/11/22/mybatis-connection-test.html)
4. [Spring MVC 구조](https://walbatrossw.github.io/spring/mvc/2017/11/25/spring-mvc-structure.html)
5. [Spring MVC - Controller 작성 연습, WAS없이 Controller 테스트 해보기](https://walbatrossw.github.io/spring/mvc/2017/11/25/spring-controller-test-without-was.html)
6. [SpringMVC + MyBatis, DAO구현 테스트](https://walbatrossw.github.io/spring/mvc/2017/11/22/spring-mvc-mybatis-dao-test.html)

Spring-MVC 게시판 예제 관련 포스팅 링크

1. [IntelliJ에서 Spring MVC Project 생성하기](https://walbatrossw.github.io/spring/mvc/2017/11/22/intellij-springmvc-create.html)
2. [Bootstrap 템플릿 세팅 (AdminLTE)](https://walbatrossw.github.io/spring/mvc/2017/11/28/02-spring-mvc-board-template.html)
3. [기본적인 CRUD 구현 : 영속계층, 비지니스계층](https://walbatrossw.github.io/spring/mvc/2018/02/26/03-spring-mvc-board-crud-persistence-business.html)
4. [기본적인 CRUD 구현 : Control, Presentation 계층](https://walbatrossw.github.io/spring/mvc/2018/02/28/04-spring-mvc-board-crud-controller-view.html)

---

# Spring MVC 게시판 예제 05 - 예외처리

## 1. 컨트롤러에서 예외 처리 방식의 종류

Spring MVC의 Controller에서 `Exception`을 처리하기 위해 다음과 같은 방식을 사용한다.

- `@ExceptionHandler`애너테이션을 이용한 처리
- `@ControllerAdvice`애너테이션을 이용한 처리
- `@ResponseStatus`애너테이션을 이용한 Http상태 코드 처리

3가지 방법 중에서 범용적으로 사용할 수 있는 방법은 `@ControllerAdvice`애너테이션을 이용한 방법인데 공통의 `Exception`처리 전용 객체를 사용하는 방식이다. 이 방식을 통해 게시글을 작성하면서 발생하는 예외를 처리하도록 해보자.

## 2. @ControllerAdvice 애너테이션을 이용한 방식 구현

### 2.1 예외를 처리할 클래스 작성

`/src/main/java/기본패키지/commons/exception`패키지를 생성하고, `CommonExceptionAdvice`클래스를 아래와 같이 작성해준다.

```java
@ControllerAdvice
public class CommonExceptionAdvice {

    private static final Logger logger = LoggerFactory.getLogger(CommonExceptionAdvice.class);

    @ExceptionHandler(Exception.class)
    public ModelAndView commonException(Exception e) {

        logger.info(e.toString());
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("exception", e);
        modelAndView.setViewName("/commons/common_error");

        return modelAndView;
    }

}
```

클래스의 선언부를 보면 `@ControllerAdvice`애너테이션을 붙여줬다. `@ControllerAdvice`애너테이션을 통해 이 클래스의 객체가 컨트롤러에서 발생하는 `Exception`을 전문적으로 처리하는 클래스라는 것을 명시해준 것이다. 그리고 `commonException()`메서드의 선언부에는 `@ExceptionHandler`라는 애너테이션을 붙여 적절한 타입의 `Exception`을 처리하도록 하였다.
일반 컨트롤러 클래스와 다르게 `Model`을 파라미터로 사용하는 것을 지원하지 않기 때문에 `ModelAndView`타입을 직접 사용하는 형태로 작성해야한다. `ModelAndView`는 하나의 객체에 `Model`데이터와 `View`의 처리를 동시에 처리할 수 있는 객체이다. 만약 예외가 발생하게 되면 예외가 발생한 내용이 담긴 데이터를 `exception`에 담고, `common_error.jsp`에 전달하게 된다.

### 2.2 예외를 출력할 페이지 작성

`/WEB-INF/views/commons`디렉토리를 생성하고, `common_error.jsp`파일에 아래와 같이 작성해준다.

```xml
<section class="content container-fluid">

    <h3><i class="fa fa-warning text-red"></i> ${exception.getMessage()}</h3>
    <ul>
        <c:forEach items="${exception.getStackTrace()}" var="stack">
            <li>${stack.toString()}</li>
        </c:forEach>
    </ul>

</section>
```

### 2.3 예외가 발생할 경우 출력된 화면의 모습
예외를 고의적으로 발생시켜보면 아래와 같이 `common_error.jsp`로 이동하여 예외가 발생한 내용을 출력하게 된다.

![common_error1](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/05_spring_mvc_board_exception/common_error1.png?raw=true)

![common_error2](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/05_spring_mvc_board_exception/common_error2.png?raw=true)

## 3. 정리
위와 같이 예외를 처리하고, 어떤 예외가 발생했는지 브라우저에 로그를 출력해줌으로써 개발할 때 보다 빠르게 예외나 에러가 발생한 원인을 빠르게 찾을 수 있다. 하지만 실서버에 배포하고, 서비스를 할 때에는 위와 같은 예외처리 출력은 바람직하지 못하다. 사용자가 웹을 사용하면서 이러한 예외를 마주치게 해서는 안되기 때문이다. 그래서 예외가 발생한 경우를 대비해 404, 500 에러 정도를 알려주는 적절한 페이지를 만들어 주는 것이 바람직하다.

---
