# Windows 에서 Docker 설치

WSL2를 설치하고 리눅스 배포를 설치했으면 이제 Docker를 설치할 수 있다.

## 설치

[Docker Desktip for Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows/)에서 다운로드 받는다.

![](<.gitbook/assets/image (11).png>)

![](<.gitbook/assets/image (8).png>)

Close and logout을 클릭한다. 로그아웃 된다. 다시 로그인 하면 아래와 같은 화면이 표시된다.

![](<.gitbook/assets/image (12).png>)

위 화면이 나온 후에 'Start' 버튼이 표시되는데 클릭한다. 그러면 다음과 같은 화면이 나온다. 

![](<.gitbook/assets/image (15).png>)

뭘 복제하라고 하는 것 같은데... 파란색의 '>>'을 클릭한다. 그럼면 github에서 복제를 한다. 'Net Step' 버튼을 클릭한다.

다음과 같은 화면이 나온다. 똑 같은 방법으로 클릭하여 명령을 실행한다.

![](<.gitbook/assets/image (9).png>)



'Next Step' 버튼을 클릭한다. 다음과 같은 화면이 나온다. 마찬가지로 명령을 실행한다.

![](<.gitbook/assets/image (17).png>)

'Next Step' 버튼을 클릭한다.

![](<.gitbook/assets/image (10).png>)

마지막 화면은 Docker 이미지를 Doker Hub에 공유하려면 Sign in하라고 하는데 일단 이 단게는 건너 뛰어도 될 것 같아서 패스한다.

'Done' 버튼을 클릭한다.

![](<.gitbook/assets/image (16).png>)

PowerShell에서 다음과 같이 입력하니까 버전이 출력된다. 일단 설치는 된 것 같음.

```shell
PS C:\> docker -v
Docker version 20.10.7, build f0df350
```

정상적으로 모두 설치했으면 WSL2 우분투 터미널에서도 도커 명령을 사용할 수 있다.

## Settings

Docker 설정은 설치한 이후에 바로 필요하지 않다. 설정이 필요할 때 그 때하기로 한다. Settings 화면에서 Resources 탭에 가면 CPU, Memory 등을 설정할 수 있는데 나의 경우 WSL2를 설치했기 때문에 Windows에 의해 제한을 받는다고 설명이 되어 있다. .wslconfig 파일에서 설정하하고 가이드가 나온다.

### General

특별한 내용이 없어서 일단 패스한다.

### Resources

![](<.gitbook/assets/image (7).png>)

### File sharing

WSL 2 모드와 Windows Container 모드는 모드 자동적으로 모든 파일이 Windows에 의해 공유되기 때문에 비활성화 된다.

### Proxies

### WSL Integration

WSL 2 모드에서는 Docker WSL 통합을 포함 할 WSL 2 배포를 구성 할 수 있다. 디폴트로 default WSL distritubtion에 통합이 된다. default WSL distro를 변경하려면, 다음을 실행한다.

```shell
wsl --set-default <distro name>
```
