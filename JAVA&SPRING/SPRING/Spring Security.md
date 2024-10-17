# Spring Security

프로젝트 내 작성된 `Spring Security`와 `JWT`관련된 코드에 대해서 어떻게 동작이 되는지 확인하고 싶어서 찾아보았다.   

![](https://i.postimg.cc/fby8gtZq/2024022315584263654.jpg)   

솔직하게 해당 코드 파일을 열어보는 순간 당황스럽고 막막해졌다.   

이 코드가 어떤 동작을 하는지 그리고 어떤 코드는 무엇을 불가능하게 하는지 정말 자세히 모르고 있었기 때문이다. 당시 `Spring Security` 관련된 지식은 어떤 객체가 특정 URL에 접속을 할 때 가능하게 하거나 불가능하게 하는 내용 뿐이였다.

그래서 이 기회로 자세히 알아 볼 생각이다.   

## 전체적인 아키택처     

전체적인 아키태처 이해를 위해 첨부 사진을 통해 설명을 해보겠다.   

![](https://i.postimg.cc/Kjf9HnKV/blogpost-spring-security-architecture.png)    

유저가 로그인을 시도를 한다.

유저가 로그인을 시도를 할 때 `AuthenticationFilter`에서 해당 요청을 가로채고, 해당 요청을 검증하기 위해서 `AuthenticationManager`로 위임을 한다.   

또 다시 `AuthenticationManager`에서는 `AuthenticationProvider(s)`으로 검증을 위임한다.   

마지막으로 `AuthenticationProvider(s)`에서 동작은 `UserDetailsService`인터페이스를 통해 내부에 있는  `loadUserByUsername` 메소드를 통하여 유저가 입력한 아이디에 대한 데이터가 있는지 찾고 만약 해당 데이터가 존재한다면 검증을 한다.   

그런데 `AuthenticationFilter`와 `UsernamePasswordAuthenticationToken` 사이는 어떤 관계가 있는가?    

해당 관계는 `AuthenticationFilter`에서 검증을 위임 할 떄 필요한 `Authentication`객체 생성을 위한 관계이다. `AuthenticationFilter`에서 `AuthenticationManager`으로 위임을 할 때 데이터를 보내는 것이 아닌 `UsernamePasswordAuthenticationToken`으로 생성된 토큰을 보내기 때문이다.   

그런데 모든 요청이 `UsernamePasswordAuthenticationToken`을 거치는 것이 아니다. 예를 들어 어떤 유저는 로그인을 통해 `JWT`나 세션을 통해 이미 검증을 받았다면 해당 인증 정보는 `SecurityContextHolder`에 저장되어 있기 때문에 인증 처리 없이 바로 요청이 된다.

* SecurityContextHolder    
  `SecurityContextHolder`는 인증 정보를 전역적으로 저장하고 관리하고 있는 클래스이다.    
  다시 말하면 현재 인증된 사용자 정보를 어딘가에 저장해두고, 그 정보를 다른 요청이나 서비스 로직에서 쉽게 가져다 쓸 수 있도록 하는 역할을 한다.
  
  구조는 아래와 같은 구조를 갖고 있다.   
  ![](https://i.postimg.cc/Xvbc5sYb/securitycontextholder.png)
  
  * SecurityContext   
    `SecurityContext`는 인증된 사용자에 대한 정보를 담고 있는 객체이다.   
    앞에서 `SecurityContextHolder`는 인증된 사용자 정보를 저장한다고 말을 했다. 여기서 인증된 사용자의 정보가 `SecurityContext`객체이다.   

    `SecurityContext`는 현재 인증된 사용자의 정보를 관리한다. 해당 정보는 `Authentication`객체 안에 있는 정보이다.   

  * Authentication
    `Authentication`객체에는 실직적인 사용자의 인증 정보가 들어있다. 해당 객체에는 사용자의 아이디와 비밀번호 그리고 권한 등의 다양한 정보가 들어있다.    

     * 주요 속성   
     `Principal`   : 주로 사용자의 정보나, 사용자명   
     `Credentials` : 사용자의 비밀번호같은 자격 증명, 인증 이후 보안상의 이유로 null값으로 수정 가능
     `Auyhorities` : 사용자가 갖는 권한

  * ProviderManager   
    `AuthenticationManager`는 실질적인 인증을 하는 매커니즘을 갖고있는 인터페이스다.    
    `ProviderManager`는 `AuthenticationManager`의 구현체 중 일부로 인증 작업을 처리한다.   

    ![](https://i.postimg.cc/gkw9JHJs/Authentication-Manager.png)
    위 사진을 보면 하나의 `ProviderManager`에는 여러 `AuthenticationProvider`를 갖고있다. 이는 `Spring Security`에서 다양한 인증 방식을 지원하기 때문에 각각의 인증 방식에 따라 다른 `AuthenticationProvider`가 필요할 수 있기 때문이다.   

    예를 들면 로그인 과정에서 아이디/비밀번호를 사용한 인증 방법과 OAuth 인증 방식이 있을 때 각각의 방법에 대한 인증을 처리하기 위해서다.  

이렇게 간단한 아키택처를 확인했다.   

## Filters

`Spring Security`는 `Servlet Filter` 바탕으로 동작을 한다.   
![](https://i.postimg.cc/MTPQsS1y/image.png)    

간단하게 확인을 하면 사용자의 요청이 들어오면 `request`형태로 서버로 전달이 된다. 이 시점이 어플리케이션 시작의 시점이다.     

해당 요청이 서버로 오면, 서버는 해당 요청을 처리하기 위해 `Filter`와 `Servlet`이 있는 `Filter Chain`은 생성한다. (이때 두 개념의 역활이 중요하다.)   

여기서 `Filter`은 `Servlet`에 도달하기 전 사전 작업? 사전 처리?를 진행한다.  여기서 `Filter`는 요청을 처리하는데 다양한 기능을 추가하여 사전 작업을 한다. (예를 들면, 특정한 요청을 차단하거나, 요청 데이터를 수정하거나 등)   

이후 `Servlet`에서는 실제 요청을 처리하고, 비지니스 로직을 수행한다. 

여기서의 `Servlet`은 `Spring MVC`에서 `DispatcherServlet`이 역활을 수행한다.   

## DelegatingFilterProxy

![](https://i.postimg.cc/2Sh9gHcq/delegatingfilterproxy.png)

**위 이미지에서 `DelegatingFilterProxy`이 등장을 했다. 이는 무엇이며 어떤 역활을 하는가??**

`DelegatingFilterProxy`는 Spring Framework에서 중요한 역할을 하는 컴포넌트다. 간단히 말해, Spring의 필터를 서블릿 컨테이너에서 사용할 수 있도록 연결해주는 다리 역할을 한다.   

서블릿 컨테이너는 우리가 웹 애플리케이션을 만들 때 사용하는 필터를 관리를 한다. `DelegatingFilterProxy`는 이 서블릿 필터 체인에 Spring의 필터를 연결하는 역활을 수행한다. 예를 들어, Spring의 보안 필터인 `UsernamePasswordAuthenticationFilter`를 호출할 수 있도록 도와줍니다. 이렇게 하면 Spring의 기능을 서블릿 환경에서도 사용할 수 있습니다.   

그리고 필터 규현체와의 관계를 보면 `DelegatingFilterProxy`는 직접적으로 요청을 처리하는 필터는 아니다. 대신, 요청을 처리할 실제 필터 구현체를 찾아서 그 필터에게 요청을 위임하는 역할을 수행한다. 즉, `DelegatingFilterProxy`는 Spring 컨텍스트에서 특정 필터를 찾고, 그 필터가 요청을 처리하도록 호출한다. 이렇게 함으로써 Spring의 필터를 효율적으로 사용할 수 있게 된다.   

서블릿 컨테이너는 표준 필터를 등록할 수 있지만, Spring의 Bean 형태로 만들어진 필터는 기본적으로 인식하지 못합니다. `DelegatingFilterProxy`는 이러한 `Spring Bean`형태의 필터를 서블릿 컨테이너에서 사용할 수 있도록 변환해준다. 

## SecurityFilterChain

![](https://i.postimg.cc/xjKLMW4F/image.png)

일단 `Filter Chain Proxy`는 어떤 역활을 수행하는지 확인을 하자.

`Spring Security`에서 제공하는 특별한 필터이다. `Filter Chain Proxy`는 `SecurityFilterChain`을 통하여 여러 필터 제공이 가능하다.   

`FilterChainProxy`는 Spring에서 제공하는 필터로, 일반적인 필터와 마찬가지로 요청을 가로채고 처리한다. 이 말은 즉, `FilterChainProxy`가 실제로 요청을 처리하고, 필요한 경우 다음 필터로 요청을 전달하는 방식으로 작동한다는 뜻이다.   

`DelegatingFilterProxy`와의 관계를 확인하면 `DelegatingFilterProxy`는 Spring의 필터를 서블릿 컨테이너에서 사용할 수 있도록 도와주는 역할을 합니다. 그리고 `FilterChainProxy`도 이러한 Spring의 필터 중 하나이기 때문에 `FilterChainProxy`는 `DelegatingFilterProxy`의 관리를 받으며, Spring의 보안 필터 체인에서 중요한 역할을 맡고 있다.

갑자기 너무 많은 내용이 나타나서 어지러우니 잠시 중간 정리를 하겠다.

> 	`DelegatingFilterProxy`는 `Spring Security`의 필터를 서블릿 컨테이너와 연결하는 다리 역할을 한다. 하지만 직접적인 보안 처리를 하는 것은 아니며, 그 역할을 `FilterChainProxy`에 위임한다.
>
>	`FilterChainProxy`는 `SecurityFilterChain`을 통해 보안 필터들의 체인을 관리합니다. 이 필터 체인에는 인증, 인가와 관련된 다양한 필터들이 포함된다.
>
>	`SecurityFilterChain`은 여러 보안 필터를 하나의 체인으로 묶어서 동작하게 하며, 각각의 요청이 해당 필터 체인을 따라 처리된다.
>
>	`FilterChainProxy`는 `DelegatingFilterProxy`에 의해 서블릿 컨테이너에 등록된다. 그러면 서블릿 요청이 들어올 때 `DelegatingFilterProxy`가 이를 가로채고, 내부적으로 `FilterChainProxy`로 위임하여 보안 필터 체인을 처리한다.

![](https://i.postimg.cc/zD9my2QB/image.png)   

## 마무리...

나는 지금까지 아래와 같이 `Spring Security`설정 파일을 작성을 했을 때 어떤 부분을 작성하는지 몰랐다. ㅎㅎ;    

```java
@Bean
protected SecurityFilterChain config(HttpSecurity http) throws Exception {

    return http
            .httpBasic(AbstractHttpConfigurer::disable)
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(requests -> {
                requests
                        .requestMatchers("/h2-console/**").permitAll()
                        .requestMatchers("/api/account/**").permitAll()
                        .requestMatchers("/login", "/api/account/signup", "/api/member/login").permitAll()
                        .anyRequest()
                        .authenticated();
            })
            .sessionManagement(
                    sessionManagement ->
                            sessionManagement.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtFilter(), UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(exceptionHandling -> {
                exceptionHandling
                        .defaultAuthenticationEntryPointFor(
                                new HttpStatusEntryPoint(HttpStatus.UNAUTHORIZED),
                                new AntPathRequestMatcher("/api/**")
                        );
            })
            .build();
}
```

이제는 알겠다 우리가 보통 작성하는 `Spring Security`코드는 대부분이  `SecurityFilterChain`을 구성하는 설정 코드라는 것을   

그리고 `Spring Security`가 동작하는 아키택처를 이제는 누군가에게 설명을 할 수 있게 되었다.   

지금까지 나는 내가 작성한 코드가 어떤 부분이며, 어떻게 동작하는지 설며을 못하는 속이 텅텅 빈 공갈 개발자가 될 뻔 했다.