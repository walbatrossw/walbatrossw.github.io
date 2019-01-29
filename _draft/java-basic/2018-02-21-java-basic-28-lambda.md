---
layout: article
title: Java - 람다(Lambda) # 제목
date: 2018-02-21 00:00:00 # 작성 시간 2019-00-00 00:00:00
category: java # 카테고리
tags: java java-basic lambda # 태그
key: 20180221a # for gitalk
---

<!--more-->

본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.


## 1. 람다식(Lambda expression)이란?

**람다식은 간다히 말해서 메서드를 하나의 식으로 표현한 것이다.** 메서드를 람다식으로 표현하면 메서드의 이름과 반환값이 없어지므로 람다식을 **익명함수(anonymous function)** 라고도 한다. 이전에 `Collection`의 `Arrays`를 정리하면서 일부 코드에서 람다식을 봤었는데 그 코드를 메서드로 표현하면 아래와 같다.

```java
int[] arr = new int[5];

// 람다식                    
Arrays.setAll(arr, () -> (int)(Math.random()*5)+1);
```

```java
// 위의 람다식을 메서드로 표현
int method() {
  return (int) (Math.ramdom()*5) + 1;
}
```

위의 코드를 보면 메서드보다 람다식이 간결하고 이해하기 쉽다는 것을 알 수 있다. 게다가 모든 메서드는 클래스에 포함되어야 하므로 클래스도 새로 만들어야 하고, 객체도 생성해야만 비로소 메서드를 호출할 수 있다. 그러나 람다식은 이 모든 과정없이 오직 **람다식 자체만으로도 이 메서드의 역할을 수행**할 수 있다. 람다식은 **메서드의 매개변수로 전달하는 것이 가능** 하고, **메서드의 결과로 반환** 될 수도 있다. 람다식으로 **메서드를 변수처럼 다루는 것이 가능** 해진 것이다.

## 2. 람다식 작성하기

```java
// 메서드
ReturnType methodName(Argument argument) {
  //...
}

// 람다식 : 반환타입과 메서드명을 없애고, 매개변수와 {} 사이에 ->를 추가
(Argument argument) -> {
  //...
}
```
람다식은 익명함수답게 **메서드에서 이름과 반환타입을 제거하고 매개변수 선언부와 몸통`{}` 사이에 `->`를 추가** 한다.

### 2.1 메서드를 람다식으로 변환 과정 및 주의 사항

```java
// 메서드
int max(int a, int b) { return a > b ? a : b; }
```
```java
// 리턴타입과 메서드명 제거, -> 추가
(int a, int b) -> { return a > b ? a : b; }
```
```java
// 코드블럭({})을 제거하고 리턴문 대신 식으로 변경할 경우, 식의 끝에 세미콜론(;)을 반드시 제거
(int a, int b) -> a > b ? a : b // OK
(int a, int b) -> a > b ? a : b; // Error
```
```java
// return문을 작성할 경우, 코드블럭을 제거할 수 없음
(int a, int b) -> { return a > b ? a : b; }// OK
(int a, int b) -> return a > b ? a : b; // Error
```
```java
// 매개변수의 타입이 추론이 가능한 경우 생략할 수 있음. 단, 하나만 생략할 수 없고, 타입이 있으면 괄호를 생략할 수 없음
(a, b) -> a > b ? a : b // OK
(int a, b) -> a > b ? a : b // Error
int a, int b -> a > b ? a : b // Error
```
```java
// 만약 매개변수가 하나일 경우, 괄호를 생략할 수 있음, 단 타입을 생략해야함
(int a) -> a * a // OK
a -> a * a // OK
int a -> a * a // Error
```

## 3. 람다식 변환 예제

### 3.1 예제 1
```java
int max(int a, int b) {
  return a > b ? a : b;
}

(int a, int b) -> { return  a > b ? a : b; }

(int a, int b) -> a > b ? a : b

(a, b) -> a > b ? a : b
```

