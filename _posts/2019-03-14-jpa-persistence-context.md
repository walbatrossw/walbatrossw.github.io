---
layout: article
title: JPA - 영속성 관리
category: JPA
tags: JPA
key: 20190314a
---

<!--more-->

> 본 글은 [자바 ORM 표준 JPA 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=9252528)를 읽고 개인적으로
학습한 내용 복습하기 위해 작성된 글로 내용상 오류가 있을 수 있습니다. 오류가 있다면 지적 부탁 드리겠습니다.


# JPA - 영속성 관리

## 1. 영속성 컨텍스트 기본 개념

**영속성 컨텍스트(Persistence Context)란 "엔티티를 영구 저장하는 환경"이라는 뜻이다.** 엔티티
매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에서 엔티티를 보관하고 관리한다.

`persist()`메서드는 엔티티 매니저를 사용해서 회원 엔티티를 영속성 컨텍스트에
저장한다.

## 2. 엔티티 생명주기

엔티티에는 4가지 상태가 존재하며 생명주기는 아래의 그림과 같다.

![entity-lifetime](https://github.com/walbatrossw/jpa-study/blob/master/docs/img/entity-lifetime.png?raw=true)

- 비영속(new/transient) : 영속성 컨텍스트와 관계가 없는 상태
- 영속(managed) : 영속성 컨텍스트에 저장된 상태
- 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed) : 삭제된 상태

### 2.1 비영속

엔티티 객체를 생성하여 순수한 객체의 상태이며 저장하지 않은 상태를 말한다.
따라서 영속성 컨텍스트나 데이터베이스와 관련이 없다.

```java
// 객체 생성 : 비영속
Member member = new Member();
member.setId("id001");
member.setUsername("doubles");
```

### 2.2 영속

앤티티 매니저를 통해 앤티티를 영속성 컨텍스트에 저장한 상태를 말한다. 영속성 컨텍스트가 관리하는 엔티티를
영속상태라고 한다.

```java
// 저장 : 영속
entityManager.persist(member);
// 한건 조회 : 영속
entityManager.find("id001");
// 목록 조회 : 영속
List<Member> members = entityManager.createQuery("SELECT m FROM Member m", Member.class).getResultList();
```

### 2.3 준영속

영속성 컨텍스트가 관리하던 영속상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면
준영속 상태가 된다. 특정 엔티티를 준영속 상태로 만드려면 아래의 메서드들을 호출하면 된다.
- `entityManager.detach()` : 분리
- `entityManager.close()` : 영속성 컨텍스트 닫음
- `entityManager.clear()` : 영속성 컨텍스트 초기화

### 2.4 삭제

엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한다.

```java
entityManager.remove();
```

## 3. 영속성 컨텍스트 특징

영속성 컨텍스트의 특징은 아래와 같다.

- 영속성 컨텍스트는 엔티티를 식별자 값으로 구분한다, 즉 영속상태는 식별자 값이 존재해야만 한다.
- 영속성 컨텍스트에 저장된 엔티티는 트랜잭션을 커밋하는 순간 데이터베이스에 반영된다. 이것을 플러시라 한다.
- 영속성 컨텍스트의 장점
  - 1차 캐시
  - 동일성 보장
  - 트랜잭션을 지원하는 쓰기 지연
  - 변경감지
  - 지연로딩

그럼 영속성 컨텍스트가 왜 필요하고 어떤 장점이 있는지 CRUD를 하면서 알아보자

### 3.1 조회 : 1차 캐시, 동일성 보장

영속성 컨텍스는 내부에 캐시를 가지고 있는데 이것을 1차 캐시라 한다. 영속상태의 엔티티는 이곳에 모두
저장된다. 쉽게 말하면 영속성 컨텍스트 내부에 Map이 있는데 키는 `@Id`로 매핑한 식별자고 값은 엔티티
컨텍스트이다.

