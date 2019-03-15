---
layout: article
title: JPA - 엔티티 매핑
category: JPA
tags: JPA
key: 20190315a
---

<!--more-->

> 본 글은 [자바 ORM 표준 JPA 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=9252528)를 읽고 개인적으로
학습한 내용 복습하기 위해 작성된 글로 내용상 오류가 있을 수 있습니다. 오류가 있다면 지적 부탁 드리겠습니다.

# JPA - 엔티티 매핑

JPA를 사용할 때 엔티티와 테이블을 정확하게 매핑하는 것이 가장 중요하다. JPA는 매핑 어노테이션을 지원하는데
아래와 같이 크게 4가지로 분류할 수 있다.

- 객체와 테이블 매핑
- 기본키 매핑
- 필드와 컬럼 매핑
- 연관관계 매핑

위의 분류를 이제 차근차근 정리해보자.

## 1. @Entity

JPA를 사용해서 테이블과 매핑할 클래스에는 `@Entity` 어노테이션을 붙인다. `@Entity`가 붙은 클래스는
JPA가 관리하는 것으로 엔티티라 부른다.

### 1.1 속성
- `name` : JPA와 사용할 엔티티 이름을 지정, 사용하지 않을 경우 클래스이름이 그대로 적용

### 1.2 주의사항
- 기본생성자 필수
- `final`, `enum`, `interface`, `inner` 클래스는 사용할 수 없음
- 저장할 필드에 `final` 사용할 수 없음

그리고 기억해야할 것은 자바에서는 기본생성자가 없을 경우 자동으로 생성해주지만 만약 파라미터가 있는 생성자가
클래스에 존재할 경우 기본생성자를 자동으로 생성해주지 않기 때문에 직접 작성해주어야한다.

## 2. @Table

`@Table` 어노테이션은 엔티티와 매핑할 테이블을 지정하고, 생략시 매핑한 엔티티 이름을 테이블 이름으로 사용한다.

### 2.1 속성
- `name` : 매핑할 테이블 이름
- `catalog` : catalog 기능이 있는 데이터베이스에서 catalog를 매핑
- `schema` : schema 기능이 있는 데이터베이스에서 schema를 매핑
- `uniqueConstraint` : DDL 매핑 시에 유니크제약조건을 만듬, 스키마 자동생성 기능을 사용해서 DDL을 만들때만 사용

## 3. 데이터베이스 스키마 자동생성

JPA는 데이터베이스 스키마를 자동으로 생성하는 기능을 제공한다. 클래스의 매핑정보를 보면 어떤 테이블에 어떤
칼럼을 사용하는지 알 수 있다. 아래는 예제에 사용할 클래스를 작성한 것이다.

### 3.1 스키마 자동 생성을 위한 클래스 작성

```java
@Entity // 클래스와 테이블 매핑
@Table(name = "MEMBER") // 매핑할 테이블 정보 명시
public class Member {

    @Id // 기본키 매핑
    @Column(name = "ID") // 필드를 컬럼에 매핑
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age; // 매핑 정보가 없을 경우 필드명이 컬럼명으로 매핑

    // 회원 타입 구분
    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    // 날짜 타입 매핑
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    // 날짜 타입 매핑
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob // 길이 제한 없음
    private String description;

    // getter, setter
}
```

### 3.2 스키마 자동 생성 옵션 설정

`persistence.xml`에서 아래와 같은 속성을 추가하면 데이터베이스 테이블을 자동으로 생성해주고, 콘솔에
DDL을 출력해준다.

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
<property name="hibernate.show_sql" value="true" />
```

예제 프로젝트 실행 후 콘솔화면을 확인해보면 아래와 같이 DDL이 출력되고, DB에 테이블이 생성되는 것을 확인
할 수 있다.

```console
Hibernate:
    drop table MEMBER if exists
Hibernate:
    create table MEMBER (
        ID varchar(255) not null,
        age integer,
        createdDate timestamp,
        description clob,
        lastModifiedDate timestamp,
        roleType varchar(255),
        NAME varchar(255),
        primary key (ID)
    )