### 3.2 예제 2
```java
int printVar(String name, int i) {
  System.out.println(name + "=" + i);
}

(String name, int i) -> { System.out.println(name + "=" + i); }

(String name, int i) -> System.out.println(name + "=" + i)

(name, i) -> System.out.println(name + "=" + i)
```

### 3.3 예제 3

```java
int squre(int x) {
  return x * x;
}

(int x) -> { return x * x; }

(int x) -> x * x

x -> x * x
```

### 3.4 예제 4
```java
int roll() {
  return (int) (Math.random()) * 6);
}

() -> { return (int) (Math.random() * 6); }

() -> (int) (Math.random() * 6)
```

### 3.5 람다식 변환 예제 5

```java
int sumArr(int[] arr) {
  int sum = 0;
  for(int i : arr)
      sum += i;
  return sum;
}

(int[] arr) -> {
  int sum = 0;
  for(int i : arr)
      sum += i;
  return sum;
}
```

## 4. 함수형 인터페이스(functional interface)

### 4.1 익명클래스와 함수형 인터페이스

자바에서는 모든 메서드는 클래스 내에 포함되어 있어야 하는데 람다식은 어떤 클래스에 포함되는 것일까? 람다식이 메서드와 동등한 것처럼 설명했지만, 사실 **람다식은 익명클래스의 객체와 동등하다.**
```java
// 람다식
(int a, int b) -> a > b ? a : b

// 익명클래스 객체
new Object() {
  int max(int a, int b) {
    return  a > b ? a : b;
  }
}
```

그렇다면 람다식으로 정의된 익명 객체의 메서드를 어떻게 호출할 수 있을까? 이미 알고 있듯이 참조변수가 있어야 객체의 메서드를 호출할 수 있으므로 **익명객체의 주소를 `f`라는 참조변수에 저장한다.**
```java
Type f = (int a, int b) -> a > b ? a : b;
```

참조변수 `f`의 타입은 어떤 것이야할까? 참조형이기때문에 **클래스 또는 인터페이스가 가능하다**. 그리고 **람다식과 동등한 메서드가 정의되어 있는 것** 이어야 한다.
```java
interface MyFunction {
  public abstract max(int a, int b);
}
```

이제 이 인터페이스를 구현한 익명 클래스의 객체는 아래의 코드와 같이 생성을 할 수 있다.
```java
MyFunction f = new MyFunction() {
  public int max(int a, int b) {
    return a > b ? a : b;
  }
};

int big = f.max(5, 3); // 익명 객체의 메서드 호출
```

`MyFunction`인터페이스에 정의된 메서드 `max()`는 람다식 `(int a, int b) -> a > b ? a : b`과 메서드 선언부가 일치하기 때문에 위 코드의 익명 객체를 람다식으로 대체할 수 있다.
```java
MyFunction f = (int a, int b) -> a > b ? a : b // 익명 객체를 람다식으로 교체

int big = f.max(5, 3); // 익명 객체의 메서드를 호출
```

`MyFunction`인터페이스를 구현한 익명 객체를 람다식으로 대체가 가능한 이유는, 람다식도 실제로 익명객체이고, `MyFunction`인터페이스를 구현한 익명 객체의 메서드 `max()`와 람다식의 매개변수 타입과 갯수, 반환값이 일치하기 때문이다.


```java
@FunctionalInterface
interface MyFunction { // 함수형 인터페이스 MyFunction을 정의
  public abstract max(int a, int b);
}
```
지금까지 살펴본 것처럼 인터페이스를 통해 람다식을 다룰수 있고, 이러한 인터페이스를 **함수형 인터페이스(Functional interface)** 라고 부른다. **함수형 인터페이스는 오직 하나의 추상메서드만 정의되어 있어야 한다는 제약이 있다.** 그 이유는 람다식과 인터페이스의 메서드가 1:1로 연결되어야 하기 때문이다. 반면에 `static`메서드와 `default`메서드의 갯수에는 제약이 없다.

