---
layout: article
title: 객체지향의 원리
category: oop
tags: OOP
key: 20180727b
---

<!--more-->

> 본 글은 [자바 객체지향과 디자인패턴](https://book.naver.com/bookdb/book_detail.nhn?bid=7467601)를
읽고 개인적으로 학습한 내용을 복습하기 위해 작성된 글로 내용상 오류가 있을 수 있습니다.
오류가 있다면 지적 부탁드리겠습니다.

## 1. 추상화

### 1.1 추상화란?

**추상화란 어떤 영역에서 필요로 하는 속성이나 행동을 추출하는 작업을 의미한다. 추상화를 통해 관심이 쏠리는 부분에 더욱 집중을 할 수 있다.**

### 1.2 추상화의 예

예를 들어 넓은 주차장에 수많은 자동차들이 주차되어있다고 가정해보자. 이 자동차들을 그룹화할 때 추상화 개념을 사용할 수 있다.
어떤 사람은 사람이 탈 수 있는 승객의 수를 기준으로 승합차와 승용차로 그룹화 할 수 있고, 또다른 어떤 사람은 문의 개수에 따라 세단과 쿠페로 그룹화 할 수 있다.

- 자동차 승객 수에 따른 분류
    - 승용차
    - 승합차

- 자동차 문의 수에 따른 분류
    - 세단
    - 쿠페

위에서 예로 든 것처럼 **구체적인 사물들의 공통적인 특징을 파악해서 이를 하나의 개념 집합으로 다루는 수단이 추상화다.**
추상화는 객체지향 프로그래밍에서 매우 중요한데 만약 추상화가 없다면 각각의 개체를 각각 구분해야할 것이다.

예를 들어 자동차마다 엔진오일을 교환하는 방식이 다르다고 가정해보자. 그렇다면 아래와 같은 코드가 작성될 것이다.

```
switch(자동차 종류)
    case 아우디: break;    // 아우디 엔진오일을 교환하는 과정
    case 벤츠: break;     // 벤츠 엔진오일을 교환하는 과정
```

이때 만약 BMW와 같은 새로운 종류의 엔진오일을 교환하는 기능을 추가하라는 요구사항이 있을 경우 아래와 같이 코드가 변경될 것이다.

```
switch(자동차 종류)
    case 아우디: break;    // 아우디 엔진오일을 교환하는 과정
    case 벤츠: break;     // 벤츠 엔진오일을 교환하는 과정
    case BMW: break;     // BMW 엔진오일을 교환하는 과정
```

위 코드의 문제점은 새로운 자동차가 추가될 때마다 새로운 코드를 작성해주어야하는 불편함이 발생한다. 이러한 불편함을 해결할 방법은 아래와 같다.

아우디, 벤츠, BMW와 같은 구체적인 자동차 대신 이들의 추상화 개념인 자동차를 이용할 경우 아래와 같이 코드를 작성할 수 있다.

```
void changeEngineOil(Car car) {
    car.changeEngineOil();
}
```


프로시저 `changeEngineOil`의 인자로 아우디, 벤츠의 추상개념인 Car를 사용하는데 인자 어느 곳에서도 구체적인 자동차 종류와 연관된 부분을 찾을 수 없게 되었다.
따라서 이 코드는 어떤 새로운 자동차가 추가되더라도 코드가 변경될 일이 없어진다.
대신 인자 `car`가 가리키는 구체적인 자동차의 종류에 따라 `changeEngineOil`메서드가 다르게 실행되어야한다. 이는 일반화 관계에서 설명할 다형성 원리에 따르게 된다.

**추상화를 통해 각 개체의 구체적인 개념에 의존하지 말고 추상적인 개념에 의존해야 설계를 유연하게 변경할 수 있다는 점에 주목하자.**

## 2. 캡슐화

### 1.1 캡슐화란?

소프트웨어 개발을 할 때 요구사항 변경에 유연하게 대처하기 위해서는 높은 응집도와 낮은 결합도를 유지하는 것이 중요하다.
- 응집도 : 클래스나 모듈 안의 요소들이 얼마나 밀접하게 관련되어 있는지를 나타낸다.
- 결합도 : 어떤 기능을 실행하는데 다른 클래스나 모듈들에 얼마나 의존적인지 나타낸다.

캡슐화는 특히 낮은 결합도를 유지할 수 있도록 해주는 객체지향의 원리이다. 캡슐화는 정보은닉을 통해 높은 응집도와 낮은 결합도를 갖도록 한다.
- 정보은닉 : 알 필요가 없는 정보는 외부에서 접근하지 못하도록 제한하는 것을 말한다.

예를 들어 자동차의 가속 페달을 밟았을 때 어떤 과정을 거쳐 속도가 올라가는지 모르더라도 운전하는데 전혀 지장이 없다. 세탁기도 역시 어떤과정을 거쳐 세탁기의 드럼이 동작하는지 모르더라도 옷을 세탁하는데 전혀 지장이 없다.

그렇다면 정보은닉은 왜 필요한 것인가?

소프트웨어는 결합이 많을수록 문제가 많이 발생하게 된다. 한 클래스가 변경이 발생하면 변경된 클래스의 비밀에 의존하는 다른 클래스도 변경해야할 가능성이 커지는 것이다.
그렇기 때문에 정보은닉을 통해 클래스간의 결합도를 낮추고, 응집도를 높여 변경되는 사항에 유연하게 대처하는 것이 바람직하다.

### 1.2 캡슐화 예제

- 캡슐화 예제 1 : `ArrayStack`클래스와 `StackClient`클래스 간의 결합도 증가

- 캡슐화 예제 2 : `ArrayStack`클래스와 `StackClient`클래스 간의 결합도 감소, 응집도 증가

## 3. 일반화

### 3.1 일반화 관계
일반적으로 객체지향 개념에서 가장 많이 오해하고, 오용하는 것이 일반화 관계이다. 일반화 관계는 객체지향 프로그래밍 관점에서 상속관계라 한다.
따라서 속성이나 기능의 재사용만을 강조해서 사용하는 경우가 많다. 이는 일반화 관계에서 극히 한정되게 바라보는 시각이다.

철학에서 말하는 일반화는 "여러 개체들이 가진 공통된 특성을 부각시켜 하나의 개념이나 법칙으로 성립시키는 과정"이라고 한다.
예를 들어 설명해보면 사과, 바나나, 오렌지 등이 가진 공통된 개념은 과일이라고 할수 있다. 과일은 사과, 바나나, 오렌지 등이 가진 공통 개념을 일반화한 개념이며 사과, 바나나, 오렌지는 과일의 한 종류이기므로 과일을 특수화한 개념이다.
그런데 만약 이러한 일반화, 특수화 관계가 없다면 냉장고에 과일이 몇개있는지 질문하기 위해서는 하나의 과일마다 따로 질문을 해야만하는 상황이 올지도 모른다.

- 일반화 관계가 없다면?
    - 냉장고에 사과가 몇개 있지?
    - 그리고 냉장고에 바나나가 몇개 있지?
    - 그리고 또 배가 몇개 있지?
    - 오렌지는 또 몇개가 있지?
- 일반화 관계를 통해 하나의 질문으로 통합할 수 있다.
    - 냉장고에 과일은 몇개가 있지?

그렇다면 과일의 전체 가격을 구하는 코드를 작성해보자.

```
가격 총합 = 0
while(장바구니에 과일이 있다) {
    switch(과일 종류)
        case 사과:
            가격총합 = 가격총합 + 사과가격
        case 배:
            가격총합 = 가격총합 + 배가격
        case 바나나:
            가격총합 = 가격총합 + 바나나가격
}
```

만약 오렌지를 추가해야된다면? 아래의 코드처럼 변경될 것이다.

```
가격 총합 = 0
while(장바구니에 과일이 있다) {
    switch(과일 종류)
        case 사과:
            가격총합 = 가격총합 + 사과가격
        case 배:
            가격총합 = 가격총합 + 배가격
        case 바나나:
            가격총합 = 가격총합 + 바나나가격
        case 오렌지:
            가격총합 = 가격총합 + 오렌지가격
}
```

위와 같이 새로운 과일이 등장할 때마다 코드를 일일히 수정해줘야하는 번거로움이 발생하게되고 변경사항에 유연하게 대처가 어렵게된다.

그렇다면 어떻게 수정되어야할까? 아래의 코드를 통해 알아보자.

```
int computeTotalPrice(LinkedList<Fruit> fruit) {
    int total = 0;
    Iterator<Fruit> iterator = fruit.iterator();

    while(hasNext()) {
        Fruit currentFruit = iterator.next();
        total = total + currentFruit.calculatePrice();
    }
    return total;
}
```
이렇게 수정된다면 어떠한 과일이 추가되더라도 코드를 수정할 필요가 없어지게 된다. 여기서`calculatePrice()`는 실제 과일 객체의 종류에 따라 각가 다르게 실행하게 된다.
이것은 다형성에 따른 것인데 추후 다형성을 정리하면서 알아보자.

지금까지 정리한 내용을 통해서 알수 있는 것은 일반화 관계는 외부에 자식 클래스를 캡슐화하는 개념으로 볼 수 있다. 캡슐화의 개념이 단순히 한 클래스 안에 있는 속성과 연산들의 캡슐화에 한정되지 않고, 일반화를 통해 클래스 자체를 캡슐화하는 것으로 확장된다.
이러한 서브 클래스 캡슐화는 외부 클라이언트가 개별적인 클래스들과 무관하게 프로그래밍을 할 수 있게 해준다.


아래는 사람이 자동차를 사용하는 상황을 묘사한 것이다.
```
사람 ----------> 자동차 <----------- BMW, 소나타, 벤츠, 아우디
```

사람 클래스 관점에서는 구체적인 자동차의 종류가 숨겨져있는데 대리운전을 한다고 가정해보자. 대리운전자는 자동차 종류에 따라 운전에 영향을 받지 않는다.
이와 같이 새로운 자동차를 운전해야 하는 경우에도 사람 클래스는 영향을 받지 않게 된다.

**일반화 관계는 자식 클래스를 외부로부터 은닉하는 캡슐화의 일종이다**

### 3.2 일반화 관계와 위임
이전에도 말한 것처럼 많은 사람들이 일반화 관계(상속)를 기능의 상속, 즉 재사용을 위해 존재한다고 오해하고 있다. 하지만 이는 사실이 아니다.

예를 들어 보자. 만약 `ArrayList`를 상속받아 `Stack`클래스를 만든다면 아마 프로그래머의 의도는 `ArrayList`클래스에 정의된 `isEmpty()`, `size()`, `add()`, `remove()` 메서드를 자신이 구현하지 않고 그대로 사용하길 원했을 것이다.
기능의 재사용 측면에서는 성공적이라고 할 수 있게지만`ArrayList`클래스에 정의된 스택과 전혀관련 없는 수많은 연산이나 속성도 함께 상속 받게 된다.
실제로도 이런 불필요한 속성이나 연산은 도움이 되기보다는 물려받고 싶지 않은 빚이 될 가능성이 많다.

실제로 위의 사례와 같이 코드를 작성해보면 아래와 같다.
```java
// 일반화(상속)을 통한 기능 재사용
public class MyStack<String> extends ArrayList<String> {

    public void push(String element) {
        add(element);
    }

    public String pop() {
        return remove(size() - 1);
    }

}
```
```java
public class Main {
    public static void main(String[] args) {

        MyStack<String> myStack = new MyStack<>();

        myStack.push("A");
        myStack.push("B");
        myStack.push("C");
        myStack.add("C");   // 허용되어서는 안됨
        myStack.set(0, "D"); // 허용되어서는 안됨

        System.out.println(myStack.pop());
        System.out.println(myStack.pop());
        System.out.println(myStack.pop());

    }
}
```
위와 같이 작성됨에 따라 `push()`, `pop()`메서드를 통하지 않고 스택의 자료구조에 직접 접근할 수 있게 되었다.
하지만 이렇게 됨에 따라 스택의 무결성 조건인 LIFO에 위배된다.

기본적으로 일반화 관계는 "IS A KIND OF 관계"가 성립이 되어야만 한다. 그런 면에서 `MyStack`클래스와 `ArrayList`클래스의 관계가 참인지 판단을 통해 알 수 있다.

> Stack is a kind of ArrayList : 참이 아니다. 왜냐하면 대부분의 프로그램에서 배열목록 대신 스택을 사용할 수 없기 때문이다.

그렇다면 어떤 클래스의 일부 기능만 재사용하고 싶은 경우 어떻게 하는 것이 좋을까?
그것은 바로 *위임* 을 사용하는 것이다. 위임은 자신이 직접 기능을 실행하지 않고 다른 클래스의 객체가 기능을 실행하도록 하는 것이다.

따라서 일반화 관계는 클래스 사이의 관계이지만 위임은 객체 사이의 관계이다.

아래는 위임을 사용하여 일반화(상속)를 대신하는 과정이다.

1. 자식 클래스에 부모 클래스의 인스턴스를 참조하는 속성을 만든다. 이 속성 필드를 `this`로 초기화한다.
2. 서브 클래스에 정의된 각 메서드에 1번에서 만든 위임 속성 필드를 참조하도록 변경한다.
3. 서브 클래스에서 일반환 관계 선언을 제거하고, 위임 속성 필드에 슈퍼 클래스의 객체를 생성해 대입한다.
4. 서브 클래스에서 사용된 슈퍼 클래스의 메서드에도 위임 메서드를 추가한다.
5. 컴파일하고 잘 동작하는지 확인한다.

실제로 `MyStack`클래스에 적용 시키보자.
```java
// 3. 서브 클래스에서 일반화된 관계선언을 제거
public class MyStack<String> {
//public class MyStack<String> extends ArrayList<String> {

    // 1. 자식 클래스에 부모클래스의 인스턴스를 참조하는 속성 생성, this로 초기화
    //private ArrayList<String> arrayList = this;

    // 4. 위임 속성 필드에 슈퍼 클래스의 객체를 생성해 대입
    private ArrayList<String> arrayList = new ArrayList<>();

    // 2. 서브 클래스에 정의된 메서드에 1번에서 만든 위임 속성 필드를 참조하도록 변경
    public void push(String element) {
        //add(element);
        arrayList.add(element);
    }

    // 2. 서브 클래스에 정의된 메서드에 1번에서 만든 위임 속성 필드를 참조하도록 변경
    public String pop() {
        //return remove(size() - 1);
        return arrayList.remove(size() - 1);
    }

    // 5. 서브 클래스에서 사용된 슈퍼 클래스의 메서드에도 위임 메서드를 추가
    public boolean isEmpty() {
        return arrayList.isEmpty();
    }

    // 5. 서브 클래스에서 사용된 슈퍼 클래스의 메서드에도 위임 메서드를 추가
    public int size() {
        return arrayList.size();
    }

}
```

**기능을 재사용할때는 일반화 관계(상속)이 아닌 위임을 이용하자**

### 3.3 집합론 관점에서 본 일반화 관계
일반화 관계는 수학에서 배우는 집합론과 매우 밀접한 관계가 있다.

아래의 그림은 집합과 일반화의 관계를 보여준다.

![집합론](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/group.png)

부모클래스 A는 전체 집합 A에 해당하고 그 부분 집합 A1, A2, A3는 각각 A의 자식 클래스에 해당한다. 이 때 아래와 같은 관계가 성립되어야 한다.

- A = A1 ∪ A2 ∪ A3
- A1 ∩ A2 ∩ A3 = ∅

아래와 같은 제약 조건도 존재한다.

![일반화 관계에서의 제약조건](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/contraint.png)

위 제약 조건을 일반화 관계에 적용하려면 제약조건 `{disjoint, complete}`를 사용한다. 제약 `{disjoint}`는 자식 클래스 객체가 동시에 두 클래스에 속할 수 없다는 의미고, `{complete}`는 자식 클래스의 객체에 해당하는 부모 클래스의 객체와 부모 클래스의 객체에 해당하는 자식 클래스의 객체가 하나만 존재한다는 의미다.

그리고 집합론 관점에서 일반화 관계를 만들면 연관관계를 단순하게 할 수 있다. 가령 어떤 쇼핑몰에서 구매액을 기준으로 회원을 VIP회원과 일반회원으로 분류했다고 가정해보자.
VIP회원과 일반회원 각각을 자식 클래스로 생각하여 상품과 연관관계를 맺게 할 수 있지만 기본적으로 회원은 상품을 회원등급에 관계없이 구매가 가능하다.
즉, 다시말하자면 상품클래스와 연관관계는 모든 자식클래스에서 공통적으로 갖는 연관관계이므로 아래와 같이 부모 클래스인 회원 클래스로 연관관계를 이동하는 것이 클래스 다이어그램을 간결하게 만들 수 있다.

![집합론을 통한 연관관계의 일반화](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/group-relation.png)

집합론적인 관점에서 일반화는 상호배타적인 부분 집합으로 나누는 과정으로 간주할 수 있다. 이를 통해 상호 배타적인 특성이 요구되는 상황에서 일반화 관계를 적용할 수 있다.
예를 들어 학생은 "놀기"와 "공부하기" 중에서 어느 한 상태에만 있을 수 있다면 학생이 "공부하기" 상태라면 책만 볼 수 있고, "놀기"상태라면 장난감만 다룰 수 있다. 이런 경우에는 전형적으로 상호 배타적인 두 상태를 모델링 해야하며 이때 일반화 관계가 유용하게 사용된다.

![일반화 관계를 이용한 상호 배타적 관계 모델링](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/mutually-exclusive-relation.png)

또한 집합을 여러 기준에서 분류할 수도 있다.
예를 들어 앞서 살펴본 쇼핑몰의 회원등급에 따라 VIP와 일반회원을 분류했지만 쇼핑몰의 동일한 지역주민이냐, 아니냐에 따라 분류할 수도 있다.
UML에서는 이러한 분류를 변별자(discriminator)라고 하며 일반화 관계를 표시하는 선 옆에 변별자 정보를 표시한다.

그런데 여러 개의 변별자를 사용해 집합을 부분집합으로 나눌 때 고려해야하는 사항이 있다.
만약 회원을 구매액과 지역주민이라는 변별자에 따라 분류하면 회원의 한 인스턴스는 VIP Member와 Ordinary Member 중 하나의 자식 클래스에 속하는 동시에 Local과 Non Local 중 하나의 자식 클래스에도 속하게 된다는 사실이다.
이와 같이 한 인스턴스가 동시에 여러 클래스에 속할 수 있는 것을 다중분류(Multiple Classification)이라 하며 '<<다중>>'이라는 스테레오 타입을 사용해 표현한다.

일반적으로 각 변별자에 따른 일반화 관계가 완전히 독립적일때는 별 문제가 발생하지 않는다. 하지만 요구사항이 변경되거나 새로운 요구사항이 추가됨에 따라 두 일반화 관계가 더 이상 독립적이지 않을 경우가 발생할 수도 있다.
예를 들어 보면 "현재 시스템은 VIP회원에게만 할인쿠폰을 제공한다"라는 말의 의미는 회원이 지역주민의 여부에 관계없이 할인 쿠폰을 지급한다는 의미다.
만약 정책이 일반회원이지만 지역주민에게는 경품을 제공하기로 변경된다면 난처한 상황이 발생된다.
연관 관계를 위한 속성을 Ordinary Member 클래스에 두면 비지역민에게도 경품이 제공될 수도 있고, Local 클래스에 두면 VIP 회원에게도 경품이 제공되기 때문이다.
이는 의도한 바와 잘못된 모델링이다.

![변별자와 다중 분류](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/discriminator.png)

이를 처리하기위한 방법으로 모든 분류 가능한 조합으로 대응하는 클래스를 만드는 방법이 있다.
Member클래스의 자식 클래스로 아래와 같은 4개의 클래스를 만들수 있다.

- VIP-Local : VIP Member ∩ Local
- VIP-Non Local : VIP Member ∩ Non Local
- Ordinary-Local : Ordinary Member ∩ Local
- Ordinary-Non Local : Ordinary Member ∩ Non Local

아래의 그림은 집합론 관점에서 클래스 관계를 최종적으로 수정한 것이다.

![변별자와 다중분류 변경](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/group-generalization.png)

위의 그림을 토대로 최종적으로 다음과 같이 클래스를 6가지로 분류할 수 있다.

- VIP-Local : VIP Member ∩ Local
- VIP-Non Local : VIP Member ∩ Non Local
- Ordinary-Local : Ordinary Member ∩ Local
- Ordinary-Non Local : Ordinary Member ∩ Non Local
- VIP Member : VIP Local ∪ VIP-Non Local
- Ordinary Member : Ordinary-Local ∪ Ordinary-Non Local
- Member : Ordinary Member ∪ VIP Member

**일반화는 자식 클래스의 적절한 합집합과 교집합으로 이루어진다.**

## 4. 다형성

### 4.1 다형성이란?
**객체지향에서 다형성은 "서로 다른 클래스의 객체가 같은 메시지를 받았을 때 각자의 방식으로 동작하는 능력"을 말한다.**

### 4.2 다형성 예제
애완동물을 예를 들어서 설명해보자. 애완동물에는 고양이, 강아지, 앵무새 등 여러 종류의 동물이 있고, 이러한 동물들에게 talk라는 명령을 내리면 각각의 애완 동물들을 아래와 같이 대답할 것이다.

- 고양이 : 야옹 야옹~
- 강아지 : 멍멍!!
- 앵무새 : 안녀엉~~

위에서 처럼 각각의 애완동물이 모두 명령을 수행하지만 행동 방식이 모두 다르다. 이게 바로 다형성의 개념이다.
다형성이 상속과 연계되어 동작하면 매우 강력한 힘을 발휘하게 된다.

위의 내용을 코드로 구현하면 아래와 같다.
```java
// 추상 클래스
public abstract class Pet {
    // 추상 메서드
    public abstract void talk();
}

// Pet 추상클래스를 상속 받은 Cat 클래스
public class Cat extends Pet {
    // 추상메서드를 재정의
    public void talk() {
        System.out.println("야옹~");
    }
}

// Pet 추상클래스를 상속 받은 Dog 클래스
public class Dog extends Pet {
    // 추상메서드를 재정의
    public void talk() {
        System.out.println("멍멍!");
    }
}

// Pet 추상클래스를 상속 받은 Parrot 클래스
public class Parrot extends Pet {
    // 추상메서드를 재정의
    public void talk() {
        System.out.println("안녀엉~~");
    }
}
```

그리고 `Parrot`클래스의 객체를 `Pet` 클래스 타입으로 지정한다.

```java
Pet p = new Parrot();
```

`p`에 바인딩된 객체에 `talk`메서드의 메시지를 전달하면 현재 `p`가 실제로 참조하는 객체에 따라 실행되는 `talk()`메서드의 동작이 달라지게 된다.
위의 경우는 `p`가 `Parrot`클래스의 객체를 참조하기 때문에`Parrot`클래스에 정의된 `talk()`메서드가 실행된다. 이것을 다형성이라고 한다.

그렇다면 다형성을 적용한 코드와 적용하지 않은 코드의 차이점을 알아보자.

```java
public class Cat {
    public void meow() {
        System.out.println("야옹~");
    }
}
public class Dog {
    public void bark() {
        System.out.println("멍멍!");
    }
}
public class Parrot {
    public void sing() {
        System.out.println("안녀엉~");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog d = new Dog();
        Cat c = new Cat();
        Parrot p = new Parrot();
        d.bark();
        c.meow();
        p.sing();
    }
}
```

```java
public abstract class Pet {
    public abstract void talk();
}

public class Cat extends Pet {
    @Override
    public void talk() {
        System.out.println("야옹~");
    }
}

public class Dog extends Pet {
    @Override
    public void talk() {
        System.out.println("멍멍!");
    }
}

public class Parrot extends Pet {
    @Override
    public void talk() {
        System.out.println("안녀엉~~");
    }
}
```

위의 다형성을 사용한 코드와 그렇지 않은 코드의 결과물은 같다.
하지만 다형성을 사용하지 않은 경우는 클래스별로 다르게 처리해주어야 하는 불편함이 있다.
다형성을 사용하는 경우에는 구체적으로 현재 어떤 클래스 객체가 참조되는지와 무관하게 프로그래밍을 할 수가 있다.
따라서 새로운 애완동물을 나타내는 클래스가 자식 클래스로 추가되더라도 코드는 영향을 받지 않게 된다.

이러한 것이 가능한 이유는 일반화 관계에 있을 때 부모 클래스의 참조 변수가 자식 클래스의 객체를 참조할 수 있기 때문이다.
단, 부모 클래스의 참조변수가 접근할 수 있는 것은 부모 클래스가 물려준 변수와 메서드뿐이다.

**다형성과 일반화 관계(상속)는 코드를 간결하게 할 뿐만아니라 변화에 유연하게 대처할 수 있게 해준다.**

- 다형성 예제1 : 형변환 오류
- 다형성 예제2 : 상속과 정적 메서드의 관계


## 5. 피터 코드의 상속 규칙

피터 코드는 상속의 오용을 막기 위해 상속의 사용을 엄격하게 제한하는 규칙 5가지를 만들었다.

- 자식 클래스와 부모 클래스 사이는 역할 수행 관계가 아니어야 한다.
- 한 클래스의 인스턴스는 다른 서브 클래스의 객체로 변환할 필요가 절대 없어야한다.
- 자식 클래스가 부모 클랫의 책임을 무시하거나 재정의하지 않고 확장만 수행해야 한다.
- 자식 클래스가 단지 일부 기능을 재사용할 목적으로 유틸리티 역할을 수행하는 클래스를 상속하지 않아야한다.
- 자식 클래스가 역할, 트랜잭션, 디바이스 등을 특수화 해야한다.

아래의 클래스 다이어그램을 통해 피터 코드의 규칙을 살펴보자

![상속으로 표현한 역할 수행관계](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/peter-coad-class-diagram.png)

### 5.1 첫번째 규칙 : 자식 클래스와 부모 클래스 사이는 역할 수행 관계가 아니어야 한다.
운전자는 어떤 순간 사람이 수행하는 역할이고, 회사원도 사람이 어떤 순간 수행하는 역할이다. 그래서 운전자나 회사원이 사람과 상속관계로 표현되어서는 안되므로 규칙에 위배된다.  

### 5.2 두번째 규칙 : 한 클래스의 인스턴스는 다른 서브 클래스의 객체로 변환할 필요가 절대 없어야한다.
운전자는 어떤 시점에 회사원이 될 수 있고, 회사원 역시 출퇴근 시에 운전자가 될 수 있다. 이런 경우 객체의 변환 작업이 필요하기 때문에 규칙에 위배된다.

### 5.3 세번째 규칙 : 자식 클래스가 부모 클랫의 책임을 무시하거나 재정의하지 않고 확장만 수행해야 한다.
현재 상태로는 규칙에 위배되는지 알 수 없다. 어떤 속성과 연산이 정의되어있는지 모르기 때문이다.

### 5.4 네번째 규칙 : 자식 클래스가 단지 일부 기능을 재사용할 목적으로 유틸리티 역할을 수행하는 클래스를 상속하지 않아야한다.
기능만 재사용할 목적으로 상속관계를 표현하지 않았으므로 규칙을 준수한다.

### 5.5 다섯번째 규칙 : 자식 클래스가 역할, 트랜잭션, 디바이스 등을 특수화 해야한다.
슈퍼 클래스가 역할, 트랜잭션, 디바이스를 표현하지 않았으므로 규칙에 위배된다.

따라서 위의 클래스 다이어그램은 아래의 클래스 다이어그램처럼 상속을 사용하지 않고, 연관 관계를 사용해 클래스 사이의 관계를 표현하는 편이 좋다.
이렇게 설계하면 사람은 종업원 역할과 운전자 역할을 수행한다는 사실이 드러나고, 어느 순간에도 두 역할도 수행하지 않을 수 있다는 다중성도 표현할 수 있다.

![연관 관계를 이용한 역할 수행 표현](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/peter-coad-class-diagram2.png)

아래의 클래스 다이어그램들의 차이점에 대해 알아보자.

![상속으로 표현한 역할 수행관계](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/peter-coad-class-diagram.png)
![관계를 이용한 역할 수행](https://raw.githubusercontent.com/walbatrossw/java-design-patterns/master/ch02-oop-principles/img/class-diagram.png)

첫번째 클래스 다이어그램은 사람의 역할이 운전자와 회사원으로 고정되어 있다. 따라서 사람에게 새로운 역할이 부가되면 클래스 코드도 변경되어야 한다.
그러나 두번째 클래스 다이어그램은 역할이라는 추상클래스를 상속받는 구조로 구체적인 역할 클래스들을 캡슐화하기 때문에 새로운 역할이 추가되더라도 기존의 코드는 영향을 받지 않는다.
만약 축구선수라는 역할이 추가되더라도 사람 클래스 코드는 변경되지 않는다. 이것을 **개방-폐쇠의 원칙(Open-Closed Principle)** 이라 한다.

그렇다면 위의 클래스 다이어그램을 코드로 작성해보자.

```java
// 사람 클래스
public class Person {

    private Role role;  // 역할 속성

    public void setRole(Role role) {
        this.role = role;
    }

    public Role getRole() {
        return role;
    }

    public void doIt() {
        role.doIt();
    }
}

// 역할 클래스
public abstract class Role {

    // 수행 추상 메서드
    public abstract void doIt();

}

// 운전자 클래스
public class Driver extends Role {

    @Override
    public void doIt() {
        System.out.println("Driving");  // 오버라이딩
    }
}

// 직장인 클래스
public class Worker extends Role {

    @Override
    public void doIt() {
        System.out.println("Working");  // 오버라이딩
    }
}

// 축구선수 클래스
public class SoccerPlayer extends Role {

    @Override
    public void doIt() {
        System.out.println("Soccer Player");
    }
}

// 메인 클래스
public class Main {
    public static void main(String[] args) {

        Person doubles = new Person();  // 사람 객체 생성

        doubles.setRole(new Driver());  // 운전자로 역할 변경
        doubles.doIt();

        doubles.setRole(new Worker());  // 직장인으로 역할 변경
        doubles.doIt();

        doubles.setRole(new SoccerPlayer()); // 축구 선수로 역할 변경
        doubles.doIt();
    }
}
```
