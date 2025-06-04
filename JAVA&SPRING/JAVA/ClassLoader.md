#Java #CalssLoader
# *CalssLoader*   
## CalssLoader? 그게 뭐 하는 놈인디…
![](https://i.postimg.cc/YCvfQ5G5/388da08665aa77ed364175496b337696.jpg)
### CalssLoader의 개념

자바에서 사용되는 **CalssLoader**는 클래스 파일(.class) 파일을 읽어 JVM의 Method Area에 적재하는 역활을 한다.  
즉, 우리가 .java 파일을 작성하고 컴파일한 .class 파일이 **실제로 JVM 메모리에 올라가서 사용될 수 있도록** 하는 **입구 역할**을 한다. 


> ## **🕵What do the .java & .class files mean??**
**CalssLoader** 개념을 이야기 할 떄 .java파일과 .class파일이 나온다. 그런데 해당 파일들은 무엇인지 모를 수 있다.  그렇기 때문에 간단하게 확인하고 넘어가면 좋을 것이다.

만약 자바 개발을 해봤다면 .java 파일과 .class파일은 한 번쯤 보았을 것이다. 여기서 자바 파일은 **자바 언어를 사용하여 소스코드를 작성할 때 소스코드가 작성된 파일**을 의미한다. 

![](https://i.postimg.cc/R0VZgpkC/scode-mtistory2-fname-https-blog-kakaocdn-net-dn-XL21-E-btrt-H06o7v-B-t-Yg5-SKuqvr-Ru-Qlo-Yq-Byy4-K-img.png)

그 다음으로 클래스 파일은 작성된 **자바 파일을 컴파일**하게 되면 생성되는 파일이 클래스 파일 또는 자바 바이트 코드라고 부른다. 이 파일은 JVM만 설치되어 있으면, 언제 어디에서 JAVA언어로 작성된 파일로 실행할 수 있다.


## **🤔 그러면 JVM에 어떤 과정으로 적재 할까?**

해당 동작 과정은 자바를 이해하는데 있어 아주 핵심적인 개념이다. 아래의 단계는 **JVM이 클래스를 로드 하는 과정**이다.

JAVA는 **동적 로딩(Dynamic Loading)**을 한다는 특징이 있다. 이는 **동적 로딩을 담당하는 부분**이 **JVM**에서 클래스 로더이다.     
즉, 클래스 로더는 JAVA로 작성된 프로그램의 런타임 중 **JVM**의 Method Area영역에 동적으로 자바 클래스를 적재하는 역활을 하는 것이다. 

적재하는 과정에서 크게 **Loading**, **Linking**, **Initialization**을 거치게 된다.

![](https://i.postimg.cc/K83NRQqz/1-Bs-LZHUe-5-Qv2tb-Hdz-U-TQ.webp)

### 1. Loading (클래스 로딩)

클래스 파일을 찾아서 메모리로 불러온다. 불러들인 클래스 파일은 적절한 바이트 코드로 만들어 **Method Area**에 저장을 하게 된다. 

여기서 메모리로 불러오는 역활은 **Class Loader**가 수행하게 된다. 이 **Class Loader**는 **부모-자식 형태의 계층 구조**로 설계되어 있으며, 로딩 과정에서 **“부모 먼저 검색”** 원칙을 따릅니다. 이를 **Parent Delegation Model**이라 부릅니다.

```
// before java version 8.
[Bootstrap ClassLoader]
        ↓
[Extension ClassLoader]
        ↓
[Application (System) ClassLoader]

// after java version 9.
[Bootstrap ClassLoader]
        ↓
[Platform ClassLoader]   ← Java 9 신설!
        ↓
[System (Application) ClassLoader]
```

자바 9 버전 이후로 부터모듈 시스템(Jigsaw)이 도입 되면서 **Platform ClassLoader**와 **System ClassLoader**로 구조가 바뀌었다.

_**Class Loader**에 대한 자세한 정보는 뒤에서 알아보자_

### 2. Linking (링킹)

**Linking**에서 3단계를 과정을 거치게 된다. 각 단계는 **Verification**, **Preparation**, **Resolution**으로 정의되어 있다.

* Verification   
  **JVM**이 읽어온 클래스 파일이 유효한 바이트 코드인지 검증한다.

* Preparation    
  클래스에 선언된 **static** 변수에 기본값을 할당한다.    

* Resolution    
  다른 클래스나 메서드 필드 등에 대한 심볼릭 참조에서 실제 참조로 변경한다.    


### 3. Initialization (초기화)

**Initialization** 단계에서 클래스의 **static**초기화 블럭과 **static** 변수에 실제 코드에서 정의된 값들이 할당하게 된다.    
즉, `static {...}`블럭과 `static 변수 = 값`같은 코드가 실행되는 시점이다.    

이 과정을 거친 후, 클래스는 **JVM**의 **Method Area**에 완전히 준비된 상태로 존재하게 되며, 해당 클래스를 사용한 객체 생성이나 메서드 호출이 가능해진다.


최종적으로 자바 클래스는 **JVM**에서 **Loading** -> **Linking** -> **Initialization** 과정을 통해서 메모리에 적재된다.    

**JVM**은 프로그램이 실행되는 도중에도 필요한 클래스를 동적으로 로드라고 실행할 수 있다.

이러한 동적 로딩 구조는 자바의 유연성과 확장성을 높이는 핵심적인 설계이자, **JVM**이 메모리와 클래스 관리를 효율적으로 수행하는 기반이 된다.    

---

## Class Loader

일단 자바 버전울 기준으로 어떻게 변경되었는지 확인을 하자    

#### 자바 8 이전의 Class Loader

1. Bootstrap ClassLoader
   * 가장 최상위 로더
   * JVM에 내장되어 있어 **자바로 구현되지 않고 C/C++로 작성됨**
   * rt.jar 등 **Java 핵심 클래스**(예: java.lang.*, java.util.*)를 로드
   * null 값으로 나타남 (ClassLoader.getSystemClassLoader().getParent() → null)

2. Extension ClassLoader (≒ Platform)
   * JAVA_HOME/lib/ext 디렉토리에 있는 JAR 파일들을 로드
   * 확장 기능이나 보조적인 라이브러리를 여기에 넣어 사용함
   * 사용자 커스터마이징 확장을 위한 용도로 제공됨

3. Application (System) ClassLoader
   * 우리가 만든 클래스(.class), 외부 라이브러리(JAR) 등을 **classpath**로부터 로드
   * 가장 일반적으로 사용되는 클래스 로더

#### 자바 9 이후의 Class Loader

1. Bootstrap ClassLoader
   * 동일하게 Java 핵심 클래스 로드
   * java.base 모듈의 클래스들이 여기에서 로드됨

2. Platform ClassLoader (💡 New in Java 9)
   * **기존 Extension ClassLoader 대체**
   * java.sql, java.xml, java.logging 등 JDK의 일부 **표준 플랫폼 API 모듈**들을 로드
   * 사용자 애플리케이션 코드와는 별개의 클래스 로딩 책임을 가짐

3. System (Application) ClassLoader
   * 사용자 코드와 라이브러리들을 로딩 (classpath에 있는 클래스)
   * Java 8과 동일한 기능이지만, 상위가 Platform ClassLoader로 변경됨

> #### 버전 8 전과 버전 9 이후 상위에 있는 Bootstrap Class Lodaer은 같은 Bootstrap Class Loader인가요? 🤔🤔

애석하게도 완벽하게 같다고 말할 수 없다. 

그러면 어떤 부분에서 차이점을 보이는 것일까? 

**Java 8 이전과 Java 9 이후의 Bootstrap ClassLoader는 기본적인 역할과 개념은 동일합니다.**
**그러나** Java 9부터 도입된 **모듈 시스템 (JPMS)** 때문에 **동작하는 방식이나 로딩하는 대상이 약간 달라졌습니다.**

최상위 클래스 로더로서 다음과 같은 동일한 작업을 수행한다.

| **역할** | **설명** |
|:-:|:-:|
| **핵심 API 클래스 로드** | java.lang, java.util 등 가장 기본이 되는 클래스 로딩 |
| **JVM 내장** | C/C++로 구현되어 있고, 자바 코드에서 직접 다룰 수 없음 |
| **부모 위임 구조의 최상단** | 모든 클래스 로더의 부모에 해당 (parent == null) |

### ✅ Java 8 이전의 특징

Java 8 이전의 **Bootstrap Class Loader**는 JAVA_HOME/jre/lib/rt.jar 경로에 위치한 **rt.jar** 파일 내의 클래스를 로드합니다.
대표적으로 java.lang.String, java.util.HashMap 등 **JDK의 핵심 클래스들**이 이에 해당됩니다.

### ✅ Java 9 이후의 변화

Java 9부터는 **모듈 시스템(JPMS: Java Platform Module System)**이 도입되면서 클래스 로딩 방식에 큰 변화가 생겼습니다.

기존에는 rt.jar 파일에 포함된 모든 핵심 클래스를 Bootstrap ClassLoader가 로딩했지만, Java 9부터는 **rt.jar 파일이 더 이상 존재하지 않으며**, 대신 **JDK 내부가 모듈 단위로 구성**되어 lib/modules에 위치한 **jimage 파일** 안에서 모듈별로 클래스들을 관리하고 로딩하게 되었습니다.    


### Class Loader 동작 방식    

자바에서 클래스는 프로그램이 실행될 때 **필요한 시점에서 동적**으로 **JVM** 메모리에 로드된다고 했가. 그리고 클래스 로딩은 **Parent Delegation Model**이라는 방식을 따른다고 했다.    

> **😭 Parent Delegation Model은 또 뭐야…ㅠㅠ**

클래스 로더가 클래스를 로드할 때 **자신보다 상위 클래스 로더에거 먼저 요청**을 한다. 
상위가 해당 클래스를 모를 때 자신이 직접 로드를 하게 된다.   

해당 과정을 거치게 되면 **보안성이 높아지고**, 핵심 JAVA API가 오염되지 않도록 보호할 수 있다.

  ```java
  ClassLoader cl = MyClass.class.getClassLoader();
  System.out.println(cl); // AppClassLoader
  System.out.println(cl.getParent()); // PlatformClassLoader
  System.out.println(cl.getParent().getParent()); // null (Bootstrap은 네이티브로 구현되어 있음)
  ```

#### 클래스 로딩의 예외 상황

**Parent Delegation Model**은 일반적으로 보안상 유리하지만, 몇몇 경우에는 이 원칙이 지켜지지 않는다.    

대표적으로 두 상황이 있다.   
* Web Application 서버
  여러 웹 어플리케이션이 동시에 실행될 수 있도록, 각 앱마다 별도의 클래스 로더를 사용하며, 일부 클래스를 직접 로딩하기도 한다.

* Custom ClassLoader 사용 시
  특정 라이브러리를 격리하거나 동적으로 코드를 로딩할 필요가 있는 경우, **Class Loader**를 직접 구현해서 로딩 방식을 바꾸기도 한다.


### Custom Class Loader    

자바의 **Class Loader**를 상속받아 **사용자 정의 방식으로 클래스를 로딩**할 수 있도록 만든 클래스이다.    
자바의 클래스 로딩 방식은 보안성과 안정성은 뛰어나지만, 제한적인 유연성 때문에 다음과 같은 경우에 직접 **Class Loader**를 정의해야 한다.    

* 플러그인 시스템 
  동적으로 플로그인 클래스를 로드 및 언로드하고, 서로 다은 클래스 로더에서 독립적으로 실행되도록 구성

* 클래스 버전 격리
  서로 다른 버전의 라이브러리를 동시에 사용해야 하는 경우

* 암호화된 클래스 파일 로딩
  클래스 파일을 암호화해서 배포한 후 로딩 시 복호화하여 실행

* 테스트 및 리플렉션 도구
  동적으로 생성된 바이트 코드를 로드하거나 런타임에 수정한 클래스를 주입  

기본적으로 커스텀 클래스 로더를 작성할 때 `Class Loader`클래스를 상속받아 `findClass(Strinf name)` 메서드를 오버라이딩 해야 한다. 대부분 `defineClass`메서드를 사용하여 바이트 배열을 객체로 변환한다.

```java
public class MyClassLoader extends ClassLoader {

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            // 클래스 이름을 기반으로 .class 파일을 읽어 byte[]로 변환
            String path = "C:/classes/" + name.replace('.', '/') + ".class";
            byte[] classData = Files.readAllBytes(Paths.get(path));
            // byte[]를 Class로 변환
            return defineClass(name, classData, 0, classData.length);
        } catch (IOException e) {
            throw new ClassNotFoundException(name, e);
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) throws Exception {
        MyClassLoader loader = new MyClassLoader();
        Class<?> clazz = loader.loadClass("com.example.Hello");
        Object obj = clazz.getDeclaredConstructor().newInstance();
        Method method = clazz.getMethod("sayHello");
        method.invoke(obj); // Hello 출력
    }
}
```

#### Parent Delegation Model     
```java
@Override
public Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    if (name.startsWith("com.plugin")) {
        return findClass(name); // 직접 로딩
    }
    return super.loadClass(name, resolve); // 부모에게 위임
}
```

-----   