```java
// 비영속 상태 : 객체 생성
Member member = new Memebr();
member.setId("id001");
member.setUsername("doubles");

// 영속 상태
// 1차 캐시에 저장
entityManager.persist(member); // 엔티티 저장

// 1차 캐시에 존재하는 엔티티를 조회할 경우
// 1. 1차 캐시에서 조회
// 2. 반환
Member member1 = entityManager.find(Member.class, "id001"); // 한 건 조회

// 1차 캐시에 존재하지 않는 엔티티를 조회할 경우
// 1. 1차 캐시에서 id002를 식별자로 가진 엔티티 조회
// 2. 1차 캐시에 존재하지 않기 때문에 DB에서 조회해서 엔티티를 생성
// 3. 1차 캐시에 저장
// 4. 반환
Member member2 = entityManager.find(Member.class, "id002"); // 한 건 조회

// 영속 엔티티 동일성
// 식별자가 같은 엔티티 2번 조회
Member a = entityManager.find(Member.class, "id001");
Member b = entityManager.find(Member.class, "id001");

// 엔티티 a와 b는 1차 캐시에 있는 같은 엔티티를 반환 받았기 때문에 같은 인스턴스
System.out.println(a == b); // true
```


### 3.2 등록 : 트랜잭션을 지원하는 쓰기 지연

```java
// memberA, memberB 객체 생성
// 엔티티 매니저 생성, 트랜잭션 획득 ...

transaction.begin(); // 트랜잭션 시작

entityManager.persist(memberA);
entityManager.persist(memberB);
// INSERT SQL을 DB에 보내지 않음

// 커밋을 수행하는 순간 INSERT SQL을 DB에 보냄
transaction.commit(); // 트랜잭션 커밋
```

엔티티 매니저는 트랜잭션을 커밋하기 전까지 DB에 엔티티를 저장하지 않고, 내부 쿼리 저장소에 INSERT SQL을
차곡차곡 모아둔다. 트랜잭션을 커밋할 때 모아둔 SQL을 DB에 보낸다. 이것을 쓰기 지연이라고 한다.

트랜잭션을 커밋하면 영속성 컨텍스트는 플러시를 하는데 여기서 플러시는 영속성 컨텍스트의 변경 내용을 DB에
동기화하는 작업을 말한다.

### 3.3 수정 : 변경 감지, 지연 로딩

```java
// 엔티티 매니저 생성, 트랜잭션 획득 ...

transaction.begin(); // 트랜잭션 시작

Member findMember = entityManager.find(Member.class, "id001"); // 영속 엔티티 조회

// 영속 엔티티 데이터 수정
findMember.setUsername("더블에스");
findMember.setAge(29);

//entityManager.update(findMember); // 이런 코드는 없다!

transaction.commit(); // 트랜잭션 커밋
```

JPA에서 엔티티를 수정할 때는 위와 같이 엔티티를 조회하고 데이터만 변경해주기만 하면 된다. 수정과 관련된
메서드는 JPA에서 따로 존재하지 않는다. 이렇게 엔티티의 변경사항을 DB에 자동으로 반영하는 것을 변경감지라고 한다.

그렇다면 변경감지 기능은 어떻게 작동하는지 알아보자.

JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해 저장해두는데 이것을 스냅샷이라 한다. 그리고
플러시 시점에 스냅샷과 엔티티를 비교하여 변경된 엔티티를 찾게된다.

1. 트랜잭션 커밋 ---> 플러시 호출
2. 엔티티와 스냅샷 비교, 변경된 엔티티를 찾음
3. 변경된 엔티티가 존재하면 수정 쿼리를 생성 ---> 쓰기지연 SQL 저장소에 보관
4. 쓰기지연 SQL을 DB에 전송
5. DB 트랜잭션 커밋

변경 감지는 영속성 컨텍스트가 관리하는 영속상태의 엔티티에만 적용되고 비영속, 준영속 상태의 엔티티는 적용되지
않는다.

변경감지로 인해 생성된 UDPATE SQL은 엔티티의 모든 필드를 업데이트 한다. 모든 필드를 DB에 보내면 데이터
전송량이 증가하는 단점이 존재하지만 아래와 같은 장점을 얻게 된다.

- 모든 필드를 사용하면 수정 쿼리가 항상 같다. 애플리케이션 로딩시점에 수정쿼리를 미리 생성하고 재사용이 가능하다.
- DB에 동일한 쿼리를 보내면 DB는 이전에 한번 파싱된 쿼리를 재사용할 수 있다.

