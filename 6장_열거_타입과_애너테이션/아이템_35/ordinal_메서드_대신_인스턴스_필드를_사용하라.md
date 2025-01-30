# ordinal 메서드 대신 인스턴스 필드를 사용하라

## ️✔️ ordinal

모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 **ordinal 메서드**를 제공한다.

```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
}

public class EnumOrdinalExample {
    public static void main(String[] args) {
        for (Day day : Day.values()) {
            System.out.println(day + "의 ordinal 값: " + day.ordinal());
        }
    }
}
```

<hr>

## ✔️ ordinal의 단점

- 유지보수하기 어렵다
  - 상수 선언 순서를 바꾸면 오작동한다
  - 이미 사용 중인 정수와 값이 같은 상수를 추가 불가능
- 중간에 값을 비워둘 수 없다

<hr>

## ✔️ 그렇다면 해결책은?

**열거 타입 상수에 연결된 값을 ordinal 메서드로 얻는 대신, <u>인스턴스 필드에 저장하자</u>**

```java
enum Day {
    MONDAY(1), TUESDAY(2), WEDNESDAY(3), THURSDAY(4),
    FRIDAY(5), SATURDAY(6), SUNDAY(7);

    private final int dayNumber;

    Day(int dayNumber) {
        this.dayNumber = dayNumber;
    }

    public int getDayNumber() {
        return dayNumber;
    }
}

public class EnumFieldExample {
    public static void main(String[] args) {
        for (Day day : Day.values()) {
            System.out.println(day + "의 번호: " + day.getDayNumber());
        }
    }
}
```

위와 같은 형식으로 인스턴스 필드를 활용하도록 코드를 작성하면 문제를 해결할 수 있다! 