```

위와 같이 스키마 자동생성 기능을 사용하면 애플리케이션 실행 시점에 테이터베이스 테이블이 자동으로 생성된다.
하지만 스키마 자동 생성 기능이 만든 DDL은 운영환경에서 사용할 만큼 완벽하지 않기 때문에 개발환경에서 사용하거나
매핑을 어떻게 해야하는지 참고정도로만 사용하는 것이 좋다.

`hibernate.hbm2ddl.auto`의 속성은 아래와 같다.

- `create` : 기존 테이블을 삭제하고 새로 생성, DROP + CREATE
- `create-drop` : 애플리케이션을 종료할 때 생성한 DDL을 제거, DROP + CREATE + DROP
- `update` : DB 테이블과 엔티티 매핑정보를 비교해서 변경사항만 수정
- `validate` : DB 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션 실행X, DDL수정 X
- `none` : 자동생성 기능을 사용하지않으려면 속성을 제거하거나, 유효하지않은 옵션값을 주면 됨

그리고 HBM2DDL에서 주의 해야할 것은 DDL을 수정하는 옵션(create, creat-drop, update)은 운영서버에서
사용을 절대 하면 안된다. 오직 개발서버나 개발단계에서만 사용해야만 한다.

- 개발 초기단계 : create, update
- 테스트 서버 : update, validate
- 스테이징, 운영서버 : validate, none

JPA는 2.1부터는 스키마 자동생성기능을 표준으로 지원하지만 HBM2DDL 속성이 지원하는 update, validate 옵션을
지원하지 않는다.

- 지원옵션 : none, create, drop-and-create, drop

### 3.3 이름 매핑 전략 변경

일반적으로 단어와 단어를 구분할 때 자바는 카멜표기법을 DB는 언더스코어를 주로 사용한다. 이러한 차이를 매핑하기
하기 위해서는 `@Column.name` 속성을 명시적으로 사용해서 이름을 지정해주어야 한다.

```java
@Column(name = "role_type")
String roleType;
```

위와 같이 이름 매핑 전략을 직접 지정해주어도 되지만 `persistence.xml`에서 아래와 같이 속성을 지정해주면
자동으로 자바의 카멜표기법을 테이블의 언더스코어 표기법으로 매핑한다.

```xml
<property name="hibernate.ejb.naming_strategy" value="org.hibernate.cfg.ImprovedNamingStrategy" />  
```

프로젝트를 다시 실행해보면 아래와 같이 카멜표기법에서 언더스코어 표기법으로 칼럼이 매핑 된 것을 확인할 수 있다.

```console
Hibernate:
    drop table member if exists
Hibernate:
    create table member (
        id varchar(255) not null,
        age integer,
        created_date timestamp,
        description clob,
        last_modified_date timestamp,
        role_type varchar(255),
        name varchar(255),
        primary key (id)
    )
```

## 4. DDL 생성기능

회원 클래스에서 회원 이름이 필수로 입력되고, 10자를 초과하면 안된다는 제약조건을 추가해보자.

```java
@Column(name = "NAME", nullable = false, length = 10) // 필수입력(null 허용X), 길이는 10자
private String username;
```

프로젝트를 다시 실행해보면 제약 조건이 아래와 같이 추가 되었다.

```console
Hibernate:
    drop table member if exists
Hibernate:
    create table member (
        id varchar(255) not null,
        age integer,
        created_date timestamp,
        description clob,
        last_modified_date timestamp,
        role_type varchar(255),
        name varchar(10) not null,  // 제약조건에 의해 컬럼 생성
        primary key (id)
    )
```


유니크 제약조건을 만들어주는 `@Table`의 `uniqueConstraints`속성에 대해 알아보자.

회원 클래스에서 테이블 어노테이션에 아래와 같이 유니크 제약조건(유일한 값만 저장)을 추가해줄 수 있다.

```java
@Entity
// 유니크 제약조건 추가
@Table(name = "MEMBER", uniqueConstraints = {@UniqueConstraint(
        name = "NAME_AGE_UNIQUE",
        columnNames = {"NAME", "AGE"} )})
public class Member {
   // ...
}
```
프로젝트 실행 후 콘솔화면을 확인 하면 아래와 같이 제약 조건이 추가된 것 을 확인 할 수 있다.

```console
Hibernate:
    drop table member if exists
Hibernate:
    create table member (
        id varchar(255) not null,
        age integer,
        created_date timestamp,
        description clob,
        last_modified_date timestamp,
        role_type varchar(255),
        name varchar(10) not null,
        primary key (id)
    )
Hibernate:
    alter table member
        add constraint NAME_AGE_UNIQUE  unique (name, age)  // 유니크 제약조건 변경
