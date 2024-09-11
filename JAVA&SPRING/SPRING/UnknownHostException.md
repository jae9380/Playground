# Failed to resolve 'DESKTOP-UBM6CI1.mshome.net

Spring Cloud를 학습하면서 코드로 실습을 노트북과 컴퓨터를 번갈아 가면서 학습을 할고 있었다.

그런데 어느날 노트북으로 학습을 하고 다시 집으로 돌아와 컴퓨터로 실행을 하는 순간 `Failed to resolve 'DESKTOP-UBM6CI1.mshome.net`라면서 API Gateway기능에 문제가 발생했다.

## 발생 원인

`java.net.UnknownHostException` 예외는 사용하고 있는 컴퓨터 Host 정보에 저장되어 있지 않아서 그렇다고 한다. 다시 말해 **호스트 이름을 IP 주소로 변환하지 못했기에** 발생했다고 말해준다.

- 서비스가 IP 주소 대신에 호스트 이름을 사용하고 있음
  Spring Cloud Gateway나 다른 서비스가 요청을 할 때 IP 주소가 아닌 호스트 이름을 사용하려고 하고 있다.

- DNS 설정의 문제
  로컬환경에서 호스트 이름을 DNS 서버가 제대로 인식하지 못할 수 있다. 이는 로컬 네트워크 설정의 문제이거나 호스트 파일의 설정에 따라 달라질 수 있다

이와 같은 두가지 경우에 발생될 수 있는 예외 상황이다.

## 해결 방안

1. `prefer-ip-address` 설정

```yaml
eureka:
  instance:
    prefer-ip-address: true
```

각 서비스의 yml파일에 `prefer-ip-address`을 설정을 해준다. 이는 서비스가 Eureka에 IP 주소를 사용하여 등록되며, 다른 서비스들이 이 해당 IP를 사용하여 통신할 수 있게 된다.

하지만 해당 설정으로 단순히 해결되지 않았다.

2. Host 파일 수정
   로컬 DNS 설정 또는 `hosts`파일에서 올바른 IP 주소 매핑을 해준다.

해당 파일 위치는

- Windows - C:\Windows\System32\drivers\etc\hosts
- Mac or Linux - /etc/hosts

해당 파일 하단 부분에

```
192.168.0.0   DESKTOP-UBM6CI1.mshome.net
```

이 처럼 추가를 해준다.
