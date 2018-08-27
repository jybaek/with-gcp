*Google Cloud Platform* 은 *GitHub* 같은 형태의 소스코드 저장소를 로컬에서 지원하기 때문에 소스 관리가 용이하다. 또한 막상 *GitHub* 을 무료계정으로 사용하고 있어서 *private* 한 내용을 별도로 보관하고 이력관리할 필요가 있다면 *GCP* 의 저장소는 무척 유용하다. 사실 사용하기 위해서 대단한 스킬이 필요한 것은 아니지만 누군가의 진입장벽을 조금이라도 더 낮추기 위해서 포스팅해둔다. 

*datalab* 을 생성하면 기본적으로 저장소가 생성되는 듯 하고, 별도로 `저장소 만들기` 메뉴를 통해 생성할 수도 있다. 우선 여기 예시에서는 아래처럼 생성되어 있는 *datalab-notebooks* 라고 하는 이름의 저장소를 이용하는 방법에 대해서 알아보겠다. (`저장소 만들기` 를 통해 새로운 저장소를 생성해서 실습해도 다를게 없다.)

![](https://t1.daumcdn.net/cfile/tistory/2719013C593DDAB81E)

일단 별거 없다. "복제 URL"을 복사하고 평소 Git 을 사용하던 것 처럼 clone 을 사용해봤다. 인증이 필요하겠지만 *google* 계정 정도이다.

![](https://t1.daumcdn.net/cfile/tistory/26358A3C593DDAB91B)

구글 계정 정보를 입력했더니 유효하지 않은 인증 정보란다. 그리고 새로운 식별정보를 생성하란다. 아마도 구글 기본 계정으로 사용은 불가능한가보다. 친절하게 링크를 제공해줬으니 따라가서 확인해본다.

![](https://t1.daumcdn.net/cfile/tistory/2516083C593DDAB926)

*Git* 계정은 구글 계정과 동일하고 패스워드는 별도의 *base64encoding* 된 듯한 문자열이 출력된다. 패스워드를 바로 사용하면 안되고 *home directory* 에 있는 *.netrc* 에 넣고 사용하란다. *machine source...* 부터 전부 복사해서 *.netrc* 파일에 써주면 된다. 파일이 없다면 생성해주도록 한다.

![](https://t1.daumcdn.net/cfile/tistory/25629A3C593DDABC16)

*clone* 에 성공했다. 기본적으로 생성되어 있는 파일들을 확인한다. 아마도 *datalab* 으로 생성된 저장소가 아니더라도 아래 두 개 파일은 기본적으로 있을 것이다.

![](https://t1.daumcdn.net/cfile/tistory/263A883C593DDABD1B)

이번에는 파일을 변경하고 *commit/push* 까지 테스트 시도해본다.

![](https://t1.daumcdn.net/cfile/tistory/252D4A38593DDABE2C)

우선 파일을 변경했으니 상태를 확인한다.

![](https://t1.daumcdn.net/cfile/tistory/2137B438593DDABE25)

그리고 나서 *commit* 을 시도해봤지만 기본적인 *Git* 환경에 필요한 환경변수의 세팅이 필요했다.

![](https://t1.daumcdn.net/cfile/tistory/24208D38593DDAC220)

*email* 주소와 사용자 이름을 입력하고,

![](https://t1.daumcdn.net/cfile/tistory/221FD838593DDABF20)

다시 한번 *commit* 을 하고 *push* 까지 진행하도록 하자.

![](https://t1.daumcdn.net/cfile/tistory/2535B138593DDAC01E)

모두 정상적으로 완료 되는 것을 확인할 수 있다. 이제 커밋 로그를 *Google Cloud Platform* 콘솔에서 확인해보도록 한다.

![](https://t1.daumcdn.net/cfile/tistory/27531038593DDAC124)

소스 저장소에 소스 코드로 들어가서 우측에 시계 아이콘을 클릭하면 커밋 로그에 대한 히스토리를 볼 수 있다.

![](https://t1.daumcdn.net/cfile/tistory/25148E3C593DDABB1E)

방금 커밋한 `Update README.md` 가 화면에 보인다.

![](https://t1.daumcdn.net/cfile/tistory/2742C33C593DDABA21)

*Google Cloud Platform* 이 저장소를 제공한다는 이야기는 들었었는데 어떤식으로 제공을 하는지, 어떻게 사용할 수 있는지에 대한 이야기는 약간 뜬 구름 같았다. 사실 구글이 하는 것이니 뭔가 엄청난 것이 숨어 있을 것 같기도 했고. (사실 이 정도면 엄청 대단한거긴 하지)

하지만 살펴보니 정작 특별한 기술 필요 없이 강력한 *Git* 저장소를 사용할 수 있었다. 이 정도면 *Git* 을 사용할 줄 아는 사용자라면 누구나 쉽게 접근할 수 있겠다. (도구 및 플러그인 메뉴를 통해 더욱 강력한 것들을 사용할 수 있을 것으로 기대되는데 이 부분은 별도의 포스팅으로 알아보자)
