---
layout: article
title: Spring-MVC 게시판 예제 14  - 게시글 조회수, 댓글 갯수의 출력 구현과 트랜잭션 처리
category: spring-mvc
tags: spring-mvc transaction
key: 20180315a
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
7. [Spring - RestController, ResponseEntity, AJAX](https://walbatrossw.github.io/spring/mvc/2018/03/08/spring-mvc-rest-ajax.html)

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
11. |[AJAX 댓글처리 : Persistence, Business, Control 계층](https://walbatrossw.github.io/spring/mvc/2018/03/11/11-spring-mvc-board-ajax-reply-persistence-business-control.html)
12. [AJAX 댓글처리 : Presentation 계층](https://walbatrossw.github.io/spring/mvc/2018/03/13/12-spring-mvc-board-ajax-reply-presentation.html)
13. [AOP를 이용한 LogAdvice 작성](https://walbatrossw.github.io/spring/mvc/2018/03/14/13-spring-mvc-board-aop-logging.html)

---

## 1. 트랜잭션(Transaction)

### 1.1 트랜잭션이란?

[관련 포스팅 참조](http://doublesprogramming.tistory.com/124)

### 1.2 스프링의 트랜잭션 처리

스프링에서는 트랜잭션을 처리하는 방식은 기본적으로 XML을 사용해 선언하는 방식과 애너테이션을 활용하는 방식으로 나누어진다.

- **XML을 사용하는 경우** : 별도의 `treansaction-context.xml`파일을 이용해 XML로 작성해서 처리한다.
- **애너테이션을 활용하는 경우** : 주로 비지니스 계층의 Service인터페이스를 구현한 클래스에 애너테이션을 붙여 처리한다.

게시판 예제에서는 애너테이션을 활용하여 다음과 같은 로직을 트랜잭션 처리를 한다.

- 게시글에 댓글이 추가/삭제되면, 댓글 갯수를 갱신(1씩 증가/감소)한다.
- 게시글을 조회하면 게시글 조회수를 1씩 증가시킨다.

## 2. 트랙잭션 처리를 위한 준비

### 2.1 트랜잭션 라이브러리 추가 : pom.xml

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
```

### 2.2 트랜잭션 매너저 설정 : applicationContext.xml

트랜잭션 매니저를 설정은 보통 DataSource의 설정이 존재하는 곳에 함께 적용하는 것이 편리하다. `<tx:annotation-driven/>`은 `@Treansactional`애너테이션을 이용한 트랜잭션 관리가 가능하게 하는 설정이다.

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<tx:annotation-driven/>
```

## 3. 게시글의 댓글에 따른 트랜잭션 처리

게시글 테이블에 댓글의 갯수를 의미하는 칼럼을 추가하고, 댓글 등록/삭제시 댓글의 갯수를 갱신하는 작업을 처리하도록 한다.

### 3.1 댓글 갯수 칼럼 추가

게시글 테이블에 댓글 갯수 칼럼을 추가하고, 게시글 등록일자와 조회수 칼럼명도 수정해준다.(이 작업은 기존의 칼럼명이 맘에 들지 않아서 변경하는 것일 뿐 트랜잭션작업과 무관하다.)

```sql
-- 댓글 갯수 칼럼 추가
ALTER TABLE tbl_article ADD COLUMN reply_cnt int DEFAULT 0;

-- 등록일자 칼럼명 변경 : 트랜잭션 작업과 무관
ALTER TABLE tbl_article CHANGE COLUMN regdate reg_date int DEFAULT 0;

-- 조회수 칼럼명 변경 : 트랜잭션 작업과 무관
ALTER TABLE tbl_article CHANGE COLUMN viewcnt view_cnt int DEFAULT 0;
```

### 3.2 게시글 도메인 클래스 수정

게시글 도메인 클래스인 `ArticleVO`에 멤버변수로 `replyCnt`를 추가해준다.

```java
public class ArticleVO {

    private Integer articleNo;
    private String title;
    private String content;
    private String writer;
    private Date regDate;
    private int viewCnt;
    private int replyCnt; // 댓글 갯수 추가

    // getter, setter, toString() 생략...
}
```

### 3.3 SQL Mapper 수정 : articleMapper.xml

댓글 갯수 칼럼이 필요한 SQL문과 `resultMap`에 댓글 갯수 칼럼을 추가시켜주고, 변경한 칼럼명은 수정해준다.

```sql
<select id="listSearch" resultMap="ArticleResultMap">
    <![CDATA[
    SELECT
        article_no,
        title,
        content,
        writer,
        reg_date,
        view_cnt,
        reply_cnt
    FROM tbl_article
    WHERE article_no > 0
    ]]>
      <include refid="search"/>
    <![CDATA[
    ORDER BY article_no DESC, reg_date DESC
    LIMIT #{pageStart}, #{perPageNum}
    ]]>
</select>
```

```xml
<resultMap id="ArticleResultMap" type="ArticleVO">
    <id property="articleNo" column="article_no"/>
    <result property="title" column="title" />
    <result property="content" column="content" />
    <result property="writer" column="writer" />
    <result property="regDate" column="reg_date" />
    <result property="viewCnt" column="view_cnt" />
    <result property="replyCnt" column="reply_cnt" />
</resultMap>
```

### 3.4 게시글, 댓글 영속 계층 수정

#### 3.4.1 ArticleDAO / ArticleDAOImpl 댓글 갯수 갱신 메서드 선언 및 구현

```java
// ArticleDAO
void updateReplyCnt(Integer articleNo, int amount) throws Exception;
```

```java
// ArticleDAOImpl
@Override
public void updateReplyCnt(Integer articleNo, int amount) throws Exception {

    Map<String, Object> paramMap = new HashMap<>();
    paramMap.put("articleNo", articleNo);
    paramMap.put("amount", amount);

    sqlSession.update(NAMESPACE + ".updateReplyCnt",paramMap);
}
```

#### 3.4.2 articleMapper.xml 댓글 갯수 갱신 SQL 작성

```sql
<update id="updateReplyCnt">
    UPDATE tbl_article
    SET reply_cnt = reply_cnt + #{amount}
    WHERE article_no = #{articleNo}
</update>
```

#### 3.4.3 ReplyDAO / ReplyDAOImpl 댓글의 게시글 번호 조회 메서드 선언 및 구현

`ReplyDAO`에는 댓글이 삭제될 때 해당 게시물의 번호를 조회하는 기능을 추가한다. 이 기능을 통해 조회한 게시글의 번호를 가지고 댓글 갯수를 갱신하는 작업을 수행하게 된다.

```java
// ReplyDAO
int getArticleNo(Integer replyNo) throws Exception;
```

```java
// ReplyDAOImpl
@Override
public int getArticleNo(Integer replyNo) throws Exception {
    return sqlSession.selectOne(NAMESPACE + ".getArticleNo", replyNo);
}
```

#### 3.4.4 replyMapper.xml 댓글의 게시글 번호 조회 SQL 작성

```sql
<select id="getArticleNo" resultType="int">
    SELECT
        article_no
    FROM tbl_reply
    WHERE reply_no = #{replyNo}
</select>
```

### 3.5 댓글 비지니스 계층의 수정 및 트랜잭션 적용 : ReplyServiceImpl

ArticleDAO인터페이스를 생성자를 통해 의존성 주입시켜준다.

```java
private final ReplyDAO replyDAO;

private final ArticleDAO articleDAO;

@Inject
public ReplyServiceImpl(ReplyDAO replyDAO, ArticleDAO articleDAO) {
    this.replyDAO = replyDAO;
    this.articleDAO = articleDAO;
}
```

댓글 등록/삭제 메서드를 아래와 같이 수정해준다. 댓글이 추가되거나 삭제되면, 게시글의 댓글 갯수를 갱신하는 작업을 동시에 수행하기 때문에 `@Transactional`애너테이션을 통해 두가지의 작업을 트랜잭션 처리해준다.

댓글이 추가되면 댓글 갯수를 1씩 증가 시켜준다.

```java
// 댓글 등록
@Transactional // 트랜잭션 처리
@Override
public void addReply(ReplyVO replyVO) throws Exception {
    replyDAO.create(replyVO); // 댓글 등록
    articleDAO.updateReplyCnt(replyVO.getArticleNo(), 1); // 댓글 갯수 증가
}
```

댓글이 삭제되면 댓글 갯수를 1씩 감소 시킨다.

```java
// 댓글 삭제
@Transactional // 트랜잭션 처리
@Override
public void removeReply(Integer replyNo) throws Exception {
    int articleNo = replyDAO.getArticleNo(replyNo); // 댓글의 게시물 번호 조회
    replyDAO.delete(replyNo); // 댓글 삭제
    articleDAO.updateReplyCnt(articleNo, -1); // 댓글 갯수 감소
}
```

### 3.6 게시글 목록 페이지 수정 : list.jsp

게시글 목록 페이지에서 게시글 제목 옆에 아래와 같이 댓글 갯수를 출력하도록 코드를 작성해준다.

```html
<td>
    <a href="${path}/article/paging/search/read${pageMaker.makeSearch(pageMaker.criteria.page)}&articleNo=${article.articleNo}">
            ${article.title}
    </a>
    <span class="badge bg-teal"><i class="fa fa-comment-o"></i> + ${article.replyCnt}</span>
</td>
```

## 4. 게시글의 조회에 따른 트랜잭션 처리

게시글을 조회하면 조회수를 의미하는 칼럼인 `viewCnt`를 1씩 증가시키는 작업을 처리하도록 한다.

### 4.1 게시글 영속 계층의 조회수 증가 메서드 선언 및 구현

#### 4.1.1 ArticleDAO

```java
void updateViewCnt(Integer articleNo) throws Exception;
```

#### 4.1.2 ArticleDAOImpl
```java
@Override
public void updateViewCnt(Integer articleNo) throws Exception {
    sqlSession.update(NAMESPACE + ".updateViewCnt", articleNo);
}
```

#### 4.1.3 articleMapper.xml

```sql
<update id="updateViewCnt">
    UPDATE tbl_article
    SET view_cnt = view_cnt + 1
    WHERE article_no = #{articleNo}
</update>
```

### 4.2 게시글 비지니스 계층의 수정 및 트랜잭션 적용 : ArticleServiceImpl

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
@Override
public ArticleVO read(Integer articleNo) throws Exception {
    articleDAO.updateViewCnt(articleNo);
    return articleDAO.read(articleNo);
}
```

## 5. 최종 기능 구현 모습 확인

댓글 추가/삭제에 따른 댓글 갯수 변화와 게시글 조회에 따른 조회수 증가 모습은 아래와 같다.

![check](https://raw.githubusercontent.com/walbatrossw/TIL/master/04_spring-framework_orm/spring-mvc-board/img/14_spring_mvc_board_transaction/check.png)

## 6. 정리

게시글의 조회수 변화와 댓글 갯수의 갱신작업을 트랜잭션을 통해 처리했는데 앞서 포스팅한 내용에 비해 간단한 내용들이어서 따로 정리할 것은 없다. 다만 불만족스러운 부분은 게시글의 조회수 중복증가 방지처리를 하지 않았다는 점이다. 이 부분은 앞으로 로그인처리를 구현하면서 Session을 통해 조회수 중복증가를 막아보도록 할 예정이다.

---