기존에는 아래의 코드 첫번째와 같이 인터페이스의 메서드 하나를 구현하는데도 복잡하게 해야 했다. 하지만 아래의 2번째 코드와 같이 람다식으로 처리하면 간단하게 처리를 할 수 있게 된다.
```java
List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaa");

Collections.sort(list, new Comparator<String>()) {
  public int compare(String s1, String s2) {
    return s2.compareTo(s1);
  }
}
```
```java
List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaa");

// 람다식으로 처리
Collections.sort(list, (s1, s2) -> s2.compareTo(s1));
```

### 4.2 함수형 인터페이스 타입의 매개변수
함수형 인터페이스 `MyFunction`이 아래의 첫번째 코드와 같이 정의가 되있고, 두번째 코드와 같이 매서드의 매개변수가 `MyFunction`타입이면, 이 메서드를 호출할 때 람다식을 참조하는 참조변수를 매개변수로 지정해야 한다.
```java
@FunctionalInterface
interface MyFunction {  // 함수형 인터페이스
  void myMethod(); // 추상메서드
}
```
```java
void aMethod(MyFunction f) { // 매개변수 타입이 함수형 인터페이스인 메서드
  f.myMethod(); // 함수형 인터페이스 MyFunction에 정의된 메서드 호출
}

// 람다식을 참조하는 참조변수 f
MyFunction f = () -> System.out.println("myMethod()");

// 람다식을 참조하는 참조변수를 인자로 넣어주고, 메서드 호출
aMethod(f);

// 또는 참조변수 없이 직접 람다식을 매개변수로 지정
aMethod(() -> System.out.println("myMethod()"));
```

### 4.3 함수형 인터페이스 타입의 반환타입
메서드의 반환타입이 함수형 인터페이스라면, 이 함수형 인터페이스의 추상메서드와 동등한 람다식을 가리키는 참조변수를 반환하거나, 람다식을 직접 반환할 수 있다.
```java
MyFunction myMethod() {
  MyFunction f = () -> {};
  return f;
  //return () -> {};
}
```


**람다식을 참조변수로 다룰수 있다는 것은 메서드를 통해 람다식을 주고 받을 수 있다는 것, 즉 변수처럼 메서드를 주고 받을 수 있다는 것을 의미한다.**

### 4.4 함수형 인터페이스 예제

```java
public class LambdaEx1 {

    // 매개변수 타입이 함수형 인터페이스 MyFunction인 메서드
    static void execute(MyFunction f) {
        f.run();
    }

    // 매개변수 타입이 함수형 인터페이스 MyFunction인 메서드
    static MyFunction getMyFunction() {
        MyFunction f = () -> System.out.println("f3.run()");
        return f;
    }

    public static void main(String[] args) {

        // 람다식으로 MyFunction의 run() 구현
        MyFunction f1 = () -> System.out.println("f1.run()");

        // 익명클래스로 run() 구현
        MyFunction f2 = new MyFunction() {

            public void run() {
                System.out.println("f2.run()");
            }

        };

        MyFunction f3 = getMyFunction();

        f1.run();
        f2.run();
        f3.run();

        execute(f1);

        execute( () -> System.out.println("run()"));

    }
}

@FunctionalInterface
interface MyFunction {
    void run();
}
```

```console
f1.run()
f2.run()
f3.run()
f1.run()
run()
```

### 4.5 람다식의 타입과 형변환

함수형 인터페이스로 람다식을 참조할 수 있는 것일 뿐, 람다식의 타입이 함수형 인터페이스의 타입과 일치하는 것은 아니다. 람다식은 익명 객체이고, 익명 객체는 타입이 없다. 정확히는 타입이 있지만 컴파일러가 임의의 이름을 정하기 때문에 알 수 없는 것이다. 그래서 대입 연산자의 양변의 타입을 일치시키기 위해 아래와 같이 형변환이 필요하다.
```java

MyFunction f = (MyFunction) (() -> {}); // 양변의 타입이 다르기때문에 형변환이 필요, 형변환 생략가능
```

**람다식은 `Object`타입으로 형변환은 할 수없다.** 그럼에도 `Object`타입으로 형변환을 하려면 함수형 인터페이스로 형변환을 먼저 해야한다.

