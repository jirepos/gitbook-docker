# Tomcat 이미지 구성

Docker를 사용하면 Tomcat을 실행하기 위해서 개발 서버, 검증 서버, 운영 서버 등에서 별도로 설정할 필요가 없다. 단지 이미지를 받아서 컨테이너로 실행하면 된다. 그러나 소스를 반영하거나, 각 서버에서 DB 접속 정보등을 따로 설정해야 하는데 이 것 때문에 새로운 이미지를 만들 수는 없다. 따라서 동일한 이미지로 컨테이너를 생성하고 실행해야 하고 각 서버마다 다른 부분만 통일된 디렉터리 구조를 가지고 컨테이너가 참조하게 하면 된다.

Tomcat의 디렉터리 중에서 webapps, conf, logs 디렉터리를 호스트의 디렉터리로 생성하고 컨테이너가 참조하게 하면 된다.

## Tomcat 8.5 Image 받기

tomcat:835 이미지를 받는다.

```shell
#> docker pull tomcat:8.5
8.5: Pulling from library/tomcat
0bc3020d05f1: Pull complete
a110e5871660: Pull complete
83d3c0fa203a: Pull complete
a8fd09c11b02: Pull complete
96ebf1506065: Pull complete
26b72ffca293: Pull complete
0bffa2ea17aa: Pull complete
d880cebcc7a6: Pull complete
810680243d4b: Pull complete
6ff9f16d00ec: Pull complete
Digest: sha256:efb7347498f9527ab8d21f4a5c807a5806785452f37f1d749287cd08717e408b
Status: Downloaded newer image for tomcat:8.5
docker.io/library/tomcat:8.5
```

## 컨테이너 실행

우선 컨테이너를 실행해 본다.

```shell
docker run --name was -d -p 8080:8080 tomcat:8.5
```

## 브라우저에서 접속

브라우저에서 접속해 본다.

```shell
http://localhost:8080/index.html
```

그런데 404 오류가 나고 페이지 없다고 나온다. tomcat:8.5 이미지의 webapps 디렉터리는 비어있기 때문이다.

## 로그확인

Docker logs 명령으로 컨테이너 로그를 확인할 수 있다. 컨테이너 이름을 was로 주었기 때문에 다음과 같이 명령을 실행한다.

```shell
docker logs -f was
```

## 파일 복사

Tomcat 사이트에서 다운로드 받은 Tomcat 8.5 버전의 압축을 해제한 디렉터리에서 webapps의 examples 디렉터리를 cp 명령을 사용하여 복사하자.

```shell
docker cp /mnt/e/tmp/examples was:/usr/local/tomcat/webapps
```

다시 브라우저로 접속하면 이번에는 페이지가 보일 것이다.

```shell
http://localhost:8080/examples/
```

## 컨테이너 내 특정 디렉토리를 로컬에 마운트

이번에는 Volume을 이용할 것이다. 컨테이너가 볼륨을 사용하기 위해서는 볼륨을 컨테이너에 마운트(mount)해줘야 한다. docker run 커맨드로 컨테이너를 실행할 때 -v 옵션을 사용하면 되는데 콜론(:)을 구분자로 해서 앞에는 마운트할 볼륨명 또는 호스트의 경로(바인드 마운트라고 함), 뒤에는 컨테이너 내의 경로를 명시해주면 된다.

Windowws에서 d:/docker/tomcat 디렉터리를 만든다.

```shell
mkdir d:/docker/tomcat
```

tomcat:8.5 이미지에서 폴더를 복사한다. webapps는 파일이 없으므로 제외한다.

```shell
docker cp was:/usr/local/tomcat/conf /mnt/d/docker/tomcat
docker cp was:/usr/local/tomcat/logs /mnt/d/docker/tomcat 
```

복사가 되었는지 확인한다.

```shell
#> ls -al /mnt/d/docker/tomcat
total 0
drwxrwxrwx 1 root root 512 Jun 29 17:15 .
drwxrwxrwx 1 root root 512 Jun 29 17:08 ..
drwxrwxrwx 1 root root 512 Jun 29 17:13 conf
drwxrwxrwx 1 root root 512 Jun 29 17:13 logs
```

Windows 탐색기에서 다운로드 받은 Tomcat 8.5 디렉터리에서 webapps/examples를 /mnt/d/docker/tomcat으로 복사한다. 명령을 입력해도 되지만 직접 복사하는 것이 편할 것이다.

컨테이를 중지하고 삭제한다.

```shell
docker stop was
docker rm was
```

컨테이너를 다음과 같이 Volume을 이용하여 실행한다.

```shell
#!/bin/sh
docker run -d -p 8080:8080 \
 -v /mnt/d/docker/tomcat/conf:/usr/local/tomcat/conf \
 -v /mnt/d/docker/tomcat/webapps:/usr/local/tomcat/webapps  \
 -v /mnt/d/docker/tomcat/logs:/usr/local/tomcat/logs \
 --name was tomcat:8.5
```

> 주의할 점은 '--name was tomcat:8.5'을 맨 뒤에 적어야 한다.

**형식**

```shell
-v [host-src]:[container-dest]
```

host-src는 절대 경로이거나 볼륨의 이름이어야 한다.

## 이미지 생성

컨테이너를 중지하고 commit 명령을 사용하여 이미지를 만든다.

## 이미지 배포

이 부분은 개인 registry에서 정리할 것이다.

## 생성된 이미지 사용

이제 다른 서버에서는 동일한 위치에 webapps,logs, conf 디렉터리를 만들고 단지 pull로 받은 이미지를 컨테이너로 실행하기만 하면 된다.
