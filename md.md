# Docker Logging

컨테이너 내부에서 무슨 일이 일어나는지 아는 것은 디버깅만 아니라 운영에 있어서도 중요하다. 어플리케이션 레벨에서 로그가 기록되도록 개발하여 별도의 로깅 서비스를 사용할 수 있지만
도커는 컨테이너의 표준 출력(StdOut)과 에러(StdErr) 로그를 별도의 메타데이터 파일로 저장하며 이를 확인하는 명령어를 제공을 한다.

## $$1$$ json-file Log

컨테이너를 생성 후 간단한 로그를 남겨보겠다.
``` shell
docker run -d --name mysql \
-e MYSQL_ROOT_PASSWORD=1234 \
mysql 
```

mysql과 같은 어플리케이션을 구동하는 컨테이너는 포그라운드 모드로 실행되므로 `-d` 옵션을 사용함으로 백그라운드 모드로 컨테이너를 생성하는 경우가 많다.   
그렇기 때문에 어플리케이샨이 잘 구동되는지에 대한 여부는 잘 알 수 없지만 docker lofs 명령어를 사용하여 컨테이너 표준 출력을 확인함으로 상태를 확인 가능하다.
(`docker logs`는 컨테이너 내부에서 출력을 보여주는 명령어다.)

![](https://i.postimg.cc/FHpSz4xQ/logs-1.png)

이번에는 `-e` 옵션을 제외하여 컨테이너를 생성 해보겠다.

해당 컨테이너는 문제가 발생되어 컨테이너가 시작되지 않는다. 이 처럼 정상적인 실행 및 동작을 하지 않고 `docker attach`명령어도 사용하지 못하는 경우에
`docker logs`를 사용하여 간단하고 빠르게 어떤 문제가 발생했는지 확인이 가능하다.
![](https://i.postimg.cc/SKcwCgJq/logs-2.png)

>
> 컨테이너의 로그가 많아서 식별하기 어려울 경우 `--tail` 옵션을 사용하여 마지막 로그줄 부터 출력할 줄의 수를 지정하여 확인이 가능하다.   
```shell
docker logs --tail 5 mysql

...
```

다음으로 `--since` 옵셔넹 대해서 알오보자   
해당 옵션은 유닉스 시간을 입력하여 특정 시간대의 이후 시점 로그를 확인할 수 있다. `-t` 옵션으로 타임스탬프를 표시할 수 있고,
`-f`옵션은 컨테이너에서 실시간으로 출력되는 내용을 확인할 수 있다.    
(책에서는 `-f` 옵션으로 어플리케이션을 개발할 때 유용하다고 한다. 아직 왜 유용한지는 모르겠다.)

```shell
docker logs --since ~~~ mysql
...

docker logs -f -t mysql
...
```

logs 명령어는 run에서 `-i -t` 옵션을 설정하여 `docker attach` 명령어를 사용할 수 있는 컨테이너에서도 사용이 가능하며, 내부에서 bash 셸 등을 입출력한 내용을 확인할 수 있다.

```shell
docker run -i -t --name logtest ubuntu:14.04
root@~~:/# echo test!

# test! 출력

docker logs logstest 
```
기보적으로 이와 같은 컨테이너 로그는 JSON형태로 도커에 저장이 된다. 이 파일은 다음 경로에 컨테이너 ID로 시작하는 파일의 이름으로 저장이 된다.   
```shell
cat /var/lib/docker/containers/${CONTAINER_ID}/${CONTAINER_ID}-json.log
```
log 파일의 내용을 cat, vi 편집기 등으로 확인을 하면 logs 명령으로 정제되지 않은 JSON형태의 데이터를 확인할 수 있다.

그러나 무작정으로 이러한 설정을 한다면 컨테이너 내부에서 log의 출력이 많을 경우 파일의 크기는 무지막지하게 커지는 경우가 나타날 수 있다. 
그러면 결국 호스트의 남은 저장 공간을 모두 차지하게되는 상황이 일어나게 된다.    
이러한 상황을 방지하기 위하여 `--log-opt` 옵션으로 컨테이너 JSON 로그 파일의 최대 크기를 지정할 수 있다. `max-size`느ㄴ 로그 파일의 최대 크기, `max-file`은 롤그 파일의 개수를 의미한다.

```shell
docker run -it \
--log-opt max-size=10k --log-opt max-file=3 \
--name log-test ubuntu:14.04
```

아무런 설정도 하지 않았다면 도커는 위와 같이 컨테이너 로그를 JSON파일로 저장하지만,
각종 로깅 드라이버를 사용한다면 설정해진 컨테이너 로그를 수집할 수 있다.   

---

## syslog logging

컨테이너 로그는 JSON뿐 아니라 syslog로 보내어 저장하도록 설정을 할 수 있다.   
syslog는 유닉스 계열 운영체제에서 로그를 수집하는 오래된 표준 중 하나이다. 대부분의 유닉스 계열 운영 체제에서는 syslog를 사용하는 인터페이스가 동일하기 때문에 체계적으로 로그를 수집하고 분석이 가능하다는 장점이 있다.

```shell
docker run -d --name syslog_container \
--log-driver=syslog \
ubuntu:14.04 \
echo syslogtest
```

이 처럼 입력하면 syslog에 로그를 저장하는 컨테이너를 생성한다.

`ubuntu:14.04`와 `RHEL`은 /var/log/message 파일에서 확인이 가능하고, `ubuntu16.04`와`CoreOS`는 journalctl -u docker.service 명령어로 확인이 가능하다.

## fluentd logging
fluentd는 각종 로그를 수집하고 저장할 수 있는 기능을 제공하는 오픈소스 도구로서, 도커 엔진의 컨테이너 로그를 fluentd를 통하여 저장할 수 있도록 플러그인을 동식적으로 제공을 한다.   
fluentd는 데이터 포맷으로 JSON을 사용하기 때문에 쉽게 이용할 수 있을 뿐더러 수집되는 데이터를 AWS S3, HDFS, MongoDB등 다양한 저장소에 저장이 가능하다는 장점이 있다.   

![](https://i.postimg.cc/dQnPPskM/fluentd.png)

간단한 실습으로 fluentd와 MongoDB를 연동하여 데이터를 저장하는 방법을 해보겠다.    

fluentd서버는 컨테이너의 로그를 수집하고, MongoDB에 로그를 수집하는 구조이다.   


