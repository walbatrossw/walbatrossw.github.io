---
layout: article
title: Design Pattern - Observer Pattern(옵저버 패턴) # 제목
date: 2018-08-28 03:00:00 # 작성 시간 2019-00-00 00:00:00
category: oop # 카테고리
tags: oop design-pattern observer-pattern # 태그
key: 20180828d # for gitalk
---

<!--more-->

## 1. 옵서버 패턴이란?

**옵서버 패턴은 데이터의 변경이 발생했을 경우 상대 클래스나 객체에 의존하지 않으면서 데이터 변경을 통보하고자 할 때 유용하다.**

예를 들어 새로운 파일이 추가되거나, 기존 파일이 삭제되었을 때 탐색기는 이를 즉시 표시할 필요가 있다. 탐색기를 복수 개 실행하는 상황이나 하나의 탐색기에서 시스템을
변경하였을 때는 다른 탐색기에게 즉각적으로 이를 통보해야한다.

옵서버 패턴은 통보 대상 객체의 관리를 `Subject`클래스와 `Observer`인터페이스로 일반화한다. 그러면 데이터 변경을 통보하는 클래스(`ConcreteSubject`)는 통보 대상
클래스나 객체(`ConcreteObserver`)에 대한 의존성을 없앨 수 있다. 결과적으로 옵서버 패턴은 통보 대상 클래스나 대상 객체의 변경에도 `ConcreteSubject`클래스를 수정없이
그대로 사용할 수 있도록 한다.

