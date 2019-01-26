---
layout: article
title: Spring-MVC 게시판 예제 08 - 페이징처리 개선, 게시글 조회시 목록페이지 정보 유지
category: spring-mvc
tags: spring-mvc paging UriComponentsBuilder
key: 20180302b
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

---

## 1. 페이징 처리 개선

![enhancement_before](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/08_spring_mvc_board_paging_enhancement_tip/enhancement_before.png?raw=true)

- `<a herf="/article/listPaging?page=2"/>` : 페이지 번호만을 전달
- `<a herf="/article/listPaging?page=2&perPageNum=10"/>` : 페이지 번호와 페이지당 출력할 게시글의 갯수도 함께 전달

이전 포스팅까지 페이징 처리 구현을 완료했다. 하지만 아쉬운 점은 위의 사진과 같이 GET파라미터을 `page`만 처리했기 때문에 `perPageNum`이나 그 이상의 정보를 전달할 수 없다는 점이다. 그래서 이번 포스팅에서 페이징 처리를 개선하고, uri를 좀더 나은 방식으로 제어할 수 있는 방법들에 대해 정리해보자.

### 1.1 UriComponentsBuilder를 이용하는 방식

스프링 MVC에서 웹개발에 필요한 유틸리티 클래스들을 제공하고 있는데 그중에 URI를 작성할 때 도움을 주는 클래스는 `UriComponentsBuilder`와 `UriComponents`클래스가 있다. 그렇다면 `UriComponentsBuilder`를 사용하기 위한 간단한 테스트 코드를 작성해보자.

```java
@Test
public void testURI() throws Exception {

    UriComponents uriComponents = UriComponentsBuilder.newInstance()
            .path("/article/read")
            .queryParam("articleNo", 12)
            .queryParam("perPageNum", 20)
            .build();

    logger.info("/article/read?articleNo=12&perPageNum=20");
    logger.info(uriComponents.toString());

}
```

```
INFO : com.doubles.mvcboard.article.ArticleDAOTest - /article/read?articleNo=12&perPageNum=20
INFO : com.doubles.mvcboard.article.ArticleDAOTest - /article/read?articleNo=12&perPageNum=20
```

위와 같이 `UriComponents`클래스를 통해 `path`나 `query`에 해당하는 문자열을 추가해서 원하는 URI를 생성할 수가 있다. 위 테스트를 실행해보면 콘솔창에 동일한 URI가 생성되는 것을 확인할 수 있다.

```java
@Test
public void testURI2() throws Exception {

    UriComponents uriComponents = UriComponentsBuilder.newInstance()
            .path("/{module}/{page}")
            .queryParam("articleNo", 12)
            .queryParam("perPageNum", 20)
            .build()
            .expand("article", "read")
            .encode();

    logger.info("/article/read?articleNo=12&perPageNum=20");
    logger.info(uriComponents.toString());

}
```

```
INFO : com.doubles.mvcboard.article.ArticleDAOTest - /article/read?articleNo=12&perPageNum=20
INFO : com.doubles.mvcboard.article.ArticleDAOTest - /article/read?articleNo=12&perPageNum=20
```

그리고 만약 미리 필요한 경로를 설정해두고 `{module}`과 같은 경로를 `article`로 `{page}`를 `read`로 변경할 수도 있다. 테스트를 실행해보면 이것도 동일한 URI가 생성되는 것도 확인할 수가 있다.

이제 `PageMaker`클래스에 `makeQuery()`메서드를 추가하고, URI를 자동으로 생성하도록 처리해보자.

```java
public String makeQuery(int page) {
    UriComponents uriComponents = UriComponentsBuilder.newInstance()
            .queryParam("page", page)
            .queryParam("perPageNum", criteria.getPerPageNum())
            .build();

    return uriComponents.toUriString();
}
```

`list_paging.jsp`에서 게시글 제목, 페이지 번호, 다음, 이전의 `<a>`태그를 아래의 코드와 같이 수정한다.

