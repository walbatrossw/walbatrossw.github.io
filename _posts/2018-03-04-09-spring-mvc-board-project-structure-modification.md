---
layout: article
title: Spring-MVC 게시판 예제 09 - 프로젝트 구조 변경 및 수정사항
category: spring-mvc
tags: spring-mvc
key: 20180304a
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
5. [예외처리](https://walbatrossw.github.io/spring/mvc/2018/02/28/05-spring-mvc-board-exception.html)
6. [페이징처리 : Persistence, Business 계층](https://walbatrossw.github.io/spring/mvc/2018/03/01/06-spring-mvc-board-paging-persistence-business.html)
7. [페이징처리 : Control, Presentation 계층](https://walbatrossw.github.io/spring/mvc/2018/03/02/07-spring-mvc-board-paging-control-presentation.html)
8. [페이징처리 개선, 목록페이지 정보 유지](https://walbatrossw.github.io/spring/mvc/2018/03/02/08-spring-mvc-board-paging-enhancement-tip.html)

---

이전 포스팅까지 게시판 프로젝트를 생성하고, 기본적인 CURD와 페이징 처리까지 구현해보았다. 이어서 게시글 검색 기능을 추가할 예정인데 그에 앞서 지금까지 구현한 프로젝트의 예제 구조를 변경하고, View페이지를 수정하려고 한다. 그 이유는 지금까지 게시글과 관련된 기능들을 구현하면서 단계별로 어떻게 수정되고 변경되었는지 차이점을 구분하기 위해서이다. 프로젝트의 변경 및 수정사항을 아래와 같다.

## 1. Controller의 변경

게시글 관련 컨트롤러의 경우 아래와 같이 기능이 추가되거나 변경될 경우 기존의 컨트롤러에 메서드를 새로 추가하거나, 변경하지 않고 새로운 컨트롤러를 추가 해서 아래와 같이 작성해준다.

### 1.1 ArticleController : 기본적인 게시글 CRUD 기능까지만 구현

```java
@Controller
@RequestMapping("/article")
public class ArticleController {

    private static final Logger logger = LoggerFactory.getLogger(ArticleController.class);

    private final ArticleService articleService;

    @Inject
    public ArticleController(ArticleService articleService) {
        this.articleService = articleService;
    }

    @RequestMapping(value = "/write", method = RequestMethod.GET)
    public String writeGET() {

        logger.info("normal writeGET() called...");

        return "article/normal/write";
    }

    @RequestMapping(value = "/write", method = RequestMethod.POST)
    public String writePOST(ArticleVO articleVO,
                            RedirectAttributes redirectAttributes) throws Exception {

        logger.info("normal writePOST() called...");
        logger.info(articleVO.toString());
        articleService.create(articleVO);
        redirectAttributes.addFlashAttribute("msg", "regSuccess");

        return "redirect:/article/normal/list";
    }

    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public String list(Model model) throws Exception {

        logger.info("normal list() called ...");
        model.addAttribute("articles", articleService.listAll());

        return "article/normal/list";
    }

    @RequestMapping(value = "/listCriteria", method = RequestMethod.GET)
    public String listCriteria(Model model, Criteria criteria) throws Exception {
        logger.info("normal listCriteria() ...");
        model.addAttribute("articles", articleService.listCriteria(criteria));
        return "article/normal/list_criteria";
    }

    @RequestMapping(value = "/read", method = RequestMethod.GET)
    public String read(@RequestParam("articleNo") int articleNo,
                       Model model) throws Exception {

        logger.info("normal read() called ...");
        model.addAttribute("article", articleService.read(articleNo));

        return "article/normal/read";
    }

    @RequestMapping(value = "/modify", method = RequestMethod.GET)
    public String modifyGET(@RequestParam("articleNo") int articleNo,
                            Model model) throws Exception {

        logger.info("normal modifyGet() called ...");
        model.addAttribute("article", articleService.read(articleNo));

        return "article/normal/modify";
    }

    @RequestMapping(value = "/modify", method = RequestMethod.POST)
    public String modifyPOST(ArticleVO articleVO,
                             RedirectAttributes redirectAttributes) throws Exception {

        logger.info("normal modifyPOST() called ...");
        articleService.update(articleVO);
        redirectAttributes.addFlashAttribute("msg", "modSuccess");

        return "redirect:/article/list";
    }

    @RequestMapping(value = "/remove", method = RequestMethod.POST)
    public String remove(@RequestParam("articleNo") int articleNo,
                         RedirectAttributes redirectAttributes) throws Exception {

        logger.info("normal remove() ...");
        articleService.delete(articleNo);
        redirectAttributes.addFlashAttribute("msg", "delSuccess");

        return "redirect:/article/list";
    }

}
```

### 1.2 ArticlePagingController : 기본적인 게시글 CRUD + 페이징처리까지만 구현

```java
@Controller
@RequestMapping("/article/paging")
public class ArticlePagingController {

    private static final Logger logger = LoggerFactory.getLogger(ArticlePagingController.class);

    private final ArticleService articleService;

    @Inject
    public ArticlePagingController(ArticleService articleService) {
        this.articleService = articleService;
    }


    @RequestMapping(value = "/write", method = RequestMethod.GET)
    public String writeGET() {

        logger.info("paging writeGET() called...");

        return "article/paging/write";
    }

    @RequestMapping(value = "/write", method = RequestMethod.POST)
    public String writePOST(ArticleVO articleVO,
                            RedirectAttributes redirectAttributes) throws Exception {

        logger.info("paging writePOST() called...");

        articleService.create(articleVO);
        redirectAttributes.addFlashAttribute("msg", "regSuccess");

        return "redirect:/article/paging/list";
    }

    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public String list(Model model, Criteria criteria) throws Exception {

        logger.info("paging list() called ...");

        PageMaker pageMaker = new PageMaker();
        pageMaker.setCriteria(criteria);
        pageMaker.setTotalCount(articleService.countArticles(criteria));

        model.addAttribute("articles", articleService.listCriteria(criteria));
        model.addAttribute("pageMaker", pageMaker);

        return "article/paging/list";
    }

    @RequestMapping(value = "/read", method = RequestMethod.GET)
    public String read(@RequestParam("articleNo") int articleNo,
                       @ModelAttribute("criteria") Criteria criteria,
                       Model model) throws Exception {

        logger.info("paging read() called ...");
        model.addAttribute("article", articleService.read(articleNo));

        return "article/paging/read";
    }

    @RequestMapping(value = "/modify", method = RequestMethod.GET)
    public String modifyGET(@RequestParam("articleNo") int articleNo,
                            @ModelAttribute("criteria") Criteria criteria,
                            Model model) throws Exception {

        logger.info("paging modifyGet() called ...");
        model.addAttribute("article", articleService.read(articleNo));

        return "article/paging/modify";
    }

    @RequestMapping(value = "/modify", method = RequestMethod.POST)
    public String modifyPOST(ArticleVO articleVO,
                                   Criteria criteria,
                                   RedirectAttributes redirectAttributes) throws Exception {

        logger.info("paging modifyPOST() called ...");
        articleService.update(articleVO);
        redirectAttributes.addAttribute("page", criteria.getPage());
        redirectAttributes.addAttribute("perPageNum", criteria.getPerPageNum());
        redirectAttributes.addFlashAttribute("msg", "modSuccess");

        return "redirect:/article/paging/list";
    }

    @RequestMapping(value = "/remove", method = RequestMethod.POST)
    public String remove(@RequestParam("articleNo") int articleNo,
                               Criteria criteria,
                               RedirectAttributes redirectAttributes) throws Exception {

        logger.info("paging remove() called ...");
        articleService.delete(articleNo);
        redirectAttributes.addAttribute("page", criteria.getPage());
        redirectAttributes.addAttribute("perPageNum", criteria.getPerPageNum());
        redirectAttributes.addFlashAttribute("msg", "delSuccess");

        return "redirect:/article/paging/list";
    }

}
```

### 1.2 ArticlePagingSearchController : 기본적인 게시글 CRUD + 페이징처리 + 검색처리까지 구현 예정

## 2. `JSP`파일들의 변경 및 수정사항

### 2.1 게시글 관련 views 디렉토리 변경 및 JSP파일 추가

```
WEB-INF/views/article : 게시글관련 JSP디렉토리
                  |
                  ├── /normal (write.jsp, list.jsp, read.jsp, modify.jsp) : 기본적인 게시글 CRUD
                  ├── /paging (write.jsp, list.jsp, read.jsp, modify.jsp) : 기본적인 게시글 CRUD + 페이징처리
                  └── /search (write.jsp, list.jsp, read.jsp, modify.jsp) : 기본적인 게시글 CRUD + 페이징처리 + 검색처리
```

`JSP`파일의 경우 기능이 추가될 경우 위와 같이 디렉토리를 새로 생성하고, 기존의 `JSP`파일을 복사해서 추가시켜준다. 그리고 기능 구현 단계에 따라 구분해준 컨트롤러의 매핑URI로 `JSP`의 코드를 수정해준다.

### 2.2 레이아웃 변경

아래와 같이 `JSP`파일의 `<body>`태그의 클래스 속성에 `layout-boxed`를 추가시켜 아래와 같이 레이아웃을 수정해준다.

`<body class="hold-transition skin-blue sidebar-mini">` : 변경 전

![before](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/09_spring_mvc_board_project_structure_modification/before.png?raw=true)

`<body class="hold-transition skin-blue sidebar-mini layout-boxed">` : 변경 후

![after](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/09_spring_mvc_board_project_structure_modification/after.png?raw=true)

### 2.3 사이드 메뉴의 변경

`left_column.jsp`의 사이드 메뉴는 아래와 같이 기능별 구현 단계에 따라 구분할 수 있도록 변경한다.

![menu](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/09_spring_mvc_board_project_structure_modification/menu.png?raw=true)


```html
<ul class="sidebar-menu" data-widget="tree">
    <li class="header">메뉴</li>
    <li class="treeview">
        <a href="#"><i class="fa fa-edit"></i> <span>게시판(기본)</span>
            <span class="pull-right-container">
        <i class="fa fa-angle-left pull-right"></i>
      </span>
        </a>
        <ul class="treeview-menu">
            <li><a href="${path}/article/write"><i class="fa fa-pencil"></i> 게시글 쓰기</a></li>
            <li><a href="${path}/article/list"><i class="fa fa-list"></i> 게시글 목록</a></li>
        </ul>
    </li>
    <li class="treeview">
        <a href="#"><i class="fa fa-edit"></i> <span>게시판(페이징)</span>
            <span class="pull-right-container">
        <i class="fa fa-angle-left pull-right"></i>
      </span>
        </a>
        <ul class="treeview-menu">
            <li><a href="${path}/article/paging/write"><i class="fa fa-pencil"></i> 게시글 쓰기</a></li>
            <li><a href="${path}/article/paging/list"><i class="fa fa-list"></i> 게시글 목록</a></li>
        </ul>
    </li>
    <li class="treeview">
        <a href="#"><i class="fa fa-edit"></i> <span>게시판(페이징+검색)</span>
            <span class="pull-right-container">
        <i class="fa fa-angle-left pull-right"></i>
      </span>
        </a>
        <ul class="treeview-menu">
            <li><a href="${path}/article/paging/search/write"><i class="fa fa-pencil"></i> 게시글 쓰기</a></li>
            <li><a href="${path}/article/paging/search/list"><i class="fa fa-list"></i> 게시글 목록</a></li>
        </ul>
    </li>
</ul>
```

---