```

## 5. 기본키 매핑

기본키 매핑의 경우 직접 할당할 수도 있고, 데이터베이스가 생성해주는 값을 사용할 수도 있다. 데이터베이스마다
기본키를 생성하는 방식이 다르기 때문에 JPA가 제공하는 데이터베이스 기본키 생성 전략이 아래와 같이 다양하다.

1. 직접할당 : 기본키를 애플리케이션에서 직접할당
2. 자동생성 : 대리키 사용방식
    - IDENTITY : 기본키 생성을 데이터베이스에 위임, 데이터베이스에 의존
    - SEQUENCE : 데이터베이스 시퀀스를 사용하여 기본키를 할당, 데이터베이스에 의존
    - TABLE : 키 생성 테이블을 사용, 데이터베이스에 의존X
    - AUTO : 자동으로 기본키 생성

### 5.1 기본키 생성 전략 설정

기본키 생성 전략을 사용하기 위해서는 `pesistence.xml`에 아래와 같이 속성을 추가해주어야한다.

```xml
<property name="hibernate.id.new_generator_mappings" value="true" />
```

### 5.2 직접할당

기본키를 직접 할당하는 방법은 아래와 같이 클래스 필드를 `@Id`로 매핑을 하고, 엔티티를 저장하기 전에
애플리케이션에서 기본키를 직접 할당해주면 된다.

```java
// 기본키 직접 할당 전략
@Entity
@Table(name = "MEMBER_DIRECT")
public class MemberDirect {

    @Id // 기본키 매핑
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    // getter, setter ...
}
```

```java
// 기본키 직접 할당 사용 코드
for (int i = 0; i < 5; i++) {
    MemberDirect member = new MemberDirect();
    member.setUsername("doubles" + i);
    member.setId("id00" + i); // 기본키 직접 할당
    entityManager.persist(member);
    System.out.println("DIRECT member id = " + member.getId());
}
```

```console
// 테이블 생성
Hibernate:
    drop table member_direct if exists
Hibernate:
    create table member_direct (
        id varchar(255) not null,
        name varchar(255),
        primary key (id)
    )

// 식별자 출력
DIRECT member id = id000
DIRECT member id = id001
DIRECT member id = id002

