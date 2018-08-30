# 터미널 배포

Google Kubernetes Engine \(GKE\) 관련해서 앞서 2개의 글을 통해 대략적인 이해를 할 수 있었을 것으로 예상된다. 이번 포스팅에서는 더 깊은 이해를 위해 터미널을 통해 실제 앱을 kubernetes 로 배포하고 업데이트를 진행 해보자. 이 예제의 내용은 [qwiklabs](https://qwiklabs.com/) - Hello Node Kubernetes 를 통해서도 확인할 수 있다. 아래 예제는 모두 클라우드 셸에서 진행한 것이지만 구글 클라우드 SDK 가 설치된 다른 환경에서 진행하더라도 무리는 없다.

## Node.js 애플리케이션 생성

우선 실습에서 사용하는 간단한 Node.js 앱을 작성해보자. 코드는 아래와 같다.

```text
// server.js
var http = require('http');
var handleRequest = function(request, response) {
  response.writeHead(200);
  response.end(`(pid:${process.pid}) Hello World!`);
}
var www = http.createServer(handleRequest);
www.listen(8080);
```

## 도커 컨테이너 이미지 생성

server.js 앱을 통해 쿠버네티스의 동작을 확인해볼텐데 우선 앱을 도커 컨테이너 이미지로 생성해주도록 하자. 생성을 위해 Dockerfile 을 생성해준다. 쿠버네티스 동작의 가장 작은 단위인 pod 을 이루는게 도커 컨테이너임을 생각하면 컨테이너 생성은 당연한 과정이다.

```text
FROM node:6.9.2
EXPOSE 8080
COPY server.js .
CMD node server.js
```

파일의 내용을 대략 살펴보면, 

* Docker Hub 에서 6.9.2의 노드 버전을 가져와서 사용할 것이고 
* 포트는 8080을 사용한다. 
* server.js 를 이미지로 사용하기 위해 복사하고, 
* 실행 시키는 방법을 명시해줬다.

아래와 같이 도커 이미지를 빌드한다. \( PROJECT\_ID 부분은 본인의 프로젝트 번호를 입력해줘야 한다 \)

```text
$ docker build -t gcr.io/PROJECT_ID/hello-node:v1 . 
```

이미지가 정상적으로 빌드 된 것을 셸 내에서 docker images 명령을 통해 확인할 수 있다.

![](https://t1.daumcdn.net/cfile/tistory/99809D3B5AEECAED32)

이제 생성한 이미지를 기반으로 도커 컨테이너를 실행시켜준다. \( 즉, 생성한 이미지로 컨테이너를 만든다고 생각하면 된다 \)

```text
$ docker run -d -p 8080:8080 gcr.io/PROJECT_ID/hello-node:v1
```

위 명령을 실행하고 화면에 출력되는 해시\( HASH \) 값은 컨테이너 고유의 ID 값이다. 이제 실행된 컨테이너에 통신을 해서 정상적으로 server.js 가 돌고 있는지 확인해보면 되겠다. 아래와 같이 curl 명령을 사용해서 노드가 사용중인 8080 포트에 패킷을 전송해보자.

```text
$ curl http://localhost:8080
```

PID 값과 함께 우리가 지정한 문구가 정상적으로 출력되는 것을 확인할 수 있다.

```text
$ curl http://localhost:8080
(pid:5) Hello World!
```

이렇게 컨테이너로 노드 서버가 동작하고 있는 것을 실제 패킷을 보내 확인하는 방법도 있겠지만 docker 명령어를 통해서도 확인이 가능하다.

```text
$ docker ps
```

추가적으로 docker ps 는 현재 동작\( running \)중인 컨테이너에 대한 내용만 출력이되고 등록되어 있는 전체 컨테이너를 보기 위해서는 뒤에 -a \(--all\) 옵션을 줘서 확인할 수 있겠다. 동작중인 컨테이너를 중지 시키기 위해서는 stop 옵션을 사용하도록 한다.

```text
$ docker stop [CONTAINER ID]
```

Google Container Registry \(GCR\) 등록

자, 여기까지 했으면 도커에 대한 기본과정은 끝났다. 본격적으로 앱을 쿠버네티스에서 사용할 수 있도록 우선 GCR \(Container Registry\) 에 등록하도록 하자. GCR 은 Docker 이미지의 비공개 저장소이다. 이 저장소에 우리의 앱을 올려놓고 쿠버네티스에서 가져다 사용할 수 있도록 할 것이다. 

```text
$ gcloud docker -- push gcr.io/PROJECT_ID/hello-node:v1
```

클라우드 콘솔에서 확인해보도록 하자. TOOLS 밑에서 Container Registry 를 확인할 수 있다.

![](https://t1.daumcdn.net/cfile/tistory/99D828395AEED1EA0B)Container Registry

아래처럼 우리가 추가한 hello-node 앱을 확인할 수 있다.  


![](https://t1.daumcdn.net/cfile/tistory/9984303E5AEED22609)Container Registry

앱 이름 \( hello-node \) 을 클릭하고 들어가면 버전\(v1\)을 포함한 상세 정보를 볼 수 있다.  


![](https://t1.daumcdn.net/cfile/tistory/99C6433B5AEED28D0A)Container Registry

## 클러스터 생성

이제 여러개의 노드[1](http://jybaek.tistory.com/739?category=696494#footnote_739_1)를 관리할 수 있는 클러스터를 아래와 같이 클라우드 셸에서 생성하도록 한다. 이때 클러스터는 도커 이미지가 있는 GCR 과 같은 영역에 만드는 것을 좋다고 한다.

```text
$ gcloud container clusters create hello-world \
                --num-nodes 2 \
                --machine-type n1-standard-1 \
                --zone us-central1-f
```

클러스터는 2개의 노드를 n1-standard-1 타입으로 us-central1-f 에 갖게된다. 여기서의 노드는 도커가 아닌 실제 인스턴스의 개념이다. 노드의 개수에 따라 인스턴스가 생성되며 비용이 발생하기 때문에 주의하도록 하자. 다음과 같이 콘솔에서 생성된 인스턴스를 확인할 수 있다.

![](https://t1.daumcdn.net/cfile/tistory/9937CE415AEED4DB19)Compute Engine

그리고 방금 생성한 클러스터는 Kubernetes Engine 카테고리에서 확인할 수 있다. 클라우드 셸과 같은 터미널에서도 모든 설정을 편집할 수 있지만 아래 보이는 콘솔\( GUI \)을 통해서도 모든 설정이 가능하다는 것을 알 수 있다. 다만, 개발의 특성상 터미널 명령어가 약간은 더 빠르게 개선되고 업데이트되는 것은 어쩔 수 없다. 가령 예를들어 새로운 기능이 나오게 되면 화면상으로 노출은 되지 않지만 터미널에서는 beta 를 달고 출시되기도 한다.  


![](https://t1.daumcdn.net/cfile/tistory/993991415AEED52A09)Kubernetes Engine

이제 kubectl 명령어를 통해 앞서 생성했던 도커 컨테이너로 생성해둔 앱을 배포하도록 하자.  


## pod 생성

pod 은 하나 또는 여러개의 컨테이너를 갖는 쿠버네티스의 핵심 개념이다. 그 안에서 실제 우리 앱의 동작이 모두 이루어지게 되는데 여기서 배포를 통해 그 의미를 파악해보도록 하자.

```text
$ kubectl run hello-node \
    --image=gcr.io/PROJECT_ID/hello-node:v1 \
    --port=8080
```

생성이 완료되면 다음과 같은 출력을 볼 수 있다.

```text
deployment "hello-node" created
```

이제 hello-node:v1 이미지를 실행하는 하나의 포드가 생성되었다. 아래 명령어를 통해 배포 상태를 확인할 수 있다.

```text
$ kubectl get deployments
```

배포에 대한 정보가 아래와 같이 출력된다.

```text
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           2m
```

또한 pod 은 다음과 같이 확인하도록 한다.

```text
$ kubectl get pods
```

pod 에 대한 정보를 확인할 수 있다.

```text
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-6f87f9797c-czt5t   1/1       Running   0          3m
```

클라우드 콘솔에서도 생성된 pod 에 대한 정보를 확인할 수 있다.

![](https://t1.daumcdn.net/cfile/tistory/99F9A9365AEF8D7431)Kubernetes Engine

## 외부 트래픽 허용 

서비스를 만들었으니 이제 외부에서 접속 가능하도록 트래픽을 허용해주도록 하자. pod 는 기본적으로 클러스터 내부 IP 로만 접근이 가능하기 때문에 서비스하기 위해서는 외부 접속을 열어줘야 한다.

```text
$ kubectl expose deployment hello-node --type="LoadBalancer"
```

이렇게 LB 를 달아주게 되면 향후 증가되는 pod 에 대해서 트래픽의 고른 분배가 가능해진다. \(pod 이 아닌 배포 개체를 노출시켰다는 점을 보자\) 이제 아래 명령어를 통해 클러스터에 있는 모든 네트워크를 확인해보도록 하자. 방금 LB 로 지정한 외부 연결이 가능한 네트워크도 확인이 가능할 것이다.

```text
$ kubectl get services
```

다음과 같은 출력을 볼 수 있다. 혹시 EXTERNAL-IP 가 pending 상태로 보인다면 잠시 기다렸다가 다시 확인해보도록 하자. IP 할당에 약 1분 가까운 시간이 소요된다.

```text
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
hello-node   LoadBalancer   10.3.243.168   35.188.107.14   8080:31903/TCP   3m
kubernetes   ClusterIP      10.3.240.1     <none>          443/TCP          17m
```

자, 이제 EXTERNAL-IP 를 통해 접속해보도록 하자.

```text
http://<external_ip>:8080
```

## 서비스 규모 확장

이제 쿠버네티스의 강점이라고 말할 수 있는 애플리케이션의 확장을 살펴볼 차례이다. 얼마나 간단하게 확장 가능한지 확인하도록 하자.

```text
$ kubectl scale deployment hello-node --replicas=4
```

즉시 결과를 확인할 수 있다.

```text
deployment "hello-node" scaled
```

업데이트된 배포에 대한 내용과 pod 에 대해서도 확인해보도록 하자.

```text
$ kubectl get deployment && kubectl get pods
```

정상적으로 replicas=4 가 적용된 것을 확인할 수 있다.

```text
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   4         4         4            4           26m
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-6f87f9797c-65hfn   1/1       Running   0          4m
hello-node-6f87f9797c-9ndw2   1/1       Running   0          4m
hello-node-6f87f9797c-czt5t   1/1       Running   0          26m
hello-node-6f87f9797c-npt4t   1/1       Running   0          4m
```

아래 그림은 클러스터의 상태를 보여주는 매우 좋은 그림이다.

![](https://t1.daumcdn.net/cfile/tistory/9907383A5AEF92600B)

## 서비스 업그레이드

우리가 배포했던 앱에서 치명적인 오류가 발생해서 패치를 해야 하는 일이 생겼다. 이런 경우 어떻게 해야할까? 우선 실습을 위해 최초 생성했던 server.js 의 소스코드의  일부분을 수정하도록 한다. \( kubernetes 라는 문구를 추가해줬다. 치명적인 오류라고 가정하자 \)

```text
response.end(`(pid:${process.pid}) Hello Kubernetes World!`);
```

이렇게 코드가 수정이 되었으면 우선 도커 이미지가 새로 빌드되어야 한다. 빌드를 하고 GCR 에 새로운 버전을 등록하도록 하자.

```text
$ docker build -t gcr.io/PROJECT_ID/hello-node:v2 . 
$ gcloud docker -- push gcr.io/PROJECT_ID/hello-node:v2
```

처음 이미지를 빌드할 때와는 다르게 매우 빠르게 빌드에 성공하는데 이건 도커 자체에서 캐시를 갖고 있기 때문이다. 변경된 부분이 많지 않기 때문에 빠른 빌드가 가능했다. 그리고 또 하나 수정을 해야 하는 부분이 있는데 바로 배포 부분이다. 현재는 hello-node:v1 로 배포가 되고 있기 때문에 이 부분을 hello-node:v2 로 변경해줘야 한다.

```text
$ kubectl edit deployment hello-node
```

위 명령어를 입력했을 때 편집기가 열릴텐데 spec.template.spec.containers.image 부분의 v1 을 v2 로 변경해주도록 하자. vim 의 경우 저장하고 빠져나오면 변경 상태가 즉시 반영이 된다. pod 의 상태를 살펴보도록 하자.

```text
$ kubectl get pods
```

여기서 재미있는 부분을 확인할 수 있게 된다.

```text
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-5559467cdb-4jvz7   0/1       Pending   0          45s
hello-node-5559467cdb-fcwrr   0/1       Pending   0          45s
hello-node-5559467cdb-h45cj   1/1       Running   0          48s
hello-node-5559467cdb-n6tbz   1/1       Running   0          48s
hello-node-6f87f9797c-czt5t   1/1       Running   0          42m
```

혹시 앞서 NAME 을 유심히 본 사람이라면 뭔가 다른게 보일텐데 6f87f9797c 는 이전 리비전을 의미하고, 5559467cdb 는 새로운 버전을 의미한다. 즉, 순차적으로 롤링 업데이트가 이루어지고 있는 과정을 kubectl get pods 명령어로 확인이 되는 것이다. 해당 명령어를 반복적으로 입력해보자. 클라우드 콘솔에서도 해당 버전에 대한 정보를 얻을 수 있다.

![](https://t1.daumcdn.net/cfile/tistory/99C5C03E5AEF96BF18)Kubernetes Engine

 Revision history 부분을 유심히 보면 각 Name 이 어떤 GCR 을 참조하고 있는지 알 수 있다. 

## 마무리

이렇게 퀵랩 실습을 통해 쿠버네티스의 메커니즘에 대한 부분을 수박 겉핥기식으로 살펴보았다. 사실 쿠버네티스 자체가 굉장히 큰 엔진이기 때문에 글 몇개로 모두 이해한다는 것은 애초에 무리가 있다. 앞으로 여러개의 포스팅을 통해 다양한 각도로 쿠버네티스를 살펴보고 이해하는 시간을 갖도록 해보자.

