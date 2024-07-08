# [JAVA] What's the diff between Comparable and Comparator?

코딩테스트 정렬 문제를 풀고 있었다. 당시 필자인 나 정렬이라면 그냥 무턱대고 사용하던 `sort()` 메소드 뿐이였다.
그런데 유일하게 사용할 줄 알았던 `sort()`에 대해서 어떻게 구동이 되는지 등 모르면서 사용을 했었다.   

누군가 해당 메소드에 관한 질문을 한다면? 라는 생각이 어쩌다가 들었는데 내 자신이 너무 한심한거 같았다.

내가 사용하는 메소드에 대하여 내가 모르다니...

그래서 관련하여 찾아보다가 `Comparable<T>`와 `Comparator<T>`에 대하여 찾아보게 되었다.

지금 알아볼 두 인터페이스는 #"객체를 비교할 수 있게 해준다."#라고 한다. 

나는 처음에 이게 무슨 뜻이지?? 하면서 완벽한 이해는 하지 못 했다.
```java
    int a = 10, b = 8;

    System.out.println(a>b); // true
```
하지만 기본형 타입을 비교를 할 때는 대부분 부등호를 사용했다.
그런데 문자열은 부등호 사용이 가능한가?를 생각했을 때 앞에서 왜 그런말을 했는지를 이해 했다.   
(물론 문자열을 비교를 한다면 이는 사전순으로 비교를 한 것이다.)

#### 그렇구나 내가 직접 만든 클래스를 비교를 할 때 사용을 하는구나...



## Comparable

[Comparable 공식 문서](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html#method.summary)

`Comparable<T>`은 자바에서 객체의 자연적인 순서를 정의하기 위한 인터페이스다.   
다시 이야기를 한다면 해당 인터페이스는 정렬 가능한 클래스를 만들기 위해 구현하는 인터페이스입니다.   
이 인터페이스를 구현한 클래스는 정렬 시 자신과 매개 변수객체를 비교할 수 있는 방법을 제공한다.   

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

- 동일 객체 타입 비교 시

  - 양수 → 크다
  - 음수 → 작다
  - 0 → 같다

- 동작
  `Comparable`인터페이스의 `compareTo()`메소드를 사용하여 간단하게 `Person`클래스를 사용하여 나이 기준으로 정렬

```java
public class Person {
  private String name;
  private int age;
  private int classNum;
}
```   

위 코드를 보면 이름, 나이, 학급 번호가 있다.   
만약 나 자신과 Person_A 두 객체가 있고 각 객체에는 각각의 정보가 입력이 되어있다. 이 때 두 객체를 비교할 때 사용할 `compareTo()`메소드를 재정의를 해줘야 한다.    

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;
    private int classNum;
    
    // 생성자, getter, setter 등
    
    @Override
    public int compareTo(Person person) {
        return this.age - person.age; // 나이를 기준으로 정렬
      /*
              양수    =>   크다
              0      =>   같다
              음수    =>   작다
       */
    }
}
```
위 처럼 해당 인터페이스의 메소드를 정의하면서 어떠한 값을 기준으로 비교를 하는지 명시해주면 된다.

이렇게 먼저 자신을 매개변수와의 비교를 하는 `Comparable<T>` 인터페이스를 확인 했다.

## Comparator

[Comparator 공식 문서](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#method.summary)    

`Comparator<T>` 인터페이스는 두 개의 객체를 비교하는 메서드를 정의한 인터페이스이다.   
이를 사용하여 정렬 기준을 유연하게 변경할 수 있다는 점이 존재한다.   

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

바로 앞에서 확인한 `compareTo()`메소드와 달리 두 개의 매개값이 필요하다. 왜냐하면 앞에서는 나와 매개변수를 비교를 했다면, 
여기서는 매개변수 끼리의 비교를 하기 때문이다.

```java
public class Person implements Comparator<Person> {
    private String name;
    private int age;
    private int classNum;
    
    // 생성자, getter, setter 등
    
    @Override
    public int compare(Person person1, Person person2) {
        return person1.classNum - person2.classNum; // 학급 번호를 기준으로 정렬
    }
}
```

`Comparable<T>`의 `compareTo()`는 자신이 매개변수보다 크거나 작거나를 비교를 한다면, 
`Comparator<T>`의 `compare()`는 첫 매개변수가 다음에 오는 매개변수 보다 크거나 작거나를 비교한다.

 그래서 두 매개변수를 비교하는데 있어서 자신은 비교에 있어서 끼치는 영향은 없다.

```java
// A, B, C 가 있다고 가정을 하자
int ac = b.compare(a,c);
// b는 a와c비교에 있어서 아무런 영향이 없다.
```

그리고 정렬하는데 있어서 어쩌다가 한 번 사용하게 될 경우라면 익명 클래스로 재정의하여 사용할 수 있다.

```java
        Collections.sort(people, new Comparator<Person>() {
            @Override
            public int compare(Person p1, Person p2) {
                return p1.getAge() - p2.getAge();
            }
        });
```

---
$Reference$   
[자바 [JAVA] - Comparable 과 Comparator의 이해](https://st-lab.tistory.com/243)   
[Comparator 와 Comparable - JAVA](https://dding9code.tistory.com/68)