```java
Object obj = (Object) (() -> {}); // Error, 함수형 인터페이스만 형변환 가능
Object obj = (Object) (MyFunction) (() -> {}); // 함수형 인터페이스로 먼저 형변환, Object타입으로 다시 형변환
String str = ((Object) (MyFunction) (() -> {})).toString();
```

### 4.6 람다식의 형변환 예제
```java
public class LambdaEx2 {
    public static void main(String[] args) {
        MyFunction2 f = () -> {};
        Object obj = (MyFunction2) (() -> {});
        String str = ((Object) (MyFunction2) (() -> {})).toString();

        System.out.println(f);
        System.out.println(obj);
        System.out.println(str);

        //System.out.println(()->{}); // Error, 람다식은 Object타입으로 형변환 불가
        System.out.println((MyFunction2) (()->{}));
        //System.out.println((MyFunction2) (()->{}).toString()); // Error
        System.out.println((Object) ((MyFunction2) (() -> {})).toString());

    }
}

@FunctionalInterface
interface MyFunction2 {
    void myMethod();
}
```

```console
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$1/1078694789@7cca494b
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$2/1023892928@7ba4f24f
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$3/558638686@448139f0
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$4/999966131@7699a589
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$5/1480010240@4dd8dc3
```

## 5. java.util.function 패키지

대부분의 메서드는 타입이 비슷하기 때문에 제네릭으로 정의하면 매개변수나 반환타입이 달라도 문제가 되지 않는다.
그래서 `java.util.function`패키지에 일반적으로 쓰이는 형식의 메서드를 함수형 인터페이스로 미리 정의해 놓았다.
매번 함수형 인터페이스를 정의하지 말고, 이 패키지의 인터페이스를 활용하는 것이 좋다.

### 5.1 기본적인 함수형 인터페이스

|함수형 인터페이스|메서드|설명|
|---|---|---|
|`java.lang.Runnable`|`void run()`|매개변수 X, 리턴값 O|
|`Supplier<T>`|`T get()`|매개변수 X, 리턴값 O|
|`Consumer<T>`|`void accept(T t)`|매개변수 O, 리턴값 X|
|`Function<T>`|`R apply(T t)`|일반적인 함수, 매개변수 1개, 리턴값 O|
|`Predicate<T>`|`boolean test(T t)`|조건식 표현에 사용, 매개변수 1개, 리턴값(`boolean`)|

### 5.1 조건식에 사용되는 Predicate
Predicate는 Function의 변형으로 리턴타입이 `boolean`이라는 것만 다르다. Predicate는 조건식을
람다식으로 표현하는데 사용된다.

```java
Predicate<String> isEmptyStr = s -> s.length() ==0;
String s = "";
if (isEmptyStr.test(s)) {
  System.out.println("s is an empty String.");
}
```

### 5.2 매개변수가 두 개인 함수형 인터페이스
매개변수가 2개인 함수형 인터페이스는 이름 앞에 접두사 "Bi"가 붙는다.

|함수형 인터페이스|메서드|설명|
|---|---|---|
|`BiConsumer<T, U>`|`void accept(T t, U u)`|매개변수 2개, 리턴값 X|
|`BiPredicate<T, U>`|`boolean test(T t, U u)`|조건식 표현에 사용, 매개변수 2개, 리턴값(boolean)|
|`BiFunction<T, U, R>`|`R apply(T t, U u)`|매개변수 2개, 리턴값 O|

만약 두 개 이상의 매개변수를 갖는 함수형 인터페이스가 필요하다면 직접 만들어 써야한다. 만일 3개의 매개변수를
갖는 함수형 인터페이스를 선언한다면 아래의 코드와 같이 작성할 수 있다.

```java
@FunctionalInterface
interface TriFunction<T, U, V, R> {
  R reply(T t, U u, V v);
}
```

### 5.3 UnaryOperator와 BinaryOperator
Function의 또 다른 변형으로 UnaryOperator와 BinaryOperator가 있는데, 이 매개변수의 타입과 반환타입의
타입이 모두 일치한다는 점만 제외하고는 Function과 같다.

