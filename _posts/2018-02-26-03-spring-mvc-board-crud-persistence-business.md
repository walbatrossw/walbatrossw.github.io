---
layout: article
title: Spring-MVC 게시판 예제 03 - 기본적인 CRUD 구현(Presistence, Business 계층)
category: spring-mvc
tags: spring-mvc crud
key: 20180226a
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

포스팅과 관련된 내용 포스팅 링크

1. [영속, 비지니스, 컨트롤러, 프레젠테이션 계층에 대한 정리 : 계층화 아키텍쳐(layered architecture)](https://walbatrossw.github.io/etc/2018/02/26/etc-layered-architecture.html)

---

## 1. 게시판 테이블 생성

기본적인 CRUD를 구현하기에 앞서 `spring_ex`스키마에 아래와 같이 `tbl_article`테이블을 생성해준다.

```sql
-- 게시판 테이블
CREATE TABLE tbl_article (
  article_no INT NOT NULL AUTO_INCREMENT,     -- 게시글 번호
  title VARCHAR(200) NOT NULL,                -- 게시글 제목
  content TEXT NULL,                          -- 게시글 내용
  writer VARCHAR(50) NOT NULL,                -- 게시글 작성자
  regdate TIMESTAMP NOT NULL DEFAULT NOW(),   -- 게시글 등록시간
  viewcnt INT DEFAULT 0,                      -- 게시글 조회수
  PRIMARY KEY (article_no)                    -- 게시글 기본키
)
```

## 2. 영속(DAO)계층 구현하기

### 2.1 ArticleVO 클래스 작성

테이블의 구조를 객체화 시킬 때 사용할 `ArticleVO`클래스를 `src/main/java/기본패키지/article/domain`패키지 경로에 작성해준다.

```java
public class ArticleVO {

    private Integer articleNo;

    private String title;

    private String content;

    private String writer;

    private Date regDate;

    private int viewCnt;

    // getter, setter, toString() 생략
}
```

### 2.2 ArticleDAO 인터페이스 작성

`src/main/java/기본패키지/article/persistence`패키지 경로에 아래와 같이 인터페이스를 생성하고, 메서드를 작성해준다.

```java
public interface ArticleDAO {

    void create(ArticleVO articleVO) throws Exception;

    ArticleVO read(Integer articleNo) throws Exception;

    void update(ArticleVO articleVO) throws Exception;

    void delete(Integer articleNo) throws Exception;

    List<ArticleVO> listAll() throws Exception;

}
```

### 2.3 ArticleDAO인터페이스를 구현한 ArticleDAOImpl클래스 작성

`src/main/java/기본패키지/article/persistence`패키지 경로에 아래와 같이 `ArticleDAO`인터페이스를 구현한 `ArticleDAOImpl`클래스를 생성하고, 메서드 오버라이딩을 해준다.

```java
@Repository
public class ArticleDAOImpl implements ArticleDAO {

    private static final String NAMESPACE = "com.doubles.mvcboard.mappers.article.ArticleMapper";

    private final SqlSession sqlSession;

    @Inject
    public ArticleDAOImpl(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public void create(ArticleVO articleVO) throws Exception {
        sqlSession.insert(NAMESPACE + ".create", articleVO);
    }

    @Override
    public ArticleVO read(Integer articleNo) throws Exception {
        return sqlSession.selectOne(NAMESPACE + ".read", articleNo);
    }

    @Override
    public void update(ArticleVO articleVO) throws Exception {
        sqlSession.update(NAMESPACE + ".update", articleVO);
    }

    @Override
    public void delete(Integer articleNo) throws Exception {
        sqlSession.delete(NAMESPACE + ".delete", articleNo);
    }

    @Override
    public List<ArticleVO> listAll() throws Exception {
        return sqlSession.selectList(NAMESPACE + ".listAll");
    }
}
```

위의 코드에서 주의하거나 주목해야할 점 3가지는 다음과 같다.

- `field injection`, `constructor injection`, `setter injection`의 차이점 : **따로 정리해서 포스팅할 예정**
- `@Repository`애너테이션을 잊지 않고 작성해야한다.
- 변수 `NAMESPACE`는 `articleMapper.xml`의 `<mappper>`태그의 `namespace`속성과 일치해야한다.

### 2.4 articleMapper.xml 작성

`src/main/resources/mappers/article/`경로에 `articleMapper.xml`을 생성하고, 아래와 같이 작성해준다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.doubles.mvcboard.mappers.article.ArticleMapper">

    <insert id="create">
        INSERT INTO tbl_article (
            article_no
            , title
            , content
            , writer
            , regdate
            , viewcnt
        ) VALUES (
            #{articleNo}
            , #{title}
            , #{content}
            , #{writer}
            , #{regDate}
            , #{viewCnt}
        )
    </insert>

    <select id="read" resultMap="ArticleResultMap">
        SELECT
            article_no
            , title
            , content
            , writer
            , regdate
            , viewcnt
        FROM
            tbl_article
        WHERE article_no = #{articleNo}
    </select>

    <update id="update">
        UPDATE tbl_article
        SET
            title = #{title}
            , content = #{content}
        WHERE
            article_no = #{articleNo}
    </update>

    <delete id="delete">
        DELETE FROM tbl_article
        WHERE article_no = #{articleNo}
    </delete>

    <select id="listAll" resultType="ArticleVO">
        <![CDATA[
        SELECT
            article_no,
            title,
            content,
            writer,
            regdate,
            viewcnt
        FROM tbl_article
        WHERE article_no > 0
        ORDER BY article_no DESC, regdate DESC
        ]]>
    </select>

    <resultMap id="ArticleResultMap" type="ArticleVO">
        <id property="articleNo" column="article_no"/>
        <result property="title" column="title" />
        <result property="content" column="content" />
        <result property="writer" column="writer" />
        <result property="regDate" column="regdate" />
        <result property="viewCnt" column="viewcnt" />
    </resultMap>

</mapper>
```

위의 `Mapper`의 코드에서 주목 해야할 점은 **`resultType`을 `typeAliases`을 통해 간결하게 작성하는 법** 과 **자바객체의 필드변수명과 DB의 칼럼명이 불일치할 경우의 처리방법** 두가지이다.

첫번째는 `Mapper`를 사용하는데 있어 매번 `parameterType`이나 `resultType`을 패키지까지 포함된 클래스명을 작성하는 것이 상당히 번거롭다. 그래서 MyBatis 설정파일인 `mybatis-config.xml`에서 `<typeAliases>`를 아래와 같이 설정하여 `Mapper`에서 `resultType`을 간결하게 작성할 수 있다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <typeAliases>
        <typeAlias type="com.doubles.mvcboard.article.domain.ArticleVO" alias="ArticleVO"/>
    </typeAliases>

</configuration>
```

두번째는 자바 객체의 필드변수명과 DB칼럼명이 불일치할 경우 발생하는 문제이다. 서두에 언급한 것처럼 이 예제는 코드로 배우는 스프링 웹프로젝트를 기반으로 하고 있다. 책에서는 자바객체의 변수와 DB칼럼명 일치하게 나오지만 나의 경우는 자바변수는 카멜표기법(camelCase)을, DB칼럼명의 경우 언더스코어(`_`) 표기법을 따르고 있다. 이렇게 별다른 설정이 없다면 `select`쿼리의 경우 원하는 결과를 가져올 수가 없게 된다. 그래서 `select`쿼리의 원하는 결과를 가져오기 위해 `SQL Aliases`를 사용해 `칼럼명 AS 필드명`와 같은 형식으로 SQL을 작성해주면 된다. 위의 코드처럼 자바 객체의 필드변수와 DB칼럼명이 불일치하는게 1개이기 때문에 `AS`를 사용함으로써 간단하게 처리할 수 있다. 하지만 만약 필드변수와 칼럼명의 불일치하는 것이 많다면 매번 SQL을 작성할 때마다 `AS`를 붙여 일치시켜야하는 번거로움이 생긴다. 이러한 번거로움을 해결하기위해 `resultMap`을 통해 자바변수명과 DB칼럼명을 일치시키는 작업을 해주고, `<select>`태그의 `resultType`속성 대신 `resultMap`속성으로 변경해주면 해결할 수 있다.
```xml
<resultMap id="ArticleResultMap" type="ArticleVO">
    <id property="articleNo" column="article_no"/>
    <result property="title" column="title" />
    <result property="content" column="content" />
    <result property="writer" column="writer" />
    <result property="regDate" column="regdate" />
    <result property="viewCnt" column="viewcnt" />
</resultMap>
```

`resultMap`을 통해 자바 객체 간의 관계(1:N, N:1, N:N)를 설정할 수도 있고, 다수의 DB테이블의 `join`쿼리의 결과를 가져올 수도 있는데 이 내용은 추후 포스팅을 통해 정리할 예정이다.

### 2.5 ArticleDAO 테스트
`/test/java/기본패키지/article`패키지에 `ArticleDAOTest`클래스를 생성하고, 게시판의 기본적인 CRUD기능 구현을 검증할 테스트 코드를 아래와 같이 작성한다.
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring-config/applicationContext.xml"})
public class ArticleDAOTest {

    private static final Logger logger = LoggerFactory.getLogger(ArticleDAOTest.class);

    @Inject
    private ArticleDAO articleDAO;

    @Test
    public void testCreate() throws Exception {
        ArticleVO article = new ArticleVO();
        article.setTitle("새로운 글 작성 테스트 제목");
        article.setContent("새로운 글 작성 테스트 내용");
        article.setWriter("새로운 글 작성자");
        articleDAO.create(article);
    }

    @Test
    public void testRead() throws Exception {
        logger.info(articleDAO.read(1).toString());
    }

    @Test
    public void testUpdate() throws Exception {
        ArticleVO article = new ArticleVO();
        article.setArticleNo(1);
        article.setTitle("글 수정 테스트 제목");
        article.setContent("글 수정 테스트 내용");
        articleDAO.update(article);
    }

    @Test
    public void testDelete() throws Exception {
        articleDAO.delete(1);
    }
}
```
테스트 결과 문제없이 DB에 게시글이 저장되고, `SELECT`, `UPDATE`, `DELETE`가 되었다.

## 3. 비지니스(Service)계층 구현하기

### 3.1 ArticleService 인터페이스 작성
`src/main/java/기본패키지/article/service`패키지에 `ArticleService`인터페이스를 생성하고, 아래와 같이 메서드를 정의해준다.
```java
public interface ArticleService {

    void create(ArticleVO articleVO) throws Exception;

    ArticleVO read(Integer articleNo) throws Exception;

    void update(ArticleVO articleVO) throws Exception;

    void delete(Integer articleNo) throws Exception;

    List<ArticleVO> listAll() throws Exception;

}
```
### 3.2 ArticleServiceImpl 클래스 작성

`src/main/java/기본패키지/article/service`패키지에 `ArticleService`인터페이스를 구현한 `ArticleServiceImpl`클래스를 생성하고, 메서드를 오버라이딩해준다.

```java
@Service
public class ArticleServiceImpl implements ArticleService {

    private final ArticleDAO articleDAO;

    @Inject
    public ArticleServiceImpl(ArticleDAO articleDAO) {
        this.articleDAO = articleDAO;
    }

    @Override
    public void create(ArticleVO articleVO) throws Exception {
        articleDAO.create(articleVO);
    }

    @Override
    public ArticleVO read(Integer articleNo) throws Exception {
        return articleDAO.read(articleNo);
    }

    @Override
    public void update(ArticleVO articleVO) throws Exception {
        articleDAO.update(articleVO);
    }

    @Override
    public void delete(Integer articleNo) throws Exception {
        articleDAO.delete(articleNo);
    }

    @Override
    public List<ArticleVO> listAll() throws Exception {
        return articleDAO.listAll();
    }
}
```
현재 기본적인 CRUD기능을 구현하기 때문에 비지니스 로직에는 특별히 주목해서 봐야할 코드는 없다. 다만 주의해야할 점은 클래스의 선언부에 스프링 bean으로 인식되기 위해 `@Service`애너테이션을 반드시 작성해줘야한다.

## 3. 간단 요약 정리
이번 포스팅에서는 게시판의 기본적인 CRUD기능 구현을 위해 DB테이블을 생성하고, 영속계층, 비지니스계층을 구현해보았다. 다음은 주의해야할 사항이나 기억해야할 점들을 간단 요약정리 정리해보자.
- DAO,Service 인터페이스 생성, 구현 클래스 작성
- 구현 클래스에 각각 `@Service`, `@Repository`애너테이션을 반드시 잊지 말고 써줄 것
- `Mapper`의 `resultType`을 위한 `mybatis-config.xml`의 alias설정
- 자바 객체 필드변수와 DB칼럼명의 불일치로 인한 문제를 해결하기 위한 SQL `AS`키워드 또는 `resultMap` 설정

---