// 엔티티 DB 테이블 저장
Hibernate:
    insert
    into
        member_direct
        (name, id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member_direct
        (name, id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member_direct
        (name, id)
    values
        (?, ?)
```

코드를 실행한 결과를 콘솔화면에서 확인하면 위와 같은 결과를 확인할 수 있다.

`@Id`에 적용가능 자바타입은 아래와 같다.
- 자바 기본형
- 래퍼형
- String
- java.util.Date
- java.sql.Date
- java.math.BigDecimal
- java.math.BigInteger

### 5.2 IDENTITY 전략

IDENTITY 전략은 기본키 생성을 데이터베이스에 위임하는 방식으로 주로 MySql, PostgreSQL, SQL Server,
DB2에서 사용한다.

회원 클래스를 아래와 같이 작성하고, id 칼럼값을 지정하지 않고 저장하면 콘솔에서 확인할 수 있듯이 자동으로
기본키가 생성되는 것을 확인할 수 있다.

```java
// 기본키 IDENTITY 전략
@Entity
@Table(name = "MEMBER_IDENTITY")
public class MemberIdentity {

    @Id // 기본키 매핑
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "NAME")
    private String username;

    // getter, setter 생략...
}
```

```java
// 기본키 IDENTITY 전략 사용 코드
for (int i = 0; i < 3; i++) {
    MemberIdentity member2 = new MemberIdentity();
    member2.setUsername("doubles" + i);
    // 기본키를 직접 할당는 코드 X
    entityManager.persist(member2);
    System.out.println("IDENTITY member id = " + member2.getId());
}
```

```console
// 테이블 생성
Hibernate:
    drop table member_identity if exists
Hibernate:
    create table member_identity (
        id bigint generated by default as identity, // 기본키 생성 전략
        name varchar(255),
        primary key (id)
    )

// 엔티티 DB 저장
Hibernate:
    insert
    into
        member_identity
        (id, name)
    values
        (null, ?)
// 식별자 출력
IDENTITY member id = 1  

// 엔티티 DB 저장
Hibernate:
    insert
    into
        member_identity
        (id, name)
    values
        (null, ?)
// 식별자 출력
IDENTITY member id = 2

// 엔티티 DB 저장
Hibernate:
    insert
    into
        member_identity
        (id, name)
    values
        (null, ?)
// 식별자 출력
IDENTITY member id = 3
```

콘솔화면을 보면 기본키를 직접할당 했을 때와 차이점을 확인할 수 있다. 기본키를 직접할당 했을 때는 엔티티의
식별자가 바로 출력되었지만 IDENTITY 전략을 사용한 경우 엔티티를 DB에 저장한 뒤에 출력된 것을 볼 수 있다.

이렇게 두 개의 전략이  차이점을 보이는 이유는 아래와 같다.

- 엔티티가 영속상태가 되려면 식별자가 반드시 필요한데 IDENTITY전략의 경우 엔티티를 데이터베이스에 저장해야먼 식별자를 구할수 있음
- `entityManager.persist()`를 호출하는 즉시 INSERT SQL이 DB에 바로 전달되기 때문에 트랜잭션을 지원하는 쓰기 지연이 동작하지 않음

### 5.3 SEQUENCE 전략

SEQUENCE 전략은 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다. 시퀀스를 지원하는 Oracle,
PostgreSQL, DB2, H2에서 사용이 가능하다.

```java
// SEQUENCE 전략
@Entity
@SequenceGenerator(
        name = "MEMBER_SEQ_GENERATOR",  // 식별자 생성기 이름
        sequenceName = "MEMBER_SEQ", // 매핑할 데이터베이스 시퀀스 이름
        initialValue = 1,  // DDL 생성시에 사용, 시퀀스 DDL을 생성할 때 처음 시작하는 수
        allocationSize = 1) // 시퀀스 한번 호출에 증가하는 수
public class MemberSequence {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "MEMBER_SEQ_GENERATOR")
    private Long id;

    @Column(name = "NAME")
    private String username;

    // getter, setter 생략
}
```

우선 사용할 데이터베이스 시퀀스를 매핑해주어야 하는데 `@SequenceGenerator`를 사용해서 `MEMBER_SEQ_GENERATOR`라는
시퀀스 생성기를 등록했다. `sequenceName`속성의 이름으로 `MEMBER_SEQ`을 지정했는데 JPA는 이 시퀀스
생성기를 실제 데이터베이스의 `MEMBER_SEQ`와 매핑한다.

그리고 키 생성 전략을 `GenerationType.SEQUENCE`로 설정하고 `generator = "MEMBER_SEQ_GENERATOR"`로
방금 등록한 시퀀스 생성기를 선택한다. 이렇게 되면 id 식별자 값은 `MEMBER_SEQ_GENERATOR` 시퀀스 생성기가
할당을 한다.

```java
// 시퀀스 사용 코드
for (int i = 0; i < 3; i++) {
    MemberSequence member3 = new MemberSequence();
    member3.setUsername("doubles" + i);
    entityManager.persist(member3);
    System.out.println("SEQUENCE member id = " + member3.getId());
}
```

위의 코드를 실행하고 나면 아래와 같이 콘솔화면을 확인 할 수 있다. 콘솔화면에 출력된 내용을 살펴보면 회원 테이블과
시퀀스를 생성하고 회원객체를 생성하고 저장할 때 마다 시퀀스를 호출해 1씩 증가시키는 것을 알 수 있다. 그리고
IDENTITY 전략과 다르게 쓰기지연이 가능한 것을 확인할 수 있다.

```console
// 테이블 생성
Hibernate:
    drop table member_sequence if exists
Hibernate:
    drop sequence if exists MEMBER_SEQ
Hibernate:
    create table member_sequence (
        id bigint not null,
        name varchar(255),
        primary key (id)
    )

// 시퀀스 생성
Hibernate:
    create sequence MEMBER_SEQ start with 1 increment by 1

// 시퀀스 호출시 1씩 증가
Hibernate:
    call next value for MEMBER_SEQ

// 기본키 할당
SEQUENCE member id = 1

// 시퀀스 호출시 1씩 증가
Hibernate:
    call next value for MEMBER_SEQ

// 기본키 할당
SEQUENCE member id = 2

// 시퀀스 호출시 1씩 증가
Hibernate:
    call next value for MEMBER_SEQ

// 기본키 할당
SEQUENCE member id = 3