|함수형 인터페이스|메서드|설명|
|---|---|---|
|`UnaryOperator<T>`|`T apply(T t)`|Function의 자손, Funtion과 다르게 매개변수와 결과 타입이 같음|
|`BinaryOperator<T>`|`R test(T t, T t)`|BiFunction의 자손, BiFunction과 달리 매개변수와 결과 타입이 같음|

### 5.4 컬렉션프레임워크와 함수형 인터페이스
컬레션 프레임워크의 인터페이스에 다수의 디폴트 메서드가 추가되었는데, 그 중에서 일부는 함수형 인터페이스를 사용한다.

|인터페이스|메서드|설명|
|---|---|---|
|`Collection`|`boolean remove(Predicate<T> filter)`|조건에 맞는 요소 삭제|
|`List`|`void replaceAll(UnaryOperator<E> operater)`|모든 요소를 변환하여 대체|
|`Iterable`|`void forEach(Consumer<T> action)`|모든 요소에 작업 action을 수행|
|`Map`|`V compute(K key, BiFunction<K, V, V> f)`|지정된 키의 값에 작업 f를 수행|
|`Map`|`V computeIfAbsent(K key, BiFunction<K, V> f)`|키가 없으면, 작업 f 수행 후 추가|
|`Map`|`V computeIfPresent(K key, BiFunction<K, V, V> f)`|지정된 키가 있을 때, 작업 f 수행|
|`Map`|`V merge(K key, V value, BiFunction<V, V, V> f)`|모든 요소에 병합작업 f를 수행|
|`Map`|`void forEach(BiConsumer<K, V> action)`|모든 요소에 작업 action을 수행|
|`Map`|`void replaceAll(BiFunction<K, V, V>) f`|모든 요소에 치환작업 f를 수행|


### 5.5 함수형 인터페이스 예제

#### 5.5.1 예제 1 : 컬렉션 프레임워크 함수형 인터페이스

```java
// 컬렉션 프레임워크 함수형 인터페이스
public class LambdaEx4 {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();

        // 0 ~ 9까지 저장
        for (int i = 0; i < 10; i++) {
            list.add(i);
        }

        // 모든 요소 출력
        list.forEach(i -> System.out.print(i + ", "));
        System.out.println();

        // 2의 배수, 3의 배수 제거
        list.removeIf(x -> x % 2 == 0 || x % 3 == 0);
        System.out.println(list);

        list.replaceAll(i -> i * 10);   // 각 요소에 10을 곱한다
        System.out.println(list);

        // map의 모든 요소를 {k, v} 형식으로 출력
        Map<String, String> map = new HashMap<>();
        map.put("1", "1");
        map.put("2", "2");
        map.put("3", "3");
        map.put("4", "4");
        map.forEach((k, v) -> System.out.print("{" + k + ", " + v + "} , "));
        System.out.println();
    }
}
```

```console
0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
[1, 5, 7]
[10, 50, 70]
{1, 1} , {2, 2} , {3, 3} , {4, 4} ,
```


#### 5.5.2 예제 1 : 함수형 인터페이스

```java
// 함수형 인터페이스
public class LambdaEx5 {
    public static void main(String[] args) {

        Supplier<Integer> s = () -> (int) (Math.random() * 100) + 1;
        Consumer<Integer> c = i -> System.out.print(i + ", ");
        Predicate<Integer> p = i -> i % 2 == 0;
        Function<Integer, Integer> f = i -> i / 10 * 10;    // i의 일의 자리 제거

        List<Integer> list = new ArrayList<>();
        makeRandomList(s, list);
        System.out.println(list);

        printEvenNum(p, c, list);
        List<Integer> newList = doSomething(f, list);
        System.out.println(newList);

    }

    private static <T> List<T> doSomething(Function<T, T> f, List<T> list) {
        List<T> newList = new ArrayList<>(list.size());
        for (T i : list) {
            newList.add(f.apply(i));
        }
        return newList;
    }

    private static <T> void printEvenNum(Predicate<T> p, Consumer<T> c, List<T> list) {
        System.out.print("[");
        for (T i : list) {
            if (p.test(i))
                c.accept(i);
        }
        System.out.println("]");
    }

    private static <T> void makeRandomList(Supplier<T> s, List<T> list) {
        for (int i = 0; i < 10; i++) {
            list.add(s.get());
        }
    }

}
```

