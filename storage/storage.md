# Storage

_Google Cloud Platform_ 저장소\(_Storage_\)에 대해서 살펴볼텐데, 여기 메뉴에는 딱히 어려운게 없다. "난 저장소를 사용할 일이 없는데?" 라고 생각할지 모르지만 여기 저장소는 _Cloud_ 에 다른 _compute engine_ 이나 특히 `CloudML` 에서 매우 유용하게 사용되므로 익혀두는 것이 좋겠다.

저장소를 사용하기 위해서는 화면 좌측 메뉴에서 _Storage_ 에 접근하도록 한다.

![](https://t1.daumcdn.net/cfile/tistory/265C7A3A59406E4619)

그럼 세 개의 메뉴가 보일텐데 설정은 _Storage_ 에 _access_ 할 수 있는 _ID_ 를 제공하고 _API_ 를 설정할 수 있는 메뉴를 제공한다. 전송은 _Amazon S3\(Simple Storage Service\)_ 나 기타 서버와의 자료 전송을 위해 사용되는 메뉴다. 가장 중요한 브라우저는 기본 페이지로서 버킷을 생성하거나 파일/폴더 를 컨트롤 할 수 있는 메뉴가 되겠다.

![](https://t1.daumcdn.net/cfile/tistory/245CF93A59406E4725)

_GCP_ 의 저장소는 버킷 단위로 관리가 되는데 저장 공간의 종류라고 생각하면 이해가 쉽다. 우선 버킷 생성 메뉴를 통해 더 살펴본다.

![](https://t1.daumcdn.net/cfile/tistory/254BD73A59406E4824)

버킷의 종류에는 _Multi-Regional, Regional, Nearline, Coldline_ 이렇게 총 네 가지가 있다. 데이터에 _access_ 하는 빈도에 따라 분류한다고 생각하면 되는데 "거의 들여다보지 않는 데이터를 비싼 돈 주고 백업할 필요가 있나"라는 개념으로 접근해보자. 여기 저장소는 그러한 갈증을 해결해준다. 로그 저장 기간 법률 등으로 인해 어쩔 수 없이 장기간 보존되어야 하는 데이터는 그게 맞는 버킷 제공으로 저렴한 가격에 이용이 가능하다.

![](https://t1.daumcdn.net/cfile/tistory/261BAD3A59406E4924)

가격정책에 대해 조금 더 살펴보면 아래와 같다. \([가격 정책](https://cloud.google.com/storage/docs/storage-classes?hl=ko&_ga=1.99812574.1723078938.1490857833)\)

![](https://t1.daumcdn.net/cfile/tistory/2312973A59406E4A22)

버킷을 적당히 선택했다면 이제 여기서 파일이나 폴더를 컨트롤할 수 있겠다.

![](https://t1.daumcdn.net/cfile/tistory/2126583A59406E4B24)

_Storage_ 는 _CloudML_ 로 넘어가는 첫 단추여서 짧지만 기록해 두도록 한다. 또한 _IDC_ 에 물리 저장소에서 _Cloud_ 로 넘어오는 시국의 경계선에 서있는 만큼, 버킷의 종류를 이해하고 적절하게 사용할 수 있는 것은 기본소양이 되겠다.

