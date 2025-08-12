# Stream

![](~https://velog.velcdn.com/images/jae9380/post/a04cc0e0-8ba3-42a6-b59a-5f1aeeed6f8b/image.png~)


지금까지 자바를 공부하면서 아주 갈략하게 스트림에 대하여 읽었던 기억이 있다. 하지만 이 기능을 자주 사용하지 않았다.   

## **🧐 Stream은 뭘까??**

컬렉션(List, Set 등)의 데이터를 좀 더 간결하고 선언적으로 처리할 수 있다면 얼마나 좋을까?   
기존 방식으로는 특정 동작을 위해서 반복문과 조건문 등 여러 코드를 직접 작성하면서 단계별 동작의 코드가 길어지기 쉬웠다.    

자바 8 이후 **Stream**이라는 강력한 기능이 추가되었다. 이런 작업을 마치 데이터 파이프라인처럼 흐름으로 처리할 수 있게 되었다.   

예를 들어서 칼로리가 400 미만인 음식을 뽑는 예제 코드를 보자.

처음 확인하는 예제 코드의 경우 자바 7 방식으로 작성되었다.
```java
List<Dish> lowCaloricDishes = new ArrayList<>();
for(Dish dish : menu) {
    if(dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
    }
}

Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
    public int compare(Dish dish1, Dish dish2) {
        return Integer.compare(dish1,getCalories(), dish2.getCalories());
    }
});

List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish : lowCaloricDishes) {
    lowCaloricDishesName.add(dish.getName());
}
```

이제 자바8에 추가된 `Stream`을 이용을 해보겠다.

```java
List<String> lowCaloricDishName =
        menu.stream()
            .filter(d -> d.getCalories() < 400)
            .sorted(comparing(Dishes::getCalories))
            .map(Dish::getName)
            .collect(toList());
```

확실하게 7 버전의 경우와 다르게 훨씬 짧고 명확해 보인다.

이 처럼 **Stream**을 사용하게 된다면 반복문 없이 선언형 코드로 처리가 되며, 전체적인 흐름을 쉽게 확인할 수 있다는 장점을 얻을 수 있다.    

스트림을 이용하게 된다면 스트림의 새로운 기능이 소프트웨어공학적으로 다양한 이득을 안겨준다.    
선언형으로 코드를 구현할 수 있다는 점과 여러 빌딩 블록 연산을 연결하여 복잡한 데이터 처리 파이프 라인을 만들 수 있어 가독성과 명확성을 유지한다는 장점이 있다.

`filter`와 같은 연산은 ****고수준 빌딩 블록(high-level building block)**** 으로 이루어져 있어 특정한 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서도 사용할 수 있다.

> #### **고수준 빌딩 블록**
> <br>
> 복잡한 시스템이나 프로그래밍을 구성하는 기본 구성요소들 중 추상화 수준이 높은것들을 의미한다.   
> 추상화, 재사용성, 조합 가능성, 가독성 이렇게 총 4가지의 특성을 갖고 있다.

## Stream 

그런데 **스트림**은 어떻게 정의할 수 있을까?    

> 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소   

이 처럼 정의할 수 있는 이유에 대해서 확인을 하자.   

* 연속된 요소
  컬렉션과 마찬가지로 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 지원한다.  컬렉션은 자료구조이므로 **컬렉션에서는 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룬다.**  반면에 **스트림의 경우 `filter`,`sorted`,`map`처럼 표현 계산식이 주를 이루고 있다.**     
  즉, 다시 말하면 **컬렉션의 경우 주제는 데이터**이고, **스트림의 경우 주제는 계산**이라고 보면 된다.   

* 소스
  스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로 부터 데이터를 소비한다.  정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지가 된다. 즉, 리스트로 스트림을 만들게 된다면 스트림의 요소는 리스트의 요소와 같은 순서를 유지한다.

* 데이터 처리 연산
  스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터 베이스와 비슷한 연산을 지원한다. 

또한 아래와 같은 중요 특징이 두 가지 존재한다.    

- 파이프라이닝
  ![](https://i.postimg.cc/6qgBCZWH/scode-mtistory2-fname-https-blog-kakaocdn-net-dn-Cj-Yji-btr-VOg-GQe-Za-d-O8-FP8-CKXQr-Hfn-Jw-PBk-Uv1-img.png)   

  대부분의 스트림 연산은 스트림 연산끼리 연결하여 커다란 파이프 라인을 구성할 수 있도록 스트림은 스트림 자신을 반환한다. 이러한 부분 때문에 `Laziness`, `short - circuiting`같은 최적화도 얻을 수 있다.

- 내부 반복
  반복자를 사용하여 명시적으로 반복하는 컬렉션과 달리 **스트림은 내부 반복을 지원**한다. 

### **외부 반복과 내부 반복**

컬렉션 인터페이스를 사용하려면 사용자가 직접 요소를 반복해야 한다.  
이를 **외부 반복**이라고 한다. 반면에스트림 라이브러리는 **내부 반복**을 사용을 한다.

```java
List<String> names = new ArrayList<>();
for(Dish dish : menu) {
    names.add(dish.getName());
}
```

`for-each`구문은 반복자를 사용하는 불편함을 어느 정도 해결해준다.  
`for-each`를 이용하면 `Iterator`객체를 이용하는 것보다 더 쉽게 컬렉션을 반복할 수 있다.

```java
List<String> names = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while(iterator.hasNext()) {
    Dish dish = iterator.next();
    names.add(dish.getName());
}
```

아래와 같이 스트림 내부 반복을 이용

```java
List<String> names = menu.stream()
                    .map(Dish::getName)
                    .collect(toList());
```

스트림 라이브러리의 내부 반복은 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다. 반면에 `for-each` 를 이용하는 외부 반복에서는 병렬성을 스스로 관리를 해야한다.

## **스트림의 연산**

스트림 인터페이스의 연산은 크게 ****중간 연산****과 ****최종 연산**** 두 가지로 구분할 수 있다.

- #### **중간 연산**
  중간 연산은 다른 스트림을 반환을 한다. 따라서 여러 중간 연산을 연결해서 질의를 만들 수 있다. 중간 연산의 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지 아무 연산도 수행하지 않는다는 것, 즉 게리르다는 것이다.

| 연산     | 형식      | 반환 형식   | 연산의 인수      | 함수 디스크립터 |
| -------- | --------- | ----------- | ---------------- | --------------- |
| filter   | 중간 연산 | `Stream<T>` | `Predicate<T>`   | T -> boolean    |
| map      | 중간 연산 | `Stream<R>` | `Function<T, R>` | T -> R          |
| limit    | 중간 연산 | `Stream<T>` |                  |                 |
| sorted   | 중간 연산 | `Stream<T>` | `Comparator<T>`  | (T, T) -> int   |
| distinct | 중간 연산 | `Stream<T>` |                  |                 |

- #### **최종 연산**
  최종 연산은 스트림 파이프라인에서 결과를 도출한다. 보통 최종 연산에 의해 `List`, `Interger` 등 스트림 이외의 결과가 반환된다.

| **연산** | **반환 형식** | **설명** |
|:-:|:-:|:-:|
| **forEach** | void | 각 요소에 대해 주어진 동작 수행 (순서 보장: forEachOrdered()) |
| **count** | long | 스트림의 총 요소 개수 반환 |
| **collect** | Collector 결과 | 결과를 리스트, 맵, 집합 등 컬렉션으로 수집 (Collectors.toList() 등) |
| **reduce** | 단일 값 (int, long, 객체 등) | 누적 연산을 통해 하나의 결과로 축약 (예: 합계, 곱, 최대값 등) |
| **min** | Optional<T> | 최소값 반환 (Comparator 필요) |
| **max** | Optional<T> | 최대값 반환 (Comparator 필요) |
| **anyMatch** | boolean | 하나라도 조건을 만족하면 true |
| **allMatch** | boolean | 모두 조건을 만족하면 true |
| **noneMatch** | boolean | 아무것도 조건을 만족하지 않으면 true |
| **findFirst** | Optional<T> | 첫 번째 요소 반환 (순차 스트림에서 주로 사용) |
| **findAny** | Optional<T> | 임의의 요소 반환 (병렬 스트림에서 빠른 반환 가능) |
| **toArray** | Object[] 또는 제네릭 배열 | 스트림의 요소를 배열로 변환 |


## 스트림 사용하기
앞에서 스트림에 대한 내용을 이야기 했다. 그리고 이제 스트림을 어떻게 사용하는지 알아보자.   

* 빈 스트림 생성
```java
Stream<Object> emptyStream = Stream.empty();
```

* 값을 넣어 생성하기
```java
Stream<String> stream = Stream.of("Java", "Python", "Go");
```

* Builder를 사용하여 생성
```java
Stream<String> stream = Stream.<String>builder()
                              .add("a")
                              .add("b")
                              .add("c")
                              .build();
```

* 컬렉션에서 생성하기
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream(); 
```

* 배열에서 스트림 생성하기
```java
int[] numbers = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(numbers);

String[] strArr = {"A", "B", "C"};
Stream<String> stringStream = Stream.of(strArr);
```

* 파일에서 스트림 생성하기
```java
Path path = Paths.get("sample.txt");
try (Stream<String> lineStream = Files.lines(path)) {
    lineStream.forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}
```

이 처럼 다양한 방법으로 스트림을 생성할 수 있다. 이렇게 생성된 스트림은 **중간 연산**과 **최종 연산**의 구조로 구성하여 사용하면 된다.  

참고 : [모던 자바 인 액션](https://www.hanbit.co.kr/store/books/look.php?p_code=B4926602499)