# cluster 생성

[Container Engine 의 기본 개념](concept.md)에 대해서 살펴보았고 이번에는 Google Cloud Platform \(GCP\) 에서 Container Engine 을 사용하기 위한 준비단계에 대해서 다뤄보도록 한다. 과정중에 Kubernetes 의 내부를 간단한 다이어그램을 통해 살펴볼 것이다. 우선 Cluster 라는 것을 생성하기 위해 다음 화면과 같이 좌측 메뉴에서 **\[Container Engine\]** 카테고리를 선택하도록 하자.

![](https://t1.daumcdn.net/cfile/tistory/243CC742596555D618)

**\[컨테이너 클러스터 만들기\]** 메뉴를 통해 클러스터를 생성하도록 한다. \(여러 개의 Container 를 묶어서 하나의 시스템처럼 동작하도록 하는 개념을 Container cluster 라고 부른다. cluster 라는 단어는 범용적으로 사용되기 때문에 이해하는게 어렵지 않다.\)

![](https://t1.daumcdn.net/cfile/tistory/251AF042596555D712)

클러스터의 기본적인 내용을 설정할 수 있다. 영역으로 리전을 선택할 수 있고 머신 유형을 통해 CPU 와 메모리를 직접 설정할 수 있다.

![](https://t1.daumcdn.net/cfile/tistory/25231142596555D915)

그리고 스크롤을 밑으로 내려보면 사실상 제일 중요한 설정이 있는데 노드의 크기를 선택할 수 있다. 위에서 Container Cluster 는 여러 개의 Container 를 하나의 시스템처럼 구성할 수 있다고 했는데 조금 더 확장해서 보면 복수 개의 Container 를 담고 있는 호스트 \(여기서는 VM 인스턴스\) 를 여러 개 묶어서 사용할 수 있다. 즉, Cluster 는 여러 개의 VM 인스턴스의 집합이라고 보면 되겠다.

![](https://t1.daumcdn.net/cfile/tistory/246E5342596555DA1A)

한번 더 주의 깊게 봐야하는 부분이 선택한 크기만큼 VM 인스턴스가 생성되기 때문에 인스턴스 개수 만큼 요금이 청구된다는 사실이다. 무작정 많이 생성한다고 좋은 것이 아니라는 이야기이므로 제공하려는 서비스의 목적에 맞게 생성해주면 되겠다. VM 인스턴스 개수는 나중에라도 **"인스턴스 그룹"** 을 편집해서 개수를 조정할 수 있으니 참고하도록 하자. 이제 **\[만들기\]** 를 통해 설정을 끝내고 나면 클러스터가 생성된다. 

![](https://t1.daumcdn.net/cfile/tistory/2158EA42596555DB1B)

**\[VM 인스턴스\]** 카테고리에서 실제로 생성된 인스턴스도 확인이 가능하다. 일반적으로 생성한 인스턴스와 동일하게 SSH 접속이나 기타 설정이 가능한 상태이다.

![](https://t1.daumcdn.net/cfile/tistory/267B3742596555DC13)

위에서 설명한 클러스터나 노드에 대해서 공식 홈페이지에는 다음과 같은 다이아그램을 제공한다. 아마도 클러스터를 가장 이상적으로 잘 표현하고 있는 그림이 아닐까. 실제로 각 노드 \(VM 인스턴스\) 의 집합이 하나의 시스템처럼 동작할 수 있도록 Master 가 존재하고 여러개의 노드는 Kubernetes API 를 통해 마스터와 통신하는 구조이다. \(각 노드는 여러개의 containerized app 을 갖을 수 있다\)

![](https://t1.daumcdn.net/cfile/tistory/2635C63359655B0013)

정확히 어느 버전부터인지 모르겠지만 Kubernetes 는 사용자 GUI 를 제공한다. 클러스터에 **\[연결\]** 버튼을 누르면 다음과 같은 팝업이 출력되고 GUI 에 접속할 수 있도록 도와주는 명령어를 보여준다.   


![](https://t1.daumcdn.net/cfile/tistory/2258E342596555DD1A)

위에 명령어를 터미널에 붙여넣고 실행 해보자. 아마도 Cloud Platform 에서 Kubernetes 를 처음 사용하는 경우라면 아래와 같은 메시지가 출력 될 것이다. 이는 터미널에서 Kubernetes 를 제어하기 위한 kubectl componets 가 인스톨 되어 있지 않기 때문이다. \(실행한 명령어는 제대로 수행되었다. 메시지는 터미널에서 클러스트를 제어하기 위해서는 kubectl 이 필요하다는 친절한 알림 정도로 생각하면 된다.\)

![](https://t1.daumcdn.net/cfile/tistory/276AD438596555E11B)

gcloud components install kubectl  명령어를 통해 kubectl 을 인스톨 하도록 한다.

![](https://t1.daumcdn.net/cfile/tistory/22379738596555E21C)

인스톨이 끝나면 kubectl proxy 명령어를 통해 웹을 실행한다. 

![](https://t1.daumcdn.net/cfile/tistory/2729F638596555E31A)

Starting to serve on 127.0.0.1:8001 이라는 메시지가 보이지만 실제로 GUI 에 접근하기 위해서는 127.0.0.1:8001/ui 까지 입력해줘야 한다. 이제 브라우저를 열고 주소를 입력해보자. 다음과 같이 Kubernetes 화면을 볼 수 있을 것이다.

![](https://t1.daumcdn.net/cfile/tistory/2264C738596555E014)

Kubernetes GUI 를 통해 다양한 설정과 모니터링이 가능하다. 상당히 편리한 부분이고 가시성이 좋지만 우리는 터미널의 명령어와 친해질 필요가 있다. 이유는 간단하다. 그것이 시스템의 내부구조를 이해하는 가장 빠른 방법이기 때문이다. 그렇기 때문에 다음 편에서는 간단한 앱을 터미널을 통해 배포하는 예제를 실습하도록 하겠다.

