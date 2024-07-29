# API Gateway

해당 서비스는 사용자가 정의한 라우팅 설정에 따라서 각각의 엔트포인트로 클라이언트를 대신하여 요청하고 응당을 받으며 다시 클라이언트한테 전달해주는 프록시 역활을 하게된다.

시스템 내부 구조는 숨기고 외부의 요청에 대하여 적절한 형태로 응답을 하도록 한다는 장점이 있다.

![](https://i.postimg.cc/26m9zyQx/APIGateway2.png)

마이크로 서비스가 3가지 있다고 가정을 하자.

기존에는 모바일나 웹이거나 클라이언트 측에서 마이크로 서비스의 주소를 직접 이용해서 파리미터를 전달하고 요청하는 것으로 볼 수 있다. 여기서 하나의 서비스가 추가되거나, 기존 서비스의 주서가 변경 되거나 등 일이 얼어났을 때 마이크로 서비스가 독립적으로 빌드와 배포가 된다. 그러면 문제는 클라이언트 사이드에서 발생한다.

클라이언트 사이드에서 직접적으로 엔트포인트를 이용해서 호출했을 경우에는 클라이언트 사이드에 있는 어플리케이션 또한 같이 수정, 배포를 해줘야 한다.

이렇게 단일 진입점을 갖고 있는 형태로의 개발이 필요하게 되었다.

![](https://i.postimg.cc/2yGspvgv/APIGateway.png)

위 처럼 서버에 Gateway역활을 수행을 할 진입점을 하나 두고 각각의 마이크로 서비스로 요청되는 모든 정보에 대하여 일괄적으로 처리할 수 있게 만들어 준다.

어떤 방식에 있어서 직접적으로 마이크로 서비스를 호출하지 않고, 클라이언트는 Gateway만을 상대하게 한다.

이러한 `API Gateway`를 이용하게 된다면

- 인증 및 권한 부여에 대한 단일 작업 가능
- 마이크로 서비스의 검색을 통합
- 응답할 수 있는 캐싱 정보를 저장
- 정책, 회로 차단기 및 Qos 다시 시도
- 속도 제한과 로드밸런싱
- 부하 분산
- 로깅, 추적, 상관관계
- 헤더, 쿼리문 문자열 및 청구 반환
- IP에 대하여 허용, 차단 목록 관리

## Netflix Ribbon

![](https://i.postimg.cc/XY5zXtfL/netflix-Ribbon.png)  
스프링 클라우드에서 마이크로 서비스간의 통신, 하나의 마이크로 서비스에서 다른 마이크로 서비스를 호출하기 위한 방법 중 대표적인 방법은 `Rest Template`와 `Feign Client`가 있다.

- Rest Template
  `Rest Template`은 전통적으로 하나의 웹 어플리케이션에서 다른 어플리케이션을 이용하기 위해서 사용한 방법이다.

```java
RestTemplate restTemplate = new RestTemplate();
restTemplate.getForObject(http://localhost:8080/",User.class,200);
```

- Feign Client
  스프링 클라우드에서는 `Feign Client`라는 API를 이용하여 호출이 가능하다.

```java
@FeignClient("stores")
public interface StoreClient {
  @RequestMapping(method = RequestMethod.GET, value="/stores")
  List<Store> getStores();
}
```

위 처럼 특정한 인터페이스를 생성을 하고, 웹으로 따로 호출하고 싶은 추가적인 마이크로 서비스의 이름을 등록을 한다.

직접적인 서버의 주소, 포트 번호 없이 마이크로 서비스의 이름을 갖고 호출할 수 있게된다.

이러한 방법으로 스프링 클라우드에서 마이크로 서비스 간의 호출을 담담했는데, 문제는 로드 발랜서를 하기 위해서 어디에 구축해서 작업을 할 것인가가 문제이다.
초기 스프링 클라우드에서는 이러한 로드 발랜서의 역확을 담당하는 별도의 서비스 프로젝트를 위해 Ribbon이라는 서비스를 제공하기 시작했다.

> Ribbon : Client side Load Balancer

그런데 리본이라는 방식은 최근 `functional API` 또는 `React JAVA`라고 하는 방식과는 호환이 많이 안되는 방식 즉, 비동기가 처리가 잘 되지 않는 방식이기 때문에 최근에 이러한 방식을 사용하지 않는다.

그리고 `Health Check`라고 해서 해당하는 서비스가 정삭적인 동작을 하는지 확인할 수 있다.

마이크로 서비스 4가지가 있다고 했을 때, 클라이언트와 서비스 사이에 `API Gateway`를 중간에 두고 동작을 해야하는데 이러한 구조가 아닌 클라이언트 내부에 `Ribbon`이라는 서비스를 구축하여 사용을 하기 시작했다. (Client Side에 위치하고 있다.)

![](https://i.postimg.cc/9f7Cmx0H/img.png)

클라이언트 프로그램 내부에서 이동하고자 하는 마이크로 서비스의 주소 값을 직접 관리를 하는 구조다.

Client Side Load Balancer 장점으로는 IP하고, 포트 번호를 명시하는 방식이 아닌 그냥 단순히 서비스의 이름만 갖고 호출이 가능하다는 장점이 있다.

## Netflix Zuul

![](https://i.postimg.cc/gJzSSJft/netflix-Zuul.png)
`Netflix Zuul`은 방금 전 확인했던 Gateway을 담당해주는 제품이다.

![](https://i.postimg.cc/WzfHJvYW/Zuul.jpg)
