# JitPack으로 라이브러리 배포_2

저번에 전체적인 내용을 아주 간략하게 메모를 했었다.    

하지만, 현재 다시 라이브러리를 만들면서 여러 문제가 발생했고 해결하기 위해서 아까운 시간을 많이 낭비한거 같아서 어떤 문제들이 있었는지 원인을 완벽하게 알고 싶어서 또 글을 남겨본다.   

![](https://i.postimg.cc/Gmx7nWjb/IMG-6233.jpg)    

## Java로 만들어?? Spring Boot로 만들어??    

라이브러리를 만들기 위해서 여러 블로그에 포스팅된 글을 읽는데 어떤 글은 자바프로젝트로 만들고 다른 글은 스프링부트로 만들고있었다.    

> 각각의 방법에 큰 차이점이 있을까? <br> 그냥 작동만 잘 된다면 큰 문제는 없겠지...   

라는 생각으로 포스팅된 글을 대충 보았다.   

하지만 아무런 생각없이 자바 프로젝트로 라이브러리를 만들면서 알았다.    

여러 어노테이션을 사용하면서 Spring의 특정 기능을 이용하고 싶어서 작성한 어노테이션이 문제를 발생시키는 것이다. 그래서 다 작성한 프로젝트를 폐기하고 다시 Spring으로 프로젝트를 작성했다... 

### 각각의 프로젝트의 장단점   

일단 먼저 자바 프로젝트로 만들었을 때는 복잡한 의존성이나 설정이 없기 때문에 보다 가볍기 때문에 단순한 클래스를 모와 JAR파일로 빌드할 수 있기에 경량화된 환경에서 사용을 한다.    
Spring Boot의존성이 없기 때문에 특정 기능을 사용이 불가능하다. 

스프링부트 프로젝트는 스프링의 여러 기능들을 사용이 가능해진다. 예를 들어 Spring Security등을 이용이 가능하다. 그리고 자동 설정 기능으로 라이브러리를 사용하는 다른 마이크로사비스들이 자동으로 설정을 적용받을 수 있다.    

이렇게 간단하게 두 방법에 대해서 알아보았다.

---    
    
## Spring Boot로 만들 때 설정    

### Build.Gradle

#### Plugins

먼저 plugins 부분을 보면 아래와 같이 설정을 했다.   

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.7' apply(false)   
    id 'io.spring.dependency-management' version '1.1.7'
    id 'maven-publish'
}
```

`id 'org.springframework.boot' version '3.3.7' apply(false)`    
해당 부분에서 마지막 부분에 `apply(false)` 같이 설정을 하는 이유는 해당 플러그인을 자동을 적용하지 않도록 하기 위해서다.    

> 자동으로 적용하지 않는 이유 <br> 일반 프로젝트일 경우 설정을 해야한다. 하지만 지금 작성하는 내용은 공통 라이브러리를 만들기 위함이다.  <br> 공통라이브러리에서는 실행가능한 어플리케이션을 만들 이유가 없으며 단순히 다른 프로젝트에서 의존성을 사용하면 된다.


다음으로 `id 'maven-publish'` 부분은 Maven 같은 저장소에 배포를 가능하도록 해주는 플러그인이다. JitPack을 사용할 때 해당 플러그인 설정으로 라이브러리를 배포할 수 있도록 해주는 설정이다.

---

#### DependencyManagement

```gradle
dependencyManagement {
    imports {
        mavenBom org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES
    }
}
```   

`org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES`는 **스프링 부트의 BOM (Bill of Materials)**을 가져와서, 프로젝트에서 사용하는 의존성의 버전들을 일관되게 관리하는 역할을 한다.    

만약 해당 설정을 하지 않는다면, 의존성 버전이 관리되지 않아 버전이 충돌하는 문제가 발생될 수 있다. 특히 공통 라이브러리를 사용하는 다른 프로젝트에서 의존성을 사용할 경우 버전 충돌로 쉽게 문제가 발생된다.     

---   

#### Publishing

```gradle
publishing {
    publications {
        maven(MavenPublication) {
            groupId = 'com.exeample.apiresponse'
            artifactId = 'Api-Response'
            version = '0.0.9'

            from components.java

            versionMapping {
                usage('java-api') {
                    fromResolutionOf('runtimeClasspath')
                }
                usage('java-runtime') {
                    fromResolutionResult()
                }
            }
        }
    }
}
```

`maven(MavenPublication)` 부분은 Maven저장소에 배포할 때 라이브러리 메타데이터를 정의하고, 배포할 실제 파일을 설정한다.     

`versionMapping`에 정의된 코드를 보자
 * usage('java-api'):
    * 이 설정은 라이브러리가 API용으로 사용될 때, 즉 runtimeClasspath에 있는 의존성 버전을 사용할 것을 정의한다.    
    fromResolutionOf('runtimeClasspath')는 런타임에 필요한 의존성 버전을 자동으로 해결하도록 한다.

    * usage('java-runtime'):    
    이 설정은 라이브러리가 실행(Runtime) 시 사용될 때, fromResolutionResult()를 사용하여 의존성 버전을 해결한다.       
    fromResolutionResult()는 의존성 해석 결과를 그대로 따르도록 합니다.

`versionMapping`은 라이브러리가 API와 실행시 요구되는 의존성 버전들을 명확하게 구분하여 의존성 버전 충돌을 방지하는데 도움을 준다.


**versionMapping** 없이 배포를 시도를 했을 때, 아래와 같은 문제가 발생되었다.
> Invalid publication 'maven':
    - Publication only contains dependencies and/or constraints without a version.
    
    

