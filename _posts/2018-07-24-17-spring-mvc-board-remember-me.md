---
layout: article
title: Spring-MVC 게시판 예제 17 - 자동로그인 기능 및 로그아웃 구현
category: spring-mvc
tags: spring-mvc remember-me
key: 20180724a
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
7. [Spring - RestController, ResponseEntity, AJAX](https://walbatrossw.github.io/spring/mvc/2018/03/08/spring-mvc-rest-ajax.html)
8. [Spring MVC Interceptor](https://walbatrossw.github.io/spring/mvc/2018/07/18/spring-mvc-interceptor.html)

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
11. [AJAX 댓글처리 : Persistence, Business, Control 계층](https://walbatrossw.github.io/spring/mvc/2018/03/11/11-spring-mvc-board-ajax-reply-persistence-business-control.html)
12. [AJAX 댓글처리 : Presentation 계층](https://walbatrossw.github.io/spring/mvc/2018/03/13/12-spring-mvc-board-ajax-reply-presentation.html)
13. [AOP를 이용한 LogAdvice 작성](https://walbatrossw.github.io/spring/mvc/2018/03/14/13-spring-mvc-board-aop-logging.html)
14. [댓글 갯수, 게시글 조회수 구현, 트랜잭션처리](https://walbatrossw.github.io/spring/mvc/2018/03/15/14-spring-mvc-board-transaction.html)
15. [AJAX방식의 게시판 첨부파일 기능 구현](https://walbatrossw.github.io/spring/mvc/2018/07/21/15-spring_mvc-board-fileupload.html)
16. [HttpSession을 이용하는 로그인 처리](https://walbatrossw.github.io/spring/mvc/2018/07/21/16-spring-mvc-board-httpsession-login.html)

---

자동로그인을 구현해보자.

## 1. 쿠키를 이용하는 자동로그인 방식 구현

자동 로그인을 처리하기 전에 앞서 사용자가 로그인한 후 쿠키를 만들어 브라우저로 전송하고, 다시 서버에 접속할 때 쿠키가 전달되는지 확인해보자.

### 1.1 로그인유지를 위한 체크박스
로그인 처리 구현하면서 `login.jsp`에 체크박스를 이미 만들어 두었다.


```html
<div class="col-xs-8">
    <div class="checkbox icheck">
        <label>
            <input type="checkbox" name="useCookie"> 로그인유지
        </label>
    </div>
</div>
```


![user_login_remember_me1](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/user_login_remember_me1.png?raw=true)

### 1.2 LoginInterceptor에서의 쿠키 생성하기

이전에 작성한 `LoginInterceptor` 클래스의 경우 `preHandle()`메서드를 이용해서 HttpSession에 `UserVO`타입의 객체를 보관한다. 이 것을 수정해 중간에 쿠키를 생성하고,
`HttpServletResponse`에 담아서 전송하도록 코드를 수정해준다.

```java
@Override
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    HttpSession httpSession = request.getSession();
    ModelMap modelMap = modelAndView.getModelMap();
    Object userVO =  modelMap.get("user");

    if (userVO != null) {
        logger.info("new login success");
        httpSession.setAttribute(LOGIN, userVO);
        //response.sendRedirect("/");

        if (request.getParameter("useCookie") != null) {
            logger.info("remember me...");
            // 쿠키 생성
            Cookie loginCookie = new Cookie("loginCookie", httpSession.getId());
            loginCookie.setPath("/");
            loginCookie.setMaxAge(60*60*24*7);
            // 전송
            response.addCookie(loginCookie);
        }

        Object destination = httpSession.getAttribute("destination");
        response.sendRedirect(destination != null ? (String) destination : "/");
    }

}
```

위의 코드를 보면 사용자가 로그인 유지를 선택한 경우 쿠키를 생성하고, `loginCookie`라는 이름의 변수에 현재 session의 아이디값을 보관한다. 여기서 session 아이디는
session cookie의 값을 의미한다. session cookie의 경우 브라우저를 종료하면 사라지지만, 작성하는 loginCookie의 경우 오랜시간 보관하기 위해 `setMaxAge()`메서드를
이용한다. 그리고 최종적으로 만들어진 쿠키는 `HttpServletResponse`에 담겨서 전송된다.

위 코드를 이용해서 로그인한 후에 게시물의 여러 페이지에서 매번 브라우저에 `loginCookie`가 전달되는 것을 개발자 도구에서 확인할 수 있다.

![cookie_check](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/cookie_check.png?raw=true)

![cookie_check2](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/cookie_check2.png?raw=true)

![cookie_check3](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/cookie_check3.png?raw=true)

여기서 JSESSIONID는 Tomcat에서 발행된 세션 쿠키의 이름이고, loginCookie는 인터셉터에 의해서 만들어진 쿠키이다. 자세히 보면 세션 쿠키의 값과 동일한 값이 loginCookie에 기록된
것을 볼 수 있다.

### 1.3 브라우저 종료 후 다시 접속

현재의 브라우저를 종료한 뒤 다시 해당 페이지를 쿠키가 유지되는 5분이내에 접속하면 아래와 같은 결과를 볼 수 있다.

![cookie_check4](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/cookie_check4.png?raw=true)

브라우저를 종료한 뒤 다시 접속하면 JSESSION은 변경되지만 일주일간 유효기간이 지정된 쿠키는 그 값을 유지하고 있는 것을 확인할 수 있다.

세션 쿠키는 브라우저가 종료될 때 같이 종료되기 때문에 매번 브라우저를 새로 실행하고, 접속하면 새롭게 만들어진다. 하지만 loginCookie의 경우 로그인할 때 브라우저를 이용해서 보관된다.

### 1.4 자동로그인 구상

자동로그인의 처리는 Session과 Cookie를 이용하여 처리하도록 하는데 아래와 같이 4가지의 상황이 있을 수 있다.

|번호|HttpSession에 login 객체의 유무|loginCookie의 유무|상황|
|:---:|:---:|:---:|:---:|
|1|X|X|사용자 로그인이 필요, 비로그인 상태|
|2|O|X|사용자 로그인 상태(접속 중), 로그인 유지 선택 X|
|3|X|O|이전에 로그인 한 적이 있음, 7일사이에 로그인한 적 있음|
|4|O|O|사용자 로그인 상태(접속 중), 로그인 유지 선택 O|

위의 4가지 상황 중에서 자동 로그인이 필요한 상황은 3번이다. HttpSession에 login이란 이름으로 보관된 객체가 없지만, loginCookie가 존재한다. 이 경우는 인터셉터에서 설정한 7일의 기간 사이에
접속한 적이 있다는 것을 의미하므로, 과거의 로그인 시점에 기록된 정보를 이용해서 다시 HttpSession에 login이란 이름으로 `UserVO` 객체를 보관해줘야 한다.

### 1.5 자동로그인을 위한 데이터베이스 설정 및 `LoginDTO`

위의 구상을 토대로 사용자가 loginCookie를 가지고 있다면 그 값은 과거에 로그인한 시점의 세션 아이디라는 것을 알 수 있다. 그래서 loginCookie의 값을 이용해서
데이터베이스에 `UserVO`의 정보를 읽어오고, 읽어본 `UserVO` 객체를 현재의 HttpSession에 보관하면 로그인이 된다. 이 과정을 정리하면 아래와 같다.

- 사용자가 로그인하면 데이터베이스에 현재의 Session ID값과 유효기간을 저장한다.
- 사용자가 로그인하지 않은 상태에서 쿠키를 가지고 접속하면 쿠키의 내용물을 추출한다.
- 쿠키의 내용물로 데이터베이스를 조회하고, 유효기간에 맞는 값인지 검증한다.
- 확인된 사용자는 Session에 로그인한 정보를 기록해서, 자동로그인 처리를 수행한다.

위의 내용을 구현하기 위해서 데이터베이스의 `tbl_user`의 컬럼들을 살펴보자. 이전에 회원가입과 로그인을 구현하면서 이미 자동로그인과 관련된 칼럼을 생성해두었다.

```sql
-- 회원 테이블
CREATE TABLE tbl_user (
  user_id VARCHAR(50) NOT NULL,
  user_pw VARCHAR(100) NOT NULL,
  user_name VARCHAR(100) NOT NULL,
  user_point INT NOT NULL DEFAULT 0,
  session_key VARCHAR(50) NOT NULL DEFAULT 'none',  -- session 아이디 보관
  session_limit TIMESTAMP,                          -- 자동로그인 유효시간 기록
  user_img VARCHAR(100) NOT NULL DEFAULT 'user/default-user.png',
  user_email VARCHAR(50) NOT NULL,
  user_join_date TIMESTAMP NOT NULL DEFAULT NOW(),
  user_login_date TIMESTAMP NOT NULL DEFAULT NOW(),
  user_signature VARCHAR(200) NOT NULL DEFAULT '안녕하세요 ^^',
  PRIMARY KEY (user_id)
);
```

### 1.6 자동로그인 영속계층

`UserDAO` 인터페이스에 `sessionKey`와 `sessionLimit`를 업데이트하는 메서드(`keepLogin()`)와 `loginCookie`에 기록된 값으로 사용자의 정보를 조회하는 메서드(`checkUserWithSessionKey()`)를 추가하고,
`UserDAOImpl` 클래스에서 구현을 완료해준다.

```java
// 로그인 유지 처리
void keepLogin(String userId, String sessionId, Date sessionLimit) throws Exception;

// 세션키 검증
UserVO checkUserWithSessionKey(String value) throws Exception;
```

```java
// 로그인 유지 처리
@Override
public void keepLogin(String userId, String sessionId, Date sessionLimit) throws Exception {
    Map<String, Object> paramMap = new HashMap<>();
    paramMap.put("userId", userId);
    paramMap.put("sessionId", sessionId);
    paramMap.put("sessionLimit", sessionLimit);

    sqlSession.update(NAMESPACE + ".keepLogin", paramMap);
}

// 세션키 검증
@Override
public UserVO checkUserWithSessionKey(String value) throws Exception {
    return sqlSession.selectOne(NAMESPACE + ".checkUserWithSessionKey", value);
}
```

`userMapper.xml`에 아래와 같이 sql을 추가해준다.

로그인 유지를 선택한 경우 현재의 세션아이디와 로그인 유지기간을 갱신해준다.

```xml
<update id="keepLogin">
    UPDATE tbl_user
    SET session_key = #{sessionId}
        , session_limit = #{sessionLimit}
    WHERE user_id = #{userId}
</update>
```

로그인 시에 loginCookie 값과 `session_key`이 일치하는 회원의 정보를 가져온다.

```xml
<select id="checkUserWithSessionKey" resultMap="userVOResultMap">
    SELECT
        *
    FROM tbl_user
    WHERE session_key = #{value}
</select>
```


### 1.7 자동로그인 서비스 계층

`UserService` 인터페이스에 로그인 유지 메서드(`keepLogin()`)와 loginCookie로 회원정보를 조회하는 메서드(`checkLoginBefore()`)를 추가하고 `UserServiceImpl` 클래스에서 구현을 완료한다.

```java
void keepLogin(String userId, String sessionId, Date next) throws Exception;

UserVO checkLoginBefore(String value) throws Exception;
```

```java
@Override
public void keepLogin(String userId, String sessionId, Date sessionLimit) throws Exception {
    userDAO.keepLogin(userId, sessionId, sessionLimit);
}

@Override
public UserVO checkLoginBefore(String value) throws Exception {
    return userDAO.checkUserWithSessionKey(value);
}
```

### 1.8 자동로그인 컨트롤러

`UserLoginController`의 로그인 매핑 메서드(`loginPost()`)를 아래와 같이 수정해준다.

```java
// 로그인 처리
@RequestMapping(value = "/loginPost", method = RequestMethod.POST)
public void loginPOST(LoginDTO loginDTO, HttpSession httpSession, Model model) throws Exception {

    UserVO userVO = userService.login(loginDTO);

    if (userVO == null || !BCrypt.checkpw(loginDTO.getUserPw(), userVO.getUserPw())) {
        return;
    }

    model.addAttribute("user", userVO);

    // 로그인 유지를 선택할 경우
    if (loginDTO.isUseCookie()) {
        int amount = 60 * 60 * 24 * 7;  // 7일
        Date sessionLimit = new Date(System.currentTimeMillis() + (1000 * amount)); // 로그인 유지기간 설정
        userService.keepLogin(userVO.getUserId(), httpSession.getId(), sessionLimit);
    }

}
```

위 코드에서 주목해서 봐야할 점은 로그인 유지기간을 7일로 설정하고, 유지기간을 실제로 DB에 저장하도록 처리하는 것이다.


### 1.9 자동로그인 인터셉터

이제 마지막으로 인터셉터를 통해 사용자가 접속을 할 경우 자동으로 로그인이 되도록 처리해보자. 책에서는 `AuthInterceptor`에 코드를 작성하였지만,
나의 경우 `RememberMeInterceptor` 클래스를 만들고 그 곳에 코드를 작성했다.

`RememberMeInterceptor` 클래스를 인터셉터로 스프링이 인식할 수 있도록 `dispatcher-servlet.xml`에 아래와 같이 설정을 추가한다.


```xml
<bean id="rememberMeInterceptor" class="com.doubles.mvcboard.commons.interceptor.RememberMeInterceptor"/>

<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**/"/>
        <ref bean="rememberMeInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```


```java
public class RememberMeInterceptor extends HandlerInterceptorAdapter {

    private static final Logger logger = LoggerFactory.getLogger(RememberMeInterceptor.class);

    @Inject
    private UserService userService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        HttpSession httpSession = request.getSession();
        Cookie loginCookie = WebUtils.getCookie(request, "loginCookie");
        if (loginCookie != null) {
            UserVO userVO = userService.checkLoginBefore(loginCookie.getValue());
            if (userVO != null)
                httpSession.setAttribute("login", userVO);
        }

        return true;
    }
}
```

위의 코드를 설명하면 아래와 같다.

- 접속한 사용자가 `loginCookie`를 가지고 있다면, `loginCookie`의 정보를 이용해 사용자 정보가 존재하는지 확인한다.
- 사용자 정보가 존재할 경우, HttpSessin에 회원의 정보를 넣어준다.

### 1.10 자동 로그인 테스트 확인

실제로 자동 로그인이 구현되었는지 확인해보자.

![remember_me_check](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/remember_me_check.gif?raw=true)

위의 화면을 보면 로그인 유지를 선택하고 로그인을 하면 실제로 DB에 쿠키값과 로그인 유지기간에 저장되고, 브라우저를 종료한 뒤 다시 접속하면 정상적으로 로그인이 유지가 되는 것을 확인할 수 있다.

## 2. 로그아웃 처리

### 2.1 로그아웃 처리 구현

로그아웃 처리는 `UserLoginController`에서 login과 같이 저장된 정보를 삭제하고 `invalidate()`를 해주는 작업과 쿠키의 유효시간을 만료시키는 작업을 해준다.

```java
// 로그아웃 처리
@RequestMapping(value = "/logout", method = RequestMethod.GET)
public String logout(HttpServletRequest request,
                     HttpServletResponse response,
                     HttpSession httpSession) throws Exception {

    Object object = httpSession.getAttribute("login");
    if (object != null) {
        UserVO userVO = (UserVO) object;
        httpSession.removeAttribute("login");
        httpSession.invalidate();
        Cookie loginCookie = WebUtils.getCookie(request, "loginCookie");
        if (loginCookie != null) {
            loginCookie.setPath("/");
            loginCookie.setMaxAge(0);
            response.addCookie(loginCookie);
            userService.keepLogin(userVO.getUserId(), "none", new Date());
        }
    }

    return "/user/logout";
}
```

위 코드의 흐름은 다음과 같다.

- HttpSession에 `login`이라는 객체가 존재하면 `login`객체를 삭제하고, `invalidate()` 처리를 한다.
- 사용자가 로그인 유지를 선택했다면, `LoginInterceptor` 인터셉터에서 설정했던 `loginCookie`값을 초기화한다.
- DB에 저장했던 세션아이디와 자동로그인 유지기간을 초기화한다.

그리고 `logout.jsp`는 아래와 같이 별다른 내용없이 로그아웃 알림 메시지와 메인 페이지(`/`)로 이동하는 코드를 작성해준다.


```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<script>
    alert("로그아웃 되었습니다.");
    self.location = "/";
</script>
</body>
</html>
```


### 2.2 로그아웃 처리 확인

실제로 로그아웃이 처리되었는지 확인해보자.

![remember_me_logout](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/remember_me_logout.gif?raw=true)

위의 화면에서 확인할 수 있듯이 로그아웃을 하면 DB의 세션아이디와 로그인 유지기간도 초기화가 되면서 로그인 유지가 풀리게 된다.

## 3. 로그인 상태에서 로그인 페이지, 회원가입 페이지 접근 제한하기

### 3.1 구현

책의 내용은 로그아웃 처리까지로 완료가 되었다. 추가적으로 이제부터는 보완해서 구현해야할 내용들은 이후의 포스팅을 통해 정리할 예정이다. 그러나 로그인과 관련된 내용에서 추가적으로 정리해야할
부분은 여기에서 정리를 했다.

지금가지 구현된 내용에서는 사용자가 로그인 상태임에도 불구하고 주소창에 로그인페이지와 회원가입 페이지로 이동이 가능하다. 인터셉터를 통해 이러한 접근이 불가능하도록 해보자.

`LoginAfterInterceptor` 인터셉터를 아래와 같이 코드를 작성해준다. 로그인 페이지나 회원가입페이지로 이동하려고 할 때 로그인 객체가 HttpSession에 존재할 경우 메인페이지로 이동하도록 처리한다.

```java
public class LoginAfterInterceptor extends HandlerInterceptorAdapter {

    private static final Logger logger = LoggerFactory.getLogger(LoginAfterInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        // 로그인 처리후 로그인페이지 or 회원가입 페이지로 이동할 경우
        HttpSession session = request.getSession();
        if (session.getAttribute("login") != null) {
            response.sendRedirect(request.getContextPath() + "/");
            return false;
        }
        return true;
    }
}
```

스프링에서 `LoginAfterInterceptor`클래스를 인터셉터로 인식하도록 `dispatcher-servlet.xml`에 아래와 같이 설정한다.

```xml
<bean id="loginAfterInterceptor" class="com.doubles.mvcboard.commons.interceptor.LoginAfterInterceptor"/>

<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/user/login"/>
        <mvc:mapping path="/user/register"/>
        <ref bean="loginAfterInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

### 3.2 구현확인

로그인 상태에서 로그인페이지(`/user/login`)과 회원가입 페이지(`/user/register`)로 이동하면 메인페이지로 리다이렉트 되는지 확인해보자.

![Login_after_redirect](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/Login_after_redirect.gif?raw=true)

위 화면에서와 같이 로그인 상태에서는 해당페이지로의 이동이 되지 않고 바로 메인페이지로 리다이렉트되는 것을 알 수 있다.

## 4. 최종 정리

지금까지 책의 내용을 바탕으로 게시판을 만들어보았다. 전체적인 내용은 책의 내용에 맞춰 게시판을 구현했지만, 극히 일부의 내용은 내방식데로 바꿨다. 이제부터는
필요하다고 생각하는 부분들을 직접 구현해보면서 포스팅을 할 예정이다. 지금까지 필요하다고 생각한 기능은 아래와 같다.

- 회원 정보 페이지
    - 회원정보 수정 기능
    - 비밀번호 찾기 기능
    - 회원 탈퇴 기능
    - 이메일 인증 기능
    - 프로필 이미지 업로드 및 수정 기능

현재까지 생각한 이 기능들은 추가적으로 구현하면서 포스팅할 예정이다.

---