### 3.4 삭제

```java
Member member1 = entityManager.find(Member.class, "id001"); // 삭제 대상 엔티티 조회
entityManager.remove(member); // 엔티티 삭제
```
엔티티를 삭제하려면 삭제 대상 엔티티를 조회하고 엔티티 매니저에 삭제 대상 엔티티를 넘겨주면 해당 엔티티를
삭제한다. 엔티티 등록과 동일하게 삭제 쿼리를 쓰기 지연 SQL 저장소에 등록한 뒤 트랜잭션을 커밋해서 플러시를
호출하면 DB에 삭제쿼리를 전달한다.

## 4. 플러시

**플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다** 플러시를 실행하면 아래와 같은 일이 일어난다.

1. 변경 감지가 동작해서 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해서 수정된 엔티티를 찾는다. 수정된
엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.
2. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송한다.

영속성 컨텍스트를 플러시하는 방법은 아래와 같이 3가지 방법이 있다.

### 4.1 직접호출

`entityManager.flush()`메서드 직접 호출해서 영속성 컨텍스트를 강제로 플러시 한다. 테스트 또는 다른
프레임워크랑 함께 JPA를 사용할 때를 제외하고 거의 쓰이지 않는다.

### 4.2 트랜잭션 커밋 시 플러시 자동 호출

DB에 변경 내용을 SQL로 전달하지 않고 트랜잭션만 커밋하면 어떤 데이터도 DB 반영되지 않는다. 그래서 트랜잭션을
커밋하기 전에 플러시를 호출해서 영속성 컨텍스트의 변경 내용을 DB에 반영해야 한다. JPA는 이러한 문제를 예방하기
위해 트랜잭션을 커밋할 때 플러시를 자동 호출한다.

### 4.3 JPQL 쿼리 실행시 플러시 자동 호출

```java
// 객체 memberA, memberB, memberC 생성
// 엔티티 매니저 생성, 트랜잭션 획득 ...

entityManager.persist(memberA); // 등록
entityManager.persist(memberB); // 등록
entityManager.persist(memberC); // 등록

// JPQL 실행
List<Member> members = entityManager.createQuery("SELECT m FROM Member m", Member.class).getResultList();

// 트랜잭션 커밋 ...
```

JPQL이나 Criteria 같은 객체지향 쿼리를 호출할 때도 플러시가 실행된다. `persist()`메서드를 호출해서
엔티티 `memberA`, `memberB`, `memberC`를 영속 상태로 만들고, 영속성 컨텍스트에 있지만 아직 DB에는
반영되지 않았다. 이 때 JPQL을 실행하면 SQL로 변환되어 데이터베이스에서 엔티티를 조회한다. 그런데 `memberA`,
`memberB`, `memberC`는 아직 데이터베이스에 없으므로 쿼리 결과로 조회가 되지 않는다. 따라서 쿼리를
실행하기 직전에 영속성 컨텍스트를 플러시해서 변경 내용을 DB에 반영해야만 한다.

JPA는 이런 문제를 예방하기 위해 JPQL을 실행할 때도 플러시를 자동 호출한다. 그래서 `memberA`,
`memberB`, `memberC`들도 쿼리 결과에 포함된다.

`find()`메서드의 경우는 플러시가 실행되지 않는다.

플러시는 영속성 컨텍스트에 보관된 엔티티를 지우는 것이 아니라 영속성 컨텍스트의 내용변경을 데이터베이스에
동기화하는 것이다.

## 5. 준영속

**영속성 컨텍스트가 관리하는 영속상태의 엔티티가 영속성 컨텍스트에서 분리된 것을 준영속 상태라고 한다.** 준영속
상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.

### 5.1 엔티티를 준영속 상태로 전환 : detach()

```java
// 영속 상태
entityManager.persist(member);

// 회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
entityManager.detach(member);

// 태랜잭션 커밋 ...
```

`detach()`메서드는 특정 엔티티를 준영속 상태로 만드는데 위의 코드를 보면 먼저 회원 엔티티를 영속화한
다음 `detach()`메서드를 호출했다. 영속성 컨텍스트에게 해당 엔티티를 관리하지 말라는 것이다. 이 메서드를
호출하면 1차 캐시, 쓰기 지연 SQL 저장소까지 해당 엔티티를 관리하기 위한 정보가 모두 제거된다.

