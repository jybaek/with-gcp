본문에서는 _Cloud Shell_ 을 이용해서 인스턴스에 접속하는 방법을 살펴봤는데, 이번에는 사용자의 머신에서 직접 _SSH_ 를 접속하는 방법에 대해서 설명한다. 환경은 리눅스(우분투)지만 다른 OS라도 크게 다르지 않을 것이다.

전체 시나리오를 요약하면 다음과 같다.

  - 로컬에서 ssh 접속을 위한 rsa key 생성
  - 생성한 key 를 Google Cloud Platform 콘솔에 등록

천천히 살펴볼텐데 우선 _GCP_ 에 등록할 _rsa key_ 를 생성해야 한다. _rsa key_ 의 _default_ 파일명은 *id_rsa* 로 지정 되는데 혹시 이미 사용중일 수도 있으니 적당한 이름으로 변경해서 사용하도록 하자.

```bash
$ ssh-keygen -t rsa -f ~/.ssh/[KEY_FILE_NAME] -C [USERNAME]
```

![](https://t1.daumcdn.net/cfile/tistory/263AFB3358E596691B)

_key_ 생성이 끝났으면 _\*.pub key_ 내용을 복사해서 _GCP_ 에 입력해야 한다. _key_ 내용을 확인하고 복사하도록 한다.

![](https://t1.daumcdn.net/cfile/tistory/2518183358E5966616)

복사한 키를 _GCP_ 에 입력한다. 위치는 `Compute Engine -> 메타데이터 -> SSH 키` 이다. 이때 사용자 이름은 자동으로 들어가니 신경쓰지말고, 저장 버튼을 누른다.

![](https://t1.daumcdn.net/cfile/tistory/2176BA4058E592340C)

이제 모든 과정이 끝났고 아래와 같이 터미널에서 인스턴스로 접속을 하도록 한다. 인스턴스의 IP는 GCP에서 미리 확인하도록 하자. 해당 IP는 기본적으로 임시 IP이기 때문에 변경될 수도 있으니 주의가 필요하다.

![](https://t1.daumcdn.net/cfile/tistory/222CCE4058E5923632)

외부IP는 기본적으로는 임시IP로 설정되지만 설정을 통해 고정IP로 사용도 가능하니 참고하도록 하자.

_SSH_ 로그인 관련해서 보다 자세한 내용은 아래 링크를 참고하면 좋다.

https://cloud.google.com/compute/docs/instances/connecting-to-instance#standardssh

혹시 계속 접속이 되지 않는다면 위 과정을 수행하는 중 *~/.ssh/known_hosts* 의 내용이 꼬였을 수 있다. 해당 파일에서 _GCP_ 에 대한 라인을 삭제하고 다시 시도해보자.
