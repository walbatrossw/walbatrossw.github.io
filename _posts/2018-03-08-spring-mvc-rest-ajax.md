---
layout: article
title: Spring Framework - RestController, ResponseEntity, AJAX
category: spring-mvc
tags: spring-mvc rest ajax ResponseEntity
key: 20180308a
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
5. [예외처리](https://walbatrossw.github.io/spring/mvc/2018/02/28/05-spring-mvc-board-exception.html)
6. [페이징처리 : Persistence, Business 계층](https://walbatrossw.github.io/spring/mvc/2018/03/01/06-spring-mvc-board-paging-persistence-business.html)
7. [페이징처리 : Control, Presentation 계층](https://walbatrossw.github.io/spring/mvc/2018/03/02/07-spring-mvc-board-paging-control-presentation.html)
8. [페이징처리 개선, 목록페이지 정보 유지](https://walbatrossw.github.io/spring/mvc/2018/03/02/08-spring-mvc-board-paging-enhancement-tip.html)
9. [프로젝트 구조 변경 및 수정사항](https://walbatrossw.github.io/spring/mvc/2018/03/04/09-spring-mvc-board-project-structure-modification.html)
10. [검색처리, 동적 SQL](https://walbatrossw.github.io/spring/mvc/2018/03/07/10-spring-mvc-board-search-dynamic-sql.html)

---

## 1. REST(Representational State Transfer)란?

### 1.1 기본 개념
**하나의 URI는 하나의 고유한 리소스를 대표하도록 설계된다는 개념** 이다. REST방식은 특정한 URI는 반드시 그에 상응하는 데이터 자체라는 것을 의미하는 방식이다. 예를 들어 `/article/120`은 120번 게시물이라는 고유한 의미를 가지도록 설계하고, 이에 대한 처리는 GET, POST 방식과 같이 추가적인 정보를 통해 결정하게된다.

### 1.2 REST API
외부에서 위와 같은 방식으로 특정 URI를 통해 사용자가 원하는 정보를 제공하는 방식이다. 최근 OPEN API에서 많이 사용되면서 REST방식으로 제공되는 외부연결 URI를 REST API라고 하고, REST 방식의 서비스 제공이 가능것을 "Restful"하다고 표현한다.

### 1.3 스프링에서의 REST
스프링의 경우 3버전부터 `@ResponseBody`애너테이션을 지원하게되면서 REST방식의 처리를 지원하고 있으며, 스프링4부터는 `@RestController`를 통해 구현이 가능하게 되었다.

## 2. @RestController 애너테이션

### 2.1 @RestController
스프링4부터 사용이 가능한 `@RestController`애너테이션은 기존의 특정한 JSP와 같은 뷰를 만들어 내는 것이 목적이 아니라 REST방식의 데이터를 처리하기 위해 사용하는 애너테이션이다. 이 때 반환되는 데이터로 사용되는 것은 단순문자열, JSON, XML이다.

### 2.2 @Controller, @ResponseBody
스프링3에서는 컨트롤러는 `@Controller`애너테이션만을 사용해서 처리했기 때문에 화면을 처리를 담당하는 JSP가 아닌 데이터자체를 서비스하려면 해당 메서드나 리턴타입에 `@ResponseBody`애너테이션을 추가하는 형태로 작성하였다.

## 3. RestController 예제

`@RestController`를 이용해서 예제 연습을 해보자.

### 3.1 RestController 작성법
기존에는 `@Controller`애너테이션을 붙여줬지만 `RestController`를 이용하기 위해서는 `@RestController`를 붙여주면 된다.
```java
@RestController
@RequestMapping("/ajax/test")
public class AjaxController {

}
```

### 3.2 단순 문자열을 리턴할 경우 메서드 작성법

```java
@RequestMapping("/hello")
public String sayHello() {

    return "Hello World";

}
```

해당 URI를 요청하면 브라우저에 아래와 같이 출력이 되는 것을 확인 할 수 있다.

![return_string](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_rest_ajax/return_string.png?raw=true)

### 3.3 객체를 JSON으로 리턴할 경우 메서드 작성법

먼저 예제에 사용될 간단한 객체를 아래와 같이 작성해준다.

```java
public class SampleVO {

    private Integer sampleNo;
    private String firstName;
    private String lastName;

    // getter, setter, toString() 생략
}
```

위에서 작성한 클래스의 객체를 반환하는 메서드를 `RestController`에 추가해준다.

```java
@RequestMapping("/sendVO")
public SampleVO sendVO() {

    SampleVO sample = new SampleVO();
    sample.setSampleNo(1);
    sample.setFirstName("더블");
    sample.setLastName("에스");

    return sample;
}
```

해당 URI를 요청하면 브라우저에 아래와 같이 출력이 되는 것을 확인 할 수 있다.
![return_json](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_rest_ajax/return_json.png?raw=true)

만약 406에러가 발생한다면 `pom.xml`에 `jackson-databind`라이브러리가 추가한다.

```xml
<!--JSON : jackson-databind 라이브러리-->
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.8.4</version>
</dependency>
```

### 3.4 컬렉션 타입(`List`)의 객체를 리턴할 경우 메서드 작성법

```java
@RequestMapping("/sendList")
public List<SampleVO> sendList() {

    List<SampleVO> samples = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        SampleVO sample = new SampleVO();
        sample.setSampleNo(i);
        sample.setFirstName("더블");
        sample.setLastName("에스" + i);
        samples.add(sample);
    }

    return samples;
}
```

해당 URI를 요청하면 브라우저에 아래와 같이 출력이 되는 것을 확인 할 수 있다. JSON의 문법상 리스트는 배열로 표현되기 때문에 아래와 같이 나오게 된다.

![return_list](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_rest_ajax/return_list.png?raw=true)

### 3.5 컬렉션 타입(`Map`)의 객체를 리턴할 경우 메서드 작성법

```java
@RequestMapping("/sendMap")
public Map<Integer, SampleVO> sendMap() {

    Map<Integer, SampleVO> sampleMap = new HashMap<>();
    for (int i = 0; i < 10; i++) {
        SampleVO sample = new SampleVO();
        sample.setFirstName("더블");
        sample.setLastName("에스"+i);
        sample.setSampleNo(i);
        sampleMap.put(i, sample);
    }

    return sampleMap;
}
```
해당 URI를 요청하면 브라우저에 아래와 같이 출력이 되는 것을 확인 할 수 있다. Map의 경우 자바스크립트의 JSON형태로 보여지게 되는데 Key와 Value로 구성되서 `{}`로 표현된다.

![return_map](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_rest_ajax/return_map.png?raw=true)

## 4. ResponseEntity 타입

### 4.1 ResponseEntity 타입이란?

[HTTP 상태코드 정리 포스팅 참고](http://doublesprogramming.tistory.com/203)

`@RestController`는 별도의 뷰를 제공하지 않는 형태로 서비스를 실행하기 때문에 리턴 데이터가 예외적인 상황에서 문제가 발생할 수가 있다. 웹의 경우 HTTP 상태코드가 이러한 정보를 나타내는데 사용된다.

**스프링에서 제공하는 `ResponseEntity`타입은 개발자가 직접 결과 데이터와 HTTP상태코드를 직접 제어할 수 있는 클래스다.** `ResponseEntity`를 사용하면 404나 500같은 에러를 전송하고 싶은 데이터와 함께 전송할 수 있기 때문에 좀더 세밀한 제어가 가능해진다.

### 4.2 ResponseEntity 예제1 : HTTP상태코드만 보내기

아래와 같이 예제를 작성하고 해당 URI를 요청해보자.

```java
@RequestMapping("/sendErrorAuth")
public ResponseEntity<Void> sendListAuth() {
    return new ResponseEntity<>(HttpStatus.BAD_REQUEST);
}
```

해당 URI를 요청하면 리턴타입으로 선언된 `ResponseEntity`는 결과 데이터로 Http상태코드 중에서 400에러를 헤더메시지로 보내게 되는데 크롬 개발자도구의 네트워크 탭을 확인해보면 아래와 같이 400에러가 발생된 것을 확인해볼 수 있다.

![return_400](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_rest_ajax/return_400.png?raw=true)

### 4.3 `ResponseEntity` 예제2 : 결과데이터와 HTTP상태코드 같이 보내기

아래와 같이 예제를 작성하고 해당 URI를 요청해보자.

```java
@RequestMapping("/sendErrorNot")
public ResponseEntity<List<SampleVO>> sendListNot() {

    List<SampleVO> samples = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        SampleVO sample = new SampleVO();
        sample.setSampleNo(i);
        sample.setFirstName("더블");
        sample.setLastName("에스" + i);
        samples.add(sample);
    }

    return new ResponseEntity<>(samples, HttpStatus.NOT_FOUND);
}
```

해당 URI를 요청하면 리턴타입에 list 데이터와 HTTP상태코드(404)를 전송하게 된다. 앞서 본 예제와 달리 화면에는 전송한 결과를 보여주면서, 상태코드도 함께 전달되는 것을 확인해볼 수 있다.

![return_404](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_rest_ajax/return_404.png?raw=true)

## 5. AJAX(Asynchronous Javascript and XML)

### 5.1 AJAX란?

웹을 통해 작업할 때 REST방식이 가장 많이 쓰이는 형태는 AJAX와 같이 결합된 형태이다. 주로 브라우저에서 대화형으로 서버와 데이터를 주고받는 형태의 메시지 전송방식을 의미한다. **AJAX는 화면의 전환이나 깜빡임없이 서버에서 데이터를 받는 방법** 이라고 생각하면 된다. 페이스북이나 네이버와 같은 포털사이트의 검색창 자동완성기능들이 AJAX를 활용한 대표적인 서비스라고 할 수 있다.

![ajax](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_rest_ajax/ajax.png?raw=true)

사진출처(http://adaptivepath.org/ideas/ajax-new-approach-web-applications/)

**비동기화는 결과의 데이터를 기다리는 방식이 아닌 결과를 통보받는 형식** 이라고 할 수 있다. 대부분의 프로그래밍은 특정로직을 호출하고, 결과를 받아서 다음 로직을 실행하는 방식이라면 비동기화된 방식은 로직을 호출하고 결과를 기다리지 않는다. 대신 결과가 통보될 때 실행할 로직을 지정하는 방식이다.

- 동기화방식은 순차적인 일을 실행하는데 적합
- 비동기방식은 일을 처리한 결과를 기다리지 않고 흐름이 지속됨
- 비동기 방식의 특징은 처리된 일의 결과를 통보받은 형태로 처리

### 5.2 AJAX의 장점
- 페이지의 이동없이 고속으로 화면을 전환할 수 있다.
- 서버처리를 기다리지 않고, 비동기 요청이 가능하다.
- 수신하는 데이터의 양을 줄일 수 있고, 클라이언트에게 처리를 위임할 수도 있다.

### 5.3 AJAX의 단점
- AJAX를 쓸 수 없는 브라우저에 대한 문제가 있다.
- HTTP 클라이언트의 기능이 한정되 있다.
- 페이지 이동없는 통신으로 인한 보안상의 문제가 발생할 수 있다.
- 지원하는 Charset이 한정되어 있다.
- 스크립트로 작성되므로 디버깅이 용이하지 않다.
- 요청을 남발하면 역으로 서버 부하가 늘 수 있음.
- 동일-출처 정책으로 인해 다른 도메인과는 통신이 불가능하다.

[AJAX 장단점 출처](https://ko.wikipedia.org/wiki/Ajax)

---
