# 아이템 14. Comparable을 구현할지 고려하라

우선 Comparable이 무엇인지 알아보자.

## ✔️ Comparable<T>

### 정의
> 자기 자신과 매개변수 객체를 비교할 수 있도록 만들어주는 인터페이스

기본형 타입인 int, double 등의 변수는 단순히 관계 연산자를 통해 비교할 수 있다.

하지만 여러 변수와 값을 가진 **객체**들은 <u>무엇을 기준으로 비교해야 할까?</u>

### 객체의 비교를 가능하게!

name과 score을 멤버 변수로 갖는 Student 클래스를 생각해보자.
name을 기준으로 정렬해야 할까, score을 기준으로 정렬해야 할까?
비교의 기준을 Comparable 인터페이스의 compareTo()를 재정의함으로써 정의할 수 있다.

```java
public class Student implements Comparable<Student> {
    private String name;
    private int score; // 점수
    
    ...
    
    // Comparable 인터페이스 구현
    @Override
    public int compareTo(Student other) {
        // 성적을 기준으로 오름차순 정렬
        return Integer.compare(this.grade, other.grade);
    }
}

```

Student 클래스의 인스턴스들을 score를 기준으로 정렬하도록 코드를 작성해보았다.

클래스가 Comparable을 구현했다는 것은 해당 클래스의 인스턴스들 사이에 자연적인 순서가 있음을 의미하며,
순서가 명확한 값 클래스(알파벳, 숫자, 연대)를 작성한다면 반드시 Comparable 인터페이스를 구현해야 한다.

이처럼, 인스턴스를 비교하기 위한 기준을 정의할 수 있도록 Comparable 인터페이스는 **compareTo()** 를 제공한다.
그리고 compareTo()에서 정의한 기준은 곧, `Collections.sort()`나 `Arrays.sort()` 의 정렬 기준이 된다.

자바에서는 같은 타입의 인스턴스를 서로 비교해야만 하는 클래스들은 모두 Comparable 인터페이스를 구현하고 있다.
따라서 Boolean을 제외한 래퍼 클래스나, String, Time, Date 클래스의 인스턴스는 모두 정렬이 가능하다.
이 때, <u>기본 정렬 순서는 **오름차순**이다.</u>

<hr>

## ✔️ compareTo()

Comparable 인터페이스를 구현한 클래스에서는 **반드시 compareTo()를 정의해야 한다.** (구현하지 않으면 컴파일 오류)
앞서 설명한 Comparable이 제공하는 유일한 메서드인 compareTo()에 대해 알아보자.

### 정의
> 객체들을 비교하여 **정렬 순서를 결정**하는 메서드

### 예시

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

### compareTo를 이용한 2가지 비교 방식 - 숫자형 & 문자열

```java
// 숫자형 비교
public int compareTo(NumberSubClass referenceName);

// 문자열 비교
public int compareTo(String anotherString);
```
compareTo()는 위의 코드에서 볼 수 있듯이, **숫자형 비교와 문자열 비교** 2가지 방식을 제공한다.

#### 1) 숫자형 비교

```java
public class Main {
  public static void main(String[] args) {
      
    Integer a = 5;
    Integer b = 10;
    Double c = 0.8;
    int a = 3;
    int b = 4;

    System.out.println(a.compareTo(b)); // -1
    System.out.println(c.compareTo(0.1)); // 1
    System.out.pritln(b.compareTo(10)); // 0
  }
}
```

- Byte, Double, Integer, Float, Long, Short등과 같은 숫자형 래퍼 클래스를 비교할 수 있다.
- 반환 값
    - `기준 값 < 비교 값` : -1
    - `기준 값 > 비교 값` : 1
    - `기준 값 == 비교 값` : 0

#### 2) 문자열 비교

