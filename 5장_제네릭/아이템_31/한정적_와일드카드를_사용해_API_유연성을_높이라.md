> [결론]
> - 타입안정성과 유연성을 극대화하려면, 입력 매개변수에 와일드 타입을 사용하자.

## 서론
- 매개변수화 타입 : 제네릭 타입을 특정 타입으로 지정한 형태
- 불공변: **타입이 자기 자신으로만 할당 가능하며, 상위 타입으로는 할당할 수 없는 성질**
    - 제네릭 타입에서는 제네릭 타입이 서로 다른 타입 파라미터를 가지면 하위 타입 관계가 성립하지 않는다
- 불공변은 타입 안정성을 위해 제네릭 타입에서 기본적으로 적용된다. 이를 적용하지 않으면 컴파일 단계에서 타입 간 불일치로 인해 발생할 수 있는 런타임 오류를 방지할 수 없게 됩니다.
- 예시
  ```java
  // Java에서는 `String`이 `Object`의 하위 타입입니다. 따라서 아래 코드는 아무 문제 X
  Object obj = "Hello, World!";
  String str = "Hello, Java!";
  obj = str; // 허용됨: String은 Object의 하위 타입

  // 불공변이 적용되지않는다면?
  List<String> stringList = new ArrayList<>();
  List<Object> objectList = stringList; // 불공변이 없다면 허용
  
  objectList.add(42); // List<String>에 Integer 추가!
  String s = stringList.get(0); // ClassCastException 발생
  
  ```

<br/>

## 한정적 와일드카드 사용
- **한정적 와일드카드**를 사용하면 불공변의 문제를 해결할 수 있습니다. 
- 와일드카드(`? extends E`, `? super E`)를 사용하면 불공변 대신 공변 또는 반공변의 동작을 구현할 수 있습니다.

### (1) 생산자 (`? extends E`)의 활용
- 주로 데이터를 읽기만 하는 경우에 사용됩니다.  
**예시: 합계를 계산하는 메서드**
```java
public static double sumOfList(List<? extends Number> list) {
    double sum = 0.0;
    for (Number num : list) {
        sum += num.doubleValue();
    }
    return sum;
}

// 사용 예시
List<Integer> intList = List.of(1, 2, 3);
List<Double> doubleList = List.of(1.1, 2.2, 3.3);

System.out.println(sumOfList(intList));   // 6.0
System.out.println(sumOfList(doubleList)); // 6.6

```

**장점**: `List<Integer>`와 `List<Double>`를 모두 처리할 수 있는 유연성을 제공.

<br/>

### (2) 소비자 (`? super E`)의 활용
- 데이터를 추가하거나 수정할 때 유용합니다.  
**예시: 리스트에 값을 추가하는 메서드**
```java
public static void addNumbers(List<? super Integer> list) {
    list.add(10);
    list.add(20);
}

// 사용 예시
List<Number> numberList = new ArrayList<>();
addNumbers(numberList);
System.out.println(numberList); // [10, 20]

```
**장점**: `List<Number>`와 같이 더 넓은 범위를 가지는 컬렉션에도 안전하게 값을 추가 가능

<br/>

### **왜 API 유연성이 높아지는가?**
와일드카드를 사용하지 않으면 API는 특정 타입에 고정되어 호출하는 쪽에서 타입 변환이 필요하거나 더 많은 제약이 생깁니다. 반면, **한정적 와일드카드**를 사용하면
1. 호출자가 다양한 타입을 전달할 수 있어 **호환성**이 증가.
2. 제네릭 타입으로 인해 컴파일 시 **타입 안정성**을 확보.

<br/>

## 관련 개념

### 공변/반공변
- 공변은 `T`의 하위 타입을 허용합니다. `? extends T`
- 반공변은 `T`의 상위 타입을 허용합니다. `? super T`

### 리스코프 치환 원칙
- 제네릭 타입에서의 불공변은 리스코프 치환 원칙 위배하는 것처럼 보인다.
- why? 하위 타입은 언제나 상위 타입으로 대체 가능해야 하나, 제네릭 타입의 하위 타입 관계가 기본적으로 적용되지 않기 때문입니다
- 한정적 와일드카드는 리스코프 치환 원칙을 만족시키는 설계다.
- 서론의 예시 충족

### 타입 파라미터 표기법
- E (Element): 주로 컬렉션의 요소 타입을 나타낼 때 사용됩니다.
- T (Type): 일반적인 타입을 나타낼 때 사용되는 가장 보편적인 표기법입니다.

### 생산자/소비자
- Java에서 생산자와 소비자라는 개념은 주로 데이터의 흐름과 처리 방식을 설명하는 데 사용 ex) 제네릭 프로그래밍과 동시성 프로그래밍
- 생산자: 작업 생성하는 부분, 데이터를 '제공'하는 역할
- 소비자: 작업 처리하는 부분, 데이터를 '받아들이는' 역할

### PESCS
- 제네릭에서 와일드카드를 사용할 때의 원칙을 요약한 규칙 = 언제 Extends와 Super를 사용할지 알려주는 공식
- Producer Extends, Consumer Super

