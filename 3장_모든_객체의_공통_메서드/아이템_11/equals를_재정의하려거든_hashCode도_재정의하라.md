
# item 11 : equals를 재정의하려거든 hashCode도 재정의하라

> [서론]
> - `equals`를 재정의할 때는 `hashCode`도 반드시 재정의해야 한다.
> - 그렇지않으면, hashCode 일반 규약을 어겨 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제 
>   - => 일반 규약 2번 사항
> - equals는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수는 있지만 
>   - Object의 기본 hashCode 메서드는 전혀 다르다고 판단하여 서로 다른 값 반환!

```java
// 문제가 되는 예시
public class HashCodeProblemExample {
    public static void main(String[] args) {
        Map<Point, String> map = new HashMap<>();
        
        Point p1 = new Point(1, 2);
        Point p2 = new Point(1, 2);

        map.put(p1, "첫번째 Point");
        
        System.out.println("p1.equals(p2): " + p1.equals(p2));
        System.out.println("map.get(p1): " + map.get(p1));
        System.out.println("map.get(p2): " + map.get(p2));
    }
}

/** 결과
     p1.equals(p2): true // 논리적으로 동등하지만 다른 hashcode
     map.get(p1): First Point
     map.get(p2): null
 */
```

---

#### 단어
- 논리적 동등성: 두 객체가 서로 다른 메모리 주소를 가지고 있더라도 객체의 내부 상태나 값이 동일
- 물리적 동등성(동일성): 두 객체가 메모리 상에서 같은 위치를 참조하고 있는지


### [hash Collection 동작 과정](https://sasca37.tistory.com/250#article-5--hash-collection-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95)