### 5.2 영속성 컨텍스트 초기화 : clear()

```java
// 엔티티 조회, 영속 상태
Member member = entityManager.find(Member.class, "id001");

// 영속성 컨텍스트 초기화
entityManager.clear();

// 준영속 상태
member.setAge("28"); // 변경 감지는 독장 하지 않기 때문에 변경되지 않는다.
```

`clear()`메서드는 영속성 컨텍스트를 초기화해서 해당 영속성 컨텍스트의 모든 엔티티를 준영속 상태로 만든다.
영속성 컨텍스트를 제거하고 새로 만든것과 같다.

### 5.3 영속성 컨텍스트 종료 : close()

```java
// 엔티티 매니저 생성, 트랜잭션 획득 ...

transaction.begin(); // 트랜잭션 시작

Member memberA = entityManager.find(Member.class, "id001");
Member memberB = entityManager.find(Member.class, "id002");

transaction.commit(); // 트랜잭션 커밋

entityManager.close(); // 영속성 컨텍스트 닫기
```

영속성 컨텍스트트 종료하면 해당 영속성 컨텍스트가 관리하던 영속 상태의 엔티티가 모두 준영속 상태가 된다.

### 5.4 준영속 상태의 특징

준영속 상태의 엔티티의 특징은 아래와 같다.

- 거의 비영속 상태에 가깝기 때문에 영속성 컨텍스트가 제공하는 어떤 기능도 동작하지 않는다.
- 비영속 상태는 식별값이 없지만 준영속 상태는 이미 영속 상태였기 때문에 식별자 값을 가지고 있다.
- 지연로딩을 할 수 없다.(추후 정리예정)

### 5.5 병합 : merge()

준영속 상태의 엔티티를 다시 영속 상태로 변경하려면 병합을 사용하면 된다. `merge()`메서드는 준영속 상태의
엔티티를 받아서 그 정보로 새로운 영속 상태의 엔티티를 반환한다.

```java
public class ExamMergeMain {

    // 엔티티 매니저 팩토리 생성
    private static EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("jpa03");

    public static void main(String[] args) {

        // 회원 엔티티 생성 --> 준영속 상태
        Member member = createMember("id001", "doubles", 20);

        // 준영속 상태 --> 회원 정보 변경 : 변경이 이루어지지 않음
        member.setUsername("더블에스");

        // 회원 엔티티 병합
        mergeMember(member);
    }

    // 회원 엔티티 생성 --> 영속성 컨텍스트 종료
    private static Member createMember(String id, String username, int age) {

        // 엔티티 매니저 생성
        EntityManager entityManager1 = entityManagerFactory.createEntityManager();

        // 트랜잭션 획득
        EntityTransaction transaction1 = entityManager1.getTransaction();

        transaction1.begin();   // 트랜잭션 시작

        // 회원 객체 생성
        Member member = new Member();
        member.setId(id);
        member.setUsername(username);
        member.setAge(age);

        // 회원 엔티티 등록
        entityManager1.persist(member);

        transaction1.commit();  // 트랜잭션 커밋

        // 영속성 컨텍스트 종료 ---> 회원 엔티티 준영속 상태로 변경됨
        entityManager1.close();

        return member;
    }

    // 병합
    private static void mergeMember(Member member) {

        // 엔티티 매니저 생성
        EntityManager entityManager2 = entityManagerFactory.createEntityManager();

        // 트랜잭션 획득
        EntityTransaction transaction2 = entityManager2.getTransaction();

        transaction2.begin(); // 트랜잭션 시작

        // 회원 엔티티 병합 ---> 회원 엔티티 변경 감지 DB 반영
        Member mergeMember = entityManager2.merge(member);

        transaction2.commit(); // 트랜잭션 커밋

        System.out.println("member = " + member.getUsername()); // 준영속 상태

        System.out.println("mergeMember = " + mergeMember.getUsername()); // 영속 상태

        System.out.println("entityManager2 contains member = " + entityManager2.contains(member));

        System.out.println("entityManager2 contains mergeMember = " + entityManager2.contains(mergeMember));

        entityManager2.close();
    }

}
```

