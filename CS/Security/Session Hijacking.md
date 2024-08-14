# Hijack a Session

![](https://i.postimg.cc/d3TmgmzR/session-hijacking.png)   

`hijack a session`은 공격자가 사용자의 세션을 탈취하여 권한을 도용하여 공격하는 기법이다. 

![](https://i.postimg.cc/tJ7Smb4f/Intro.png)   

어플리케이션 개발자가 자체적으로 세션 ID를 생성할 때 보안에 필요한 복잡성과 무작위성을 제대로 구현하지 않으면, 해당 어플리케이션은 세션 기반 브루트 포스 공격에 매우 취약해진다고 한다.

이번 파트의 목표는 다른 다른 사용자의 인증된 세션에 접근하는 것을 목표로 한다.   

![](https://i.postimg.cc/fLq5dPfv/img-1.png)   

`Username`과 `Password` 부분에 임의의 값을 입력 후 제출을 한 뒤, Burp Suite에서 HRRP history에서 어떤 요청과 응답이 있는지 확인을 하면 

![](https://i.postimg.cc/BvSnL76b/img-2.png)     
위 이미지에서 Request부분을 보면 아래에 임의로 작성한 입력값을 확인 가능하다. 그리고 Response부분을 보면 `Set-Cookie header`에 찾던 hijack_cookie의 값이 보인다.   

하지만 해당 값은 임의로 입력에 대한 쿠키값이기 때문에 권한이 있는 사용자의 쿠기가 아닌 인증에 실패한 쿠기값이다. 그렇기 때문에 해당 쿠키는 어떤 형태로 생성이 되는지 파악을 해야한다.   

Intruder에서는 자동화 공격 도구로 동일한 HTTP 요청을 여러 번 반복적으로 보내는 무차별 공격을 지원하는데 해당 기능을 설정을 위해서 어느 부분에 임의의 값을 어디에 대입을 할 것인지 명시를 해줘야한다.

해당 Response을 Intruder에서 name, password, cookie를 추가하여 
![](https://i.postimg.cc/qR86Vb1G/img-3.png)   

이 처럼 위치를 지정해줘야 한다. 

```
Cookie: JSESSIONID=7cSRfd-e_qJZUZtRdv00YsTMzj3V8RxJ3Ad4SllF
Connection: keep-alive

username=test&password=§test§
```

![](https://i.postimg.cc/1z8YH6vS/img-4.png)   

이렇게 지정을 해주고, 0부터 9의 값을 1씩 증가시키면서 추가하기 위해서 같이 설정을 해준다. 그리고 공격을 시작한다.   

공격을 통해서 어떤 값들이 나왔는지 확인을 해보자 5번째 요청 부터 7번째 요청까지의 값들을 순서대로 첨부하였다.   
![](https://i.postimg.cc/1RD6Ftcj/img-5.png)   
![](https://i.postimg.cc/zX5hFrxX/img-6.png)   
![](https://i.postimg.cc/vHzVPNK7/img-7.png)

hijack-cookie 부분을 보면 '-' 기준으로 앞 뒤 값을 비교하면 각 요청마다 뒷 부분만 변경된 것을 파악할 수 있다.   

5번째 요청과 6번째 요청을 보면 다른 요청들은 '-'기준 앞 부분 숫자에서 +1씩 증가된 것을 확인 가능한데, 5번쨰 요청과 6번째 요청에서는 +2가 증가되었다.  그러면 앞 부분에서 550을 고정을 하고, 뒷 부분은 446에서 
768로 증가되었다. 그렇기 때문에 뒷 부분은 447에서 767 사이의 값으로 공격을 해보자.   

다시 Intruder으로 돌아와서 hijack_cookie를 추가하고 복사한 값을 입력을 하고 앞 부분 숫자는 550으로 수정을 하고 뒷 부분은 3자리를 임의의 수로 지정하기 위해서 설정을 하고 증가 범위도 설정을 하여 다시 공격을 하자   

결과들을 확인을 하면 공격을 성공한 사례를 찾을 수 있을 것이다.   
![](https://i.postimg.cc/wMc3t1J8/img-8.png)   


이 같이 Web Application Server에서 관리하는 세션을 사용하는 것이 아닌 개발자가 직접 개뱔한 인증 세션 혹은 토큰을 사용할 경우 무작위 대입 공격 또는 유추를 통하여 탈취를 미연에 방지하기 위하여 영문과 숫자를 섞어 무작위로 값을 생성해야 하며, 그 길이는 12byte이상으로 만들어 미연에 방지하는 것을 추천한다.


