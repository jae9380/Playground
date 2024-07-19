# Change Maven to Gradle

프로젝트 만들 때 원래 계획으로는 Gradle을 사용 할 계획이였는데 실수로 프로젝트 생성 할 때 Maven으로 생성을 하였다.

그러면 프로젝트 생성 후 Maven에서 Gradle로 변경을 어떻게 하지??

## Step. 1 Gradle 다운로드

[Gradle 다운로드](https://gradle.org/releases/)  
![](https://velog.velcdn.com/images/jae9380/post/481728b6-462e-4865-a505-f0cf2d491b9a/image.png)

만약 기존에 다운했던 Gradle이 있는지 확인을 먼저 해준다.

일단 위 주소로 들어가서 그래들은 다운 받아야 한다.

CMD 창을 열어서 아래의 명령어를 입력한다.

```cmd
gradle -v
```

![](https://velog.velcdn.com/images/jae9380/post/29b1d773-2aa0-4d30-937c-9b3905c90604/image.png)

이 처럼 버전이 나타난다면 Gradle이 있다는 것이다.

없다고 가정을 하고, 제일 위 링크를 들어가서 최신 버전의 Gradle을 다운을 받는다. 해당 파일을 다운 받을 때 .zip형태로 다운이 된다. 그래서 해당 파일을 다운 받은 후 압축을 풀어준다..

## Step. 2 환경변수 등록

환경변수 설정으로 들어가서 시스템 변수 부분에 Path로 들어간다. 그리고 방금 전에 다운로드한 Gradle을 등록을 한다.

![](https://velog.velcdn.com/images/jae9380/post/ad366233-899a-4ea7-90de-7985fd846873/image.png)

제일 아래에 Gradle의 bin 폴더로 설정을 해주면 환경변수 등록까지 완료 했다.

이제 다시 cmd를 열어서 버전확인 명령어를 입력하면 잘 나타날 것이다.

```cmd
gradle -v
```

## Step. 3 Gradle 변경

이제 프로젝트로 돌아가서, 콘솔에서

```cmd
gradle init
```

명령어를 입력을 해주고 몇 가지 설정만 해주면 Maven에서 Gradle로 변경이 완료가 된다.
![](https://velog.velcdn.com/images/jae9380/post/655971cf-b9a6-4567-9c05-9ee27ac4bacb/image.png)