```console
[64, 9, 29, 40, 70, 90, 76, 33, 85, 75]
[64, 40, 70, 90, 76, ]
[60, 0, 20, 40, 70, 90, 70, 30, 80, 70]
```

### 5.6 기본형을 사용하는 함수형 인터페이스

함수형 인터페이스는 매개변수와 리턴값의 타입이 모두 제네릭이었지만 기본형 타입의 값을 처리할 때도 래퍼(Warpper)
클래스를 사용해왔다. 기본형 대신 래퍼클래스를 사용하는 것은 당연히 비효율적이다. 그래서 보다 효율적으로 처리할 수
있도록 기본형을 사용하는 함수형 인터페이스들이 제공된다.

|함수형 인터페이스|메서드|설명|
|---|---|---|
|`DoubleToIntFunction`|`int applyAsInt(double d)`|AtoBFunction은 입력이 A타입 출력이 B타입|
|`ToIntFunction<T>`|`int applyAsInt(T value)`|ToBFunction은 출력이 B타입, 입력은 제네릭 타입|
|`IntFunction<R>`|`R apply(int value)`|AFunction은 입력이 A타입, 출력은 제네릭 타입|
|`ObjIntConsumer<T>`|`void accept(T t, int i)`|ObjAFuction은 입력이 T, int타입, 출력은 X|

#### 5.6.1 예제

```java
public class LambdaEx6 {
    public static void main(String[] args) {
        IntSupplier s = () -> (int) (Math.random() * 100);
        IntConsumer c = i -> System.out.print(i + ", ");
        IntPredicate p = i -> i % 2 == 0;
        IntUnaryOperator op = i -> i / 10 * 10;

        int[] arr = new int[10];

        makeRandomList(s, arr);
        System.out.println(Arrays.toString(arr));

        printEvenNum(p, c, arr);

        int[] newArr = doSomething(op, arr);
        System.out.println(Arrays.toString(newArr));
    }

    private static void makeRandomList(IntSupplier s, int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            arr[i] = s.getAsInt();
        }
    }

    private static void printEvenNum(IntPredicate p, IntConsumer c, int[] arr) {
        System.out.print("[");
        for (int i : arr) {
            if (p.test(i))
                c.accept(i);
        }
        System.out.println("]");
    }

    private static int[] doSomething(IntUnaryOperator op, int[] arr) {
        int[] newArr = new int[arr.length];
        for (int i = 0; i < newArr.length; i++) {
            newArr[i] = op.applyAsInt(arr[i]);
        }
        return newArr;
    }
}
```

```console
[23, 37, 72, 44, 40, 27, 89, 14, 13, 42]
[72, 44, 40, 14, 42, ]
[20, 30, 70, 40, 40, 20, 80, 10, 10, 40]
```

## 6. Function의 합성과 Predicate의 결합

```java
// Function
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after);
default <V> Function<V, R> compose(Function<? super V, ? extends T> before);
static <T> Function<T, T> identify();
```

```java
// Predicate
default Predicate<T> and(Function<? super T> other);
default Predicate<T> or(Function<? super T> other);
default Predicate<T> negate();
static <T> Function<T, T> isEqual(Object targetRef);
```

### 6.1 Function의 합성
수학에서 두 함수를 합성해서 하나의 새로운 함수를 만들어 낼 수 있는 것처럼, 두 람다식을 합성해서 새로운 람다식을
만들 수 있다. 두 함수의 합성은 어느 함수를 먼저 적용하느냐에 따라 달라진다. 함수 `f`, `g`가 있을 때, `f.andThen(g)`는
함수 `f`를 먼저 적용하고, 그 다음에 함수 `g`를 적용한다. 그리고 `f.compose(g)`는 반대로 `g`를 먼저 적용하고
`f`를 적용한다.


### 6.2 Predicate의 결합

## 7. 메서드 참조