![observer-pattern-collaboration](http://www.plantuml.com/plantuml/png/ZP112eCm44NtESKijIKUGAGR3z229uXnR5CaASaebDgxDw2n6g4qgmp__ydx8KKTmhbsQ0UqS154Q3MKfkj4RQmWPJHZiXFEGTNNTvG4BmR-7C5xXRmb9eJpPK_gxuDHpMniy4-ZDgJQPn1TiiXlnfJsPAsGAze0qZyQGRXnJyIbqT8YuPQSWvlcYdpCXwhLEAzqE8t93si4VmsHD5wvu_cdGM0kHixZ8el8Jz_Vo7spyCKE3YmkN_3z-pbm6QrUj6BCibs2VG00)

옵서버 패턴에서 나타나는 역할이 수행하는 작업은 아래와 같다.

- `Observer` : 데이터의 변경을 통보 받는 인터페이스, 즉 `Subject`에서는 `Observer`인터페이스의 `update()`메서드를 호출함으로써 `ConcreteSubject`의 데이터 변경을 `ConcreteObserver`에게 통보한다.
- `Subject` : `ConcreteObserver`객체를 관리하는 요소, `Observer`인터페이스를 참조해서 `ConcreteObserver`를 관리하므로 `ConcreteObserver`의 변화에 독립적일 수 있다.
- `ConcreteSubject` : 변경관리 대상이 되는 데이터가 있는 클래스로 데이터 변경을 위한 `setState()`메서드가 있으며, `setState()`에서는 자신의 데이터인 `subjectState()`를 변경하고, `Subject`의 `notifyObservers()`메서드를 호출해서 `ConcreteObserver`객체에 변경을 통보한다.
- `ConcreteObserver` : `ConcreteSubject`의 변경을 통보받는 클래스로 `Observer`인터페이스의 `update()`메서드를 구현함으로써 변경을 통보받는다. 변경된 데이터는 `ConcreteSubject`의 `getState()`메서드를 호출함으로써 변경을 조회한다.

아래는 옵서버 패턴의 순차 다이어그램이다.

![observer-pattern-sequence](http://www.plantuml.com/plantuml/png/hPAz3i8W58LtdeAn7Jh8Q8oB3-3G9-3dLZ4H6hWq-lQscce350STB-VoSSu9Pws0TjQYi3V20reJhW8SwRb3BNNF3TA3DT81GXl41IHcjQFmu0PmcwBAICIYkoP5q2trW2roXCg_zfx3UDukndgS1wiLuHBrCEcnqVYnPX-lZ0XZKNVt-N5V_OHTPWYzcXAyUGfMYOdmgJCbPrESviBggANr60K_6IbrL8ZFCl5NYOoarxzdN7wCab_shm9cwG40)

`ConcreteSubject`가 자신의 상태, 즉 데이터 변경을 통보하려면 `ConcreteObserver`가 미리 등록되어 있어야 하는데 위의 순차 다이어그램을 살펴보면 `ConcreteSubject`에 `ConcreteObserver1`과 `ConcreteObserver2`가 등록되어 있는 상태이다.
1. 이때 `ConcreteObserver1` `ConcreteSubject`의 상태를 변경하면 `ConcreteSubject`는 등록된 모든 `ConcreteObserver`에게 자신이 변경되었음을 통보한다.
2. 변경통보는 실제로 `ConcreteSubject`의 상위 클래스인 `Subject`클래스의 `notifyObservers()` 메서드를 호출해 이루어진다.
3. 그러면 `notifyObservers()`메서드는 등록된 각 `ConcreteObserver`의 `update()`메서드를 호출한다.
4. 마지막으로 통보받은 `ConcreteObserver1`과 `ConcreteObserver2`는 `ConcreteSubject`의 `getState()`메서드를 호출함으로써 변경된 상태나 데이터를 구한다.

## 2. 옵서버 패턴 예제 : 성적 출력 기능

성적을 출력하는 기능을 구현하면서 옵서버 패턴을 이해해보자.

### 2.1 성적출력기능

아래는 성적을 출력하는 기능을 구현하기 위해 필요한 클래스 다이어그램과 설명이다.

![observer-pattern-score-class-diagram](http://www.plantuml.com/plantuml/png/PP2x3i8m34NtV8N7LD0Vg10BB4XCL6AF6gj4GWcgtJ8W_XqtVIWf6H8vru_ZspmD4THDwF26SbluA91J0er_11LS7V7XuXbRx8vHt04XdmsR-e78TuTlRTD8YZArcAkCjN6IZhqbotakv1c2itDAYvp0wE9l_bUf9Z9d4rRgGO9Jw3rFtUUkEIibsrRfxljj-GBUlsG_r2khGAIMbHkOQh5lqbMkbfdTOsvcdle1)

- `ScoreRecord` 클래스는 입력된 점수를 저장하는 역할을 수행한다.
- `DataSheetView` 클래스는 입력된 점수를 목록형태로 출력하는 역할을 수행한다.
- `ScoreRecord` 클래스의 `addScore()`메서드가 실행될 때 성적을 출력하려면 `ScoreRecord`클래스는 `DataSheetView`클래스의 객체를 참조해야한다.

아래의 그림은 `ScoreRecord`와 `DataSheetView` 클래스 사이의 상호작용을 순차다이어그램으로 표현한 것이다.

![observer-pattern-score-sequence](http://www.plantuml.com/plantuml/png/ZP313e8m44Jl_OeUCP4VC8OBNamyIU9zeIjiWYqfMzI_jsBKLY3nqYRJpipRRKYXF1l3fRa9S6oqkvHeXUZ0CbNKWMQPjuIQ8wceZCKZ-bD5-WuOYWQHJuHN8LvEcQPPw90R2KgDGZAUNY3DAtyDXfGGK34Dm1ZLX07FmAZAMrsdl2Nvf2YSZVc6nwnnt9IuHWw4iUP0FM_tch56cyr3Bq1CotwdKTHtBTn7Kv_ODqLKkPQ3_vmRvrSSRt1Xne3cpuS7)

- 점수가 추가되면, 즉 다시말해 `ScoreRecord`클래스의 `addScore()`메서드가 호출되면 `ScoreRecord`클래스는 자신의 필드인 `scores`객체에 점수를 추가한다.
- 그리고 `DataSheetView`클래스의 `update()`메서드를 호출함으로써 성적을 출력하도록 요청한다.
- `DataSheetView`클래스는 `ScoreRecord`클래스의 `getScoreRecord()`메서드를 호출해 출력할 점수를 구한다.
- 이때 `DataSheetView`클래스의 `update()`메서드에서는 구한 점수 중에서 명시된 개수만큼 점수만 출력한다.

위에서 설명한 것을 바탕으로 작성된 코드는 아래와 같다.

```java
public class ScoreRecord {

    private List<Integer> scores = new ArrayList<>();   // 점수 저장
    private DataSheetView dataSheetView;                // 목록 형태로 점수를 출력하는 클래스 참조 변수

    public void setDataSheetView(DataSheetView dataSheetView) {
        this.dataSheetView = dataSheetView;
    }

    // 새로운 점수 추가
    public void addScore(int score) {
        scores.add(score);  // scores 목록에 주어진 점수를 추가
        dataSheetView.update(); // scores 변경 통보
    }

    // 점수 목록 가져오기
    public List<Integer> getScoreRecord() {
        return scores;
    }

}
```

```java
public class DataSheetView {

    private ScoreRecord scoreRecord;    // 점수 저장 클래스 참조변수
    private int viewCount;              // 저장된 점수의 갯수

    // 생성자
    public DataSheetView(ScoreRecord scoreRecord, int viewCount) {
        this.scoreRecord = scoreRecord;
        this.viewCount = viewCount;
    }

    // 점수의 변경을 통보받아 갱신하는 메서드
    public void update() {
        List<Integer> record = scoreRecord.getScoreRecord(); // 점수 조회
        displayScores(record, viewCount);
    }

    // 점수 출력 메서드
    private void displayScores(List<Integer> record, int viewCount) {
        System.out.println("List of " + viewCount + " entries ");
        for (int i = 0; i < viewCount &&  i < record.size(); i++) {
            System.out.println(record.get(i));
        }
        System.out.println();
    }

}
```

```java
public class Client {
    public static void main(String[] args) {

        ScoreRecord scoreRecord = new ScoreRecord();    // 점수 저장 객체 생성
        DataSheetView dataSheetView = new DataSheetView(scoreRecord, 3);    // 3개까지 점수만 출력

        scoreRecord.setDataSheetView(dataSheetView);    // 점수 시트 설정

        for (int i = 1; i <= 5; i++) {
            int score = i * 10;
            System.out.println("adding " + score);  // 점수추가
            scoreRecord.addScore(score);    // 저장된 점수목록 출력
        }

    }
}
```

```
adding 10
List of 3 entries
10

adding 20
List of 3 entries
10
20

adding 30
List of 3 entries
10
20
30

adding 40
List of 3 entries
10
20
30

adding 50
List of 3 entries
10
20
30
```

### 2.2 성적출력기능의 문제점

성적 출력 기능을 위와 같이 구현했지만 만약 기능이 추가되거나, 요구사항이 바뀐다면 어떻게 될까? 구체적인 변경사항이나 문제점에 대해 알아보자.

- 성적을 다른형태로 출력하고 싶다면? 예를 들어서 성적을 목록형태가 아닌 최소/최대로 출력하려면?
- 여러가지 형태의 성적을 동시 혹은 순차적으로 출력하려면? 예를 들어서 성적이 입력되었을 때 최대 3개, 5개, 최소/최대 값을 동시에 출력하거나 처음에는 목록으로 출력하고, 나중에는 최소/최대 값을 출력하려면?

일단 먼저 성적을 다른 형태로 출력하는 경우에 대해 알아보자.

점수 목록 출력대신 최소/최대 값만을 출력하려면 기존의 `DataSheetView`클래스 대신 `MinMaxView`클래스를 추가할 필요가 있다. 그리고 `ScoreRecord` 클래스는 `DataSheetView` 클래스가 아니라 `MinMaxView`
클래스에 성적 변경을 통보할 필요가 있다.

아래는 위에서 설명한 것과 같이 성적의 최소/최대 값을 출력하는 코드이다.

```java
public class ScoreRecord {

    private List<Integer> scores = new ArrayList<>();   // 점수 저장
    private MinMaxView minMaxView;  // MinMaxView 클래스 객체 참조 변수

    // MinMaxView 설정 추가
    public void setMinMaxView(MinMaxView minMaxView) {
        this.minMaxView = minMaxView;
    }

    // 새로운 점수 추가
    public void addScore(int score) {
        scores.add(score);  // scores 목록에 주어진 점수를 추가
        //dataSheetView.update(); // scores 변경 통보
        minMaxView.update();    // scores 변경 통보 변경
    }

    // 점수 목록 가져오기
    public List<Integer> getScoreRecord() {
        return scores;
    }

}
```

```java
public class MinMaxView {

    private ScoreRecord scoreRecord; // 점수 저장 객체 참조 변수

    public MinMaxView(ScoreRecord scoreRecord) {
        this.scoreRecord = scoreRecord;
    }

    public void update() {
        List<Integer> record = scoreRecord.getScoreRecord(); // 점수 조회
        displayMinMax(record);  // 최소 / 최대 값 출력
    }

    private void displayMinMax(List<Integer> record) {
        int min = Collections.min(record, null);
        int max = Collections.max(record, null);
        System.out.println("Min : " + min + ", Max : " + max);
    }
}
```

```java
public class Client {
    public static void main(String[] args) {

        ScoreRecord scoreRecord = new ScoreRecord();    // 점수 저장 객체 생성
        MinMaxView minMaxView = new MinMaxView(scoreRecord);

        scoreRecord.setMinMaxView(minMaxView);

        for (int i = 1; i <= 5; i++) {
            int score = i * 10;
            System.out.println("adding " + score);  // 점수추가
            scoreRecord.addScore(score);    // 저장된 점수목록 출력
        }
    }
}
```

```
adding 10
Min : 10, Max : 10
adding 20
Min : 10, Max : 20
adding 30
Min : 10, Max : 30
adding 40
Min : 10, Max : 40
adding 50
Min : 10, Max : 50
```

코드는 원하던 결과대로 출력이 되었다. 하지만 이는 OCP법칙에 위반된다. 그 이유는 점수가 입력되었을 때 지정된 특정 대상 클래스(`DataSheetView`)에게 고정적으로 통보하도록 코드가 짜여있었는데 다른 대상 클래스(`MinMaxView`)에게 점수가 입력되었음을
통보하려면 `ScoreRecord` 클래스의 변경이 불가피하기 때문이다.

이제 두번째 변경된 요구사항에 대해서 알아보자. 성적이 입력되었을 때 최대 3개 목록, 최대 5개의 목록, 최소/최대 값을 동시에 출력하거나 처음에는 목록으로 출력하고, 나중에는 최소/최대 값을 출력하려면 어떻게 해야할지 알아보자.

목록을 출력하는 것은 `DataSheetView` 클래스를 활용하고, 최소/최대 값을 출력하는 것은 `MinMaxView` 클래스를 활용할 수 있다. 그래서 `ScoreRecord` 클래스는 2개의 `DataSheetView` 객체(3개, 5개의 목록)와 1개의 `MinMaxView` 객체에게
성적 추가를 통보할 필요가 있다. 이를 위해 `ScoreRecord` 클래스는 다시 변경되어야한다.

아래의 코드는 위의 설명을 반영한 내용으로 작성되었다.

```java
public class ScoreRecord {

    private List<Integer> scores = new ArrayList<>();                   // 점수 저장
    private List<DataSheetView> dataSheetViews = new ArrayList<>();     // 목록 형태로 점수 출력 리스트 참조변수
    private MinMaxView minMaxView;                                      // 최소/최대값 출력 참조변수

    public void addDataSheetView(DataSheetView dataSheetView) {
        dataSheetViews.add(dataSheetView);
    }

    // MinMaxView 설정 추가
    public void setMinMaxView(MinMaxView minMaxView) {
        this.minMaxView = minMaxView;
    }

    // 새로운 점수 추가
    public void addScore(int score) {
        scores.add(score);  // scores 목록에 주어진 점수를 추가
        // 각 dataSheetView에 값의 변경을 통보
        for (DataSheetView dataSheetView : dataSheetViews) {
            dataSheetView.update();
        }
        minMaxView.update();    // MinMaxView에 값의 변경 통보 변경
    }

    // 점수 목록 가져오기
    public List<Integer> getScoreRecord() {
        return scores;
    }

}
```

```java
public class MinMaxView {

    private ScoreRecord scoreRecord;

    public MinMaxView(ScoreRecord scoreRecord) {
        this.scoreRecord = scoreRecord;
    }

    public void update() {
        List<Integer> record = scoreRecord.getScoreRecord();
        displayMinMax(record);
    }

    private void displayMinMax(List<Integer> record) {
        int min = Collections.min(record, null);
        int max = Collections.max(record, null);
        System.out.println("Min : " + min + ", Max : " + max);
        System.out.println();
        System.out.println("===============================");
    }
}
```

```java
public class DataSheetView {

    private ScoreRecord scoreRecord;    // 점수 저장 클래스 참조변수
    private int viewCount;              // 저장된 점수의 갯수

    // 생성자
    public DataSheetView(ScoreRecord scoreRecord, int viewCount) {
        this.scoreRecord = scoreRecord;
        this.viewCount = viewCount;
    }

    // 점수의 변경을 통보받아 갱신하는 메서드
    public void update() {
        List<Integer> record = scoreRecord.getScoreRecord(); // 점수 조회
        displayScores(record, viewCount);
    }

    // 점수 출력 메서드
    private void displayScores(List<Integer> record, int viewCount) {
        System.out.println("List of " + viewCount + " entries ");
        for (int i = 0; i < viewCount &&  i < record.size(); i++) {
            System.out.println(record.get(i));
        }
        System.out.println();
    }

}
```

```java
public class Client {
    public static void main(String[] args) {

        ScoreRecord scoreRecord = new ScoreRecord();    // 점수 저장 객체 생성

        DataSheetView dataSheetView3 = new DataSheetView(scoreRecord, 3);

        DataSheetView dataSheetView5 = new DataSheetView(scoreRecord, 5);
        MinMaxView minMaxView = new MinMaxView(scoreRecord);

        scoreRecord.addDataSheetView(dataSheetView3);
        scoreRecord.addDataSheetView(dataSheetView5);
        scoreRecord.setMinMaxView(minMaxView);

        for (int i = 1; i <= 5; i++) {
            int score = i * 10;
            System.out.println("adding " + score);
            scoreRecord.addScore(score);
        }
    }
}
```

```
adding 10
List of 3 entries
10

List of 5 entries
10

Min : 10, Max : 10

===============================
adding 20
List of 3 entries
10
20

List of 5 entries
10
20

Min : 10, Max : 20

===============================
adding 30
List of 3 entries
10
20
30

List of 5 entries
10
20
30

Min : 10, Max : 30

===============================
adding 40
List of 3 entries
10
20
30

List of 5 entries
10
20
30
40

Min : 10, Max : 40

===============================
adding 50
List of 3 entries
10
20
30

List of 5 entries
10
20
30
40
50

Min : 10, Max : 50

===============================
```

일단은 요구사항의 변경에 따라 코드를 변경하여 원하는 결과를 얻을 수 있었다. 하지만 성적의 통보 대상이 변경된 것을 반영하려고 `ScoreRecord` 클래스의 코드를 수정하게 됨에 따라 이것도 역시 OCP를 위배하게 되었다.
이러한 문제를 옵서버 패턴을 통해 해결하기 위한 해결책을 아래에서 알아보자.

### 2.3 해결책

문제 해결의 핵심은 성적 통보 대상이 변경되더라도 `ScoreRecord` 클래스를 그대로 재사용할 수 있어야한다는 점이다. 따라서 `ScoreRecord` 클래스에서 변화되는 부분을 식별하고 일반화시켜야 한다.

`ScoreRecord` 클래스에서는 통보 대상인 객체를 참조하는 것을 관리해야하며 `addScore()`메서드는 통보 대상인 객체의 `update()` 메서드를 호출할 필요가 있다. 이런 통보 대상 객체의 관리와 각 겍체에 `update()`메서드를 호출하는 기능은
성적 변경뿐만 아니라 임의의 데이터가 변경되었을 때 이에 관심을 가지는 모든 대상 객체에 통보하는 경우에도 동일하게 발생하는 기능이다. 따라서 이러한 공통 기능을 상위 클래스 및 인터페이스로 일반화하고, 이를 활용해 `ScoreRecord`를 구현하는
방식으로 설계를 변경하는 편이 좋다.

아래는 `DataSheetView`와 `MinMaxView` 클래스에게 성적 변경을 통보할 수 있도록 개선한 `ScoreRecord` 클래스 다이어그램과 이에 대한 설명이다.

![observer-pattern-enhanced-score-class-diagram](http://www.plantuml.com/plantuml/png/ZP91RiCW44NtFWLBfouvW6Lbqsug9QfKIzqJc2HK4OvWx2HgUlTE3DZOKfMoG3Rp-V_188_2ELQtso-GoBupHgDW0b78Gzvi7TWEB2lPU_XSq7VNQ1M42lufD0tgtJKMyw7waz6G7a8s5Zw0PZM2ADKlv-u-qoPjSEQy1qnszivhxR1wCmXxiAjxX0zu5IZg0m1-QZY72Cuw-dbfMeFUFHuirVhqW5Qce8jWehGx7SMrhxZSHkK4v7aUbLw29znein6N1Az8bXvH5Amz4JRaynvvDc_q1rkAGcHWTqB2qCi1PXv0wlXqgXldATDGuAlHfwIhYc_5iQTisX5EqdfoHiwICsV9xJoI3edLRRcSDdvM6qspNm00)

- `Subject`클래스는 성적변경에 관심이 있는 대상 객체를 관리하는 기능을 수행한다.
- `Subject`클래스는 `attach()`, `detach()`메서드로 성적 변경에 관심이 있는 대상 객체를 추가하거나, 제거한다.
- 성적변경의 통보 수신이라는 측면에서 `DataSheetView`, `MinMaxView`클래스는 동일하므로 `Subject`클래스는 `Observer` 인터페이스를 구현함으로써 성적 변경에 관심이 있음을 보여준다.
- `ScoreRecord`클래스의 `addScore()`메서드가 호출되면 자신의 성적 값을 저장한 후 `Subject`클래스의 `notifyObservers()`메서드를 호출해 `DataSheetView`, `MinMaxView`클래스에게 성적 변경을 통보한다.
- 이후 `Subject`클래스는 `Observer`인터페이스를 통해 `DataSheetView`, `MinMaxView` 객체의 `update()`메서드를 호출하게 된다.

아래의 코드는 위와 같은 방식으로 설계한 내용이다.

```java
// 추상화된 통보대상
public interface Observer {

    // 데이터의 변경을 통보했을 때 처리하는 메서드
    public abstract void update();

}
```

```java
// 추상화된 변경 관심 데이터
public abstract class Subject {

    // 추상화된 통보 대상 목록
    private List<Observer> observers = new ArrayList<>();

    // 옵서버(통보 대상) 추가
    public void attach(Observer observer) {
        observers.add(observer);
    }

    // 옵서버(통보 대상) 제거
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    // 통보대상 목록에서 각 옵서버에게 변경을 통보
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update();
        }
    }

}
```

```java
// 구체적인 변경 감시 대상 데이터
public class ScoreRecord extends Subject {

    private List<Integer> scores = new ArrayList<>();

    // 점수 추가
    public void addScore(int score) {
        scores.add(score);
        notifyObservers();  // 데이터의 변경을 각 옵서버에게 통지
    }

    public List<Integer> getScoreRecord() {
        return scores;
    }

}
```

```java
// 통보대상
public class DataSheetView implements Observer {

    private ScoreRecord scoreRecord;    // 점수 저장 클래스 참조변수
    private int viewCount;              // 저장된 점수의 갯수

    // 생성자
    public DataSheetView(ScoreRecord scoreRecord, int viewCount) {
        this.scoreRecord = scoreRecord;
        this.viewCount = viewCount;
    }

    // 점수의 변경을 통보받아 갱신하는 메서드
    @Override
    public void update() {
        List<Integer> record = scoreRecord.getScoreRecord(); // 점수 조회
        displayScores(record, viewCount);
    }

    // 점수 출력 메서드
    private void displayScores(List<Integer> record, int viewCount) {
        System.out.println("List of " + viewCount + " entries ");
        for (int i = 0; i < viewCount &&  i < record.size(); i++) {
            System.out.println(record.get(i));
        }
        System.out.println();
    }

}
```

```java
// 통보대상 클래스
public class MinMaxView implements Observer {

    private ScoreRecord scoreRecord;

    public MinMaxView(ScoreRecord scoreRecord) {
        this.scoreRecord = scoreRecord;
    }

    @Override
    public void update() {
        List<Integer> record = scoreRecord.getScoreRecord();
        displayMinMax(record);
    }

    private void displayMinMax(List<Integer> record) {
        int min = Collections.min(record, null);
        int max = Collections.max(record, null);
        System.out.println("Min : " + min + ", Max : " + max);
        System.out.println();
        System.out.println("==============================");
    }

}
```

```java
public class Client {
    public static void main(String[] args) {
        ScoreRecord scoreRecord = new ScoreRecord();
        DataSheetView dataSheetView3 = new DataSheetView(scoreRecord, 3);
        DataSheetView dataSheetView5 = new DataSheetView(scoreRecord, 5);
        MinMaxView minMaxView = new MinMaxView(scoreRecord);

        scoreRecord.attach(dataSheetView3);

        scoreRecord.attach(dataSheetView5);

        scoreRecord.attach(minMaxView);

        for (int i = 1 ; i <= 5 ; i ++ ) {
            int score = i * 10 ;
            System.out.println("adding " + score) ;
            scoreRecord.addScore(score) ;
        }
    }
}
```

```
adding 10
List of 3 entries
10

List of 5 entries
10

Min : 10, Max : 10

==============================
adding 20
List of 3 entries
10
20

List of 5 entries
10
20

Min : 10, Max : 20

==============================
adding 30
List of 3 entries
10
20
30

List of 5 entries
10
20
30

Min : 10, Max : 30

==============================
adding 40
List of 3 entries
10
20
30

List of 5 entries
10
20
30
40

Min : 10, Max : 40

==============================
adding 50
List of 3 entries
10
20
30

List of 5 entries
10
20
30
40
50

Min : 10, Max : 50

==============================
```

위와 같이 코드를 작성하게 되면 성적 변경에 관심이 있는 대상 객체들의 관리는 `Subject`클래스에서 구현하고 `ScoreRecord`클래스는 `Subject`클래스를 상속받게 함으로써 `ScoreRecord`클래스는 이제 `DataSheetView`와
`MinMaxView`를 직접 참조할 필요가 없게 되었다. 그러므로 `ScoreRecord`클래스의 코드를 변경하지 않고도 새로운 관심 클래스 및 객체를 추가/제거하는 것이 가능해졌다.

이제 추가적으로 합계/평균을 출력하는 `StatisticsView`클래스를 작성하고, 새로운 관심 객체를 추가 시켜보자.

```java
public class StatisticsView implements Observer {

    private ScoreRecord scoreRecord;

    public StatisticsView(ScoreRecord scoreRecord) {
        this.scoreRecord = scoreRecord;
    }

    @Override
    public void update() {
        List<Integer> record = scoreRecord.getScoreRecord();
        displayStatistics(record);
    }

    private void displayStatistics(List<Integer> record) {

        int sum = 0;
        for (int score : record) {
            sum += score;
        }
        float average = (float) sum / record.size();
        System.out.println("sum " + sum + ", average " + average);

    }
}
```

```java
public class Client {
    public static void main(String[] args) {

        ScoreRecord scoreRecord = new ScoreRecord();
        DataSheetView dataSheetView3 = new DataSheetView(scoreRecord, 3);
        scoreRecord.attach(dataSheetView3);
        MinMaxView minMaxView = new MinMaxView(scoreRecord);
        scoreRecord.attach(minMaxView);

        for (int i = 1; i <= 5; i++) {
            int score = i * 10;
            System.out.println("adding " + score);
            scoreRecord.addScore(score);
        }

        scoreRecord.detach(dataSheetView3);
        StatisticsView statisticsView = new StatisticsView(scoreRecord);
        scoreRecord.attach(statisticsView);

        for (int i = 1; i <= 5; i++) {
            int score = i * 10;
            System.out.println("adding " + score);
            scoreRecord.addScore(score);
        }
    }
}
```

```
adding 10
List of 3 entries
10

Min : 10, Max : 10

==============================
adding 20
List of 3 entries
10
20

Min : 10, Max : 20

==============================
adding 30
List of 3 entries
10
20
30

Min : 10, Max : 30

==============================
adding 40
List of 3 entries
10
20
30

Min : 10, Max : 40

==============================
adding 50
List of 3 entries
10
20
30

Min : 10, Max : 50

==============================
adding 10
Min : 10, Max : 50

==============================
sum 160, average 26.666666
adding 20
Min : 10, Max : 50

==============================
sum 180, average 25.714285
adding 30
Min : 10, Max : 50

==============================
sum 210, average 26.25
adding 40
Min : 10, Max : 50

==============================
sum 250, average 27.777779
adding 50
Min : 10, Max : 50

==============================
sum 300, average 30.0
```

이전 코드(`ScoreRecord`)를 수정하지 않고 클래스(`StatisticsView`) 하나만을 추가함으로써 합계/평균을 구할 수 있게 되었다.
