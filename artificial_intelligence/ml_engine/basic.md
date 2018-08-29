# 기본

앞서 [CloudML 개념](concept.md)에 대해서 살펴봤고, 이제 본격적으로 사용하는 방법에 대해서 기술하고자 한다. 우선 모든 예제는 우분투 16.04 에 _Cloud SDK_ 를 설치한 환경에서 진행 되었다. _CloudML_ 의 모든 것이 _gcloud_ 명령어를 통해 진행되기 때문에 OS 의 환경에서 오는 차이는 없을 것이다.

이번 글에서는 _CloudML_ 의 기본으로, 간단하게 _Google Cloud Platform \(GCP\)_ 에 _CloudML_ 서비스를 맛보는 것으로 마무리 하겠다. 사실 _CloudML_ 자체가 너무 거대한 서비스이기 때문에 하나의 글에 모두 담는다면 처음 접하는 입장에서 스크롤이 상당한 압박으로 다가올 것으로 예상된다. 조금 더 우아하게 사용하는 방법이나 고급 스킬에 대한 부분은 별도의 포스팅으로 하는게 좋겠다.

_CloudML_ 서비스는 _gcloud_ 명령어에서 시작된다. 그리고 로컬 테스트와 클라우드 테스트로 나눌 수 있다. 이 말은, 클라우드에서 수행되는 모든 서비스는 크레딧을 소모하기 때문에 모델을 학습하는 과정 중에 코드의 실수나 로직의 문제로 인해 `Error` 가 발생한다면 여러번 반복적인 _train_ 을 시도해야 하고 크레딧이 줄줄 흐르게 될 것이다. 그렇기 때문에 우선 모델이 최소한 _Logical_ 한 문제가 없는지 로컬에서 테스트가 이뤄져야 한다.

## Local 환경에서의 CloudML 테스트

테스트를 위해 우선 예제코드를 작성하도록 한다. 다음과 같은 디렉터리를 구성한다. 예제일 뿐이기 때문에 익숙해지면 변경해서 사용하면 되겠다. \(_trainer_ 라는 디렉터리 안에 두 개의 파일이 있다\)

```bash
$ tree
.
└── trainer
    ├── __init__.py
    └── oops.py

1 directory, 2 files
```

그리고 `oops.py` 라는 파일에는 다음과 같이 단순하게 _TensorFlow_ 의 버전을 출력하는 코드만 담아놨다. \(`__init__.py` 는 단순히 _python module_ 을 나타내기 위한 용도로 예제에서는 _size_ 가 0 인 파일이다.\)

```python
import tensorflow as tf 

if __name__ == '__main__': 
  print tf.__version__
```

이 예제 코드를 통해 _CloudML_ 의 동작을 확인하게 될 것이다. 우선 로컬 환경에서 정상적으로 동작하는지 테스트는 다음과 같이 진행하면 된다.

```bash
$ gcloud ml-engine local train \
    --module-name trainer.oops \    
    --package-path trainer
```

`$ python trainer/oops.py` 이런 방식의 테스트를 생각했어도 상관 없다. 단순하게 _TensorFlow_ 의 동작만 테스트해야 한다면 상관없겠지만 지금은 _Cloud_ 환경에서 동작될 수 있는 여건인지 테스트가 진행되어야 하므로 위에서 언급한 `gcloud` 명령어를 통해 테스트를 진행하면 된다. 기본적으로 `gcloud ml-engine` 이 _CloudML_ 을 위한 메인 명령어가 될 것이고 그 이후에 서브 옵션들이 어떤 환경에서 어떤 방식으로 모델을 학습시킬 것인지 결정하게 된다. 예제는 _local_ 로 지정되어 있기 때문에 _Cloud_ 환경에서 돌린다는 가정으로 로컬에서 모델이 동작하게 된다. \(한 가지 주의할 것은 _TensorFlow_ 가 실행 가능한 환경이어야 한다는 것이다. 로컬에 _install_ 이 되어 있든, _virtualenv_ 환경이 구축되어 있든지 말이다.\)

위 예제에서 사용된 옵션에 대해 각각 살펴보면 다음과 같다.

* --module-name: 학습시킬 모델 지정
* --package-path: python 모듈로 지정할 경로

매우 간단하다. 여기에 학습데이터의 위치나 출력 결과 지정 등은 모델의 인자로 넘길 수 있는데 공식 홈페이지에 있는 예제를 살펴보면 아래와 같다.

```bash
$ gcloud ml-engine local train \
    --module-name trainer.task \
    --package-path trainer/ \
    -- \
    --train-files $TRAIN_DATA \
    --eval-files $EVAL_DATA \
    --train-steps 1000 \
    --job-dir $MODEL_DIR
```

여기 예제를 보면 방금 우리가 사용했던 것 과 동일하게 _trainer_ 라는 디렉터리가 있고, 그 하위에 `task.py` 를 이용했다는 사실을 알 수 있다. 조금 전에 다뤘던 예제와의 차이점은 굵은 빨간색으로 마킹된 --  부분을 경계로 밑에 내용들이 추가되어 있다는 것이다. -- 는 일반적인 경우에는 기본적으로 명령어 옵션을 지칭할 때 사용하지만 `ml-engine` 명령에서 지금과 같이 인자를 지정하지 않았을 때는 `task.py` 에서 사용하는 인자로 전달된다. \(여기서는 모델에서 사용할 학습 데이터의 위치와 출력위치, 학습 _step_ 정도라고 이해하고 넘어가자\)

