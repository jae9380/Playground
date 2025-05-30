# 🧐 JAR vs. WAR: What's the Difference??

프젝트를 생성하는데 있어서 지금까지 아무런 관심없이 패키징 부분에서 `Jar`를 체크해왔다.

그런데 `Jar`와 `War`이 두개는 어떠한 차이점이 있을까에 대한 의문이 생겼다.

![](https://i.postimg.cc/bNpN3R6z/jar-war.png)

## Packaging

`Jar`와 `War` 개념에 대해서 알아보기 전에 Packaging은 무엇인지 알아보는게 좋겠다고 생각이 들었다.

`Packaging`은 소프트웨어 개발 과정에서 작성된 코드나 리소스 파일, 설정 파일 등을 하나로 묶어서 배포가 가능하고 실행이 가능한 형태로 만드는 과정으로 소프트웨어 배포 및 실행 등 관리를 보다 쉽게 만들어 준다.

이 처럼 패키징을 통하여 모든 파일을 하나로 묶어 배포를 함으로 환경에 상관없이 일관된 형태로 소프트웨어를 제공할 수 있어 일관성 유지에 이점이 있다.  
또한 패키징을 통하여 배포에 있어서 용이점이 있으며, 필요한 라이브러리나 설정파일을 함께 패키징을 하기에 의존성 관리 측면에서도 이점을 챙길 수 있다.

## Jar (Java Archive)

JAVA 어플리케이션 및 라이브러리를 패키징을 하는데 사용이된다. `Jar`파일은 여러 JAVA클래스 파일 및 관련 리소스를 하나로 압축된 파일로 묶어 배포 및 실행을 간단하게 해준다.  
일반적으로 어플리케이션 배포에 있어 하나의 `Jar`파일에 모든 클래스를 포함시켜 어플리케이션을 배포를 하며, 다른 어플리케이션에서 사용할 수 있는 라이브러리나 프레임 워크를 배포할 때 `Jar`로 묶어서 배포를 한다.

독립형 자바 응용 프로그램 뿐 아니라 다른 자바 프로그램에서 또한 사용이 가능할 수 있는 라이브러리 파일에도 사용된다.

### 구조

`Jar`파일은 기본적으로 Zip 형식의 아카이브 파일이다.

```arduino
myapp.jar
├── META-INF/
│   └── MANIFEST.MF
├── com/
│   └── example/
│       ├── MyClass.class
│       └── AnotherClass.class
└── resources/
    └── config.properties

```

`META-INF`는 메타데이터를 포함하는 디렉토리이다. 해당 디레토리 안에 위치하는 `MANIFEST.MF`파일은 `Jar` 파일의 메타데이터가 내포되어 있다.

`jar` 명령어를 사용하여 파일을 생성할 수 있으며, 실행은 `java -jar`명령어를 사용한다.

## War (Web Archive)

`War`파일은 웹 어플리케이션을 패키징을 하는데 사용이 된다. `Jar`파일과 유사하지만, 웹 어플리케이션에 필요한 추가적인 구조와 설정 파일을 포함한다.  
해당 방법은 웹 서버나 서블릿 컨테이너(예를 들어 Apache Tomcat)에 배포하기 위해 모든 웹 관련 자원을 하나로 묶어서 배포를 한다, 그리고 `War`파일은 특정 디렉토리 구조를 가지고 있어, 웹 어플리ㅔ이션이 올바르게 동작할 수 있도록 설정한다.

### 구조

`War`은 아래와 같은 디렉토리 구조를 갖는다.

```vbnet
myapp.war
├── META-INF/
├── WEB-INF/
│   ├── web.xml
│   ├── classes/
│   │   └── com/example/MyServlet.class
│   ├── lib/
│   │   └── some-library.jar
│   └── views/
│       └── index.jsp
├── css/
│   └── styles.css
├── js/
│   └── script.js
└── images/
    └── logo.png
```

`META-INF`파일 또한 메타데이터를 포함하는 디렉터리입니다. 그리고 `WEB-INF`은 웹 어플리케이션의 핵심 파일이 포함된 디렉토리이다. 내부에 있는 `web.xml` 파일은 웹 어플리케이션의 설정에 대한 정보다 들어있다.

## Jar와 War의 차이점

| 요소       | JAR                                  | WAR                                             |
| ---------- | ------------------------------------ | ----------------------------------------------- |
| 용도       | 독립 실행형 애플리케이션, 라이브러리 | 웹 애플리케이션                                 |
| 구조       | 단순한 디렉터리 구조                 | 특정 디렉터리 구조 필요                         |
| 실행 방식  | java -jar 명령어로 실행              | 서블릿 컨테이너에 배포하여 실행                 |
| 메타데이터 | MANIFEST.MF 파일 사용                | web.xml 파일 사용                               |
| 포함 파일  | Java 클래스 파일, 리소스 파일        | Java 클래스 파일, JSP, HTML, CSS, JS, 설정 파일 |

## 상황

- Jar

  - 독립 실행형 어플리케이션
    - 단일 Jar파일로 배포하여 다양한 환경에서 쉽게 실행할 수 있는 어플리케이션
  - 라이브러리 배포
    - 다른 프로젝트에서 사용될 라이브러리나 유틸리티 배포
  - 테스트 및 유틸리티 어플리케이션
    - 간단한 유틸리티 프로그램이나 테스트 프로그램 배포

- War
  - 동적 웹 어플리케이션
    - 웹 서버나 서블릿 컨테이너에서 실행될 필요가 있는 동적 웹 어플리케이션
  - 엔터프라이즈 웹 어플리케이션
    - 대규모의 복잡한 웹 어플리케이션 배포
  - 서블릿 및 JSP 기반 어플리케이션
    - 서블릿과 JSP를 사용하는 웹 어플리케이션 배포
