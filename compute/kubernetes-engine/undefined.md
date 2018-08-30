# 개념

## **Google 의 Container 기술**

구글은 10년 이상 전부터 리눅스 컨테이너 기술을 관리해 오면서 3개의 Container management systems 을 구축 했는데, 통합 Container Cluster Manager **Borg** 로 시작해서 차세대 Container Cluster Manager **Omega**, 그리고 이제부터 살펴볼 **Kubernetes** \(보통 쿠버네티스 라고 읽음\)가 있겠다. Kubernetes 는 오픈소스 프로젝트 이기 때문에 [GitHub 에 소스가 공개](https://github.com/kubernetes/kubernetes)되어 있고 필요하다면 contribute 하거나 Apache License 2.0 에 맞게 사용하면 된다.

사실 Kubernetes 를 사용하기 위해서는 가상화 기술이나 Container 에 대한 최소한의 지식이 필요하기 때문에 언급하지 않고 넘어갈 수가 없겠다. 다음 그림은 Container 의 특징을 매우 잘 묘사하고 있다. 좌측이 기존 시스템이고 우측이 Container 를 사용했을 때의 모습을 형상화 한 것이다.

![](https://t1.daumcdn.net/cfile/tistory/260EBF33595EC2E633)

[https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)  


일반적으로 리눅스 시스템은 커널이 존재하고 그 위에 시스템에 필요한 모든 Libraries 가 널려\(?\)있다. 그리고 Applications 은 널려 있는 Libraries 중에 필요한 library 만 가져다 쓰면 되는구조. 시장과도 같은 개념이겠다. 사실 이 프로세스는 오래 전부터 사용된 전통적인 방식이라 이해 하는데 문제가 없다. 이 전통적인 방식에서 오는 문제는 library 를 공통으로 사용하다 보니 library 의 업데이트가 필요한 경우 의존성의 문제를 생각하지 않을 수가 없었다. 또한 리눅스 베이스로 개발해본 사람이라면 이러한 부분에서 오는 side-effect 에 골머리 한번씩은 썩어 봤을 듯.

여튼 이러한 시국에 혜성처럼 등장한 것이 Container 시스템이다. Application 을 위해 한 개의 독립적인 환경을 부여한다고 보면 이해가 쉽다. 각각의 Application 은 독립적인 공간에 자신에게 필요한 library 만 사용하기 때문에 고려해야 하는 사항들이 줄어 패치가 간단하다. 사실 더 핵심은 OS 레벨에서의 가상화 개념인데 가령 예를 들어 Mips 플랫폼에 x64 로 컴파일 된 Application 을 올릴 수 있다는 것. 꿈만 같았던 이야기가 아닌가? cross-compile 을 위해 무던한 노력을 하지 않았던가. \(혜성처럼 등장했다고 표현했지만 사실 엄청 나게 많은 실패가 존재했고 오래 전부터 가상화나 Container 는 존재해왔다. 다만 너무 무거워서 사용하지 못할 수준이라는 점이 문제.\)

이렇게 동작하는 Container 를 더욱 빛나게 만드는 것이 GCP 에서 사용되는 Kubernetes 되시겠다. 

## **GCP 에서의 Container Engine** 

Kubernetes 라는 이야기를 하다가 Container 로 넘어왔기 때문에 Container Engine 의 의미가 추상적으로 그려진다. Google Cloud Platform \(GCP\) 에 Container Engine 내부적으로 사용되고 있는 Cluster 시스템이 결국 Kubernetes 이다. 그래서 Google Container Engine 는 GCE 가 아닌 GKE 로 호명된다. \( GCE 로 불리지 않는 이유는 Google Compute Engine 과 겹치는 이유가 더 크겠지만. \)

![](https://t1.daumcdn.net/cfile/tistory/223B1750595EC75030)

일반적인 가상화 기술에서 Host 는 여러 개의 container 를 실행할 수 있는데 Google Cloud 로 들어오게 되면서 클라우드의 특징처럼 사용량이나 트래픽에 대해 유연하게 동작할 수 있도록 Container orchestration 이 가능해진다. 이 말은 모든 Host \(여기서는 인스턴스로 생각하면 된다\)  를 management 해서 특정 Host 로 부하가 쏠려 container 가 제대로 된 성능을 내지 못하는 것을 방지하고 균등한 성능을 낼 수 있도록 보장해준다는 의미다. 또한 당연한 이야기지만 헬스체크까지 지원되니 안심하고 사용할 수 있겠다.

그 정도는 다른 management systems 에서도 다 되지 않나? 라는 질문을 할 수 있다. 맞다, 다른 시스템에서도 다 된다. 하지만 부연설명을 보태자면, Kubernetes 는 Linux Foundation 프로젝트이다. 그게 무슨 의미냐고? 유연한 가상화를 위한 여러가지 기술이 Linux kernel 에 상당한 분량으로 contributes 되어 있고, 이것은 Container 간의 memory share 등에서 다른 Container management systems 과 상당한 차이를 보일 것이다.  


개념을 정리 했으니 다음 편에서는 Google Cloud 에서 Container Engine 을 활용하는 방법과 그 안에서 무엇이 가능한지에 대해 더 자세히 살펴보겠다. 사실 GKE 나 다른 Container management systems 에서 가장 중요한 것은 auto-scaling 과 load-balancing 정도일텐데, 이러한 것들은 Google 의 첫 번째 Container management systems 인 Borg 에서부터 가능했다. 무려 10년이 넘는 기간 동안 얼마나 많은 노하우가 축적되었겠는가.

## **Update. 2017.11.21**

현재는 Container Engine 이 Kubernetes Engine 으로 변경되었다. 굿!!

