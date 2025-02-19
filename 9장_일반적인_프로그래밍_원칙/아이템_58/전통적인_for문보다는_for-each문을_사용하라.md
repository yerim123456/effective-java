### 결론
- 따로 반복자(index)에 접근할 일이 없고, `for` 문을 이용하는 것이 유리한 몇 가지 경우가 아니라면, `for-each` 를 우선적으로 고려하자.
- 전통적인 for 문보다 for-each 문을 사용하는 것이 좋다. for-each 문은 더 간결하고 오류 가능성이 적으며, 특히 중첩 반복에서 장점이 두드러짐.

### 전통적인 `for`문의 약점
- `while`보다는 낫지만, 그래도 여전히 인덱스가 잘못 쓰일 위험이 있다.
    - 인덱스를 굳이 쓰지 않는다면, 외부로 노출시킬 필요는 없다.

### for-each 문의 주요 이점
1. 코드가 간결해져 가독성이 향상
2. 인덱스 관련 오류를 방지 가능
3. 컬렉션이나 배열을 순회할 때 더 효율적


### 하지만 다음과 같은 경우에는 전통적인 for 문을 사용!
1. 파괴적인 필터링 (원소를 제거해야 할 때)
    - 반복을 돌며 원소를 하나씩 지워나가는 필터링을 의미한다.
    - `Collection` 의 `removeIf()` 를 통해 구현해나가자.
        - `Collection.removeIf()` 메서드는 특정 조건을 만족하는 원소를 지울 수 있다.
2. 변형 (원소의 값을 변경해야 할 때)
    - 리스트 혹은 배열을 순회하며 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용하자.
3. 병렬 반복 (여러 컬렉션을 동시에 순회해야 할 때)
    -  여러 컬렉션을 병렬로 순회해야 한다면, 반복자와 인덱스 변수를 사용해 엄격하게 제어하는 편이 좋다.


### `Iterable` 인터페이스
```java
public interface Iterable<E> {
  Iterator<E> iterator();
}
```
- `for-each` 를 사용하기 위해서는 `Iterable` 을 구현해야 한다.
- `Iterable` 만 간단히 구현한다면, `for-each` 를 사용할 수 있다.

<br/>

### `for` 문을 잘못 사용했을 때 생기는 버그 찾기
```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }

// 외부 루프에 이용된 `enum Suit`이 `enum Rank`보다 짧아서 `NoSuchElementException`을 던짐
@Test
@DisplayName("NoSuchElementException 을 던지는 예제")
public void test() {
    Collection<Suit> suits = Arrays.asList(Suit.values());
    Collection<Rank> ranks = Arrays.asList(Rank.values());

    Assertions.assertThrows(NoSuchElementException.class, () ->{
        for (Iterator<Suit> i = suits.iterator(); i.hasNext();) {
            for (Iterator<Rank> j = ranks.iterator(); j.hasNext();) {
                System.out.println(i.next() + ", " + j.next());
            }
        }
    });
}

------

enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }

// 예상 결과: 6 x 6 = 36쌍, 실제 결과: 6쌍
// 이는 외부 루프의 `i.next()`가 내부 루프에서도 소비되어 매 외부 반복마다 두 개의 요소를 건너뛰기 때문
@Test
@DisplayName("코드가 의도대로 동작하지 않음에도 예외가 안뜨는 케이스")
public void test2() {
    Collection<Face> faces = EnumSet.allOf(Face.class);

    for (Iterator<Face> i = faces.iterator(); i.hasNext();) {
        for (Iterator<Face> j = faces.iterator(); j.hasNext();) {
            System.out.println(i.next() + ", " + j.next());
        }
    }
}
```
<br/>

### 코드 예시
```java

public class ForEachExample {
    public static void main(String[] args) {
        // 중첩 반복 예시
        String[] suits = {"Hearts", "Diamonds", "Clubs", "Spades"};
        String[] ranks = {"Ace", "2", "3", "4", "5", "6", "7", "8", "9", "10", "Jack", "Queen", "King"};

        // 전통적인 for 문 (오류 가능성 있음)
        // 여기서는 문제가 없으나, suits 배열에 j를 쓰는 실수 발생할 수도?
        for (int i = 0; i < suits.length; i++) {
            for (int j = 0; j < ranks.length; j++) {
                System.out.println(suits[i] + " of " + ranks[j]);
            }
        }

        // for-each 문 (더 안전하고 간결함)
        for (String suit : suits) {
            for (String rank : ranks) {
                System.out.println(suit + " of " + rank);
            }
        }

        // Iterable 인터페이스 구현 예시
        CustomIterable<String> customList = new CustomIterable<>();
        customList.add("Item 1");
        customList.add("Item 2");
        customList.add("Item 3");

        for (String item : customList) {
            System.out.println("Custom Iterable: " + item);
        }
    }

    // Iterable 인터페이스 구현 예시
    static class CustomIterable<T> implements Iterable<T> {
        private List<T> list = new ArrayList<>();

        public void add(T item) {
            list.add(item);
        }

        @Override
        public Iterator<T> iterator() {
            return list.iterator();
        }
    }
}


```
