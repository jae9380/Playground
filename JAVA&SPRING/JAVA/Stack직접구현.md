# Stack 직접 구현하기

## Stack이란?
사전적인 의미로 스택은 "샇다"라는 의미를 갖고 있다. 그리고 지금 우리가 직접 구현할 Stack이라는 자료 구조는 `LIFO`이라고 칭한다. 해당 구조는 흔하게 후입선출이라고 생각하면 이해하기 쉽다.    

> **후입선출**
> 추가될 것을 가장 뒤에 추가를 하고, 요소를 하나 뽑을 때 가장 뒤에 것을 뽑는 것을 의미한다.

## Java에서 제공되는 Stack
`Java`에서는 기본적으로 `Stack` 자료 구조를 지원하고 있다. 그렇기 떄문에 직접 구현하기 전에 `Stack`은 어떻게 구성되어 있는지를 알아야 조금 더 비슷하게 구현할 수 있을 것이라고 생각이 되어서 간단하게 알아보자

일단 자바에서 `Stack`을 선언해서 내부를 확인을 해야겠다.
```java
Stack<Element> stack = new Stack();
```
위 처럼 선언을 하고 들여보면 아래와 같이 메서드가 있다는 것을 확인할 수 있다.
    
![](https://i.postimg.cc/0QxQY5xh/2025-05-12-14-47-54.png)

해당 자료 구조에서 지원하는 메서드는 `push()`, `pop()`, `peek()`, `empty()`, `search()` 총 5개가 있다.     

* push( )
  ```java
  public E push(E item) {
  	addElement(item);
  
  return item;
  }
  ```
  해당  메서드를 통하여 **스택의 가장 상위 부분에 데이터를 추가**하게 된다.  추가하는 과정에서 내부적으로 `Vector`클래스의 메서드인 `addElement()`을 호출한다.

* pop( )
  ```java
  public synchronized E pop() {
    E obj;
    int len = size();
    obj = peek();
    removeElementAt(len - 1);
    return obj;
  }
  ```
  스택의 **맨 위에 있는 객체를 제거하고**, 그 객체를 반환한다. 만약 스택이 비어 있을 경우에 `EmptyStackException`이 발생하게 된다고 설명하고 있다.
  * **반환값**: 스택의 가장 위에 있는 객체

* peek( )
  ```java
  public synchronized E peek() {
    int len = size();
  
    if (len == 0) throw new EmptyStackException();
  
    return elementAt(len - 1);
  }
  ```
  스택의 맨 위에 있는 객체를 **제거하지 않고 반환한다**. `pop()`처럼 스택이 비어 있을 경우 동일한 예외를 발생 시킨다.

* empty( )
  ```java
  public boolean empty() {
    return size() == 0;
  }
  ```
  해당 메서드는 메서드명 처럼 스택이 비어있다면 참, 그렇지 않을 경우 거짓을 반환하는 기능을 한다.

* search( )
  ```java
  public synchronized int search(Object o) {
    int i = lastIndexOf(o);
  
    if (i >= 0) {
        return size() - i;
    }
    return -1;
  }
  ```
  이는 객체가 스택에서 **어디에 위피해 있는지 1부터 시작하는 기준으로 반환**한다. 쉽게 말하여 스택 상위 1에서 시작하여 특정 객체까지 어느정도 떨어져 있는지 거리를 반환한다. 하지만 **찾지를 못하면 -1을 반환** 한다.
  그리고 객체 비교는 `equals()`를 사용하여 비교를 하게 된다고 설명을 한다.

## Stack 직접 구현
위에서 전반적인 `Stack`의 기능들을 확인을 했고, 이제 이를 바탕으로 해당 기능을 직접 만들어 볼 것이다.     

`Stack`을 구현하는 방법으로 크게 두 가지 방법이 있다. 구현 방법으로는 **배열을 이용하는 방법**과 **리스트를 이용하는 방법**으로 쉽게 만들 수 있다.     

#### 1. 배열(Array)을 활용한 Stack 구현

배열을 사용하는 경우, 우리는 다음과 같은 정보들을 직접 관리해야 한다
* 데이터를 담을 배열 (int[] stackArray 또는 제네릭)
* 현재 `Stack`의 **상단(top)**을 가리키는 인덱스 변수 (int top)
* 스택의 최대 크기 (int capacity)

**힌트:**
* 데이터를 넣을 때는 top변수를 증가시키며 배열에 값을 넣어야 한다.
* 데이터를 뺄 때는 top변수를 감소시키며 가장 마지막에 추가한 데이터를 반환해야 한다.
* `peek()`는 제거 없이 현재 top변수의 위치 값을 반환해야 한다.
* `isEmpty`는 top == -1 일 때 참

추가적으로 배열을 이용하여 구현을 했을 때, 배열이란 자료 구조는 최초에 선언된 길이를 수정하지 못 하기 때문에 `isFull()` 같은 메서드를 만들어 예외상황을 막을 수 있다.

배열 기반은 메모리 낭비 없이 빠르게 접근 가능하지만, 크기가 고정된다는 단점이 있다.

⸻

#### 2. 리스트(List)를 활용한 Stack 구현

`ArrayList`나 `LinkedList` 같은 리스트를 사용하는 경우, 배열과 다르게 리스트의 길이가 늘어나거나 줄어들기 때문에 top 위치에 대한 인덱스를 신경 쓸 필요가 없다.

**힌트:**
* `push()` 시에는 리스트의 끝(list.add(value))에 요소를 추가한다.
* `pop()`은 리스트의 마지막 요소를 제거하고 반환하면 된다. (list.remove(list.size() - 1))
* `peek`도 리스트의 마지막 요소를 참조하면 된다. (list.get(list.size() - 1))

리스트 기반은 크기를 동적으로 조절할 수 있어 유연하지만, 내부적으로 배열 복사가 일어나거나 구조에 따라 속도 차이가 있을 수 있다.