![image](https://user-images.githubusercontent.com/81945553/180597415-8ad1840a-8446-448e-a631-caf600007770.png)

- 컬렉션의 `HashMap`, `HashSet`, `HashTable`은 객체가 논리적으로 같은지 비교할 때 다음과 같은 과정을 거친다.
  - hashCode() 호출 (해시코드 비교)
      - 먼저 객체의 hashCode() 메서드를 호출하여 hash값을 얻고
      - 이를 사용하여 객체가 저장될 칸(버킷)을 결정 
      - 같은 칸에 이미 객체가 존재하는 경우, 다음 단계로 넘어간다.
  - equals() 호출 
    - 같은 칸에 있는 객체들과 equals() 메서드를 사용하여 동등성 비교. 
    - equals() 메서드가 true를 반환하면 두 객체는 동일한 것으로 간주

<br/>

### HashCode 일반 규약 3가지
1.  **`equals` 비교에 사용되는 정보가 변경되지 않았다면, `hashCode` 도 변하면 안 된다.**  
    (애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없음)
2. **`equals`가 두 객체가 같다고 판단했다면, 두 객체의 `hashCode`는 똑같은 값을 반환한다.**  
   → 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
3. **`equals`가 두 객체를 다르다고 판단했더라도, `hashCode`는 꼭 다를 필요는 없다.**  
   → 하지만, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다. ~ 해시충돌 <br/>
   ** why? 같은 해시코드를 가진 객체들은 같은 칸에 저장 → 검색,삽입,삭제 연산의 시간복잡도 영향

<br/>

### 좋은 hashCode 메서드를 작성하는 방법

좋은 해시 함수라면 서로 다른 인스턴스에 대해서 다른 해시코드를 반환한다. <br/>
이상적인 해시 함수는 주어진 인스턴스들을 32비트 정수 범위(0~40억)에 균일하게 분배해야 한다.

```java

public class Person {
    private String name;
    private int age;
    private String[] hobbies;
    private transient int tempField; // equals에서 사용되지 않는 필드

    public Person(String name, int age, String[] hobbies) {
        this.name = name;
        this.age = age;
        this.hobbies = hobbies;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age &&
               Objects.equals(name, person.name) &&
               Arrays.equals(hobbies, person.hobbies);
    }

    @Override
    public int hashCode() {
        int result = 17; // 1. int 변수 result를 선언하고 초기화

        // 2. 적절하게 각 필드에 대한 해시코드 값을 구함
        result = 31 * result + Objects.hashCode(name);
        result = 31 * result + Integer.hashCode(age);
        result = 31 * result + Arrays.hashCode(hobbies);
        // 31인 이유는 홀수이면서 소수이다. 숫자가 짝수이고 오버플로우가 발생한다면 정보를 잃게 된다
        // 추가적으로 equals 비교에 사용되지 않는 필드는 해시코드 계산 로직에서 반드시 제외

        // 3. result 값을 갱신하고 반환
        return result;
    }
}

```

<br/>

### 다른 방법들
```java
/** 1. 간단한 한 줄짜리 hashCode 메서드
 * Java 7 이상에서는 다음과 같이 한줄짜리 가능 but 성능의 아쉬움
 * 입력 인수를 담기 위한 배열이 필요하고 입력 값 중에 기본 타입이 있다면 박싱과 언박싱도 필요 **/

@Override 
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
// 사용: return Objects.hash(name, age, Arrays.hashCode(hobbies));

/** 2. hashCode 지연 초기화 => 캐싱!
 * 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 캐싱을 고려해야 한다.
 * 해시의 키로 자주 사용될 것 같으면 인스턴스가 만들어질 때 해시 코드를 계산해둬야 한다.
 * 해시의 키로 사용되지 않는다면 지연 초기화가 좋다.
 * 대신, 지연초기화는 스레드의 안정성을 고려해야 한다. 
 * 여러개의 쓰레드가 동시에 hashCode를 호출하는 경우 여러번의 hashCode 연산이 발생할 수 있다. 
 * 지연 초기화를 사용하기 위해서는 쓰레드 동기화 시점을 신경 써야 한다.
 */

public class Person {
    private String name;
    private int age;
    private String[] hobbies;
    private int hashCode; // 해시코드를 저장할 변수

    @Override
    public int hashCode() {
        if (hashCode == 0) { // 해시코드가 계산되지 않았다면
            int result = 17; // 소수로 시작
            result = 31 * result + areaCode;
            result = 31 * result + prefix;
            result = 31 * result + lineNum;
            hashCode = result; // 계산된 해시코드 저장
        }
        return hashCode;
    }

    // equals 메서드는 생략
}
```

### 실무에서 권장하는 방법
- 실무에서는 HashCodeBuilder나 @EqualsAndHashCode를 사용하는 것을 추천한다
- 장점: 코드 간소화, 일관성 유지, 오류 감소, 최적화된 알고리즘
- 단점: 다만 리플렉션을 사용하기 때문에 성능에 이슈가 있는지 고민을 해 볼 필요도 있다.
```java
// HashCodeBuilder
public int hashCode() {
    return HashCodeBuilder.reflectionHashCode(this);
}

// equals 메서드와 hashCode 메서드를 생성해주는데, static과 transient가 아닌 모든 필드들이 대상이 된다.
@EqualsAndHashCode
public class Example {
    private transient int transientVal = 10;
    private String name;
    private int id;
}
```

### HashCode 작성시 주의 점
1. equals 비교에 사용되지 않는 필드는 계산에서 무시하자
2. 성능을 높이기 위해 해시코드 계산시 핵심 필드를 생략해서는 안된다.
   속도는 빨라지지만 해시 품질이 나빠져, 해시테이블 성능을 떨어뜨릴 수 있다.
3. hashCode가 반환하는 값의 생성규칙을 API 사용자에게 자세히 공표하지 말자
---

> [결론]
> - `equals`를 재정의할 때는 `hashCode`도 반드시 재정의!
> - 재정의한 `hashCode`는 `Object`의 API 문서에 기술된 일반 규약을 따라야한다.
> - 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.
> - AutoValue 프레임워크를 사용하면 멋진 `equals`와 `hashCode`를 자동 만들어준다.