위의 코드는 준영속 상태의 엔티티가 어떻게 영속상태로 변경되는지 알아보기 위해 작성된 것으로 큰 흐름은 아래와 같다.

1. `createMember()`메서드에서 회원 엔티티를 생성하고, 등록한 뒤에 영속성 컨텍스트를 종료시켜 준영속 상태로 만든다.
2. 준영속 상태의 회원 엔티티를 변경하면 변경사항이 적용되지 않는다.
3. `mergeMember()`매서드에서 회원 엔티티를 병합하고, 변경사항을 감지하여 DB에 동기화한다.
4. 반환된 `mergeMember`와 `member`가 영속성 컨텍스트에 존재하는지 `contains()`메서드를 통해 확인한다.

위 코드를 실행하고 나면 콘솔에서 아래와 같이 결과를 확인할 수 있다.

```console
Hibernate:
    /* insert com.doubles.jpa03.Member
        */ insert
        into
            MEMBER
            (age, NAME, ID)
        values
            (?, ?, ?)
Hibernate:
    /* load com.doubles.jpa03.Member */ select
        member0_.ID as ID1_0_0_,
        member0_.age as age2_0_0_,
        member0_.NAME as NAME3_0_0_
    from
        MEMBER member0_
    where
        member0_.ID=?
Hibernate:
    /* update
        com.doubles.jpa03.Member */ update
            MEMBER
        set
            age=?,
            NAME=?
        where
            ID=?
member = 더블에스
mergeMember = 더블에스
entityManager2 contains member = false
entityManager2 contains mergeMember = true
```

콘솔 화면을 통해 확인할 수 있는 것은 `merge()`메서드를 통해 반환 받은 `mergeMember`와 `member`는
다른 인스턴스라는 것을 알 수 있고, `mergeMember`는 영속상태이고, `member`는 여전히 준영속 상태라는 것을
알 수 있다. 이렇게 준영속 엔티티를 참조하던 변수는 영속 엔티티를 참조하도록 변경하는 것이 안전하고 바람직하다.

```java
//Member mergeMember = entityManager2.merge(member); 코드 변경
member = entityManager2.merge(member); // 참조를 변경
```

`merge()`메서드는 비영속 엔티티도 영속상태로 만들 수도 있다.

```java
Member member = new Member(); // 비영속 상태
Member newMember = entityManager.merge(member); // 엔티티 병합
```

병합은 파라미터로 넘어온 엔티티의 식별자로 영속성 컨텍스트를 조회하고 없을 경우 DB를 조회한다. 만약 DB에도
존재하지 않는다면 새로운 엔티티를 생성해서 병합을 하게 된다.

병합은 준영속, 비영속을 가리지 않고 식별자로 조회가 가능하면 불러와 병합하고 없다면 새로 생성해서 병합한다.


## 6. Summery / Conclusion

- 영속성 컨텍스트는 엔티티를 영구히 저장하는 환경
- 엔티티 생명주기
  - 비영속 : 관계 없음
  - 영속  : 저장된 상태, `persist()`
  - 준영속 : 분리된 상태, `detach()`
  - 삭제  : 삭제된 상태, `remove()`
- 영속성 컨텍스트의 장점과 특징
  - 1차캐시 : 영속성 컨텍스트에 존재하는 일종의 가상DB, Map처럼 key와 value를 가짐
  - 동일성 보장 : 같은 엔티티를 조회할 경우, 컨텍스트에 존재하는 엔티티를 바로 반환, 속도에 이점
  - 쓰기지연 : SQL 내부저장소에 모아두었다가 트랜잭션 커밋할 때 DB에 전달
  - 변경감지 : 엔티티의 변경사항을 스냅샷을 통해 감지하고, DB에 동기화, UPDATE SQL은 모든필드를 수정하는 SQL을 사용
  - 지연로딩 :
- 플러시 : 트랜잭션 커밋할 때 영속성 컨텍스트가 플러시되고 DB에 변경사항을 동기화
- 준영속 상태에서는 영속성 컨텍스트가 제공하는 기능을 사용하지 못함
