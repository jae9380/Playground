# [JAVA] What's the diff between Comparable and Comparator?

## Comparable

[Comparable 공식 문서](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html#method.summary)

`Comparable<T>`은 자바에서 객체의 자연적인 순서를 정의하기 위한 인터페이스다.

`Comparable`은 자바에서 객체의 자연적인 순서를 정의하기 위한 인터페이스다. 또한 구현한 객체는 자신이 정의한 자연적인 순서에 따라서 정렬이 가능하다.

- 동일 객체 타입 비교 시

  - 양수 → 크다
  - 음수 → 작다
  - 0 → 같다

- 동작
  `Comparable`인터페이스의 `compareTo()`메소드를 사용하여 간단하게 `Person`클래스를 사용하여 나이 기준으로 정렬

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Person otherPerson) {
        return Integer.compare(this.age, otherPerson.age);
    }

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>();
        people.add(new Person("Alice", 30));
        people.add(new Person("Bob", 25));
        people.add(new Person("Charlie", 35));

        System.out.println("Before sorting: " + people);

        Collections.sort(people);

        System.out.println("After sorting: " + people);
    }
}

```

해당 인터페이스가 구현하고 있는 `compareTo()`메소드를 재정의하여 사용을 한다. 또한 해당 메소드는 현재객체(`this`)와 동일한 타입의 객체를 비교하여 순서를 결정하는 정수를 반환을 해준다.

## Comparator
