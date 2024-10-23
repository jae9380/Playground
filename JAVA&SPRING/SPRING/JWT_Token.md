# 인증, 인가 구현하기

현재 개인적으로 진행하는 프로젝트에 대하여 인증과 인가에 대한 기술을 구현을 해야했다.   
그리고 이를 바탕으로 검증된 유저에게 접근 허가를 해줘야 했다.

![](https://i.postimg.cc/K8TMDxNH/LAXvd.jpg)

## HTTP 통신

인증, 인가 구현 방법에 대하여 설명하기 전에 `Http`에 대하여 간단하게 확인하자.   


**HTTP(하이퍼텍스트 전송 프로토콜, HyperText Transfer Protocol)**

`HTTP`는 클라이언트와 서버 사이 데이터를 주고받기 위한 프로토콜로, 기본적으로 `Stateless`을 가지고 있다. 이는 클라이언트와 서버가 한번의 요청과 응답으로 연결을 끊기 때문에, 매번 새로운 요청에 대해 독립적으로 처리하는 특징이 있다.   

![](https://i.postimg.cc/jjv5TqQC/http-communicate-process-1.gif)

## 왜 HTTP 통신에서 인증과 인가가 중요할까? 🤔

`HTTP`통신에 대하여 간단하게 설명을 할 때 필요한 이유가 있다.   

> ### 클라이언트와 서버가 한번의 요청과 응답으로 연결을 끊기 때문에, 매번 새로운 요청에 대해 독립적으로 처리하는 특징이 있다.

`HTTP`는 `Stateless`프로토콜이기 때문에 각 처리에 대하여 독립적으로 행하고, 이전의 상태를 기억하지 않기 때문이다.    

이전의 상태를 기억하지 않는다면, 우리가 생각하는 기존 웹 사이트의 동작 방식과 달라지게 된다.    

> #### "에이, 달라져 봤자 얼마나 달라지겠어요?"   

크게 변화는 것은 없겠지만, 우리가 최초의 로그인을 통해 받았던 인증 정보 또한 기억을 하지 않기 때문에 페이지 이동이나 모든 동작에 대하여 인증 정보를 요구를 할 때 우리는 계속 로그인을 해줘야 한다는 동작이 추가가 된다.   

상당히 귀찮게 돌아갈 것이다.   

이러한 문제점을 해결하기 위해서 `Cookie`, `Session`, `Token`이 등장하게 되었다.   

## Cookie   

쿠키는 클라이언트에 저장되는 작은 데이터다.    
서버가 클라이언트에게 쿠키를 전송하면, 클라이언트는 이를 저장하고 이후 요청 시마다 서버에 쿠키를 포함해 전송하여 상태를 유지할 수 있다.

![](https://i.postimg.cc/qBZdThP9/scode-mtistory2-fname-https-t1-daumcdn.png)

해당 동작 처럼 클라이언트는 서버로 부터 요청을 보낼 때, 서버는 응답과 함께 쿠키를 하나 보내주게 된다. 그러면 클라이언트는 다음 요청 부터는 받았던 쿠키를 같이 보내주게 되면서 인증 정보를 같이 보내주는 것이다.   

## Session

`Session` 방식은 서버에서 클라이언트의 상태(인증 정보)를 저장하여 클라이언트가 접근할 때 저장된 정보를 바탕으로 검증을 하는 방법이다.   

### 동작 방식

![](https://i.postimg.cc/MXJ6S8yH/session-1.jpg)

`Session`의 방식은 위 이미지를 보면 쉽게 이해가 될 것이다.    
간단하게 비유를 하면 해당 유저가 출입 명단에 의거하여 검증을 한다.

해당 방법은 서버에서 직접 `S

## Token

## 동작 방식

간단한 전체적인 동작은 이와 같다. (아래의 이미지 번호와는 상관 없음)

1. 유저 로그인 시도
2. ID/PW 검증을 통하여 로그인을 하면, Access Token & Refresh Token을 Client에게 지급
3. Client는 지급 받은 토큰을 저장을 하고, 매번의 요청에 Access Token을 같이 전달을 하여 유효기간 동안은 인증되었음을 증명
4. 만약 유효기간이 초과된 Access Token일 경우 서버는 클라이언트에게 유효기간이 초과 되었음을 알려준다.
5. 클라이언트는 Access Token을 재발급을 위해 Refresh Token을 전달
6. 서버는 Refresh Token을 바탕으로 Access Token과 Refresh Token을 재발급 해준다.

![](https://i.postimg.cc/RhnSxVNr/oauth-refresh-token-diagram.png)

자세한 내용은 [OAut 2.0 rfc 1.4 ~ 1.5 (Access Token & Refresh Token)](https://datatracker.ietf.org/doc/html/rfc6749#section-1.4) 확인 가능