```java
public class Main {
  public static void main(String[] args) {
    String str = "abcd";
    
    System.out.println(str.compareTo("abcd")); // 0 (두 문자열이 일치)
    System.out.println(str.compareTo("ab")); // 4 - 2 (기준 값의 앞자리와 일치)
    System.out.println(str.compareTo("c")); // 'a' - 'c' = -2
  }
}

```
- 기준 값과 같은 문자열이 파라미터로 들어올 경우 0을 반환한다.
- 기타 반환 값
    - 기준 값의 앞자리부터 일치하는 문자열을 파라미터가 포함할 경우 : `(기준 문자열 길이 - 파라미터 문자열 길이)`
    - 첫 번째 위치의 문자열부터 비교가 실패할 경우 : `(기준 문자열의 첫번째 문자 아스키코드 값 - 파라미터 문자열의 첫번째 문자 아스키코드 값)`

### compareTo()와 equals()의 차이점

| **특징**             | **compareTo()**                              | **equals()**                                  |
|----------------------|----------------------------------------------|----------------------------------------------|
| **용도**            | 두 객체의 순서 비교 (정렬 기준)              | 두 객체의 동치성(Equality) 확인               |
| **메서드 정의 위치** | `Comparable<T>` 인터페이스                  | `Object` 클래스                              |
| **반환값**           | 음수, 0, 양수 (순서 정보 제공)              | `true` 또는 `false` (동치성 결과만 제공)      |
| **타입 검사**        | 같은 타입의 객체만 비교 가능 (`ClassCastException` 가능) | 모든 타입의 객체와 비교 가능                  |
| **주요 사용처**      | 정렬: `Collections.sort()`, `Arrays.sort()` | 동치성 검사: `HashMap`, `HashSet`, `List` 검색 등 |

compareTo()는 비교 대상이 동일한 타입임을 전제로 하며, 타입이 다르면 ClassCastException을 던지며 다른 객체를 신경쓰지 않아도 된다.

반면, equalsTo()는 타입이 달라도 같은 객체인지를 논리적으로 판단해야 한다.

<hr> 

## ✔️ compareTo()의 일반 규약

compareTo()의 규약을 지키면서 구현해야 객체들이 올바르게 비교되고 정렬된다.

### 1️⃣ 기준 객체와 주어진 객체의 비교

- 기준 객체 < 주어진 객체 : 음수(-1) 반환
- 기준 객체 == 주어진 객체 : 0 반환
- 기준 객체 > 주어진 객체 : 양수(+1) 반환

만약 두 객체의 타입이 일치하지 않으면, ClassCastException을 던진다.

### 2️⃣ 대칭성, 추이성, 반사성을 충족해야 한다

#### 대칭성

Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다.

따라서, compareTo(x)가 예외를 던지는 경우 x.compareTo(y)가 예외를 던져야 한다.

#### 추이성