```html
<tr>
    <td>${article.articleNo}</td>
    <%--<td><a href="${path}/article/read?articleNo=${article.articleNo}">${article.title}</a></td>--%>
    <td>
      <a href="${path}/article/read${pageMaker.makeQuery(pageMaker.criteria.page)}&articleNo=${article.articleNo}">
        ${article.title}
      </a>
    </td>
    <td>${article.writer}</td>
    <td><fmt:formatDate value="${article.regDate}" pattern="yyyy-MM-dd a HH:mm"/></td>
    <td><span class="badge bg-red">${article.viewCnt}</span></td>
</tr>
```

```html
<ul class="pagination">
    <c:if test="${pageMaker.prev}">
        <%--<li><a href="${path}/article/listPaging?page=${pageMaker.startPage - 1}">이전</a></li>--%>
        <li><a href="${path}/article/listPaging${pageMaker.makeQuery(pageMaker.startPage - 1)}">이전</a></li>
    </c:if>
    <c:forEach begin="${pageMaker.startPage}" end="${pageMaker.endPage}" var="idx">
        <li <c:out value="${pageMaker.criteria.page == idx ? 'class=active' : ''}"/>>
            <%-- <a href="${path}/article/listPaging?page=${idx}">${idx}</a> --%>
            <a href="${path}/article/listPaging${pageMaker.makeQuery(idx)}">${idx}</a>
        </li>
    </c:forEach>
    <c:if test="${pageMaker.next && pageMaker.endPage > 0}">
        <%--<li><a href="${path}/article/listPaging?page=${pageMaker.endPage + 1}">다음</a></li>--%>
        <li><a href="${path}/article/listPaging?${pageMaker.makeQuery(pageMaker.endPage + 1)}">다음</a></li>
    </c:if>
</ul>
```

이제 어떻게 바뀌었는지 크롬 개발자 도구를 통해 수정한 화면의 페이지 링크를 확인해보면 이전과 다르게 `page`와 `perPageNum`이 같이 연동되는 것을 볼 수 있다.

![enhancement_uri](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/08_spring_mvc_board_paging_enhancement_tip/enhancement_uri.png?raw=true)


### 1.2 javascript를 이용하는 방식

자바스크립트를 이용하는 방식은 페이지 번호를 클릭했을 때 이벤트를 제어하는 방식이다. 이벤트를 제어하기 위해서는 페이지 번호에 걸어둔 `<a>`태그의 `herf`속성을 단순히 페이지 번호만을 의미하도록 변경해줘야 한다.

```html
<ul class="pagination">
    <c:if test="${pageMaker.prev}">
        <%--<li><a href="${path}/article/listPaging?page=${pageMaker.startPage - 1}">이전</a></li>--%>
        <%--<li><a href="${path}/article/listPaging${pageMaker.makeQuery(pageMaker.startPage - 1)}">이전</a></li>--%>
        <li><a href="${pageMaker.startPage - 1}">이전</a></li>
    </c:if>
    <c:forEach begin="${pageMaker.startPage}" end="${pageMaker.endPage}" var="idx">
        <li <c:out value="${pageMaker.criteria.page == idx ? 'class=active' : ''}"/>>
            <%--<a href="${path}/article/listPaging?page=${idx}">${idx}</a>--%>
            <%--<a href="${path}/article/listPaging${pageMaker.makeQuery(idx)}">${idx}</a>--%>
            <a href="${idx}">${idx}</a>
        </li>
    </c:forEach>
    <c:if test="${pageMaker.next && pageMaker.endPage > 0}">
        <%--<li><a href="${path}/article/listPaging?page=${pageMaker.endPage + 1}">다음</a></li>--%>
        <%--<li><a href="${path}/article/listPaging?${pageMaker.makeQuery(pageMaker.endPage + 1)}">다음</a></li>--%>
        <li><a href="${pageMaker.endPage + 1}">다음</a></li>
    </c:if>
</ul>
```

위와 같이 코드를 변경해주고, `<form>`태그를 추가해 페이지 이동시 `page`와 `perPageNum`값을 넘겨주도록 한다.

```html
<form id="listPageForm">
    <input type="hidden" name="page" value="${pageMaker.criteria.page}">
    <input type="hidden" name="perPageNum" value="${pageMaker.criteria.perPageNum}">
</form>
```

이제 마지막으로 페이지 번호를 클릭하면 이벤트를 처리할 자바스크립트 코드를 작성해준다.

