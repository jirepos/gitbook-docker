# Windows에서 Docker 설치(2)

보통은 리눅스 컨테이너 개발 환경을 필요로 하기 때문에, 처음 Docker for Desktop을 설치하면 리눅스 컨테이너 호스트로 실행된다. 그래서 Windows 컨테이너 모드로 전환을 해야 한다.

트레이에서 우측 마우스 버튼을 클릭하면 Switch to Windows containsers 메뉴가 나온다. 이것을 클릭하면 되는데... 음 Windows Pro에서만 지원해서 비활성화 되어 있다.

## Windows 컨테이너 호스트로 전환

![](<.gitbook/assets/image (14).png>)

이건 나중에 하기로 하자.

그러다가 이런 내용이 있는 글을 읽었는데... 도커 생태계가 위협받고 있는 것 같다.

> 하지만 컨테이너의 표준화가 이뤄진 후 컨테이너 시장은 OCI와 CRI를 중심으로 성장하고 있습니다. 그 과정에서 사실상 컨테이너 표준으로 사용되던 도커의 역할을 대신하는 다양한 기술이 나오고 있으며 그 중 Buildah, Podman, Skopeo는 도커의 기능을 역할별로 나눠 구현하고 있습니다. 이들은 또한 도커의 보안 관련 단점을 보완하며 기존에는 없던 편리한 기능도 추가로 제공합니다.

이 글은 Docker에 관한 글이지만 급 OCI에 관심이 생겼다. 관련 글은 여기 http://www.opennaru.com/kubernetes/open-container-initiative/
