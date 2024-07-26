# Docker Logging

컨테이너 내부에서 무슨 일이 일어나는지 아는 것은 디버깅만 아니라 운영에 있어서도 중요하다. 어플리케이션 레벨에서 로그가 기록되도록 개발하여 별도의 로깅 서비스를 사용할 수 있지만
도커는 컨테이너의 표준 출력(StdOut)과 에러(StdErr) 로그를 별도의 메타데이터 파일로 저장하며 이를 확인하는 명령어를 제공을 한다.

## $$1$$. json-file Log

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
이러한 상황을 방지하기 위하여 `--log-opt` 옵션으로 컨테이너 JSON 로그 파일의 최대 크기를 지정할 수 있다.

```shell
docker run -it \
--log-opt max-size=10k --log-opt max-file=3 \
--name log-test ubuntu:14.04
```

아무런 설정도 하지 않았다면 도커는 위와 같이 컨테이너 로그를 JSON파일로 저장하지만,
각종 로깅 드라이버를 사용한다면 설정해진 컨테이너 로그를 수집할 수 있다.   
드라이버의 예로 곧 알아볼 syslogs, journald, fluentd, awslogs 등이 있으며, 어플리케이션의 특징에 접합한 로깅 드라이버를 선택한다.