```js
$(".pagination li a").on("click", function (event) {
    event.preventDefault();

    var targetPage = $(this).attr("href");
    var listPageForm = $("#listPageForm");
    listPageForm.find("[name='page']").val(targetPage);
    listPageForm.attr("action", "/article/listPaging").attr("method", "get");
    listPageForm.submit();
});
```

`<a>`태그를 클릭하면 발생하는 이벤트는 `preventDefault()`를 통해 페이지 이동을 막는다. 그리고 이벤트가 발생한 `<a>`태그의 페이지 번호 값을 `<from>`태그의 `page`에 넣어 전송하게 된다.

실제로 크롬 개발자도구를 통해 확인해보면 `<a>`태그의 링크는 앞서 설명한 것처럼 페이지 번호만을 의미하도록 나오고 페이지 이동시 `page`와 `perPageNum`도 연동이 되는 것을 확인할 수 있다.

![enhancement_js](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/08_spring_mvc_board_paging_enhancement_tip/enhancement_js.png?raw=true)

## 2. 목록페이지 정보 유지

페이징 처리 구현이 완료가 되었다. 이제 특정 목록페이지에서 특정 게시글을 조회, 수정, 삭제한 뒤 다시 목록을 이동할 때 특정 목록페이지로 다시 이동하도록 구현해보자.

```
3번의 목록페이지 -> 80번 게시물 조회 -> 목록 페이지 이동 버튼 클릭 -> 3번 목록 페이지 이동
3번의 목록페이지 -> 80번 게시물 조회 -> 80번 게시글 수정 -> 게시글 수정 처리 완료 -> 3번 목록 페이지 이동
3번의 목록페이지 -> 80번 게시물 조회 -> 80번 게시글 삭제 버튼 클릭 -> 게시글 삭제 처리 완료 -> 3번 목록 페이지 이동
```

바로 위의 내용은 페이지 이동의 흐름을 나타낸 것으로 조회, 수정, 삭제 처리 후에 다시 특정 목록 페이지로 이동하기 위해서는 아래와 같은 정보를 각 페이지를 이동할 때마다 가지고 있어야 한다.

- 현재 목록페이지의 번호(`page`)
- 페이지당 출력할 게시글의 갯수(`perPageNum`)
- 조회게시글의 번호(`articleNo`)

그래서 `AritcleController`의 게시글 조회, 수정, 삭제 메서드에 `page`, `perPageNum`를 가진 `Criteria`타입의 `criteria`변수를 파라미터로 추가 시켜주고, 각 `JSP`페이지에 값들을 `<form>`태그안에 `hidden`값으로 세팅해줘야 한다.

### 2.1 ArticleController : 조회, 수정, 삭제 메서드 추가
기존의 조회, 수정, 삭제 메서드들을 수정해도 되지만 지금까지 작성한 메서드들과 차이점을 구분하기 위해서 새로 작성했다.

```java
@RequestMapping(value = "/readPaging", method = RequestMethod.GET)
public String readPaging(@RequestParam("articleNo") int articleNo,
                         @ModelAttribute("criteria") Criteria criteria,
                         Model model) throws Exception {

    model.addAttribute("article", articleService.read(articleNo));

    return "/article/read_paging";
}
```

```java
@RequestMapping(value = "/modifyPaging", method = RequestMethod.GET)
public String modifyGETPaging(@RequestParam("articleNo") int articleNo,
                              @ModelAttribute("criteria") Criteria criteria,
                              Model model) throws Exception {

    logger.info("modifyGetPaging ...");
    model.addAttribute("article", articleService.read(articleNo));

    return "/article/modify_paging";
}
```

```java
@RequestMapping(value = "/modifyPaging", method = RequestMethod.POST)
public String modifyPOSTPaging(ArticleVO articleVO,
                               Criteria criteria,
                               RedirectAttributes redirectAttributes) throws Exception {

    logger.info("modifyPOSTPaging ...");
    articleService.update(articleVO);
    redirectAttributes.addAttribute("page", criteria.getPage());
    redirectAttributes.addAttribute("perPageNum", criteria.getPerPageNum());
    redirectAttributes.addFlashAttribute("msg", "modSuccess");

    return "redirect:/article/listPaging";
}
```