## Cloud 환경에서의 CloudML 테스트

이제 `gcloud` 명령어를 통해 _local_ 에서 학습을 테스트하는 방법은 끝났다. 이제 실제 _cloud_ 에서 학습을 테스트해야 할 텐데 명령어는 다음과 같이 간단하다.

### Shell 환경 변수로 몇 가지 지정해주는 부분

```bash
$ BUCKET_NAME=jybaek_cloudml_test
$ JOB_NAME=census_single_7
$ REGION=us-central1
$ OUTPUT_PATH=gs://$BUCKET_NAME/$JOB_NAME
```

### 위에서 지정된 환경변수를 기반으로 Cloud 에 모델을 전송해서 학습하는 부분

```bash
$ gcloud ml-engine jobs submit training $JOB_NAME \
--job-dir $OUTPUT_PATH \
--module-name trainer.oops \
--package-path trainer/ \
--region $REGION \
--runtime-version 1.2
```

사실 복잡해 보이지만 빨간색으로 마킹된 부분은 이미 _local_ 테스트할 때 사용했던 부분과 동일하다. 또한 `ml-engine` 에 인자로 넘어가는 것들이 직관적이기 때문에 매우 이해하기 쉽다. 가장 주목해야 할 옵션은 --runtime-version 이다. 이 옵션을 통해 우리는 _TensorFlow_ 의 버전을 지정해서 사용할 수 있다. _TensorFlow_ 가 버전업 될 때마다 어느순간 _CloudML_ 의 TF 버전도 변경이 되기 때문에 --runtime-version 을 지정하지 않으면 어느순간 우리의 모델이 정상적으로 학습되지 않을 수도 있다. 꼭 지정하는 습관이 필요하겠다. 다음으로는 위에서 지정한 환경변수를 살펴보자.

`BUCKET_NAME` 은 _Google Cloud Platform \(GCP\)_ 에 존재하는 _Storage_ 의 단위이다. 학습 결과나 출력되는 모든 내용이 기록되기 때문에 지정이 필요하다. `JOB_NAME` 은 _bucket_ 안에서 유니크한 이름이어야 한다. 이미 제출된 이력이 있는 _job_ 과는 겹쳐서는 안되기 때문이다. \(어차피 중복되면 에러 문구가 출력된다.\) `REGION` 은 _CloudML_ 이 수행 될 리전이다. 리전에 따라 가격 정책이 다르기 때문에 살펴보고 사용하도록 하자. 아래는 _GCP_ 콘솔에서 확인한 버킷 정보이다. 실제 예제에서 사용된 `jybaek_cloudml_test` 가 등록되어 있다.

![](https://t1.daumcdn.net/cfile/tistory/2618854F59543B4531)

## CloudML 결과 확인

예제처럼 학습을 제출하고 나면 아래 명령어를 통해 _Shell_ 상에서 로그를 확인할 수 있다.

```bash
$ gcloud ml-engine jobs stream-logs census_single_7
```

또한 _GCP_ 의 콘솔에서도 다음과 같이 확인이 가능하다. 최초 작업을 전송하고 학습이 진행중인 화면이다.

![](https://t1.daumcdn.net/cfile/tistory/2606E04B595439DD2E)

로그 보기를 선택하면 _GCP_ 에서 실제 로그가 저장되는 _stackdriver_ 로 이동이 되고, _job_ 에 대한 자세한 로그를 확인할 수 있다. 좀전에 로컬에서 확인할 수 있었던 로그와 같은 내용이다. \(다음은 작업이 진행중인 상태의 화면이다.\)

![](https://t1.daumcdn.net/cfile/tistory/2135A34B595439DE2E)

작업이 끝나면 결과를 직관적으로 확인할 수 있다. 작업이 실패한 경우에는 로그를 통해 원인을 파악하도록 하자. _python_ 에서 흔히 볼 수 있는 _Traceback_ 형태이기 때문에 분석이 어렵지 않다.

![](https://t1.daumcdn.net/cfile/tistory/2438D54B595439DF30)

## 마무리

이번 글에서는 단순하게 _CloudML_ 의 학습을 로컬 환경과 _Cloud_ 환경에서 돌려보는 것을 살펴보았다. 사실 많은 블로그에 소개되는 _CloudML_ 의 글은 고급 스킬이나 내용이 한번에 다뤄지고 있어서 초급자가 접근하는데 쉽지 않다는 생각을 늘 갖고 있었다. 불필요한 옵션은 최대한 제거하고 쉽게 설명하려고 글을 썼는데 역시나 쉽지는 않았다. 아무튼, 다음 글에서는 _CloudML_ 의 고급 설정을 통해 모델을 학습 시키는 것에 대해 다뤄보고자 한다.

