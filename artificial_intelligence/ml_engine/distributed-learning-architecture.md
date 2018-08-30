# 분산학습 아키텍처

이전까지 우리는 _CloudML_ 이 무엇인지 살펴보고 모델을 클라우드에 던지고 결과를 받는 등 _CloudML_ 전체 시나리오에 대한 이야기만 했다. 이번 글에서는 _ML job_ 의 분산학습 아키텍처를 살펴보고 이해하도록 한다. 

## Scale tier

_CloudML_ 은 _training_ 을 돌릴 서버의 사양을 조정할 수 있는 `Scale-tier` 옵션을 제공하는데 그 안에는 _BASIC, STANDARD\_1, PREMIUM\_1, BASIC\_GPU_ 그리고 _CUSTOM_ 까지 총 다섯가지 종류가 있다. _CUSTOM_ 을 알아보기 전에 다른 네 가지 방식의 사양을 보자.

|  Cloud ML Engine scale tier |  Compute Engine machine type |
| :--- | :--- |
|   BASIC |  ● A single worker instance.  ● n1-standard-4 |
|   STANDARD\_1 |  ● 1 master, 4 workers, 3 parameter servers.  ● **master**: n1-highcpu-8, **workers**: n1-highcpu-8, **parameter servers**: n1-standard-4 |
|   PREMIUM\_1 |  ● 1 master, 19 workers, 11 parameter servers.  ● **master**: n1-highcpu-16, **workers**: n1-highcpu-16, **parameter servers**: n1-highmem-8 |
|   BASIC\_GPU |  ● A sigle worker instance.  ● n1-standard-8 with one k80 GPU |

_scale tier_ 별로 _GCE_ 인스턴스의 사양이 정의되는데 자세한 스펙은 인스턴스 생성 페이지를 통해 살펴볼 수 있다. 여기서 중요한 것은 STARNDARD\_1, PREMIUM\_1 에 있는 master, worker, parameter server 에 대해 인지하는 것이다. 이것들에 대해 알게되면 CUSTOM 도 자연스럽게 알 수 있다.

그렇다면 master, worker, parameter server 는 각각 무엇인가? 기본적으로 CloudML 을 통해 분산학습을 하게 되면 training cluster 는 여러개의 노드\(인스턴스\)에 할당되어 작업을 수행하게 되는데 여기서 노드는 흔히 replica 로 부르게 된다. 결국 replica 가 master, worker, parameter server 로 구분되어 사용 되는데 이때 **분산학습에 해당하는 scale tier 는 single worker 가 아닌 STANDARD\_1 과 PREMIUM\_1** 이 되겠다. \(추가로 CUSTOM 까지\)

**Master**  
다른 작업들을 관리하고 전체 상태를 모니터링 하게 된다. 모든 작업이 완료될 때까지 수행된다.

**Worker**  
training 을 수행하는 replica 이다. 한편 worker 간 작업을 조정하는 특수한 replica 는 chief worker 라고 부른다.

**parameter server**  
worker 간에 공유된 모델 상태를 조정하게 된다.

master 와 worker 의 경우 직관적이기 때문에 이해하기 쉬운 반면에 parameter server 는 분산학습 개념이 없으면 이해하는데 어려움이 있을 수 있다. 아래 그림을 통해 상세히 알아보도록 하자.  


![](https://t1.daumcdn.net/cfile/tistory/99AA9C3A5A5297D313)

Parameter node \(replica\) 는 Worker node 에서 갱신되는 모델의 정보\(이를테면 Gradient vectors\)를 주기적으로 업데이트해서 다시 worker 에 전달하는 역할을 수행한다. 모든 동작은 비동기로 병렬처리 되는데 예를들어 10,000개의 배치 작업을 10개의 worker node 가 수행한다면 대략 1,000 개씩 맡아서 수행을 하게 되는 개념이다. 정확하게 나누어 떨어지지는 않고 rough 하게 처리된다. 거창해보이지만 걱정할 것 없다. 우리가 신경 쓸 건 없으니까.

#### CUSTOM

custom 은 앞에 실전 심화에서도 살펴본 것과 같이 replica 를 상세히 설정할 수 있는 scale-tier 이다. 다시 노멀하게 사용되는 config.yaml 의 각 설정을 하나씩 살펴보자. 

```text
trainingInput: 
  scaleTier: CUSTOM 
  masterType: complex_model_m 
  workerType: complex_model_m 
  parameterServerType: large_model 
  workerCount: 9 
  parameterServerCount: 3
```

masterType, workerType, parameterServerType 은 각각 사전 정의된 사양의 GCE 인스턴스 \(replica\) 이고 workerCount 와 parameterServerCount 는 worker node 와 parameter server node 의 개수를 지정하는 설정이 되겠다. 여기서 다뤄지는 서버들의 종류는 밑에 표를 참고하도록 하자.

|  Machine type |   | Cloud ML Engine machine type |  CPU | GPUs | Memory |
| :--- | :--- | :--- | :--- | :--- | :--- |
|  standard | n1-standard-4 |  XS |  - |  M |  |
|  large\_model | n1-highmem-8 |  S |  - |  XL |  |
|  complex\_model\_s  | n1-highcpu-8 |  S |  - |  S |  |
|  complex\_model\_m | n1-highcpu-16 |  M |  - |  M |  |
|  complex\_model\_l | n1-highcpu-32 |  L |  - |  L |  |
|  starndard\_gpu  | n1-standard-8  |  XS |  1 \(K80\) |  M |  |
|  complex\_model\_m\_gpu | n1-standard-16  |  M |  4 \(K80\)  |  M |  |
|  complex\_model\_l\_gpu | n1-standard-32 |  L |  8 \(K80\)  |  L |  |
|  stardard\_p100 \(Beta\) | n1-standard-8 |  XS |  1 \(P100\)  |  M |  |
|  complex\_model\_m\_p100 \(Beta\)  | n1-standard-16 |  M |  4 \(P100\)  |  M |  |

설정에 따라 CPU 개수나 메모리가 약 두 배로 증가한다고 보면 되는데 XS, S, M, L, XL 순이다. 이것을 참고해서 CPU 와 메모리의 가용 용량을 가늠하면 되겠다. 더 좋은 성능을 갈구할 수록 비례적으로 [비용](https://cloud.google.com/ml-engine/pricing)이 증가하게 되니 적당한 선에서 타협할 수 있는 지혜가 필요하겠다.

여러가지를 준비했지만 이번 글에서 제일 중요한 것은 위에 그림이다. parameter  server 와 worker 의 관계를 이해해야 CUSTOM 모드를 사용할 때 적당한 replica 개수를 지정할 수 있기 때문이다. 실제로 사용할 때 어떤 비율로 parameter server 와 worker 를 구성하는 것이 이상적인지 알 수 없다. 모델마다 다를 수도 있고, 그저 경험과 삽질을 통해 알 수 있을 것이라고 추측할 뿐이다.

#### 참고자료

[https://cloud.google.com/ml-engine/docs/training-overview\#job\_configuration\_parameters](https://cloud.google.com/ml-engine/docs/training-overview#job_configuration_parameters)  
[https://cloud.google.com/ml-engine/docs/distributed-tensorflow-mnist-cloud-datalab](https://cloud.google.com/ml-engine/docs/distributed-tensorflow-mnist-cloud-datalab)  
[https://cloud.google.com/ml-engine/pricing](https://cloud.google.com/ml-engine/pricing)