```java
@RequestMapping(value = "/removePaging", method = RequestMethod.POST)
public String removePaging(@RequestParam("articleNo") int articleNo,
                           Criteria criteria,
                           RedirectAttributes redirectAttributes) throws Exception {

    logger.info("remove ...");
    articleService.delete(articleNo);
    redirectAttributes.addAttribute("page", criteria.getPage());
    redirectAttributes.addAttribute("perPageNum", criteria.getPerPageNum());
    redirectAttributes.addFlashAttribute("msg", "delSuccess");

    return "redirect:/article/listPaging";
}
```

### 2.2 목록 페이지의 정보를 저장하기 위한 JSP 추가

컨트롤러에서 작성했던 것과 마찬가지로 기존의 조회, 수정/삭제의 `JSP`수정하지 않고 각각의 `JSP`파일을 복사해 `_paging`이라는 접미사를 붙여 새로 작성해준다.

`read_paging.jsp`

```html
<div class="box-footer">
    <form role="form" method="post">
        <input type="hidden" name="articleNo" value="${article.articleNo}">
        <input type="hidden" name="page" value="${criteria.page}">
        <input type="hidden" name="perPageNum" value="${criteria.perPageNum}">
    </form>
    <button type="submit" class="btn btn-primary listBtn"><i class="fa fa-list"></i> 목록</button>
    <div class="pull-right">
        <button type="submit" class="btn btn-warning modBtn"><i class="fa fa-edit"></i> 수정</button>
        <button type="submit" class="btn btn-danger delBtn"><i class="fa fa-trash"></i> 삭제</button>
    </div>
</div>
```

`<form>`태그 안에 `<input>`태그를 `hidden`으로 `page`와 `perPageNum`을 추가해준다.

```js
$(document).ready(function () {

    var formObj = $("form[role='form']");
    console.log(formObj);

    $(".modBtn").on("click", function () {
        formObj.attr("action", "/article/modifyPaging");
        formObj.attr("method", "get");
        formObj.submit();
    });

    $(".delBtn").on("click", function () {
       formObj.attr("action", "/article/removePaging");
       formObj.submit();
    });

    $(".listBtn").on("click", function () {
       formObj.attr("method", "get");
       formObj.attr("action", "/article/listPaging");
       formObj.submit();
    });

});
```

수정, 삭제, 목록 페이지 URI을 컨트롤러에서 작성한 것과 같이 수정해준다.

`modify_paging.jsp`

```html
<input type="hidden" name="articleNo" value="${article.articleNo}">
<input type="hidden" name="page" value="${criteria.page}">
<input type="hidden" name="perPageNum" value="${criteria.perPageNum}">
```

`<form>`태그 안에 `<input>`태그를 `hidden`으로 `page`와 `perPageNum`을 추가해준다.

```js
$(document).ready(function () {

    var formObj = $("form[role='form']");
    console.log(formObj);

    $(".modBtn").on("click", function () {
        formObj.submit();
    });

    $(".cancelBtn").on("click", function () {
        history.go(-1);
    });

    $(".listBtn").on("click", function () {
        self.location = "/article/listPaging?page=${criteria.page}&perPageNum=${criteria.perPageNum}";
    });

});
```

목록 버튼 클릭했을 때 이동할 uri를 위와 같이 수정해준다.

## 3. 간단 요약정리

지금까지 정리한 내용 중에서 기억해야할 내용을 정리해보자.
- 페이징 처리 개선 작업 요약
  - `UriComponentsBuilder`를 이용하여 원하는 URI를 자동 생성할 수 있다.
  - 자바스크립트의 클릭이벤트 발생시 `attr()`을 이용하여 URI를 제어할 수 있다.
- 목록 페이지 정보 유지 작업 요약
  - 현재 목록페이지 번호(`page`), 목록 페이지 당 출력할 게시글 갯수(`perPageNum`), 게시글 번호(`articleNo`)를 가지고 페이지를 이동해야한다.
  - 컨트롤러에 위의 정보를 파라미터로 받아 각각의 `JSP`페이지에 세팅해준다.

---
