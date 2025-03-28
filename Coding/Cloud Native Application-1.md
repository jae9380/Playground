# 클라우드 네이티브 어플리케이션 (Cloud Native Applications)   
    
## 클라우드 네이티브 어플리케이션이란? 🤔   
   
> 일단 클라우드 네이티브 어플리케이션이 어떤 의미를 내포하고 있는지에 대해서 알아보자.   

안타깝게도 해당 의미를 일반적인 정의는 존재하지 않는다고 한다. 각각의 사람이나 기관마다 해당 의미는 서로 틀릴수 있다고 한다.    

그래도 일반적인 정의에 가까운 클라우드 네이티브 컴퓨팅 재단(CNCF)에서 정의라고 한다.    

> **CNCF의 정의**    <br> 클라우드 네이티브 기술은 퍼블릭, 프라이빗, 하이브리드 클라우드와 같은 실행 환경에서 그 크기를 자유롭게 조절할 수 있는 어플리케이션을 만들고 실행할 수 있는 능력을 제공한다. 컨테이너, 서비스 메시, 마이크로 서비스, 불변 인프라스트럭쳐, 선언적 API 들을 활용하여 구현할 수 있다. <br> 이 기술들을 통하여 관리나 관찰이 쉽고 보다 탄력성 있는 느슨한 결합된 시스템을 제작할 수 있다. <br> 강력한 자동화와 더불어 엔지니어들이 최소한의 노력으로 많은 영향울 미칠 수 있는 변경 작업을 더 자주 할 수 있도록 만들어 준다.

위 CNCF에서 내린 정의를 보았을 떄 클라우드 네이티브 어플리케이션의 장점은 이와 같다고 생각이 든다.    

* #### 마이크로 서비스들 사이 느슨한 결합    
* #### 크기조절이 가능하고 탄력적이며, 관리가 용이하다    

(아직 클라우드 네이티브 어플리케이션에 대한 자세한 학습을 하지 않았기 떄문에 틀릴 수 있다.)

솔직하게 지금은 왜 CNCF에서 이렇게 정의했는지 완벽하게 이해가 되지 않는다.    

좀 더 알아보자.

---

### 마이크로 서비스로 구성   

클라우드 네이티브 어플리케이션은 독립적으로 만들어진 서비스들의 느슨하게 결합하는 것으로 설계가 된다. 각 서비스는 비즈니스 요구에 잘 부합해야 한다. 이런 조건을 만족하는 서비스들이 바로 **마이크로 서비스**라고 부른다.   

<details>
<summary>  느슨한 결합을 하는 이유 </summary>
<div markdown="1">

여기서 왜 결합이 느슨해야 하는지 모를 수 있다. 그래서 간단하게 설명하면    

느슨한 결합(loose coupling)을 하면 각 서비스가 독립적으로 배포 및 확장될 수 있어 유지보수와 변경이 용이하며, 장애가 발생해도 전체 시스템에 미치는 영향을 최소화할 수 있기 때문

</div>
</details>
<br>
클라우드 네이티브 어플리케이션에서 마이크로서비스는 필수적인 기본 요소이다. 여기서 마이크로 서비스 아키택처에 대한 개념 없이 클라우드 네이티브 어플리케이션을 설계하는 것은 불가능에 가깝다.   

<details>
<summary>  MSA에 대한 개념이 필요한 이유 </summary>
<div markdown="1">

마이크로서비스 아키텍처는 클라우드 네이티브 애플리케이션의 대표적인 설계 방식 중 하나로, 서비스의 독립적 배포, 확장성, 장애 격리, 자동화된 운영 등을 가능하게 한다.

클라우드 네이티브 애플리케이션은 느슨하게 결합된 독립적 서비스들로 구성되므로, 이를 효과적으로 구현하려면 마이크로서비스 아키텍처의 원칙(분산 시스템 설계, API 기반 통신, 서비스 독립성 등) 을 이해하고 적용해야 한다.

즉, 마이크로서비스 아키텍처 없이 클라우드 네이티브 애플리케이션을 설계하는 것은 그 핵심 원칙을 따르지 않는 것과 같으며, 결국 클라우드 환경의 이점을 제대로 활용할 수 없기 떄문이다.

</div>
</details>
<br>    

기존 모놀리식 아키택처의 한계를 해결한 **서비스 지향 아키택처(Service-Oriented Architecture, SOA)** 는 특정 비즈니스 기능을 지원하는 서비스들을 조합하여 모듈성 개념을 보여준다.    

SOA에 기반하여 설계된 것들은 여러 서비스를 만들고 개방형 표준과 **엔터프라이즈 서비스 버스(Enterprise Service Bus, ESB)** 로 알려진 중앙 집중형 모놀리식 통합 계층을 사용하여 이들을 하나로 묶는다. 그리고 API관리 계층을 배치하여 여러 비즈니스 기능을 API로 제공한다.

![](https://i.postimg.cc/8C6wvdTX/SOA.gif)

만은 엔터프라이즈 솔루션이 해당 구조를 채택을 했었으며 아직도 많은 엔터프라이즈 소프트웨어가 SOA에 기반하며 설계되어 졌다. 하지만 SOA역시 근본적인 복잡성과 한계 때문에 개발에 있어 민첩성을 떨어트린다. SOA구조로 만들어진 소프트웨어는 독립적으로 크기 조절이 힘들며, 어플리케이션 내부 의존성 때문에 제공하는 서비스를 독립적으로 개발하고 배포하는데 있어서 힘든 부분이 존재한다. 추가적으로 중앙 집중화 어플리케이션이기 떄문에 신뢰성이 낮아지며 어플리케이션 개발 시 서로 다른 다양한 기술을 추가하기 어렵다는 점이 야기된다.   

반면에 MSA는 비즈니스 중심적 서비스를 도입하고 SOA에 있던 ESB와 같은 중앙 컴포넌트를 제거하여 SOA의 한계를 뛰어 넘었다. 중앙 집중화가 아닌 MSA는 비즈니스 핵심 기능을 구현한 독립적인 서비스들로 구성되며 독립적으로 개발하여 배포하고 관리할 수 있게 되었다.

![](https://i.postimg.cc/RV33vv49/SOA-vs-MSA.png)

서비스를 통합하기 위해 MSA에서는 첨부된 이미지 처럼 ESB를 사용하는 대신 각각의 마이크로서비스가 다른 마이크로서비스와 내부에서 알아서 통신하도록 만든다. 추가적으로 이미지에 나타나지 않지만 마이크로서비스는 외부 다른 시스템과 연결하거나 다른 시스템에서 사용할 수 있는 간단한 인터페이스를 제공하기도 한다.    

MSA에서는 단일의 데이터베이스를 사용하지 않으며, 오직 서비스 인터페이스를 거쳐야만 데이터에 접근이 가능하다. 마이크로 서비스는 탄력성과 보안성 등 많은 기본 요건을 충족시킬 수 있는 서비스 간 통신 기능과 비즈니스 로직을 구현해야 한다.   

---