(x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이어야 한다.

#### 반사성

크기가 같은 객체들끼리는 다른 객체와 비교한 결과가 모두 동일해야 한다.

x.compareTo(y)==0이면, sgn(x.compareTo(z)) == sgn(y.compareTo(z))여야 한다.

### 3️⃣ compareTo() 동치성 테스트의 결과를 equals()의 결과와 같게 하라

(x.compareTo(y)==0) == (x.equals(y))를 만족해야 한다. (필수는 아니지만 되도록)
만약 이를 만족하지 못한다면 equals 메서드와 일관되지 않음을 명시해야 한다.

```java
public class Main{
  public static void main(String[] args) {
    BigDecimal a = new BigDecimal("1.0");
    BigDecimal b = new BigDecimal("1.00");

    Set<BigDecimal> set1 = new HashSet<>();
    Set<BigDecimal> set2 = new TreeSet<>();

    set1.add(a);
    set1.add(b);

    set2.add(a);
    set2.add(b);

    System.out.println(set1.size()); // 2
    System.out.println(set2.size()); // 1
  }
}
```

예를 들어 BigDecimal 객체를 원소로 가지는 HashSet과 TreeSet이 있다고 하자.

HashSet은 equals()를 기준으로, TreeSet은 compareTo를 기준으로 중복 원소를 검사하기 때문에 다르게 동작한다.

<hr>

## ✔️ compareTo() 작성 요령

- Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일 타임에 정해진다.
    - 따라서, 타입을 확인하거나 형변환할 필요가 없다.
- 인수 타입이 잘못되면 컴파일 에러가 발생한다.
- null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.
- compareTo()는 동치가 아니라 순서를 비교한다.
- 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출해야 한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교할 경우 **Comparator**을 사용한다.

<hr>

## ✔️ 기본 타입 필드 비교

기본형 타입인 필드를 비교할 때는 **래퍼 클래스**에 존재하는 **compare()** 를 사용하자.

예를 들어, `Integer.compare()`, `Double.compare()` 등이 있다.

```java
public class Main{
  public static void main(String[] args) {
    int a = 10;
    int b = 20;
    
    int result = Integer.compare(a, b);
    if(result < 0) {
      System.out.println("a < b");
    }else if(result > 0) {
      System.out.println("a > b");
    }else {
      System.out.println("a = b");
    }
  }
}
```

<hr>

## ✔️ 여러 개의 필드를 비교하는 경우

### Comparable의 compareTo() 구현

클래스에 핵심 필드가 여러 개라면, 비교의 우선순위를 정해 가장 중요한 필드부터 비교해나간다.
비교 결과가 0이 아니라면 순서가 결정되고, 결과를 곧바로 반환하자!

```java
public int compareTo(Person p) {
    int result = Integer.compare(age, p.age);
    if(result == 0) {
        result = Integer.compare(height, p.height);
    }
    return result;
}
```

### Comparator의 비교자 생성 메서드 구현

자바 8에서는 **Comparator** 인터페이스가 제공하는 **비교자 생성 메서드**를 통해 메서드 연쇄 방식으로 compareTo 메서드를 구현할 수 있다.
성능 저하 이슈가 있지만, 아래 코드와 같이 정적 임포트 방식을 적용하면 된다.

```java
private static final Comparator<Person> COMPARATOR =
        comparingInt((Person p) -> p.age)
                .thenComparingInt(p -> p.height);

public int compareTo(Person p) {
    return COMPARATOR.compare(this, p);
}
```
처음 comparingInt를 호출할 때 람다를 이용하면서 타입을 명시해줘야 한다.
그러나 thenComparingInt부터는 타입을 명시해주지 않아도 자바가 타입을 추론할 수 있다.

#### comparingX()

Comparator는 타입 별로 `Comparator.comparingX()` 보조 생성자 메서드를 지원한다.
- `Comparator.comparingInt()`
- `Comparator.comparingDouble()`
- `Comparator.comparingLong()`

#### comparing()

Comparator는 객체 참조를 기반으로 비교하는 `Comparator.comparing()`을 지원한다.

```java
public class Person {
  String name;
  int age;
  
  public Person(String name, int age) {
      this.name = name;
      this.age = age;
  }

  public String getName() {
    return name;
  }
  
  @Override
  public String toString() {
      return name + ": " + age;
  }
}

public class Main{
  public static void main(String[] args) {
    List<Person> people = new ArrayList<>();
    people.add(new Person("Alice", 25));
    people.add(new Person("Bob", 30));
    
    people.sort(Comparator.comparing(Person::getName));
    
    System.out.println(people); // [Alice: 25, Bob: 30]
  }
}
```
<hr>

## ✔️ 해시코드 값의 차를 기준으로 비교하지 말 것

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
}
```
위의 방식은 사용하면 안 된다. 대신에 **정적 compare 메서드** 혹은 **비교자 생성 메서드**를 활용한 비교자를 구현하자.

### 정적 compare 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
}
```

### 비교자 생성 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder =
    Comparator.comparingInt(o -> o.hashCode());
```
<hr>

## 🔗 Reference
[[JAVA] compareTo() 사용법, 문자열/숫자 비교](https://doitdoik.tistory.com/92)

[TCP SCHOOL](https://www.tcpschool.com/java/java_collectionFramework_comparable)

