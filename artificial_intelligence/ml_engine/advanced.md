# 실전 심화

앞서 [CloudML 의 개념](concept.md)과 [기본적인 사용 방법](basic.md)에 대해서 살펴보았고 이번에는 조금 더 실전에 가까운 글을 발행하도록 한다. 기본편에서 로컬 테스트와 클라우드 테스트를 진행했지만 코드의 내용이 단순히 _TensorFlow_ 의 버전을 확인하는 수준이라서 실전에서 사용하려면 막막하거나 여러 장애 요소가 숨어있다. 이번 글에서는 _TensorFlow_ 홈페이지에 있는 [샘플 예제](https://www.tensorflow.org/versions/master/tutorials/audio_recognition) 실습을 통해 _CloudML_ 의 울렁증을 극복할 수 있도록 한다.

## **예제코드 준비**

우선 샘플 예제 사용을 위해 _TensorFlow_ 소스코드를 다운로드 받도록 하자. 다운로드 방법은 무엇이든 상관없고 여기서는 [GitHub](https://github.com/tensorflow/tensorflow) 을 통해 받는 것으로 한다. 소스코드를 받았으면 이제 샘플 예제를 살펴보아야 하는데 예제에 대한 자세한 설명은 [홈페이지](https://www.tensorflow.org/versions/master/tutorials/audio_recognition)를 참고하도록 하자. 간단하게 말하자면 미리 준비된 음성 세트를 학습시키고 새로운 음성을 분류해내는 모델이다.

소스코드의 위치는  `tensorflow/examples/speech_commands`  이다. \(아마도\) _TensorFlow 1.4_ 에 들어오면서 생긴 예제기 때문에 오래전에 받아놓은 소스코드에서는 해당 경로를 찾지 못할 수도 있으니 최신 코드를 받는 것이 좋겠다. 우리에게 필요한건 _TensorFlow_ 전체코드가 아니라 _speech\_commands_ 예제기 때문에 _examples_ 경로로 이동해서 작업을 진행하도록 한다. 혹은 _speech\_commands_ 를 다른 작업 디렉터리로 복사해서 사용해도 좋다. 개인적으로는 GitHub 에서 받는 트리가 지저분해지는 것을 좋아하지 않기 때문에 다른 작업 디렉터리로 복사해서 사용하는 것을 선호한다.

이제 본격적인 _CloudML_ 내용을 살펴볼텐데 [개념편](concept.md)에서 이야기한 것처럼 _CloudML_ 은 모델을 구글 클라우드에 던지고 클라우드 환경에서 학습이 되도록 지원을 한다. 또한 학습된 내용을 기반으로 _product_ 형태로 사용도 가능하다. 자, 모델을 올바른 형태로 잘 던져주기만 하면 내 로컬환경과 상관없는 클라우드에서 작업이 된다는 의미인데 클라우드의 무한한 자원을 사용할 수 있게 되므로 고통받던 로컬 PC 는 모델을 던지고 정상적으로 동작하는 것이 확인되면 잠시 꺼두셔도 좋다. 

## **Local 테스트**

우선 로컬 환경에서 모델이 정상적으로 수행되는지 확인을 해야 한다. \(실제 예제가 돌아가게 하려면 약간의 수정이 필요하다. 우선 _speech\_commands_ 디렉터리에 `__init__.py` 를 생성해서 모듈로 인식될 수 있도록 하고, `train.py` 내부에 _input\_data_ 와 _models_ 를 _import_ 하는 부분을 _speech\_commands_ 하위에 있는 개념으로 수정해줘야 한다. `import speech_commands.input_data as input_data` 처럼 말이다\)

```text
$ gcloud ml-engine local train \
--module-name speech_commands.train \
--package-path speech_commands
```

앞서 [기본편](basic.md)에서 언급된 내용이지만 _Local_ 테스트는 로컬에 있는 _TensorFlow_ 를 이용하기 때문에 _TensorFlow_ 가 정상적으로 _import_ 되어 사용될 수 있는 환경이 보장되어야 한다. 아래는 실제 수행되는 과정을 담은 예제 스크린샷이다.

![](https://t1.daumcdn.net/cfile/tistory/999EA8405A3472F121)

## **Cloud 테스트**

CloudML 에 모델을 던지는 방법은 이전 [기본편](basic.md)에서 살펴본 것과 동일하게 진행한다. 우선 아래처럼 필요한 환경변수를 설정하도록 하자.

```text
$ BUCKET_NAME=jybaek_cloudml_test
$ JOB_NAME=tf_ex_1
$ REGION=us-central1
$ OUTPUT_PATH=gs://$BUCKET_NAME/$JOB_NAME
```

각 요소에 대해서 살펴보면 `BUCKET_NAME` 은 결과를 저장하거나 실행 중간에 파생되는 내용을 저장하는 용도로 사용될 것이다. `JOB_NAME` 은 _CloudML_ 로 지금 던진 모델을 구분할 수 있도록 설정하면 된다. 내 프로젝트 안에서 동일한 _JOB_ 이름을 갖지 않아야 하므로 뒤쪽에 인덱스 등을 넣어주면 좋다 \(_JOB_ 이름이 충돌나면 어차피 에러가 난다\). `REGION` 은 모델이 _training_  될 지역을 선택하는 부분이다. _CloudML_ 은 사용하는 _Scale Tier_ 에 따라 가격이 책정되는데 그 가격이 _REGION_ 별로 또 다르다. 참고로 _asia_ 가 제일 비싼데 설정하지 않으면 _REGION_ 은 _asia-east1_ 로 지정된다 \(현재 IP 지역 기준일 듯\). 마지막으로 `OUTPUT_PATH` 는 앞에 생성한 `BUCKET_NAME` 과 `JOB_NAME` 을 조합해서 _Google Storage_ 를 지정해주는 부분이다 \( **gs** 가 _Google Storage_ 를 의미하며 이처럼 터미널에서 바로 접근할 수 있는 형태로 제공된다\). 우리의 소스코드\(아래쪽에서 _package_ 로 지정되는\)와 기타 저장공간으로 사용될 예정이다.

환경설정 이후에는 바로 테스트가 가능하다.

```text
$ gcloud ml-engine jobs submit training $JOB_NAME \
--job-dir $OUTPUT_PATH \
--module-name speech_commands.train \
--package-path speech_commands \
--region $REGION \
--runtime-version 1.4
```

여기서 주의깊게 살펴보아야 할 부분은 `--runtime-version` 이다. _CloudML_ 은 내부적으로 다양한 환경을 제공하는데 뒤에 숫자\(1.4\)는 기본적으로 _TensorFlow_ 의 버전과 동일하다. 각 버전에 따른 기타 모듈의 상세 버전은 아래 링크에서 확인하면 되겠다

[https://cloud.google.com/ml-engine/docs/runtime-version-list](https://cloud.google.com/ml-engine/docs/runtime-version-list)

 지금까지 _CloudML_ 은 _python2_ 만 제공을 해왔는데 2018.12.11 드디어 _python 3.5_ 를 지원한다. 여러 사람이 무척 애타게 기다렸던 부분이다. 각 모듈에 대한 버전을 별도로 설정하는 방법도 있지만 본 글에서는 다루지 않는다.

아래는 실제 수행되는 과정을 담은 예제 스크린샷이다. 

![](https://t1.daumcdn.net/cfile/tistory/996A8B3B5A34769703)

이 명령이 실행되면 우선 _package_ 로 지정된 디렉터리가 _tar_ 형태로 묶여서 _storage_ 로 전송된다. 물론 여기서 _storage_ 는`--job-dir` 로 지정된 _Google Storage_ 이다. 그후 클라우드 어딘가에 우리의 모델을 작업할 공간\(인스턴스따위\)이 할당이 되고 _storage_ 로 부터 _package_ 를 다운로드 받는다. 그리고 나서 `--module-name` 으로 지정된 _speech\_commnands.train_ 모듈을 실행한다.

명령어를 입력하면 바로 _prompt_ 로 떨어지고 모든 작업은 클라우드에서 진행된다. 중간에 보이는 것처럼 실행시킨 모델에 대한 실시간 로그는 아래처럼 확인할 수 있다. 

```text
$ gcloud ml-engine jobs stream-logs tf_ex_1
```

여러개의 모델을 연속해서 던질 수 있기 때문에 `JOB_NAME` 을 구분해서 로그를 확인하면 되겠다. 여기서는 _tf\_ex\_1_ 은 우리가 앞서 설정 했던 `JOB_NAME` 이다.

클라우드 콘솔에서도 작업 내용을 확인할 수 있는데 아래처럼 완료된 작업, 중지된 작업, 실패한 작업, 그리고 방금 던져져서 실행중인 작업을 한눈에 볼 수 있다. 

![](https://t1.daumcdn.net/cfile/tistory/99A0F4505A34793C33)

각 항목을 선택해서 보면 작업에 대해 조금 더 상세한 정보를 볼 수 있는데, 작업 ID 부분 \(여기서는 _tf\_ex\_1_\) 을 클릭해보자. 환경설정 했던 내용들과 함께 작업에 대한 내용을 확인할 수 있다.

![](https://t1.daumcdn.net/cfile/tistory/99A953455A34790C34)

여기서 더 상세한 전체 로그\(실시간 현황\)를 보고 싶다면 **로그 보기** 링크를 선택하면 된다. 조금전 터미널에서 확인하는 명령어보다 더 상세한 로그를 확인할 수 있는데 이 로그는 _stackdriver_ 를 통해 제공된다. 무료티어에서 기본 저장일이 7일이라는 점을 감안해서 사용하면 되겠다.

지금 돌린 모델의 경우 클라우드에서 24시간 이상 걸릴 수 있는 예제다. 클라우든데 뭐가 이렇게 오래 걸려? 할 필요 없다. 로컬 PC 에서 이 모델을 돌려볼 수 있는 환경이 된다면 돌려보시라. 뚝딱 결과가 나오는 만만한 예제가 아니다. 뭐 아무리 그래도 _**CloudML**_ **이** _**GCP**_ **의 특장점**인데 실망했다고? 당연히 여기서 끝이 아니다. 위에서 잠깐 언급한 것처럼 작업을 던질때 우리는 Scale Tier 를 선택할 수 있다. 이것은 우리 작업의 처리 속도, 처리량, 그리고 비용 \(...\) 까지 모두 몇 단계 상승시켜준다. 바로 사용방법을 알아보도록 하자.

![http://www.fmkorea.com/best/834940553](https://t1.daumcdn.net/cfile/tistory/996D94395A34FCB716)

## **Scale Tier**

위에서 모델이 동작한 _Cloud_ 의 환경은 가장 낮은 등급이다. 각 등급은 아래와 같이 정의된다.

| Enums |  |
| :--- | :--- |
| `BASIC` | A single worker instance. This tier is suitable for learning how to use Cloud ML, and for experimenting with new models using small datasets. |
| `STANDARD_1` | Many workers and a few parameter servers. |
| `PREMIUM_1` | A large number of workers with many parameter servers. |
| `BASIC_GPU` | A single worker instance [with a GPU](https://cloud.google.com/ml-engine/docs/tensorflow/using-gpus?hl=ko). |
| `BASIC_TPU` | A single worker instance with a [Cloud TPU](https://cloud.google.com/ml-engine/docs/tensorflow/using-tpus?hl=ko). |
| `CUSTOM` | The CUSTOM tier is not a set tier, but rather enables you to use your own cluster specification. When you use this tier, set values to configure your processing cluster according to these guidelines:You _must_ set `TrainingInput.masterType` to specify the type of machine to use for your master node. This is the only required setting.You _may_ set `TrainingInput.workerCount` to specify the number of workers to use. If you specify one or more workers, you _must_ also set `TrainingInput.workerType` to specify the type of machine to use for your worker nodes.You _may_ set `TrainingInput.parameterServerCount` to specify the number of parameter servers to use. If you specify one or more parameter servers, you _must_ also set `TrainingInput.parameterServerType` to specify the type of machine to use for your parameter servers.Note that all of your workers must use the same machine type, which can be different from your parameter server type and master type. Your parameter servers must likewise use the same machine type, which can be different from your worker type and master type. |

[https://cloud.google.com/ml-engine/reference/rest/v1/projects.jobs?hl=ko\#scaletier](https://cloud.google.com/ml-engine/reference/rest/v1/projects.jobs?hl=ko#scaletier)

_default_ 로 사용되는 _BASIC_ 은 작은 데이터셋이나 _CloudML_ 맛보기 용으로 적합하다. 이후 _STANDARD\_1, PREMIUM\_1_ 은 10배씩 더 큰 리소스를 사용할 수 있게된다. 그러니까 _STANDARD\_1_ 은 _BASIC_ 의 10배, _PREMIUM\_1_ 은 _BASIC_ 의 100배에 해당한다 \(가격도 마찬가지다\). 여기까지가 _CPU_ 에 해당되는 단계이다. 이후 _BASIC\_GPU, BASIC\_TPU_ 는 각각 _GPU_ 와 _TPU_ 를 사용할 수 있도록 지원한다. 하지만 아쉽게도 _GPU_ 와 _TPU_ 는 무료티어에서 사용할 수 없다. _quota_ 제한 때문인데, 무료티어에서는 사용할 수 있는 _GPU_ 의 개수를 늘릴 수가 없다. 사용해보고 싶다면 과감히 계정 업그레이드를 진행해야 할 것이다. _quota_ 정책이 더 궁금하다면 아래 링크를 참고하도록 하자.

[https://cloud.google.com/ml-engine/quotas](https://cloud.google.com/ml-engine/quotas)

앞서 살펴본 것들이 프리셋 같은 개념이고 입맛에 맞게 사용할 수 있는 고급단계가 _CUSTOM_ 에 해당된다. 우선 _CUSTOM_ 을 보기 전에 가격부터 살펴보고 가도록 하자. 

[https://cloud.google.com/ml-engine/pricing](https://cloud.google.com/ml-engine/pricing)

링크로 접속해서 _US_ 와 _EUROPE, ASIA region_ 별로 각각 가격을 확인해보면 US 가 확실히 저렴한 것을 알 수 있다. 이제 정말 _BASIC_ 이 아닌 다른 _Scale Tier_ 를 사용해보도록 하자. _gcloud ml-engine_ 명령어는 기존 테스트와 모두 동일하고 뒤에 `--scale-tier` 옵션 하나만 추가해주면 된다.

```text
$ gcloud ml-engine jobs submit training $JOB_NAME \
--job-dir $OUTPUT_PATH \
--module-name speech_commands.train \
--package-path speech_commands \
--region $REGION \
--runtime-version 1.4 \
--scale-tier STANDARD_1
```

_CUSTOM_ 의 경우에는 `--scale-tier` 옵션 대신 `--config` 옵션을 사용해주면 된다. 이때는 _yaml_ 를 생성하고 --_config_ 에 달아주면 된다. 꼭 _yaml_ 파일을 사용해야 하는 것은 아니고 `--scale-tier` 처럼 옵션을 풀어서 써도 되지만 정신건강에 그다지 좋지 않다. 아래와 같이 _config.yaml_ 을 생성해서 사용하도록 하자.

```text
trainingInput:
  scaleTier: CUSTOM
  masterType: complex_model_m
  workerType: complex_model_m
  parameterServerType: large_model
  workerCount: 9
  parameterServerCount: 3
```

`--config` 옵션은 아래처럼 사용해주면 된다.

```text
$ gcloud ml-engine jobs submit training $JOB_NAME \
--job-dir $OUTPUT_PATH \
--module-name speech_commands.train \
--package-path speech_commands \
--region $REGION \
--runtime-version 1.4 \
--config config.yaml
```

참고로 여기서 다룬 _TensorFlow_ 예제 코드는 _STANDARD\_1_ 로 했을 때 10시간이 걸린다. 위쪽 클라우드 콘솔 스크린샷에서 작업 ID _kaggle\_2_ 가 바로 그것이다. 사용 요금은 _US region_ 에 명시된 것처럼 계산되어 $20~ 정도 청구 되었다. \(_BASIC\_GPU_ 의 경우 2시간 8분 소요. 약 $3~ 청구\)

## **마무리**

비싼 그래픽카드로 무장한 장비를 구매해도 시장의 빠른 변화에 대응할 정도의 성능을 발휘하지는 못한다. 학업 목적이나 작은 데이터를 다루는 것이라면 상관없지만 대규모 프로젝트나 _product_ 용도라면 이제 클라우드를 고려하지 않을 수 없는 세상이 되었다. 더욱이 모델을 연속적으로 계속 던질 수 있다는 것은 엄청난 장점이 아닐 수 없겠다. 어떤 클라우드를 사용할지는 사용자가 고민해야 하는 부분이겠다. \(일단 _Google Cloud_ 는 계정을 계속 새롭게 생성해서 $300 크레딧을 사실상 무한하게 사용할 수 있다. 공부를 위한 목적이라면 이보다 좋은 클라우드는 없을지도\)

_Google Cloud Platform_ 자체가 워낙 급변하고 있는 시장이라 이 글을 쓰여지고 얼마 안되어 터미널 명령이나 옵션등이 변경될 수도 있다. 혹시 예제가 정상적으로 수행되지 않는다면 말씀을 부탁드린다. 함께 고민하고 해결 할 수 있도록 노력하겠다.