// 엔티티 DB 저장
Hibernate:
    insert
    into
        member_sequence
        (name, id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member_sequence
        (name, id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member_sequence
        (name, id)
    values
        (?, ?)
```

`@SequenceGenerator`의 속성을 정리해보면 아래와 같다.

- name : 식별자 생성기 이름
- sequenceName : 데이터베이스에 등록된 시퀀스 이름, 기본값은 hibernate_sequence
- initialValue : DDL 생성시에만 사용되는 속성, 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정, 기본값은 1
- allocationSize : 시퀀스 호출에 증가하는 수, 기본값은 50
- catalog, schema : 데이터베이스 catalog, schema 이름

SEQUENCE 사용코드는 IDENTITY와 다를게 없지만 동작방식은 다르다. SEQUENCE 전략이 이루어지는 과정은 아래와 같다.

1. `entityManager.persist()`를 호출할 때 먼저 데이터베이스 시퀀스를 사용해 식별자를 조회
2. 조회한 식별자를 엔티티에 할당한 뒤 영속성 컨텍스트에 저장
3. 트랜젝션 커밋해서 플러시가 일어나면 엔티티를 저장

### 5.4 TABLE 전략

TABLE 전략은 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 칼럼을 만들어 데이터베이스
시퀀스를 흉내내는 전략으로 모든 DB에서 사용이 가능하다.

```java
// TABLE 전략
@Entity
@TableGenerator(name = "MEMBER_SEQ_GENERATOR",  // 식별자 생성기 이름
        table = "MY_SEQUENCES", // 키 생성 테이블 명
        pkColumnValue = "MEMBER_SEQ", // 키로 사용할 값 이름
        allocationSize = 1) // 시퀀스 한번 호출에 증가하는 수
public class MemberTable {

    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "MEMBER_SEQ_GENERATOR")
    private Long id;

    // 생략 ...
}
```

위의 코드를 살펴보면 먼저 `@TableGenerator`를 사용하여 `MEMBER_SEQ_GENERATOR`라는 이름을 가진
테이블 키 생성기를 등록한다. `MY_SEQUENCES`라는 테이블을 키 생성 테이블로 매핑하고, 키로 사용할 값의
이름을 `MEMEBR_SEQ`라고 지정했다.

그리고 기본키를 매핑한 곳에 테이블 전략을 사용한다고 명시하고, 키 생성기는 `MEMBER_SEQ_GENERATOR`으로
지정해주었다.

```java
// 기본키 TABLE 전략 사용 코드
for (int i = 0; i < 3; i++) {
    MemberTable member4 = new MemberTable();
    member4.setUsername("doubles" + i);
    entityManager.persist(member4);
    System.out.println("TABLE member id = " + member4.getId());
}
```

위의 코드를 실행하면 아래와 같은 결과를 출력한다. 테이블 전략은 시퀀스 대신에 테이블을 사용한다는 것만 제외하면
시퀀스 전략과 내부 동작 방식이 같다. TABLE 전략은 값을 조회하면서 SELECT 쿼리를 사용하고, 다음 값을 증가
시키기 위해 UPDATE 쿼리를 사용한다.

```console
// 회원 테이블, 시퀀스 테이블 생성
Hibernate:
    drop table member_table if exists
Hibernate:
    drop table MY_SEQUENCES if exists
Hibernate:
    create table member_table (
        id bigint not null,
        name varchar(255),
        primary key (id)
    )
Hibernate:
    create table MY_SEQUENCES (
         sequence_name varchar(255) not null ,
        next_val bigint,
        primary key ( sequence_name )
    )

// 값 조회
Hibernate:
    select
        tbl.next_val
    from
        MY_SEQUENCES tbl
    where
        tbl.sequence_name=? for update

// 테이블에 값 존재하지 않을 경우 삽입
Hibernate:
    insert
    into
        MY_SEQUENCES
        (sequence_name, next_val)  
    values
        (?,?)

// 다음값 증가를 위해 업데이트
Hibernate:
    update
        MY_SEQUENCES
    set
        next_val=?  
    where
        next_val=?
        and sequence_name=?
TABLE member id = 1

// 값 조회
Hibernate:
    select
        tbl.next_val
    from
        MY_SEQUENCES tbl
    where
        tbl.sequence_name=? for update

// 다음값 증가를 위해 업데이트
Hibernate:
    update
        MY_SEQUENCES
    set
        next_val=?  
    where
        next_val=?
        and sequence_name=?
TABLE member id = 2

// 값 조회
Hibernate:
    select
        tbl.next_val
    from
        MY_SEQUENCES tbl
    where
        tbl.sequence_name=? for update

// 다음값 증가를 위해 업데이트
Hibernate:
    update
        MY_SEQUENCES
    set
        next_val=?  
    where
        next_val=?
        and sequence_name=?
TABLE member id = 3

// 앤타타 DB에 저장
Hibernate:
    insert
    into
        member_table
        (name, id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member_table
        (name, id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member_table
        (name, id)
    values
        (?, ?)
```

### 5.5 AUTO 전략

AUTO 전략은 선택한 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다.

```java
// 기본키 AUTO 전략
@Entity
@Table(name = "MEMBER_AUTO")
public class MemberAuto {

    @Id // 기본키 매핑
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(name = "NAME")
    private String username;

    // 생략 ...
}
```

```java
// H2 DB : 기본키 Auto 전략
for (int i = 0; i < 3; i++) {
    MemberAuto member5 = new MemberAuto();
    member5.setUsername("doubles" + i);
    entityManager.persist(member5);
    System.out.println("AUTO member id = " + member5.getId());
}
```

```console
// 테이블 생성
Hibernate:
    drop table member_auto if exists
Hibernate:
    drop sequence if exists hibernate_sequence
Hibernate:
    create table member_auto (
        id bigint not null,
        name varchar(255),
        primary key (id)
    )

// 시퀀스 생성
Hibernate:
    create sequence hibernate_sequence start with 1 increment by 1

// 시퀀스 호출
Hibernate:
    call next value for hibernate_sequence
AUTO member id = 1
Hibernate:
    call next value for hibernate_sequence
AUTO member id = 2
Hibernate:
    call next value for hibernate_sequence
AUTO member id = 3

// 엔티티 DB 저장
Hibernate:
    insert
    into
        member_auto
        (name, id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member_auto
        (name, id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member_auto
        (name, id)
    values
        (?, ?)
```


AUTO 전략의 장점은 데이터베이스를 변경해도 코드를 수정할 필요가 없다. 특히 키 생성 전략이 확정되지 않은
개발 초기단계나 프로토 타입 개발시 편리하게 사용할 수 있다.


### 5.6 기본키 매핑 요약

1. 직접할당 : `em.persist()`를 호출하기전에 애플리케이션에서 직접 식별자 값을 할당해야한다. 값이 없으면 예외발생
2. SEQUENCE : 데이터베이스 시퀀스에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장
3. TABLE : 데이터베이스 시퀀스 생성요 테이블에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장
4. IDENTITY : 테이터베이스에 엔티티를 저장해서 식별자 값을 획득한 후 영속성 컨텍스트에 저장(테이블에 데이터를 저장해야 식별자 값 획득 가능)

## 6. 필드와 칼럼 매핑 : 레퍼런스

필드와 칼럼 매핑을 간단하게 분류하면 아래와 같다.

- 필드와 컬럼 매핑
  - `@Column` : 컬럼 매핑
  - `@Enumerated` : 자바 enum 타입 매핑
  - `@Temporal` : 날짜 타입 매핑
  - `@Lob` : BLOB, CLOB 타입을 매핑
  - `@Transient` : 특정 필드를 DB에 매핑하지 않음
- 기타
  - `@Access` : JPA가 엔티티에 접근하는 방식을 지정

이 분류에 따라 좀더 자세히 알아보자.

### 6.1 @Column

`@Column`은 객체 필드를 테이블 칼럼에 매핑한다.

#### 6.1.1 속성

- `name` 속성 - 필드와 매핑할 테이블의 컬럼 이름, 기본값은 객체의 필드 이름
- `nullable` 속성 - null 값의 허용여부를 설정, 기본값 true
- `unique` 속성 - 한 컬럼에 간단한 유니크 제약조건을 걸 때 사용, 만약 두 컬럼이상일 경우 클래스 레벨에서 사용해야함
- `length` 속성 - 문자 길이 제약조건, `String` 타입일 경우만 사용, 기본값 255

#### 6.1.2 주의사항

주의해야 할 것은 자바의 기본타입의 경우 `@Column`을 사용하면 `nullable = false`로 지정하는 것이
안전하다. 그 이유는 자바 기본타입의 경우 null값을 입력할 수 없기 때문이다.

### 6.2 @Enumerated

`@Enumerated`는 자바의 enum 타입을 매핑할 때 사용한다.

#### 6.2.1 예

```java
// enum 클래스
enum RoleType {
  ADMIN, USER
}
```

```java
// enum 매핑
@Enumerated(EnumType.STRING)
private RoleType roleType1;

@Enumerated(EnumType.ORDINAL)
private RoleType roleType2;
```

```java
// enum 사용
member1.setRoleType(roleType1.ADMIN); // DB에 문자 ADMIN으로 저장

member2.setRoleType(roleType2.ADMIN); // DB에 숫자 0으로 저장
```


#### 6.2.2 속성

- `EnumType.ORDINAL`
  - enum 순서를 데이터베이스에 저장
  - DB에 저장되는 크기가 작음
  - 저장된 enum의 순서변경 불가

- `EnumType.STRING`
  - enum 이름을 데이터베이스에 저장
  - DB에 저장되는 크기가 비교적 큼
  - 저장된 enum의 순서가 바뀌거나 추가되도 안전

### 6.3 @Temporal

날짜 타입(`java.util.Date`, `java.util.Calendar`)을 매핑할 때 사용한다.

#### 6.3.1 속성

- `TemporalType.DATE` : 날짜, 데이터베이스 date 타입과 매핑, ex) 2017-01-01
- `TemporalType.TIME` : 시간, 데이터베이스 time 타입과 매핑, ex) 01:00:00
- `TemporalType.TIMESTAMP` : 날짜와 시간 데이터베이스 timestamp 타입과 매핑, ex) 2017-01-01 01:00:00

#### 6.3.2 예

```java
@Temporal(TemporalType.DATE)
private Date date; // 날짜

@Temporal(TemporalType.TIME)
private Date time; // 시간

@Temporal(TemporalType.TIMESTAMP)
private Date timestamp; // 날짜와 시간
```


### 6.4 @Lob

데이터베이스 BLOB, CLOB 타입과 매핑한다.

#### 6.4.1 속성

- 지정할 수 있는 속성이 없음
- 매핑하는 필드 타입이 문자면 CLOB으로 매핑, 나머지는 BLOB으로 매핑
  - CLOB - String, char[], java.sql.CLOB
  - BLOB - byte[], java.sql.BLOB

#### 6.4.2 예

```java
@Lob
private String lobString;

@Lob
private byte[] lobByte;
```

### 6.5 @Transient

`@Transient`는 특정 필드를 DB에 매핑하지 않는다.

- 데이터베이스에 저장하지 않고, 조회도 하지않음
- 객체에 임시로 어떤 값을 보관하고 싶을 경우 사용

### 6.6 @Access

`@Access`는 JPA가 엔티티에 접근하는 방식을 지정한다.

#### 6.6.1 접근 방식

- 필드 접근 : `AccessType.FIELD`로 지정, 필드에 직접 접근, 접근권한이 `private`이어도 가능
- 프로퍼티 접근 : `AccessType.PROPERTY`로 지정,  접근자(`getter`)를 사용

#### 6.6.2 예

```java
// 필드 접근
@Entity
@Access(AccessType.FIELD) // 생략 가능
public class Member {

  @Id
  private String id;

  // ....
}
```

```java
// 프로퍼티 접근
@Entity
@Access(AccessType.PROPERTY) // 생략 가능
public class Member {

  private String id;

  // ....

  @Id
  public String getId() {
    return id;
  }

  // ....
}
```

```java
// 필드와 프로퍼티 접근 혼용
@Entity
public class Member {

  @Id
  private String id;  // 필드 접근 방식

  @Transient
  private String firstName;

  @Transient
  private String lastName;

  // ....

  @Access(AccessType.PROPERTY)  // 프로퍼티 접근 방식
  public String getFullName() {
    return firstName + lastName;
  }

}
```

## 7. Summery / Conclusion

- 객체 매핑 : `@Entity`
- 테이블 매핑 : `@Table`
- 스키마 자동생성
- 기본키 매핑 전략
  - 직접할당
  - IDENTITY
  - SEQUENCE
  - TABLE
  - AUTO
- 필드 / 칼럼 매핑
  - `@Column`
  - `@Enumerated`
  - `@Temporal`
  - `@Lob`
  - `@Transient`
- 필드, 프로퍼티 접근
  - `@Access`
