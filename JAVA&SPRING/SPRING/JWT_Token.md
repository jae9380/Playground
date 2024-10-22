# Access Token, Refrsh Token

현재 개인 프로젝트를 만들면서 특정 유저가 접근이 가능한지 불가능한지 검증을 해야한다.
그렇기 때문에 Token을 이용한 방식으로 인증/인가를 구현을 했다.

![](https://i.postimg.cc/K8TMDxNH/LAXvd.jpg)

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

