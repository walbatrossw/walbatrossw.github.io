> 본 글은 [자바 ORM 표준 JPA 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=9252528)를 읽고 개인적으로
학습한 내용 복습하기 위해 작성된 글로 내용상 오류가 있을 수 있습니다. 오류가 있다면 지적 부탁 드리겠습니다.

# JPA - 연관관계 매핑 기초

이번에는 객체의 참조와 테이블의 외래키를 매핑하는 법에 대해서 정리해보자.

## 1. 연관관계 매핑 핵심 키워드

- 방향 : 단방향, 양방향
- 다중성 : 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:N)
- 연관관계의 주인 : 양방향 연관관계인 경우 연관관계의 주인이 존재해야함

## 2. 단방향 연관관계

회원과 팀의 관계를 통해 단방향 연관관계 그리고 객체와 테이블의 연관관계의 차이점이 무엇인지 알아보자.

![n-to-one-one-way](http://www.plantuml.com/plantuml/png/RP31JeD048RlynHZlPH4gju92KQQUC13nNC3ChKReJIxImBH9ZtepQE9Uk0T6kNWCo6-Wy1IR1CdsTrClld_sHd9P4XORlBgewZUr_Zpz5pbnVVNIvM_OVMwAOirrgVyV0HmREVCdk0grlNu0LtRSW1kGay8T2ZoYUCzO1qiQ1-Nl0XbVRLP8eWll8XWjTMqV4-nvPqRwhg9WvFHw7JG5WCS13HZ2qBf-I4DKmsLKug6WKFtw70yjkhGq3JxqYLdaSUpwHrbZS3mpTGuzl_uBER4RXSzJngUc31d0M54SXu7GYL24gTOPaiI0Ee7QY5QDyLkAqMc5-TQYtQxD7lhmVMLFRcq7BiPMtas-3_7FLad-GK0)

- 회원(N), 팀(1)
- 회원은 하나의 팀에만 소속

### 2.1 객체와 테이블의 연관관계 차이점

객체 연관관계의 특징은 아래와 같다.
- 회원객체는 `Member.team`으로 팀 객체와 연관관계를 맺음
- 회원객체와 팀 객체는 단방향 연관관계, 팀 객체는 회원을 알수 없음, 팀에서 회원을 접근하는 필드가 없음
- 참조를 통한 연관관계는 언제나 단방향
- 양방향이 되기 위해서는 반대쪽에서도 참조를 보관하는 필드가 존재해야함
- 엄밀히 말해 양방향이 아니라 단방향 관계 2개
- 객체는 참조로 연관관계를 맺음

```java
// 객체의 단방향 관계
class Member {
  // 팀 객체의 참조를 보관
  Team team;  
}

class Team {
  // 회원객체를 보관하는 필드가 존재X
}

// 객체의 양방향 관계 : 단방향 2개
class Member {
  // 팀 객체의 참조를 보관
  Team team;  
}

class Team {
  // 회원 객체의 참조를 보관
  Member member;
}
```

테이블 연관관계의 특징은 아래와 같다.
- 회원 테이블은 `TEAM_ID` 외래키로 팀 테이블과 연관관계를 맺음
- 회원 테이블과 팀 테이블은 양방향 관계
- `TEAM_ID`를 통해 회원과 팀, 팀과 회원을 조인할 수 있음
- 테이블은 외래키로 연관관계를 맺음

```sql
-- 회원과 팀을 조인
SELECT * FROM MEMBER M JOIN TEAM T ON M.TEAM_ID = T.ID;

-- 팀과 회원을 조인
SELECT * FROM TEAM T JOIN MEMBER M ON T.ID = M.TEAM_ID;
```

### 2.1 순수한 객체 연관관계

JPA를 사용하지 않고 순수한 객체의 연관관계는 아래와 같이 코드를 작성할 수 있고, 연관관계를 맺어줄 수 있다.

```java
// 순수한 객체의 연관관계
// 회원
public class Member {

  private String id;
  private String username;

  private Team team; // 팀의 참조 보관

  // getter, setter
}

// 팀
public class Team {

  private String id;
  private String name;

  // getter, setter
}
```

```java
Member member1 = new Member("m01", "kim");  // 회원1
Member member2 = new Member("m02", "lee");  // 회원2
Team team = new Team("t01", "team01");  // 팀

// 회원을 팀에 소속
member1.setTeam(team);
member2.setTeam(team);
```

그리고 객체의 참조를 통해 연관관계를 탐색할 수 있는데 이것을 객체 그래프 탐색이라 한다.

```java
// 객체 그래프 탐색
Team team = member1.getTeam();
```

### 2.2 테이블 연관관계

회원, 팀 테이블의 관계를 설정하고, 2명의 회원을 팀에 소속시켜보자.

```sql
-- 회원 테이블
CREATE TABLE MEMBER (
  MEMBER_ID VARCHAR(255) NOT NULL,
  TEAM_ID VARCHAR(255),
  USERNAME VARCHAR(255),
  PRIMARY KEY (MEMBER_ID)
)

-- 팀 테이블
CREATE TABLE TEAM (
  TEAM_ID VARCHAR(255) NOT NULL,
  NAME VARCHAR(255),
  PRIMARY KEY (TEAM_ID)
)

-- 외래키 설정
ALTER TABLE MEMBER ADD CONSTRAINT FK_MEMBER_TEAM
  FOREIGN KEY (TEAM_ID) REFERENCES TEAM
```

```sql
-- 팀 입력
INSERT INTO TEAM(TEAM_ID, NAME) VALUES ('t01', 'team01');

-- 회원 입력
INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, NAME) VALUES ('m01', 't01', 'kim');
INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, NAME) VALUES ('m01', 't01', 'lee');

-- kim이 소속된 팀 조회
SELECT T.*
FROM MEMBER M
  JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
WHERE M.MEMBER_ID = 'm01';
```

### 2.3 객체 관계 매핑

JPA를 사용하여 회원과 팀을 매핑해보자.

```java
// 회원 클래스
@Entity
public class Member {

  @Id
  @Column(name = "MEMBER_ID")
  private String id;

  private String username;

  // 연관관계 매핑 : 다대일
  @ManyToOne
  @JoinColumn(name = "TEAM_ID")
  private Team team;

  // getter, setter

  // ...

  // 팀 연관관계 설정
  public void setTeam(Team team) {
    this.team = team;
  }

}
```

```java
// 팀 클래스
@Entity
pubic class Team {

  @Id
  @Column(name = "TEAM_ID")
  private String id;

  private String name;

  // getter, setter
}
```

위의 코드에서 주목해야할 점을 정리해보자.

- 객체의 연관관계 : 회원 객체의 `Member.team`필드
- 테이블의 연관관계 : 회원 테이블의 `MEMBER.TEAM_ID` 외래키 칼럼을 사용
- `@ManyToOne` : 다대일(N:1) 관계라는 매핑정보 지정
- `@JoinColumn(name = "TEAM_ID")` : 외래키를 매핑할 때 사용, `name` 속성은 외래키 이름을 지정

### 2.4 @JoinColumn

`@JoinColumn`은 외래키를 매핑할 때 사용하며 주요 속성은 아래와 같다.

- `name` : 매핑할 외래키 이름 지정, 기본값은 `필드명_참조 테이블의 기본키 컬럼명`
- `referencedColumnName` : 외래키가 참조하는 대상 테이블의 칼럼명, 기본값은 참조 테이블의 기본키 칼럼명
- `foreignKey(DDL)` : 외래키 제약조건 직접 지정, 테이블 생성시에만 사용

### 2.5 @ManyToOne

`@ManyToOne`은 다대일(N:1) 관계에서 사용한다.

- `optional` : `false`로 설정하면 항상 연관된 엔티티가 존재해야함, 기본값은 `true`
- `fetch` : 글로벌 패치 전략 설정, 기본값은 `@ManyToOne = FetchType.EAGER`, `@ManyToOne = FetchType.LAZY`
- `cascade` : 영속성 전이 기능 사용
- `targetEntity` : 연관된 엔티티 타입 정보 설정, 거의 사용하지 않음

## 3. 연관관계 사용

### 3.1 테스트 코드 작성을 위한 설정

연관관계를 사용하기에 앞서 테스트 코드를 작성하기 위한 설정을 추가해보자.

`pom.xml`에 junit을 추가해준다.

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

테스트 디렉토리에 엔티티 매니저 팩토리, 매니저, 트랜잭션을 생성하는 테스트 클래스를 아래와 같이 작성해준다.
```java
public class JPAHibernateTest {

    protected static EntityManagerFactory factory;  // 엔티티 매니저 팩토리
    protected static EntityManager manager;         // 엔티티 매니저
    protected static EntityTransaction transaction; // 엔티티 트랜잭션


    @BeforeClass // 대상 클래스에서 테스트가 실행되기 전 딱 한번만 수행되는 메서드
    public static void init() throws Exception {
        factory = Persistence.createEntityManagerFactory("jpa05"); // 팩토리 생성
        manager = factory.createEntityManager(); // 매니저 생성
        transaction = manager.getTransaction(); // 트랜잭션 획득
    }

    @AfterClass // 대상 클래스에서 테스트가 실행된 후 딱 한번만 수행되는 메서드
    public static void tearDown() throws Exception {
        manager.clear(); // 매니저 초기화
        manager.close(); // 매니저 닫음
        factory.close(); // 팩토리 닫음
    }

}
```

테스트 클래스를 생성하고 아래와 같이 이전에 작성한 클래스를 상속받으면 엔티티 매니저 팩토리, 매너저, 트랜잭션을
반복 작성하지 않고 사용이 가능하다.

```java
public class OneWayTest extends JPAHibernateTest {

  // 테스트 메서드
  @Test
  public testMethod() {
    transaction.begin(); // 트랜잭션 시작

    // 테스트 로직 작성

    manager.persist(); // 저장
    manager.find(); // 조회
    manager.remove(); // 삭제

    transaction.commit(); // 트랜잭션 커밋
  }

}
```

### 3.2 저장

연관관계를 설정한 엔티티가 잘 저장되는 테스트 코드를 작성하고, 코드를 실행시켜보자.

```java
// 저장 테스트
@Test
public void testSave() {

    // GIVEN
    transaction.begin(); // 트랜잭션 시작

    // 팀 객체 생성
    Team team1 = new Team("t01", "team01");
    // 팀 저장
    manager.persist(team1);

    // 회원 객체 생성 1
    Member member1 = new Member("m01", "kim");
    member1.setTeam(team1);     // 팀 연관 관계 설정
    manager.persist(member1);   // 회원 저장

    // 회원 객체 생성 2
    Member member2 = new Member("m02", "lee");
    member2.setTeam(team1);     // 팀 연관 관계 설정
    manager.persist(member2);   // 회원 저장

    transaction.commit(); // 트랜잭션 커밋

    // WHEN
    Team findTeam = manager.find(Team.class, team1.getId());
    Member findMember1 = manager.find(Member.class, member1.getId());
    Member findMember2 = manager.find(Member.class, member2.getId());

    // THEN
    assertEquals(team1, findTeam);
    assertEquals(member1, findMember1);
    assertEquals(member2, findMember2);

}
```

```console
Hibernate:
    insert
    into
        team
        (name, team_id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
```

정상적으로 테스트를 통과하고 콘솔에 `INSERT`쿼리가 출력된 것을 확인할 수 있다.

이제 위 테스트 코드에서 눈여겨볼 점은 JPA에서 엔티티를 저장할 때는 연관된 모든 엔티티는 영속상태이어야 한다는 것이다.

```java
member1.setTeam(team1); // 회원 --> 팀 연관관계 설정
manager.persist(member1); // 저장
```

### 3.3 조회

연관관계가 있는 엔티티를 조회하는 방법은 크게 2가지로 나눌 수 있다.

- 객체 그래프 탐색(객체 연관관계를 사용한 조회)
- 객체지향 쿼리 사용(JPQL)

객체 그래프 탐색 테스트를 진행해보자.

```java
// 객체 그래프 탐색 조회 테스트
@Test
public void testFind() {
    // GIVEN

    // WHEN
    Member findMember = manager.find(Member.class, "m01");
    Team team = findMember.getTeam(); // 객체 그래프 탐색

    // THEN
    assertEquals("t01", team.getId());
    assertEquals("team01", team.getName());
}
```

```console
Hibernate:
    select
        member0_.member_id as member_i1_0_0_,
        member0_.team_id as team_id3_0_0_,
        member0_.username as username2_0_0_,
        team1_.team_id as team_id1_1_1_,
        team1_.name as name2_1_1_
    from
        member member0_
    left outer join
        team team1_
            on member0_.team_id=team1_.team_id
    where
        member0_.member_id=?
```

객체 그래프 탐색을 통해 연관된 엔티티를 조회한 결과 위와 같이 정상적으로 테스트를 통과하고, 콘솔에 쿼리가
출력되었다.

이번에는 객체지향 쿼리인 JPQL에서 연관관계를 어떻게 사용하는지 코드를 작성해보자.

```java
// JPQL 쿼리 조회 테스트
@Test
public void testFindJPQL() {
    // GIVEN
    String jpql = "SELECT m FROM Member m JOIN m.team t WHERE t.name=:teamName"; // JPQL 쿼리

    // WHEN
    List<Member> results = manager.createQuery(jpql, Member.class) // 조회
            .setParameter("teamName", "team01")
            .getResultList();

    // THEN
    assertThat(results.get(0), Matchers.<Member>hasProperty("username", is("kim")));
    assertThat(results.get(1), Matchers.<Member>hasProperty("username", is("lee")));
}
```

```console
Hibernate:
    select
        member0_.member_id as member_i1_0_,
        member0_.team_id as team_id3_0_,
        member0_.username as username2_0_
    from
        member member0_
    inner join
        team team1_
            on member0_.team_id=team1_.team_id
    where
        team1_.name=?
Hibernate:
    select
        team0_.team_id as team_id1_1_0_,
        team0_.name as name2_1_0_
    from
        team team0_
    where
        team0_.team_id=?
```

JPQL을 통해 팀이름을 통해 회원을 조회한 테스트가 통과된 것을 확인할 수 있다. JPQL에 대해 따로 정리할 예정이다.

### 3.4 수정

```java
@Test
public void testUpdateRelation() {

    // GIVEN
    transaction.begin();
    Team newTeam = new Team("t02", "team02"); // 팀 객체 새로 생성
    manager.persist(newTeam);   // 새로 생성한 팀 저장

    // WHEN
    Member member1 = manager.find(Member.class, "m01"); // 회원1 조회
    member1.setTeam(newTeam);   // 새로운 팀 연관관계 설정
    transaction.commit();       // 수정 처리
    Member findMember = manager.find(Member.class, "m01"); // 회원1 다시 조회

    // THEN
    assertEquals(findMember.getTeam().getName(), "team02");

}
```

```console
Hibernate:
    select
        member0_.member_id as member_i1_0_0_,
        member0_.team_id as team_id3_0_0_,
        member0_.username as username2_0_0_,
        team1_.team_id as team_id1_1_1_,
        team1_.name as name2_1_1_
    from
        member member0_
    left outer join
        team team1_
            on member0_.team_id=team1_.team_id
    where
        member0_.member_id=?
Hibernate:
    insert
    into
        team
        (name, team_id)
    values
        (?, ?)
Hibernate:
    update
        member
    set
        team_id=?,
        username=?
    where
        member_id=?
```

update의 경우 따로 `update()`와 같은 메서드가 없다. 단순히 불러온 엔티티의 값만 변경하고 트랜잭션이
커밋할 때 플러시가 일어나면서 변경감지 기능이 작동하게 된다. 그리고 데이터베이스에 자동으로 반영된다.

연관관계를 수정할 경우에도 참조하는 대상만 변경하면 나머지는 JPA가 알아서 자동으로 처리한다.

### 3.5 연관 관계 제거

```java
// 연관관계 제거 테스트
@Test
public void testDeleteRelation() {

    // GIVEN
    transaction.begin();
    Member member1 = manager.find(Member.class, "m01"); // 조회

    // WHEN
    member1.setTeam(null); // 연관 관계 제거
    transaction.commit();

    Member findMember = manager.find(Member.class, "m01"); // 회원1 다시 조회

    // THEN
    assertNull(findMember.getTeam());
}
```

```console
Hibernate:
    select
        member0_.member_id as member_i1_0_0_,
        member0_.team_id as team_id3_0_0_,
        member0_.username as username2_0_0_,
        team1_.team_id as team_id1_1_1_,
        team1_.name as name2_1_1_
    from
        member member0_
    left outer join
        team team1_
            on member0_.team_id=team1_.team_id
    where
        member0_.member_id=?
Hibernate:
    update
        member
    set
        team_id=?,
        username=?
    where
        member_id=?
```

연관관계를 제거하는 방법은 간단하다. 연관관계를 `null`로 설정해주면 된다. 콘솔에서 테스트를 성공적으로 통과하고 SQL을
실행한 것을 확인할 수 있다.

### 3.6 연관된 엔티티 삭제

```java
// 연관된 엔티티 제거 테스트
@Test
public void testRemoveEntity() {

    // GIVEN
    transaction.begin();
    Team team1 = manager.find(Team.class, "t01"); // 팀1 조회
    Member member2 = manager.find(Member.class, "m02"); // 회원1 조회

    // WHEN
    member2.setTeam(null);  // 연관관계 제거
    manager.remove(team1); // 팀1 삭제
    transaction.commit();

    Team findTeam = manager.find(Team.class, "t01"); // 팀1 조회

    // THEN
    assertNull(findTeam);
}
```

```console
Hibernate:
    select
        team0_.team_id as team_id1_1_0_,
        team0_.name as name2_1_0_
    from
        team team0_
    where
        team0_.team_id=?
Hibernate:
    select
        member0_.member_id as member_i1_0_0_,
        member0_.team_id as team_id3_0_0_,
        member0_.username as username2_0_0_,
        team1_.team_id as team_id1_1_1_,
        team1_.name as name2_1_1_
    from
        member member0_
    left outer join
        team team1_
            on member0_.team_id=team1_.team_id
    where
        member0_.member_id=?
Hibernate:
    update
        member
    set
        team_id=?,
        username=?
    where
        member_id=?
Hibernate:
    delete
    from
        team
    where
        team_id=?
Hibernate:
    select
        team0_.team_id as team_id1_1_0_,
        team0_.name as name2_1_0_
    from
        team team0_
    where
        team0_.team_id=?
```

연관된 엔티티를 삭제하려면 기존의 연관관계를 제거하고, 삭제를 해야한다. 그렇지 않으면 외래키 제약조건으로 DB에서 오류가
발생한다. 테스트 결과 정상적으로 테스트를 통과하고 팀이 DB에서 삭제 된 것을 확인할 수 있다.

## 4. 양방향 연관관계

회원에서 팀으로만 접근하는 다대일 단방향에서 이제는 팀에서 회원으로 접근하는 관계를 추가하여 양방향 연관관계를 매핑해보자.

![n-to-one-two-way-relation](http://www.plantuml.com/plantuml/png/VL6xJiCm5Dtz5PUmD8WKM54KAOXKGY8TsZ9N2RbjBVAeR4TA19KO-06MCZ4oi4NGhohX7x3h40ggi2nPznpVSywr1HKnGhry_AXUFvSliAwUwxSLr4_LUdMlzli38Rxddta3M6IMbTt1o7DzGgxZW7FmCGcHmIq1kIoW4Pp2K32QJXiavyZI8C6_kC0WaTX87eGilkLK-OzOVuTGiad09Nn9kL1IJIRZm03RiXmmZhlT4wEnsejmGbC44OGnTWeJj741jWsTqU5-h-VAPOBZj5VdZ2O1AyStMAe6EtGA-oZyuISPGpfDMpvNVEwG6OrGnXEpBEAwu08PPgAS8oT49z1FQpP3RrjfPdnnRcweRHVr0ornDVG6VTVtrEXQKz6x57-rDIrV)

객체의 양방향 연관관계에서 변경 사항은 `Team.members`를 `List`컬렉션으로 추가한 것이다. 테이블의의 경우 외래키 하나로
양방향을 조회할 수 있기 때문에 변경사항은 없다.

### 4.1 양방향 연관관계 매핑

```java
// 회원 클래스
// 다대일 양방향 매핑
@Entity
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    // 다대일 매핑
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")   // 외래키 설정
    private Team team;

    // constructor, getter, setter
}
```

회원 엔티티에서는 변경사항이 없다.

```java
// 팀 클래스
// 다대일 양방향 매핑
@Entity
public class Team {

    @Id
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    @OneToMany(mappedBy = "team") // 반대쪽 매핑의 필드명 지정
    private List<Member> members = new ArrayList<Member>();

    // constructor, getter, setter
}
```

팀 엔티티에서는 팀과 회원을 일대다 관계로 매핑하기 위해 팀 엔티티에 `List<Member> members`필드를 추가했다. `mappedBy`
속성은 양방향 매핑일 때 사용하는데 반대쪽 매핑의 필드 이름을 값으로 지정하면된다.

### 4.2 일대다 컬렉션 조회

```java
// 양방향 그래프 탐색 테스트
@Test
public void testBiDirection() {
    // GIVEN
    Team team = manager.find(Team.class, "t01"); // 팀 조회
    // WHEN
    List<Member> members = team.getMembers(); // 회원 객체 그래프 탐색
    // THEN
    assertThat(members.get(0), Matchers.<Member>hasProperty("username", is("lee")));
    assertThat(members.get(1), Matchers.<Member>hasProperty("username", is("kim")));
}
```

```console
Hibernate:
    select
        team0_.team_id as team_id1_1_0_,
        team0_.name as name2_1_0_
    from
        team team0_
    where
        team0_.team_id=?
Hibernate:
    select
        members0_.team_id as team_id3_1_0_,
        members0_.member_id as member_i1_0_0_,
        members0_.member_id as member_i1_0_1_,
        members0_.team_id as team_id3_0_1_,
        members0_.username as username2_0_1_
    from
        member members0_
    where
        members0_.team_id=?
```

조회한 팀에서 회원 컬렉션으로 객체 그래프 탐색을 하면 정상적으로 테스트가 통과하는 것을 확인할 수 있다.

## 5. 연관관계의 주인

### 5.1 양방향 매핑 규칙 : 연관관계의 주인

양방향 관계 매핑 시 지켜야할 규칙은 두 연관관계 중 하나를 연관관계의 주인으로 정해야 한다. 연관관계의 주인만이 연관관계와
매핑하고, 외래키를 관리(등록, 수정, 삭제)할 수 있다. 반면에 주인이 아닌 쪽은 읽기만 가능하다.

연관관계의 주인을 정하는 방법은 `mappedBy`속성을 사용하면 된다.

- 주인은 `mappedBy`속성 사용 X
- 주인이 아니면 `mappedBy` 속성을 사용해 속성의 값으로 연관관계의 주인을 지정

`Member.team`, `Team.members` 둘중에서 어떤 것을 연관관계의 주인으로 지정해야할지 알아보자.

```java
// 회원 --> 팀 방향
class Member {

  // ...

  @ManyToOne
  @JoinColumn(name = "TEAM_ID")
  private Team team;

  // ...

}

// 팀 --> 회원 방향
class Team {

  // ...

  @OneToMany
  private List<Member> members = new ArrayList<>();

  // ...
}
```

연관관계의 주인을 정한다는 것은 외래키 관리자를 선택하는 것으로 회원 테이블에 있는 `TEAM_ID` 외래키를 관리할 관리자를
선택해야 한다.

- 회원 엔티티의 `Member.team`을 주인으로 선택할 경우 자기 테이블의 외래키를 관리함
- 팀 엔티티의 `Team.members`를 주인으로 선택하면 물리적으로 전혀 다른 테이블의 외래키를 관리해야함


### 5.2 연관관계의 주인은 외래키가 있는 곳

**연관관계의 주인은 테이블에 외래키가 있는 곳으로 정해야 한다.** 회원 테이블이 외래키를 가지고 있으므로 `Member.team`이
주인이 되고, 주인이 아닌 `Team.members`에는 `mappedBy = "team"`속성을 사용해 주인이 아니라는 것을 명시해준다.

```java
@OneToMany(mappedBy = "team") // 주인 아님을 명시하고, 주인인 Member.team을 지정
private List<Member> members = new ArrayList<>();
```

**항상 기억해야할 것은 항상 다(N) 쪽이 외래키를 갖는다.** 다(N) 쪽은 항상 연관관계의 주인이기때문에 `mappedBy` 속성을
지정할 수 있다.

## 6. 양방향 연관관계 저장

양방향 연관관계를 사용해 팀과 회원들을 저장해보자.

```java
// 양방향 연관관계 저장 테스트
@Test
public void testSave() {

    // GIVEN
    transaction.begin();
    Team team1 = new Team("t01", "team01"); // 팀01 저장
    manager.persist(team1);

    Member member1 = new Member("m01", "kim"); // 회원01 저장
    member1.setTeam(team1); // 연관관계 지정
    manager.persist(member1);

    Member member2 = new Member("m02", "lee"); // 회원02 저장
    member2.setTeam(team1); // 연관관계 지정
    manager.persist(member2);
    transaction.commit();

    // WHEN
    Team findTeam = manager.find(Team.class, team1.getId());
    Member findMember1 = manager.find(Member.class, member1.getId());
    Member findMember2 = manager.find(Member.class, member2.getId());

    // THEN
    assertEquals(team1, findTeam);
    assertEquals(member1, findMember1);
    assertEquals(member2, findMember2);
}
```

```console
Hibernate:
    insert
    into
        team
        (name, team_id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
```

위의 코드는 이전에 단방향 연관관계 팀, 회원 저장 코드와 동일하다.

## 7. 양방향 연관관계 주의사항

양방향 연관관계에서 주의해야할 점은 연관관계의 주인에는 값을 저장하하지 않고, 주인이 아닌 곳에만 값을 입력하는 경우이다.
만약 DB에 외래키 값이 정상적으로 저장되지 않을 경우 이것부터 의심해봐야한다.

```java
// 양방향 연관관계 저장 테스트2 : 주의 사항 - 주인이 아닌 곳에만 연관관계 설정
@Test
public void testSaveNonOwner() {

    // GIVEN
    transaction.begin();
    Member member1 = new Member("m01", "kim"); // 회원01 저장
    manager.persist(member1);
    Member member2 = new Member("m02", "lee"); // 회원02 저장
    manager.persist(member2);
    Team team1 = new Team("t01", "team01"); // 팀01 저장
    team1.getMembers().add(member1); // 연관관계에서 주인이 아닌 곳에만 연관관계를 설정
    team1.getMembers().add(member2); // 연관관계에서 주인이 아닌 곳에만 연관관계를 설정
    manager.persist(team1);
    transaction.commit();

    // WHEN
    Member findMember1 = manager.find(Member.class, member1.getId());
    Member findMember2 = manager.find(Member.class, member2.getId());

    // THEN
    assertNull(findMember1.getTeam());
    assertNull(findMember2.getTeam());
}
```

```console
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
Hibernate:
    insert
    into
        team
        (name, team_id)``
    values
        (?, ?)
```

테스트가 정상적으로 통과한 것을 확인할 수 있다. 회원을 조회하면 TEAM_ID 칼럼 값들이 `null`인 것을 알 수 있는데 연관관계의
주인이 아닌 `Team.members`에만 값을 저장했기 때문이다. 연관관계의 주인만이 외래키의 값을 변경할 수 있기 때문에 `TEAM_ID`
외래키의 값에 `null`이 저장된다.

### 7.1 순수한 객체까지 고려한 양방향 연관관계

방금 위에서 살펴본 연관관계의 주인에만 값을 저장하고, 주인이 아닌 곳에는 저장하지 않았다. 그렇다면 실제로 이렇게 해도 될까?
사실 객체 관점에서는 양쪽 방향 모두에 값을 입력해주는 것이 가장 안전하다. JPA를 사용하지 않는 순수한 객체 상태에서 심각한 문제가
발생하기 때문이다.

```java
// 순수한 객체 연관관계 설정 : 단방향만 설정
@Test
public void testPureBiDirection() {

    transaction.begin();
    // 팀, 회원 객체 생성
    Team team1 = new Team("t01", "team01");
    Member member1 = new Member("m01", "kim");
    Member member2 = new Member("m02", "lee");

    // 연관 관계 설정 : 한쪽만 설정
    member1.setTeam(team1); // member1 --> team1
    member2.setTeam(team1); // member2 --> team1

    List<Member> members = team1.getMembers();

    assertThat(members.size(), is(2));
}
```

위의 테스트 코드를 실행하면 아래와 같이 오류가 발생한다. `Member.team`에만 연관관계를 설정하고, 반대방향은 연관관계를 설정하지
않았기 떄문이다. 즉 다시말해 양방향 연관관계가 제대로 설정되었다고 볼 수 없다.

```console
java.lang.AssertionError:
Expected: is <2>
     but: was <0>
Expected :is <2>
Actual   :<0>
```

```java
// 순수한 객체 연관관계 설정 2 : 양방향 설정
@Test
public void testPureBiDirection2() {

    // GIVEN
    // 팀, 회원 객체 생성
    Team team1 = new Team("t01", "team01");
    Member member1 = new Member("m01", "kim");
    Member member2 = new Member("m02", "lee");
    // 연관 관계 설정 : 양쪽 다 설정
    member1.setTeam(team1); //member1 --> team1
    team1.getMembers().add(member1); //team1 --> member1
    member2.setTeam(team1); // member2 --> team1
    team1.getMembers().add(member2); //team1 --> member2

    // WHEN
    List<Member> members = team1.getMembers();

    // THEN
    assertThat(members.size(), is(2));
}
```

위 테스트 코드를 실행하면 오류가 발생하지 않고, 기대한데로 테스트를 통과하게 된다.

```java
// JPA 양방향 설정
@Test
public void testORMBiDirection() {

    // GIVEN
    transaction.begin();
    // 팀 생성 및 저장
    Team team1 = new Team("t01", "team01");
    manager.persist(team1);
    // 회원 객체 생성 및 저장
    Member member1 = new Member("m01", "kim");
    Member member2 = new Member("m02", "lee");
    // 연관 관계 설정 : 양쪽 다 설정
    member1.setTeam(team1); //member1 --> team1
    team1.getMembers().add(member1); //team1 --> member1
    member2.setTeam(team1); // member2 --> team1
    team1.getMembers().add(member2); //team1 --> member2

    manager.persist(member1);
    manager.persist(member2);
    transaction.commit();

    // WHEN
    Team team = manager.find(Team.class, "t01");
    List<Member> members = team.getMembers();

    // THEN
    assertThat(members.size(), is(2));
}
```

```console
Hibernate:
    insert
    into
        team
        (name, team_id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
```

이번에는 JPA를 사용하여 양쪽에 연관관계를 설정하고 테스트를 진행하면 정상적으로 테스트가 이뤄졌다.
순수한 객체 상태에서도 작동하고, 테이블의 외래키도 정상적으로 입력이 된다.

### 7.2 연관관계 편의 메서드

양방향 연관관계는 양쪽 모두를 신경써야하는데 실수로 둘중하나만 설정해주게 되면 양방향이 깨질 수 있다.

```java
// 양방향 설정, 둘중 하나라도 누락하면 양방향성이 깨진다.
member.setTeam(team); // 회원 --> 팀
team.getMembers().add(member); // 팀 --> 회원
```

두 코드를 하나인 것 처럼 사용하는 것이 안전하다. 그래서 `Member`클래스의 `setTeam()`메서드를 아래와
같이 수정해보자.

```java
public class Member {

  // ...

  // 다대일 매핑
  @ManyToOne
  @JoinColumn(name = "TEAM_ID")   // 외래키 설정
  private Team team;

  // ...

  // 연관관계 편의 메서드
  public void setTeam(Team team) {
    this.team = team; // 팀 설정
    team.getMembers().add(this); // 팀 --> 회원
  }

}
```

`setTeam()`메서드 하나로 양방향 관계를 설정하도록 변경했다. 이제 연관관계 설정을 할 때에는 `Member.setTeam()`
메서드만 호출하여 사용하면 된다.

```java
// JPA 양방향 설정 : 리팩토링
@Test
public void testORMBiDirectionRefactoring() {

    // GIVEN
    transaction.begin();

    // 팀 생성 및 저장
    Team team1 = new Team("t01", "team01");
    manager.persist(team1);

    // 회원 객체 생성 및 저장
    Member member1 = new Member("m01", "kim");
    Member member2 = new Member("m02", "lee");

    // 연관 관계 설정 : 양쪽 다 설정
    member1.setTeam(team1); //member1 --> team1, team1 --> member1
    manager.persist(member1);

    member2.setTeam(team1); // member2 --> team1, team1 --> member2
    manager.persist(member2);

    transaction.commit();

    // WHEN
    Team team = manager.find(Team.class, "t01");
    List<Member> members = team.getMembers();

    // THEN
    assertThat(members.size(), is(2));
}
```

```java
Hibernate:
    insert
    into
        team
        (name, team_id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
Hibernate:
    insert
    into
        member
        (team_id, username, member_id)
    values
        (?, ?, ?)
```

정상적으로 테스트가 통과하고 데이터가 DB에 저장되었다. 이렇게 한번에 양방향 관계를 설정하는 메서드를
편의 메서드라 한다.

### 7.3 편의 메서드 작성시 주의사항

현재 `setTeam()`메서드의 버그가 존재한다. 회원1을 새로운 팀에 설정하였을 경우, 이전 팀과의 연관관계를
수정하지 않았기 때문에 아래와 같이 팀1을 통해 회원을 조회하였을 경우에 여전히 회원1 존재하게 된다.

```
member1 <-----> team1
                team2
```

```
member1 <------ team1
member1 <-----> team2
```

이러한 문제를 해결하기 위해 아래와 같이 코드를 수정해야한다.

```java
// 연관관계 편의 메서드
public void setTeam(Team team) {

    // 기존의 팀이 존재할 경우, 팀관계를 제거
    if (this.team != null) {
        this.team.getMembers().remove(this);
    }

    // 새로운 연관관계 설정
    this.team = team;
    team.getMembers().add(this);

}
```

## 8. Summery / Conclusion

- 연관관계 매핑 키워드 : 방향, 다중성, 주인
- 객체와 테이블의 연관관계 차이점
  - 객체는 참조를 통해 연관관계를 맺음
  - 객체의 양방향 연관관계는 참조 2개로 이루어지는 것
  - 테이블은 외래키를 통해 연관관계를 맺으며 어느방향에서든 조인이 가능
- 연관관계 매핑 애너테이션
  - `@JoinColumn` : 외래키 매핑
  - `@ManyToOne` : 다대일(N:1) 매핑, 연관관계의 주인
  - `@OneToMany` : 일대다(1:N) 매핑, `mappedBy`속성을 통해 주인이 아님을 지정
- 단방향 연관관계
  - 저장시에는 모든 엔티티가 영속상태이어야함
  - 조회시에는 역방향 그래프 탐색이 불가
  - 삭제에는 제약조건 때문에 연관관계를 제거하고, 삭제해야함
- 양방향 연관관계
  - 다대일(`@ManyToOne`) <--> 일대다(`@OneToMany`)
- 연관관계의 주인은 외래키가 있는 곳
- 양방향 연관관계를 설정하기 위해 편의 메서드를 작성하는 것이 실수를 방지하는 방법
- 편의 메서드 작성할 때는 양방향 연관관계를 설정하는 것은 물론 기존의 연관관계를 제거하는 로직도 추가해야함
