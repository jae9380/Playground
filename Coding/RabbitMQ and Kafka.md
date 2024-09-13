# RabbitMQ와 Kafka

![](https://i.postimg.cc/gJvCZCgZ/Kafka-vs-Rabbit-MQ.jpg)

`RabbitMQ`와 `Kafka` 이들은 무엇인지 그리고 어떠한 차이점이 있는지에 대하여 확인하겠다.

`MSA`에서는 한 쪽에서 여러 대상에게 데이터를 보내야 하는 경우가 자주 발생된다. 그런데 해당 데이터를 받아야하는 대상에게 직접 하나씩 전달을 해주는 방법은 상당히 부적합한 방법이 된다. 그리고 또한 받는 쪽에서 특정한 데이터만을 받아야 하고, 데이터를 받는 어플리케이션이 늘어나거나 등의 여러 상황에 따라 전체적인 구조가 복잡해지거나 구현에 있어서 어려워진다.

이를 해결하기 위한 `Message Queue`라는 개념 이해가 필요하다.

## Message Queue, MQ

`MQ`는 크게 분류를 하면 메세지를 생성하는 **생성자,발행자 (Publisher, Producer)** 와 생성된 메세지를 보관하는 **Queue** 마지막으로 해당 메세지를 소비하는 **소비자 (Consumer)** 이렇게 구성된다.  
간단하게 동작을 표현을 하면 아래와 같은 이미지의 순서이다.  
![](https://i.postimg.cc/K8XWN9Yj/images-cho876-post-1c2a806d-163a-4d08-b627-4b0034c832e8-2022-03-12-12-04-32.png)

기존에는 Producers와 Consumers 사이에서 발생되고 소비되는 메세지or데이터 직접 전달을 하는 **동기적** 방식이 아닌, 둘 사이에 `Queue`로 만들어진 우편함?을 이용하여 **비동기적** 방식을 이용한다. 이렇게 되면 Producers는 Consumers가 자신이 발행한 정보에 대하여 수신을 했는지 상관없이 자신의 할 일들에 대해 집중을 하면 된다. Consumers 또한 필요할 때마다 우편함에서 정보들을 갖고오면 된다.

이렇게 `MQ`를 이용하게 되면 **비동기**로의 동작이 가능해지며, 각각의 어플리케이션끼리 높은 결합도를 **낮은 결합도**의 환경에서 운영이 가능해진다. 그리고 기존의 방식으로 추가적인 Producers와 Consumers가 있을 때 보다 편리하게 **확장을 하거나 변경**이 가능해지는 장점들이 존재 한다.

## RabbitMQ

`RabbitMQ`는 `Message Broker`의 구체이다.
전통적인 `Message Queue`방식을 지원한다. 그리고 `Message Exchange`를 사용하여 `Pub/Sub` 방식도 구현을 한다.

간단한 동작 순서는 아래의 이미지와 같다.
![](https://i.postimg.cc/Df1FKbbS/img.png)

여기서 `Exchange`는 발행자가 전달한 메세지를 `Queue`에 전달을 하는 역활을 한다. 그래서 소비자는 해당 메세지를 받아 소비하게 된다.

> ### 왜 Producer가 메세지를 바로 Queue에 추가하지 않고, Exchange가 메세지를 추가를 하지??
>
> 이는 메세지 라우팅의 유연성을 제공하기 위해서라고 한다. 여기서 Exchange는 해당 메세지를 어느 큐로 보낼지 결정하는 라우팅 매커니즘을 제공해준다.

## Kafka

`Kafka`는 분산 스트리밍 플렛폼이다.  
기본 동작에 있어서 큰 부류는 `Producer`, `Consumer`, `Cluster`로 구성이 된다. 간단한 동작의 순서는 아래와 같다.

![](https://i.postimg.cc/dVQhWZ9y/cluster-architecture.png)

생산자는 전달하고자 하는 메세지를 `Topic`을 통하여 카테고리화를 한다. 이렇게 소비자는 원하는 `Topic`을 구독하여 데이터를 소비하게 된다.

앞에서 확인한 `RabbitMQ`처럼 생산자와 소비자 모두 서로에 대하여 신경을 쓸 필요없이 자신이 얻고자 하는 `Topic`에만 집중하면 된다.

그리고 각각의 `Broker`들이 하나의 클러스토 구성되어 동작하도록 설계를 한다. 클러스터 내 `Broker`에 대한 분산 처리는 `Zookeeper`가 담당을 하게된다.

실시간 스트림 처리를 위한 StreamAPI 등 다양한 데이터의 쉬운 통합을 위해 `Connector API`를 제공한다.  
![](https://i.postimg.cc/jdm1cyT8/kafka-partition-count.png)

> ### Topic은 정확하게 무엇인가??
>
> `Kafka`는 `RabbitMQ`처럼 메세지를 담을 `Queue`자료구조를 사용하지 않는다. 대신 토픽(Topic)을 사용하여 카테고리에 데이터를 저장을 한다.
> 각각의 토픽에 메세지 로그를 갖고 있다. 하나의 토픽에 하나의 파티션 또는 여러개의 파티션을 가질 수 있다. 각 파티션은 메세지가 지속적으로 추가되며 데이터 순서가 정해져있고 내용이 변경되지 않는다.  
> <br>
> 만약 특정 토픽에 메세지를 저장할 때, 해당 포틱 내부에 여러 파티션이 존재한다면 해당 토픽 내 `Round-Robin`방식으로 분산한다.

이러한 방식으로 `Kafka`는 메세지를 파일 시스템에 저장을 함으로 **영속성을 보장**한다. 그리고 `Consumer`은 `Broker`에게 직접 메세지를 갖고가는 `Pull`방식으로 동작을 하고, `Producer` 중심으로 많은 양의 데이터를 파티셔닝하는데 초점을 맞추었다.

[Message Broker - Kafka and RabbitMQ by 얄팍한 코딩사전](https://www.youtube.com/watch?v=0lyrd5FlETQ&pp=ygUacmFiYml0bXEgdnMga2Fma2EgdnMgcmVkaXM%3D)
